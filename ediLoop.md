下面提供一個 「單一程式循序掃描 + 狀態機」 的示例，用來解析含有 4 種 Loop (N9、N1、PO1、CTT) 的 EDI 內容。此方法概念簡單且直覺：

1. 逐段讀取 EDI (用分隔符號切割出每個 segment)。


2. 根據 Segment ID 決定當前是進入哪個 Loop，或是結束哪個 Loop。


3. 每一個 Loop (例如 N9 Loop) 的內部段，都暫時收集在一個清單 (List) 裡。


4. 當遇到下一個 Loop 的起始 (或交易結束) 時，代表上一個 Loop 收集結束，呼叫對應的處理函式，把它整理或寫入資料庫。



此範例僅示範「如何分段 + 儲存 + 偵測 Loop 開始/結束」，實際上你可以依照這個架構，進一步把收集到的欄位值寫進資料庫。請依實際業務需求調整欄位及程式邏輯。


---

程式碼範例 (VB.NET)

Imports System.IO

Module EDIParserLoops
    Sub Main()
        ' 假設你有一個含 N9、N1、PO1、CTT 等段落的 EDI 檔案
        Dim ediFilePath As String = "C:\path\to\your\file.edi"
        Dim ediContent As String = File.ReadAllText(ediFilePath)

        ' 一般常見：段分隔 "~"，元素分隔 "*"
        Dim segmentDelimiter As String = "~"
        Dim elementDelimiter As Char = "*"c

        ' 切割成 segment
        Dim segments As String() = ediContent.Split(New String() {segmentDelimiter}, StringSplitOptions.RemoveEmptyEntries)

        ' 建立 4 個狀態來追蹤是否在某個 Loop 裡
        Dim inN9Loop As Boolean = False
        Dim inN1Loop As Boolean = False
        Dim inPO1Loop As Boolean = False
        Dim inCTTLoop As Boolean = False

        ' 用來暫存各 Loop 的段落
        Dim N9Segments As New List(Of String)
        Dim N1Segments As New List(Of String)
        Dim PO1Segments As New List(Of String)
        Dim CTTSegments As New List(Of String)

        For Each rawSeg As String In segments
            Dim segment As String = rawSeg.Trim()
            If segment = "" Then Continue For

            Dim elements As String() = segment.Split(elementDelimiter)
            Dim segID As String = elements(0).Trim()

            Select Case segID
                '======================
                ' N9 Loop
                '======================
                Case "N9"
                    ' 若之前正在其他 loop，需要先結束
                    If inN1Loop Then
                        ProcessN1Loop(N1Segments)
                        N1Segments.Clear()
                        inN1Loop = False
                    ElseIf inPO1Loop Then
                        ProcessPO1Loop(PO1Segments)
                        PO1Segments.Clear()
                        inPO1Loop = False
                    ElseIf inCTTLoop Then
                        ProcessCTTLoop(CTTSegments)
                        CTTSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 開始 N9 Loop
                    inN9Loop = True
                    N9Segments.Add(segment)

                Case "N1"
                    ' 若之前在 N9 Loop，先結束它
                    If inN9Loop Then
                        ProcessN9Loop(N9Segments)
                        N9Segments.Clear()
                        inN9Loop = False
                    ElseIf inPO1Loop Then
                        ProcessPO1Loop(PO1Segments)
                        PO1Segments.Clear()
                        inPO1Loop = False
                    ElseIf inCTTLoop Then
                        ProcessCTTLoop(CTTSegments)
                        CTTSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 開始 N1 Loop
                    inN1Loop = True
                    N1Segments.Add(segment)

                Case "PO1"
                    ' 若之前在 N9 Loop 或 N1 Loop，先結束
                    If inN9Loop Then
                        ProcessN9Loop(N9Segments)
                        N9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inN1Loop Then
                        ProcessN1Loop(N1Segments)
                        N1Segments.Clear()
                        inN1Loop = False
                    End If
                    If inCTTLoop Then
                        ProcessCTTLoop(CTTSegments)
                        CTTSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 開始 PO1 Loop
                    inPO1Loop = True
                    PO1Segments.Add(segment)

                Case "CTT"
                    ' 若之前在 PO1 Loop，就先結束
                    If inPO1Loop Then
                        ProcessPO1Loop(PO1Segments)
                        PO1Segments.Clear()
                        inPO1Loop = False
                    End If
                    If inN9Loop Then
                        ProcessN9Loop(N9Segments)
                        N9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inN1Loop Then
                        ProcessN1Loop(N1Segments)
                        N1Segments.Clear()
                        inN1Loop = False
                    End If

                    ' 開始 CTT Loop
                    inCTTLoop = True
                    CTTSegments.Add(segment)

                Case "SE", "GE", "IEA"
                    ' 通常遇到 SE (Transaction Set Trailer) 或 GE/IEA (結尾) 就代表所有 Loop 都可結束
                    If inN9Loop Then
                        ProcessN9Loop(N9Segments)
                        N9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inN1Loop Then
                        ProcessN1Loop(N1Segments)
                        N1Segments.Clear()
                        inN1Loop = False
                    End If
                    If inPO1Loop Then
                        ProcessPO1Loop(PO1Segments)
                        PO1Segments.Clear()
                        inPO1Loop = False
                    End If
                    If inCTTLoop Then
                        ProcessCTTLoop(CTTSegments)
                        CTTSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 處理結束段
                    Console.WriteLine($"Segment {segID} found, end of transaction.")
                    ' 依需求做最後收尾 (e.g. Transaction set 結束)

                Case Else
                    ' 不是 loop 起始，也可能是屬於已進入的某個 loop 裡的段 (如 N9 Loop 下的其他 REF、MSG…)
                    If inN9Loop Then
                        N9Segments.Add(segment)
                    ElseIf inN1Loop Then
                        N1Segments.Add(segment)
                    ElseIf inPO1Loop Then
                        PO1Segments.Add(segment)
                    ElseIf inCTTLoop Then
                        CTTSegments.Add(segment)
                    Else
                        ' 不屬於任何 loop，就直接看要不要特別處理
                        Console.WriteLine($"Other segment: {segID}")
                    End If
            End Select
        Next

        ' 如果檔案結尾前還在某個 Loop 中，也要處理殘留
        If inN9Loop Then
            ProcessN9Loop(N9Segments)
        End If
        If inN1Loop Then
            ProcessN1Loop(N1Segments)
        End If
        If inPO1Loop Then
            ProcessPO1Loop(PO1Segments)
        End If
        If inCTTLoop Then
            ProcessCTTLoop(CTTSegments)
        End If

        Console.WriteLine("=== EDI Parsing Complete ===")
        Console.ReadLine()
    End Sub

    '=====================
    ' 以下為各 Loop 的範例處理函式
    '=====================
    Private Sub ProcessN9Loop(loopSegments As List(Of String))
        Console.WriteLine("Processing N9 Loop...")
        ' TODO: 這裡把 N9 Loop 的資料切割後，寫入資料庫或做其他處理
        For Each seg In loopSegments
            Dim elements = seg.Split("*"c)
            ' N9(0)*RefID(1)*FreeForm(2)...
            If elements(0) = "N9" Then
                Console.WriteLine($"  N9 Reference: {elements(1)} / {elements(2)}")
            Else
                ' 可能是 Loop 下的其他段，如 MSG、REF 等
                Console.WriteLine($"  -> {elements(0)}: {seg}")
            End If
        Next
        Console.WriteLine("N9 Loop Processed.")
    End Sub

    Private Sub ProcessN1Loop(loopSegments As List(Of String))
        Console.WriteLine("Processing N1 Loop...")
        For Each seg In loopSegments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N1"
                    Console.WriteLine($"  N1: {elements(1)} / {elements(2)}")
                Case "N3"
                    Console.WriteLine($"  N3: {elements(1)}")
                Case "N4"
                    Console.WriteLine($"  N4: {elements(1)} / {elements(2)} / {elements(3)}")
                Case "REF"
                    Console.WriteLine($"  REF: {elements(1)} / {elements(2)}")
                Case Else
                    Console.WriteLine($"  -> {elements(0)}: {seg}")
            End Select
        Next
        Console.WriteLine("N1 Loop Processed.")
    End Sub

    Private Sub ProcessPO1Loop(loopSegments As List(Of String))
        Console.WriteLine("Processing PO1 Loop...")
        For Each seg In loopSegments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "PO1"
                    Console.WriteLine($"  PO1: LineNo={elements(1)}, Qty={elements(2)}, Price={elements(4)}")
                Case "PID"
                    Console.WriteLine($"  PID: Description={elements(4)}")
                Case "SAC"
                    Console.WriteLine($"  SAC: Charge/Allowance={elements(1)} / {elements(2)}")
                Case Else
                    Console.WriteLine($"  -> {elements(0)}: {seg}")
            End Select
        Next
        Console.WriteLine("PO1 Loop Processed.")
    End Sub

    Private Sub ProcessCTTLoop(loopSegments As List(Of String))
        Console.WriteLine("Processing CTT Loop...")
        For Each seg In loopSegments
            Dim elements = seg.Split("*"c)
            If elements(0) = "CTT" Then
                Console.WriteLine($"  CTT: TotalLineItems={elements(1)}, SumOfQty={If(elements.Length > 2, elements(2), "N/A")}")
            Else
                Console.WriteLine($"  -> {elements(0)}: {seg}")
            End If
        Next
        Console.WriteLine("CTT Loop Processed.")
    End Sub

