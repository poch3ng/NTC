下面給你一個「單階段 (Single-Phase)」解析並直接寫入資料庫的 VB.NET 範例程式碼**(不再使用任何 PurchaseOrder850 等類別)**，示範如何：

1. 讀取整個 EDI 檔案。


2. 找到每個交易 (ST~SE)。


3. 建立對應的 EdiMain 紀錄 (一個 ST~SE 對應一筆主表)。


4. 直接把區間內的所有 Segment 逐一拆解後，插入到 EdiDetail。



> 此程式僅示範「整體架構與做法」，無完整防呆與錯誤處理，請依實務需求增修。
同時亦簡化許多 Segment 欄位(例如只示範 N9 的 N901、N902 等)，實務中請自行補足。




---

SinglePhaseEdiParser.vb

Option Explicit On
Option Strict On
Imports System
Imports System.IO
Imports System.Data.SqlClient
Imports System.Linq

Module SinglePhaseEdiParser

    '--------------------------------------------------------------
    ' 你在程式中指定的資料庫連線字串
    '--------------------------------------------------------------
    Private Const CONNECTION_STRING As String = "Data Source=YOUR_SERVER;Initial Catalog=YOUR_DB;Integrated Security=True;"

    Sub Main()
        ' 1) 指定 EDI 檔案路徑
        Dim ediFilePath As String = "C:\path\to\your\file.edi"

        ' 2) 解析 & 寫入
        ParseAndSaveEdiFile(ediFilePath)

        Console.WriteLine("=== EDI Parsing & Saving Complete ===")
        Console.ReadLine()
    End Sub

    ''' <summary>
    ''' 單階段：讀整份 EDI → 找出 ST~SE → 建立 EdiMain → 直接把每個段插入 EdiDetail
    ''' </summary>
    Private Sub ParseAndSaveEdiFile(filePath As String)
        Dim ediContent As String = File.ReadAllText(filePath)

        ' 典型 X12 做法：用 "~" 當段分隔。若你有其他分隔符(如環境不同)，請自行調整
        Dim segments As String() = ediContent.Split({"~"c}, StringSplitOptions.RemoveEmptyEntries)

        ' 用來暫存「目前正在解析的 交易 (ST~SE)」資訊
        Dim currentSegments As New List(Of String)
        Dim currentTransactionID As String = Nothing   ' ST-01
        Dim currentControlNumber As String = Nothing   ' ST-02

        ' 我們要在資料庫開啟連線、整批寫入
        Using conn As New SqlConnection(CONNECTION_STRING)
            conn.Open()

            ' 也可以用 Transaction 包起來，保證原子性
            Using tran = conn.BeginTransaction()
                Try
                    For Each rawSeg In segments
                        Dim seg As String = rawSeg.Trim()
                        If String.IsNullOrEmpty(seg) Then Continue For

                        ' 拆解元素
                        Dim elements As String() = seg.Split("*"c)
                        Dim segID As String = elements(0).ToUpper()

                        Select Case segID
                            Case "ISA"
                                ' 通常是一份檔案的最外層交換起點，可在這裡視需要存入 EdiMain 或做其他紀錄
                                ' (範例略過，若要存就自行 INSERT)

                            Case "GS"
                                ' 功能群組起點 (範例略過)

                            Case "ST"
                                ' 遇到新的交易起點
                                currentSegments.Clear()
                                currentSegments.Add(seg)
                                currentTransactionID = elements.ElementAtOrDefault(1) ' ST-01
                                currentControlNumber = elements.ElementAtOrDefault(2) ' ST-02

                            Case "SE"
                                ' 交易結束，把 SE 也加進去
                                currentSegments.Add(seg)

                                ' 3) 直接寫入資料庫：先建 EdiMain，再把 ST~SE 內所有段寫入 EdiDetail
                                Dim mainId As Integer = InsertEdiMain(conn, tran, currentTransactionID, currentControlNumber, currentSegments)
                                InsertAllSegmentsIntoDetail(conn, tran, mainId, currentSegments)

                                ' 結束後清空，以便讀取下個交易
                                currentSegments.Clear()
                                currentTransactionID = Nothing
                                currentControlNumber = Nothing

                            Case "GE", "IEA"
                                ' 功能群組結束、交換結束 (範例略過)
                            Case Else
                                ' 一般段
                                If currentTransactionID IsNot Nothing Then
                                    ' 表示我們正在 ST~SE 之間
                                    currentSegments.Add(seg)
                                End If
                        End Select
                    Next

                    tran.Commit()
                Catch ex As Exception
                    Console.WriteLine("發生例外：" & ex.Message)
                    tran.Rollback()
                End Try
            End Using
        End Using
    End Sub

    '--------------------------------------------------------------
    ' (A) 建立/插入一筆 EdiMain
    '     假設：每個 ST~SE 交易一筆 EdiMain
    '--------------------------------------------------------------
    Private Function InsertEdiMain(conn As SqlConnection,
                                   tran As SqlTransaction,
                                   st01 As String,   ' TransactionSetID
                                   st02 As String,   ' ControlNumber
                                   segs As List(Of String)) As Integer

        ' 你可能會想從 segs 裡找出 BEG、CUR 等欄位，填進去 EdiMain
        ' 這裡純範例：只把 ST-01, ST-02 存入
        Dim sql As String =
        "INSERT INTO EdiMain (StControlNumber, StSegment, CreateDate)
         OUTPUT INSERTED.MainId
         VALUES (@st02, @stSegment, GETDATE());"

        ' 將 ST 原始文字找出來 (segs[0] 就是 ST)
        Dim stRaw As String = If(segs.Count > 0, segs(0), "")
        Using cmd As New SqlCommand(sql, conn, tran)
            cmd.Parameters.AddWithValue("@st02", If(st02, CType(DBNull.Value, Object)))
            cmd.Parameters.AddWithValue("@stSegment", stRaw)
            Dim newId As Integer = Convert.ToInt32(cmd.ExecuteScalar())
            Return newId
        End Using
    End Function

    '--------------------------------------------------------------
    ' (B) 將 ST~SE 之間所有 segment 分析後，逐條插入 EdiDetail
    '--------------------------------------------------------------
    Private Sub InsertAllSegmentsIntoDetail(conn As SqlConnection,
                                            tran As SqlTransaction,
                                            mainId As Integer,
                                            segs As List(Of String))

        ' 用來計算各 segment 出現的順序 (每種 segment 各自一組計數)
        Dim loopSeq As New Dictionary(Of String, Integer)(StringComparer.OrdinalIgnoreCase)

        For Each segment In segs
            Dim parts = segment.Split("*"c)
            Dim segID = parts(0).ToUpper()

            ' (1) 先決定要INSERT哪些欄位
            '     範例：只示範 N9、N1、REF、DTM、MSG…等
            '     若要包含 SAC、ITD、FOB…等，請自行加上 CASE。
            Dim loopType As String = segID
            If Not loopSeq.ContainsKey(loopType) Then
                loopSeq(loopType) = 0
            End If
            loopSeq(loopType) += 1

            ' (2) 依 segID 決定要填哪些欄位
            Dim n901 As String = Nothing
            Dim n902 As String = Nothing
            Dim n903 As String = Nothing
            Dim n1_01 As String = Nothing
            Dim n1_02 As String = Nothing
            Dim ref01 As String = Nothing
            Dim ref02 As String = Nothing
            Dim dtm01 As String = Nothing
            Dim dtm02 As String = Nothing
            Dim msg01 As String = Nothing

            Select Case segID
                Case "N9"
                    ' N9 的元素：N9*Qualifier*N902*N903*N904...
                    n901 = parts.ElementAtOrDefault(1)
                    n902 = parts.ElementAtOrDefault(2)
                    n903 = parts.ElementAtOrDefault(3)

                Case "N1"
                    ' N1 的元素：N1*EntityIDCode*Name*IDCodeQual*IDCode...
                    n1_01 = parts.ElementAtOrDefault(1) ' e.g. ST, BT...
                    n1_02 = parts.ElementAtOrDefault(2) ' e.g. "Some Name"

                Case "REF"
                    ' REF*Qualifier*Value
                    ref01 = parts.ElementAtOrDefault(1)
                    ref02 = parts.ElementAtOrDefault(2)

                Case "DTM"
                    ' DTM*Qualifier*Date*(Time)...
                    dtm01 = parts.ElementAtOrDefault(1)
                    dtm02 = parts.ElementAtOrDefault(2)

                Case "MSG"
                    ' MSG*Free-form text
                    msg01 = parts.ElementAtOrDefault(1)

                Case "ST"
                    ' 已在 InsertEdiMain 處理，這裡可以視需求再次存
                Case "SE"
                    ' 已在 InsertEdiMain 處理
                Case Else
                    ' 其他 Segment (FOB, ITD, SAC, PID, PER...) 都可在這裡加
                    ' 範例先略過
            End Select

            ' (3) INSERT 到 EdiDetail
            Dim sqlDetail As String =
            "INSERT INTO EdiDetail (
                MainId,
                LoopType,
                LoopSequence,
                N901, N902, N903,
                N101, N102,
                REF01, REF02,
                DTM01, DTM02,
                MSG01,
                CreateDate
            ) VALUES (
                @MainId,
                @LoopType,
                @LoopSeq,
                @N901, @N902, @N903,
                @N101, @N102,
                @REF01, @REF02,
                @DTM01, @DTM02,
                @MSG01,
                GETDATE()
            );"

            Using cmd As New SqlCommand(sqlDetail, conn, tran)
                cmd.Parameters.AddWithValue("@MainId", mainId)
                cmd.Parameters.AddWithValue("@LoopType", loopType)
                cmd.Parameters.AddWithValue("@LoopSeq", loopSeq(loopType))

                cmd.Parameters.AddWithValue("@N901", If(n901, CType(DBNull.Value, Object)))
                cmd.Parameters.AddWithValue("@N902", If(n902, CType(DBNull.Value, Object)))
                cmd.Parameters.AddWithValue("@N903", If(n903, CType(DBNull.Value, Object)))

                cmd.Parameters.AddWithValue("@N101", If(n1_01, CType(DBNull.Value, Object)))
                cmd.Parameters.AddWithValue("@N102", If(n1_02, CType(DBNull.Value, Object)))

                cmd.Parameters.AddWithValue("@REF01", If(ref01, CType(DBNull.Value, Object)))
                cmd.Parameters.AddWithValue("@REF02", If(ref02, CType(DBNull.Value, Object)))

                cmd.Parameters.AddWithValue("@DTM01", If(dtm01, CType(DBNull.Value, Object)))
                cmd.Parameters.AddWithValue("@DTM02", If(dtm02, CType(DBNull.Value, Object)))

                cmd.Parameters.AddWithValue("@MSG01", If(msg01, CType(DBNull.Value, Object)))

                cmd.ExecuteNonQuery()
            End Using
        Next
    End Sub

End Module

程式重點說明

1. 不再使用任何自訂類別 (例如 PurchaseOrder850)

直接在 InsertAllSegmentsIntoDetail() 裡，針對每個 Segment 做 SELECT CASE segID 分析，然後立刻 INSERT 到資料表。



2. EdiMain

每偵測到一個交易 (ST~SE) 就新增一筆。

可在這個流程中抓取需要的欄位 (如 BEG-03 當作 PoNumber)，再填入 EdiMain。上面程式為簡化，只存了 ST-02。



3. EdiDetail

每個 Segment 都會 INSERT 一次。

LoopType 直接用 SegmentID (N9, N1, REF, DTM...)；

其餘欄位 (N901、N902、N903... / N101, N102... / REF01, REF02… / DTM01, DTM02… / MSG01…) 根據 SELECT CASE segID 填值，其它皆 NULL。



4. LoopSequence

用一個 Dictionary(Of String, Integer) 來記錄 同類型 Segment 已經插入幾次，以方便區分多筆 (像多個 N9, 多個 DTM…)。

這裡未示範巢狀父子關係 (ParentDetailId)，若要處理 N9 下的 N1 就需要額外的邏輯來決定 parent Id；此範例走最簡化的「單層紀錄」。



5. 要插更多段

例如 SAC, ITD, FOB, PID… 只需在 Select Case segID 中加 Case "SAC": ...，並對應資料表欄位 (SAC01, SAC02…)，然後 INSERT。



6. 交易控制

程式中使用 Transaction 保護整個 ST~SE 的寫入過程，若中途出錯，可 Rollback。





---

結論

以上程式碼展示一個**「單階段」**作法：

1. 讀檔並逐段解析


2. 立即 INSERT 至 EdiMain / EdiDetail



優點：程式邏輯簡單，沒有額外的資料結構，段一讀到就可立刻寫入。

缺點：如果 EDI 結構很複雜或有多層巢狀，必須在 InsertAllSegmentsIntoDetail() 或先前階段就寫更多判斷 (例如誰是誰的父層)。


若你確定不需要多階層巢狀(或不需要保留父子結構)，此程式就非常直接可用。若要支援更複雜的 Loop 巢狀，也能在此基礎上自行擴充。祝開發順利!

