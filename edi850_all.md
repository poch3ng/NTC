以下示範「如何將第二階段解析後的資料 (例如 PurchaseOrder850 物件) 寫入到先前設計好的 EdiDetail 資料表」的概念做法。
舉例來說：

1. 已經解析完的結果裝在 poResult As PurchaseOrder850。


2. 先把對應的 主表 (EdiMain) 資訊 (如 BEG、CUR 等) 寫入後，拿到 MainId。


3. 然後將要寫入的各種段 (N9、N1、REF、SAC、ITD…等) 插入 EdiDetail。



> 以下程式碼為 VB.NET 範例，示意重點在「如何組織 INSERT」，細節如 DB 連線、Transaction、Exception 處理請自行依專案實作。




---

一、在 PO850Parser 解析完後，呼叫寫入流程

假設在 Main Module 裡：

For Each tx As EDITransaction In allTransactions
    If tx.TransactionSetID = "850" Then
        Dim poResult As PurchaseOrder850 = PO850Parser.ParseTransaction850(tx.Segments)
        
        ' (1) 先寫入 EdiMain 取得主表 ID
        Dim mainId As Integer = SaveEdiMain(connectionString, poResult)

        ' (2) 寫入 EdiDetail （N9、N1、REF、SAC、ITD、DTM 等等）
        SaveEdiDetail(connectionString, mainId, poResult)
    End If
Next


---

二、寫入 EdiMain (簡化範例)

Private Function SaveEdiMain(connStr As String, po As PurchaseOrder850) As Integer
    Dim newId As Integer = 0
    Using conn As New SqlClient.SqlConnection(connStr)
        conn.Open()
        Dim sql As String = "
            INSERT INTO EdiMain (BEG03, BEG05, CUR01, ...)
            OUTPUT Inserted.MainId
            VALUES (@PoNumber, @PoDate, @Currency, ... )"

        Using cmd As New SqlClient.SqlCommand(sql, conn)
            cmd.Parameters.AddWithValue("@PoNumber", po.PoNumber)
            cmd.Parameters.AddWithValue("@PoDate", po.PoDate)
            cmd.Parameters.AddWithValue("@Currency", po.Currency)
            ' ... 依實際欄位補齊

            newId = CInt(cmd.ExecuteScalar())
        End Using
    End Using
    Return newId
End Function

> 實際上你需要對應主表 (EdiMain) 的欄位寫入，例如 BEG03, BEG05, CUR01 等。




---

三、寫入 EdiDetail

3.1 整體結構

Private Sub SaveEdiDetail(connStr As String, mainId As Integer, po As PurchaseOrder850)
    Using conn As New SqlClient.SqlConnection(connStr)
        conn.Open()

        ' 建議整個寫入放在交易 (Transaction) 中
        Using tran = conn.BeginTransaction()
            Try
                ' 1) 寫入 Header 層 (N9, REF, SAC, ITD, DTM...)  
                InsertN9Loops(conn, tran, mainId, po.N9Loops)
                InsertHeaderREFs(conn, tran, mainId, po.HeaderREFs)
                InsertHeaderSACs(conn, tran, mainId, po.HeaderSACs)
                InsertHeaderITDs(conn, tran, mainId, po.Terms)  '範例中 ITD 只有一欄，如多筆可自行改寫
                InsertHeaderDTMs(conn, tran, mainId, po.Dates)
                InsertMSGIfNeeded(conn, tran, mainId, po) ' 若有 Header MSG

                ' 2) 寫入頂層 N1
                InsertTopN1Loops(conn, tran, mainId, po.TopN1Loops)

                ' 3) 寫入行項 (PO1 Loop)
                For Each lineItem In po.LineItems
                    InsertPO1Loop(conn, tran, mainId, lineItem)
                Next

                ' 4) 寫入 CTT (若需要)
                InsertCTT(conn, tran, mainId, po.CTTSummary)

                tran.Commit()
            Catch ex As Exception
                tran.Rollback()
                Throw
            End Try
        End Using
    End Using
End Sub

3.2 寫入 N9 Loop (舉例)