End Module


---

解析邏輯重點

1. 狀態機 (State Machine)

用四個 Boolean 變數 (inN9Loop, inN1Loop, inPO1Loop, inCTTLoop) 來代表「是否正位於某 Loop」

每次看到新的 Loop 起始 (N9、N1、PO1、CTT)，先結束(處理)舊的 Loop，再啟動新的



2. 收集段落

在進入某個 Loop 之後，把該 Loop 裡的所有段都加入對應的 List(Of String) (如 N9Segments)

碰到下一個 Loop 或交易結尾 (SE/GE/IEA)，就把收集好的段一次丟給 ProcessN9Loop() / ProcessN1Loop() … 等函式進行解析或入庫



3. 各 Loop 的 Process 函式

ProcessN9Loop(), ProcessN1Loop(), ProcessPO1Loop(), ProcessCTTLoop()

先把段用 Split("*"c) 切割成欄位，再依照特定段 (N9、N3、REF、PO1、PID、CTT… 等) 做對應的邏輯

真實環境下，可把這些欄位值插入資料庫



4. 簡化

若 Loop 順序總是固定 (N9 → N1 → PO1 → CTT)，可以更簡單：每個 Loop 結束時自動進入下一個，不用四個布林變數同時判斷。

不過，一般 EDI 可能同類的 Loop 會重複多次，所以還是需要具備判斷「何時開啟 / 何時結束」的機制。





---

總結

最簡單的思路 就是用 「單一程式循序掃描」+「狀態機」。

看見下個 Loop 的 Segment ID，就關閉上個 Loop，並開啟新 Loop。

Loop 內的多個段收集起來，一次性處理或寫入資料庫。

這在 VB.NET 實作上，就是維護幾個布林旗標 + 幾個 List，搭配一個 Select Case 判斷哪個 Segment 屬於哪個 Loop。

程式碼中已示範 N9 / N1 / PO1 / CTT Loop 的切換與處理，你可以根據實際業務需求進一步優化、拆分或加上錯誤處理。


這樣即可達到「多個 Loop」的簡易解析，讓你在 EDI 850（或其他交易）裡能對特定 Loop 做分段處理。祝順利完成!



以下以 EDI 850 常見的 「Loop」（例如 N1 Loop、PO1 Loop）為例，示範如何在程式邏輯中處理「重複段 (Loops)」。所謂 Loop，通常是指由一系列 segment 組合而成，且可能重複出現的區塊，例如：

N1 Loop：包含 N1（Party Identification）、N3（Address Information）、N4（Geographic Location）、REF… 等段

PO1 Loop：包含 PO1（Line Item Detail）及延伸段（PID、SAC…等）


以下提供兩種解析思路：


---

1. 條件判斷式：偵測「Loop 起始」和「結束」

最直覺的方式是：

偵測到某個特定 segment (例如 N1)，代表進入一個新的 N1 Loop

持續讀取下一個或多個 segment (N3、N4…)，直到遇到「下個 N1」或「PO1」或「SE」等，視為 N1 Loop 結束


範例程式 (VB.NET)

Imports System.IO