Private Sub InsertN9Loops(conn As SqlClient.SqlConnection, 
                          tran As SqlClient.SqlTransaction, 
                          mainId As Integer, 
                          n9Loops As List(Of N9Loop))
    Dim sql As String = "
        INSERT INTO EdiDetail (
            MainId, ParentDetailId, LoopType, LoopSequence,
            N901, N902, N903
            -- 其他欄位如 N904, N905... 依需求
        )
        VALUES (
            @MainId, @ParentDetailId, @LoopType, @LoopSequence,
            @N901, @N902, @N903
        );
        SELECT SCOPE_IDENTITY();"

    Dim sequence As Integer = 1
    For Each n9 In n9Loops
        Using cmd As New SqlClient.SqlCommand(sql, conn, tran)
            cmd.Parameters.AddWithValue("@MainId", mainId)
            cmd.Parameters.AddWithValue("@ParentDetailId", DBNull.Value) ' 預設掛在最外層
            cmd.Parameters.AddWithValue("@LoopType", "N9")
            cmd.Parameters.AddWithValue("@LoopSequence", sequence)
            cmd.Parameters.AddWithValue("@N901", n9.RefID)
            cmd.Parameters.AddWithValue("@N902", n9.FreeForm)
            cmd.Parameters.AddWithValue("@N903", DBNull.Value) ' 若需要更多欄位可填

            Dim n9DetailId As Integer = CInt(cmd.ExecuteScalar())

            ' 如果 N9 Loop 裡還有 MSG、REF 等子段，可在此呼叫 Insert 子段:
            InsertN9LoopMessages(conn, tran, mainId, n9DetailId, n9.Messages)
            InsertN9LoopRefs(conn, tran, mainId, n9DetailId, n9.Refs)

            sequence += 1
        End Using
    Next
End Sub

' 範例：N9 Loop 下的 MSG
Private Sub InsertN9LoopMessages(conn As SqlClient.SqlConnection,
                                 tran As SqlClient.SqlTransaction,
                                 mainId As Integer,
                                 parentId As Integer,
                                 messages As List(Of String))
    Dim sql As String = "
        INSERT INTO EdiDetail (
            MainId, ParentDetailId, LoopType, LoopSequence,
            MSG01
        )
        VALUES (
            @MainId, @ParentDetailId, @LoopType, @LoopSequence,
            @MSG
        )"

    Dim seq As Integer = 1
    For Each msg In messages
        Using cmd As New SqlClient.SqlCommand(sql, conn, tran)
            cmd.Parameters.AddWithValue("@MainId", mainId)
            cmd.Parameters.AddWithValue("@ParentDetailId", parentId) ' 掛在 N9
            cmd.Parameters.AddWithValue("@LoopType", "MSG")
            cmd.Parameters.AddWithValue("@LoopSequence", seq)
            cmd.Parameters.AddWithValue("@MSG", msg)

            cmd.ExecuteNonQuery()
            seq += 1
        End Using
    Next
End Sub

' 範例：N9 Loop 下的 REF
Private Sub InsertN9LoopRefs(conn As SqlClient.SqlConnection,
                             tran As SqlClient.SqlTransaction,
                             mainId As Integer,
                             parentId As Integer,
                             refs As List(Of String))
    Dim sql As String = "
        INSERT INTO EdiDetail (
            MainId, ParentDetailId, LoopType, LoopSequence,
            REF01, REF02
        )
        VALUES (
            @MainId, @ParentDetailId, @LoopType, @LoopSequence,
            @RefQual, @RefVal
        )"

    Dim seq As Integer = 1
    For Each r In refs
        ' 假設 "VR:ABC123" 這種格式 -> 自行 split
        Dim parts = r.Split(":"c)
        Dim ref01 = parts(0)
        Dim ref02 = If(parts.Length > 1, parts(1), "")

        Using cmd As New SqlClient.SqlCommand(sql, conn, tran)
            cmd.Parameters.AddWithValue("@MainId", mainId)
            cmd.Parameters.AddWithValue("@ParentDetailId", parentId)
            cmd.Parameters.AddWithValue("@LoopType", "REF")
            cmd.Parameters.AddWithValue("@LoopSequence", seq)
            cmd.Parameters.AddWithValue("@RefQual", ref01)
            cmd.Parameters.AddWithValue("@RefVal", ref02)

            cmd.ExecuteNonQuery()
            seq += 1
        End Using
    Next
End Sub

> 以上示例顯示「N9 Loop」及其子段 (MSG, REF) 的寫法；其它段 (FOB, ITD, SAC, DTM, N1…) 都類似。只要將 LoopType、對應欄位、ParentDetailId 設定正確即可。




---

3.3 寫入 N1 (頂層)

Private Sub InsertTopN1Loops(conn As SqlClient.SqlConnection,
                             tran As SqlClient.SqlTransaction,
                             mainId As Integer,
                             n1Loops As List(Of N1Loop))
    Dim sql As String = "
        INSERT INTO EdiDetail (
            MainId, ParentDetailId, LoopType, LoopSequence,
            N101, N102, N103, N104, N301, N302, N401, N402, N403, N404
        )
        VALUES (
            @MainId, @ParentId, @LoopType, @LoopSequence,
            @N101, @N102, @N103, @N104, 
            @N301, @N302, @N401, @N402, @N403, @N404
        );
        SELECT SCOPE_IDENTITY();"

    Dim seq As Integer = 1
    For Each n1 In n1Loops
        Using cmd As New SqlClient.SqlCommand(sql, conn, tran)
            cmd.Parameters.AddWithValue("@MainId", mainId)
            cmd.Parameters.AddWithValue("@ParentId", DBNull.Value) ' 頂層
            cmd.Parameters.AddWithValue("@LoopType", "N1")
            cmd.Parameters.AddWithValue("@LoopSequence", seq)
            cmd.Parameters.AddWithValue("@N101", n1.PartyCode)
            cmd.Parameters.AddWithValue("@N102", n1.PartyName)
            cmd.Parameters.AddWithValue("@N103", DBNull.Value)
            cmd.Parameters.AddWithValue("@N104", DBNull.Value)
            cmd.Parameters.AddWithValue("@N301", n1.Address)
            cmd.Parameters.AddWithValue("@N302", DBNull.Value)
            cmd.Parameters.AddWithValue("@N401", n1.City)
            cmd.Parameters.AddWithValue("@N402", n1.State)
            cmd.Parameters.AddWithValue("@N403", n1.Zip)
            cmd.Parameters.AddWithValue("@N404", DBNull.Value)

            Dim n1DetailId As Integer = CInt(cmd.ExecuteScalar())

            ' 如果 N1 Loop 有 REF，可繼續呼叫 Insert REF
            InsertN1LoopRefs(conn, tran, mainId, n1DetailId, n1.Refs)

            seq += 1
        End Using
    Next
End Sub


---

3.4 寫入 PO1 行項 + 子 N1 Loop

Private Sub InsertPO1Loop(conn As SqlClient.SqlConnection,
                          tran As SqlClient.SqlTransaction,
                          mainId As Integer,
                          line As PO1LineItem)
    ' 1) 寫入 PO1 自己
    Dim sqlPO1 As String = "
        INSERT INTO EdiDetail (
            MainId, ParentDetailId, LoopType, LoopSequence,
            PO101, PO102, PO104
        )
        VALUES (
            @MainId, NULL, 'PO1', @LoopSequence,
            @LineNumber, @Qty, @UnitPrice
        );
        SELECT SCOPE_IDENTITY();"

    ' 先假設每個PO1的LoopSequence就是它在 List 中的index (此處簡化直接寫1)
    Using cmd As New SqlClient.SqlCommand(sqlPO1, conn, tran)
        cmd.Parameters.AddWithValue("@MainId", mainId)
        cmd.Parameters.AddWithValue("@LoopSequence", 1)  ' 若多筆再傳入正確序號
        cmd.Parameters.AddWithValue("@LineNumber", line.LineNumber)
        cmd.Parameters.AddWithValue("@Qty", line.Quantity)
        cmd.Parameters.AddWithValue("@UnitPrice", line.UnitPrice)

        Dim po1Id As Integer = CInt(cmd.ExecuteScalar())

        ' 2) 寫入 PID (Descriptions)
        InsertPID(conn, tran, mainId, po1Id, line.Descriptions)

        ' 3) 寫入 SAC
        InsertLineSAC(conn, tran, mainId, po1Id, line.LineSACs)

        ' 4) 寫入行項下的 N1 Sub-Loop
        InsertLineN1SubLoops(conn, tran, mainId, po1Id, line.N1SubLoops)
    End Using
End Sub


---

四、簡要總結

1. 概念核心

每個段 (N9, N1, REF, SAC, ITD, DTM, MSG, PO1…) -> 都對應一筆 INSERT INTO EdiDetail。

LoopType 決定這筆是什麼段；LoopSequence 區分第幾筆同類段；ParentDetailId 決定是否掛在父層。



2. 巢狀邏輯

若該段掛在「上一層」(例如某個 N9) 下，就把 ParentDetailId 設成上層的 DetailId；否則填 NULL 表示最外層。



3. 大量 INSERT

通常會包在 Transaction 裡面，避免出錯時留下不完整資料。



4. 程式結構

可像上面範例：用一堆 InsertXXX() 函式來分段處理（N9 Loop、N1 Loop、PO1 Loop…），插完再插子段 (MSG、REF…)。

解析好的資料 (Class) -> 呼叫對應的 Insert 函式 -> 撰寫適合的 SQL (或 ORM) -> 填入欄位。





---

這樣的做法就能在「第二階段解析」完成後，將各種段 (Header、Loop、Line) 寫入資料庫的 EdiDetail 內，並可確保巢狀關係 (ParentDetailId) 與 LoopType 一致、順序正確。