Module EDIParserWithLoop
    Sub Main()
        Dim ediFilePath As String = "C:\path\to\your\file.edi"
        Dim ediContent As String = File.ReadAllText(ediFilePath)

        Dim segmentDelimiter As String = "~"
        Dim elementDelimiter As Char = "*"c

        ' 切割段
        Dim segments As String() = ediContent.Split(
            New String() {segmentDelimiter}, 
            StringSplitOptions.RemoveEmptyEntries
        )

        ' 用來存放 N1 Loop 的臨時區段
        Dim currentN1LoopSegments As New List(Of String)

        ' 狀態紀錄：是否目前在 N1 Loop 中
        Dim inN1Loop As Boolean = False

        For Each seg As String In segments
            Dim segment = seg.Trim()
            If segment = "" Then Continue For

            ' 切割欄位
            Dim elements As String() = segment.Split(elementDelimiter)
            Dim segmentID As String = elements(0).Trim()

            Select Case segmentID
                Case "N1"
                    ' 如果偵測到新的 N1，就代表先前 N1 Loop (若有) 可以收集完
                    If inN1Loop Then
                        ' 這裡可以把之前收集的 currentN1LoopSegments 解析存到資料庫
                        ProcessN1Loop(currentN1LoopSegments)
                        currentN1LoopSegments.Clear()
                    End If

                    ' 開始新的 N1 Loop
                    inN1Loop = True
                    currentN1LoopSegments.Add(segment)

                Case "N3", "N4", "REF"
                    ' 若目前在 N1 Loop，就繼續收集
                    If inN1Loop Then
                        currentN1LoopSegments.Add(segment)
                    Else
                        ' 若不在 N1 Loop，這些段可以根據實際需求另作處理
                    End If

                Case "PO1", "SE"
                    ' 遇到 PO1 或 SE，代表 N1 Loop 應該結束
                    If inN1Loop Then
                        ProcessN1Loop(currentN1LoopSegments)
                        currentN1LoopSegments.Clear()
                        inN1Loop = False
                    End If

                    ' 再處理 PO1 或 SE
                    If segmentID = "PO1" Then
                        ' 這裡可以進入 PO1 Loop 或直接處理
                        Console.WriteLine("PO1 segment found.")
                    ElseIf segmentID = "SE" Then
                        Console.WriteLine("Transaction Set Trailer (SE) found.")
                    End If

                Case Else
                    ' 其它段 (ISA, GS, ST, BEG, GE, IEA, 或不屬於某個 Loop 的)
                    If inN1Loop Then
                        ' 如果目前還在 N1 Loop，但又遇到不屬於 N1 Loop 的段，也視為 N1 Loop 結束
                        ProcessN1Loop(currentN1LoopSegments)
                        currentN1LoopSegments.Clear()
                        inN1Loop = False
                    End If
            End Select
        Next

        ' 若檔案最後結束前還在 N1 Loop，也要處理殘餘段
        If inN1Loop Then
            ProcessN1Loop(currentN1LoopSegments)
        End If

        Console.WriteLine("=== Parsing Complete ===")
        Console.ReadLine()
    End Sub

    ''' <summary>
    ''' 範例：處理 N1 Loop 裡的段
    ''' </summary>
    Private Sub ProcessN1Loop(n1LoopSegments As List(Of String))
        Console.WriteLine("Processing N1 Loop...")

        ' 此時 n1LoopSegments 中包含 N1、N3、N4、REF (若有)
        For Each seg In n1LoopSegments
            ' 可以再用前面相同方法，按 elementDelimiter 拆解
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N1"
                    Console.WriteLine($"  => N1 Found. Code: {elements(1)}, Name: {elements(2)}")
                Case "N3"
                    Console.WriteLine($"  => N3 Found. Address: {elements(1)}")
                Case "N4"
                    Console.WriteLine($"  => N4 Found. City/State/Zip: {elements(1)}/{elements(2)}/{elements(3)}")
                Case "REF"
                    Console.WriteLine($"  => REF Found. Type: {elements(1)}, Value: {elements(2)}")
            End Select
        Next

        ' 這裡可依需求把解析出的資料寫入資料庫
        Console.WriteLine("N1 Loop Processed.")
    End Sub
End Module

重點

1. 狀態機 (State Machine) 思維

用 inN1Loop 等布林來追蹤「現在是否位在某個 Loop」。

如果遇到新一個 N1，就把前一個 N1 Loop 結束 (如果存在)。

若遇到 PO1 或 SE 或其他會中斷 N1 Loop 的段，就結束該 N1 Loop。



2. 收集完整段後再解析

暫存在 List(Of String)，最後一次性處理、寫入資料庫。

更複雜的 Loop（如 PO1 下可能還有 PID、SAC 等子段），也可以類似做法處理。





---

2. 逐段解析 + 即時判斷

另一種方法是「每遇到一個 segment，就即時處理」。

在解析到 N1 時，開一個物件或資料結構暫存 N1 內容；

如果隨後遇到 N3、N4、REF，就把它附加到同一個物件；

遇到下一個 N1 (或 PO1、SE) 時，就代表前面 N1 Loop 結束，將該物件寫入資料庫，再開新物件。


這和上面程式邏輯類似，只是將「收集後再一次處理」改成「即時解析、處理」。整體還是需要 判斷何時開始 / 結束 Loop。


---

3. 小結

Loop 本質上就是一組會重複出現的 segment 群。你需要一個機制辨認「Loop 的開始」(例如 N1) 與「Loop 的結束」(可能下一個 N1、或下一個交易段 PO1、或 SE)。

常見做法是利用 狀態機 (State Machine) 或 收集清單 (List) 的方式，先收集完整段，再一次解析寫庫。

針對 PO1 Loop，可用同樣邏輯：如果偵測到 PO1，就進入 PO1 Loop，直到遇到 PO1 或 SE 才結束這段。若 PO1 下還有 PID、SAC… 等段，也都收集進相同清單中。


只要掌握「解析 segment → 判斷 loop 開始/結束 → 收集 & 寫庫」的流程，就能彈性處理各種 EDI Loop。希望以上能幫助你理清 Loop 的解析做法。

