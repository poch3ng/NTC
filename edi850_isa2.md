以下示範一個「較直觀好懂」的 VB.NET 程式範例，用來解析 EDI 段，包含：

ISA/IEA（Interchange）

GS/GE（Functional Group）

ST/SE（Transaction Set）

N9、N1、PO1、CTT 等「頂層 Loop」

PO1 中的 N1 Sub-Loop

REF、DTM 等常見補充段


整體邏輯盡量簡化成「單一狀態變數 (activeLoop) + 巢狀狀態 (inPO1SubLoop)」，輔以多個 List 來收集段，最後一次處理。請依實際需求（欄位、順序、可能重複性）再作調整。


---

程式概念 (更直觀)

1. 單一變數 activeLoop

用一個 String 變數來記錄當前在哪個「頂層 Loop」："N9", "N1", "PO1", "CTT", 或 "" (無)。

遇到新的頂層段（N9/N1/PO1/CTT）時，先關閉之前的 Loop，再開啟新的。



2. 巢狀狀態 inPO1SubLoop

若 activeLoop="PO1"，再遇到 N1 時，表示進入或切換到「PO1 裡的 N1 Sub-Loop」。

用 inPO1SubLoop As Boolean 簡單標示「目前是否在 PO1 的 N1 子區塊」。



3. 一個函式 CloseActiveLoop()

當想「切換 Loop」或「交易結束 (SE/GE/IEA)」時，就呼叫它，依照 activeLoop 的值，去處理收集到的段。

收完後，把對應的暫存清單清空、activeLoop 設為空字串。



4. 針對外層結構 (ISA/GS/ST)

簡化處理：看到 ISA/IEA、GS/GE、ST/SE 時，先「關閉當前 Loop」，然後把段印出或暫存即可。

沒有做多重交易或多重 Functional Group 配對，因為要保持程式簡短易懂；實際上可以視需要擴充。



5. REF、DTM 等

依照「目前在哪個 Loop (或子 Loop)」來決定把段丟到哪裡處理。





---

完整程式碼 (VB.NET)

Imports System.IO

Module EDISimpleParser
    '=== 狀態：紀錄目前在哪一個「頂層」Loop ===
    Dim activeLoop As String = ""  ' 可為 "N9", "N1", "PO1", "CTT", 或 ""

    '=== 巢狀狀態：PO1 底下的 N1 Sub-Loop ===
    Dim inPO1SubLoop As Boolean = False

    '=== 暫存各頂層 Loop 的段落 ===
    Dim N9Segments As New List(Of String)
    Dim N1Segments As New List(Of String)
    Dim PO1Segments As New List(Of String)
    Dim CTTSegments As New List(Of String)

    '=== 暫存 PO1 下的 N1 Sub-Loop ===
    Dim PO1N1Segments As New List(Of String)

    Sub Main()
        Dim ediFilePath As String = "C:\path\to\your\file.edi"
        Dim ediContent As String = File.ReadAllText(ediFilePath)

        ' 常見分隔：段分隔 "~"，欄位分隔 "*"
        Dim segments = ediContent.Split({"~"}, StringSplitOptions.RemoveEmptyEntries)

        For Each rawSeg In segments
            Dim seg = rawSeg.Trim()
            If seg = "" Then Continue For

            Dim elements = seg.Split("*"c)
            Dim segID = elements(0).Trim()

            Select Case segID
                '---------------------------------------------------------
                '  (A) 交換層 & 群組層 & 交易層 (ISA/GS/ST... etc.)
                '---------------------------------------------------------
                Case "ISA"
                    CloseCurrentLoop()  ' 先關閉可能正在做的 Loop
                    Console.WriteLine($"[ISA] {seg}")  ' 簡化處理
                Case "IEA"
                    CloseCurrentLoop()
                    Console.WriteLine($"[IEA] {seg}")

                Case "GS"
                    CloseCurrentLoop()
                    Console.WriteLine($"[GS] {seg}")
                Case "GE"
                    CloseCurrentLoop()
                    Console.WriteLine($"[GE] {seg}")

                Case "ST"
                    CloseCurrentLoop()
                    Console.WriteLine($"[ST] {seg}")
                Case "SE"
                    CloseCurrentLoop()
                    Console.WriteLine($"[SE] {seg}")

                '---------------------------------------------------------
                '  (B) 頂層 Loop: N9, N1, PO1, CTT
                '---------------------------------------------------------
                Case "N9"
                    ' 切到 N9 Loop (先關閉舊的)
                    CloseCurrentLoop()
                    activeLoop = "N9"
                    N9Segments.Add(seg)

                Case "N1"
                    ' 可能是頂層 N1 或 PO1 下的 N1
                    If activeLoop = "PO1" Then
                        ' => PO1 裡的 N1 Sub-Loop
                        If inPO1SubLoop Then
                            ' 關閉前一個 Sub-Loop
                            ProcessPO1N1Loop()
                        End If

                        inPO1SubLoop = True
                        PO1N1Segments.Clear()
                        PO1N1Segments.Add(seg)
                    Else
                        ' => 頂層 N1
                        CloseCurrentLoop()
                        activeLoop = "N1"
                        N1Segments.Add(seg)
                    End If

                Case "PO1"
                    ' 切到 PO1 Loop
                    CloseCurrentLoop()
                    activeLoop = "PO1"
                    PO1Segments.Add(seg)
                    inPO1SubLoop = False  ' 初始時還沒進到 N1 子 Loop

                Case "CTT"
                    ' 切到 CTT Loop
                    CloseCurrentLoop()
                    activeLoop = "CTT"
                    CTTSegments.Add(seg)

                '---------------------------------------------------------
                '  (C) 子段 (REF, DTM, N3, N4...) 分配到當前所在的 Loop
                '---------------------------------------------------------
                Case "REF", "DTM", "N3", "N4", "PID", "SAC"
                    ' 若正在 PO1 Loop 內，且也在 N1 Sub-Loop
                    If activeLoop = "PO1" AndAlso inPO1SubLoop Then
                        PO1N1Segments.Add(seg)
                    Else
                        ' 否則歸到頂層所在的 Loop
                        Select Case activeLoop
                            Case "N9"
                                N9Segments.Add(seg)
                            Case "N1"
                                N1Segments.Add(seg)
                            Case "PO1"
                                PO1Segments.Add(seg)
                            Case "CTT"
                                CTTSegments.Add(seg)
                            Case Else
                                Console.WriteLine($"[Outside any loop] {seg}")
                        End Select
                    End If

                '---------------------------------------------------------
                '  (D) 其它：直接依 activeLoop 丟進對應暫存
                '---------------------------------------------------------
                Case Else
                    If activeLoop = "PO1" AndAlso inPO1SubLoop Then
                        PO1N1Segments.Add(seg)
                    Else
                        Select Case activeLoop
                            Case "N9"  : N9Segments.Add(seg)
                            Case "N1"  : N1Segments.Add(seg)
                            Case "PO1" : PO1Segments.Add(seg)
                            Case "CTT" : CTTSegments.Add(seg)
                            Case Else  : Console.WriteLine($"[?? Outside] {seg}")
                        End Select
                    End If
            End Select
        Next

        '=== 迴圈結束後，如有未處理的 Loop，要做最後收尾 ===
        CloseCurrentLoop()

        Console.WriteLine("=== Parsing Complete ===")
        Console.ReadLine()
    End Sub

    ''' <summary>
    ''' 關閉目前的頂層 Loop，並做一次處理 + 清空
    ''' </summary>
    Private Sub CloseCurrentLoop()
        If activeLoop = "" Then Return  ' 表示本來就沒有在任何 Loop

        ' 若目前在 PO1 Loop，還可能在 N1 Sub-Loop，需要先收尾
        If activeLoop = "PO1" AndAlso inPO1SubLoop Then
            ProcessPO1N1Loop()
        End If

        Select Case activeLoop
            Case "N9"
                ProcessN9Loop()
            Case "N1"
                ProcessN1Loop()
            Case "PO1"
                ProcessPO1Loop()
            Case "CTT"
                ProcessCTTLoop()
        End Select

        ' 清除狀態
        activeLoop = ""
        inPO1SubLoop = False
    End Sub

    '--------------------------------------------------------------------
    ' 各 Loop 的處理函式（範例：將收集到的段印出，或寫入資料庫）
    '--------------------------------------------------------------------

    Private Sub ProcessN9Loop()
        Console.WriteLine("[ProcessN9Loop] Start")
        For Each seg In N9Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N9"
                    Console.WriteLine($"  N9 => RefID={elements.ElementAtOrDefault(1)}, Desc={elements.ElementAtOrDefault(2)}")
                Case "REF"
                    Console.WriteLine($"  REF => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case "DTM"
                    Console.WriteLine($"  DTM => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case Else
                    Console.WriteLine($"  N9-other => {seg}")
            End Select
        Next
        N9Segments.Clear()
        Console.WriteLine("[ProcessN9Loop] End")
    End Sub

    Private Sub ProcessN1Loop()
        Console.WriteLine("[ProcessN1Loop] Start")
        For Each seg In N1Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N1"
                    Console.WriteLine($"  N1 => Code={elements.ElementAtOrDefault(1)}, Name={elements.ElementAtOrDefault(2)}")
                Case "REF"
                    Console.WriteLine($"  REF => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case "DTM"
                    Console.WriteLine($"  DTM => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case "N3"
                    Console.WriteLine($"  N3 => Address={elements.ElementAtOrDefault(1)}")
                Case "N4"
                    Console.WriteLine($"  N4 => City/State/Zip={elements.ElementAtOrDefault(1)}/{elements.ElementAtOrDefault(2)}/{elements.ElementAtOrDefault(3)}")
                Case Else
                    Console.WriteLine($"  N1-other => {seg}")
            End Select
        Next
        N1Segments.Clear()
        Console.WriteLine("[ProcessN1Loop] End")
    End Sub

    Private Sub ProcessPO1Loop()
        Console.WriteLine("[ProcessPO1Loop] Start")

        For Each seg In PO1Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "PO1"
                    Console.WriteLine($"  PO1 => LineNo={elements.ElementAtOrDefault(1)}, Qty={elements.ElementAtOrDefault(2)}, Price={elements.ElementAtOrDefault(4)}")
                Case "REF"
                    Console.WriteLine($"  REF => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case "DTM"
                    Console.WriteLine($"  DTM => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case "N3"
                    Console.WriteLine($"  N3 => Address={elements.ElementAtOrDefault(1)} (但通常 PO1 下的 N3 會在子 Loop)")
                Case "N4"
                    Console.WriteLine($"  N4 => City/State/Zip={elements.ElementAtOrDefault(1)}/{elements.ElementAtOrDefault(2)}/{elements.ElementAtOrDefault(3)}")
                Case "PID"
                    Console.WriteLine($"  PID => Desc={elements.ElementAtOrDefault(4)}")
                Case "SAC"
                    Console.WriteLine($"  SAC => Code={elements.ElementAtOrDefault(1)}, Amt={elements.ElementAtOrDefault(5)}")
                Case Else
                    Console.WriteLine($"  PO1-other => {seg}")
            End Select
        Next
        PO1Segments.Clear()

        Console.WriteLine("[ProcessPO1Loop] End")
    End Sub

    ''' <summary>
    ''' PO1 下的 N1 Sub-Loop
    ''' </summary>
    Private Sub ProcessPO1N1Loop()
        Console.WriteLine("    => [ProcessPO1N1Loop] Start")
        For Each seg In PO1N1Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N1"
                    Console.WriteLine($"       N1 => Code={elements.ElementAtOrDefault(1)}, Name={elements.ElementAtOrDefault(2)} (PO1-Sub)")
                Case "N3"
                    Console.WriteLine($"       N3 => Address={elements.ElementAtOrDefault(1)}")
                Case "N4"
                    Console.WriteLine($"       N4 => City/State/Zip={elements.ElementAtOrDefault(1)}/{elements.ElementAtOrDefault(2)}/{elements.ElementAtOrDefault(3)}")
                Case "REF"
                    Console.WriteLine($"       REF => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case "DTM"
                    Console.WriteLine($"       DTM => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case Else
                    Console.WriteLine($"       Sub-other => {seg}")
            End Select
        Next
        PO1N1Segments.Clear()
        inPO1SubLoop = False  ' 關閉子 Loop 狀態
        Console.WriteLine("    => [ProcessPO1N1Loop] End")
    End Sub

    Private Sub ProcessCTTLoop()
        Console.WriteLine("[ProcessCTTLoop] Start")
        For Each seg In CTTSegments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "CTT"
                    Console.WriteLine($"  CTT => TotalLineItems={elements.ElementAtOrDefault(1)}, SumOfQty={elements.ElementAtOrDefault(2)}")
                Case "REF"
                    Console.WriteLine($"  REF => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case "DTM"
                    Console.WriteLine($"  DTM => {elements.ElementAtOrDefault(1)} / {elements.ElementAtOrDefault(2)}")
                Case Else
                    Console.WriteLine($"  CTT-other => {seg}")
            End Select
        Next
        CTTSegments.Clear()
        Console.WriteLine("[ProcessCTTLoop] End")
    End Sub
End Module


---

這樣做法的特點

1. 單一狀態 activeLoop：

減少程式中重複的 If/Else 判斷，只要知道「目前在哪個 Loop」，就把段加入對應的 List。

遇到新頂層段 (N9, N1, PO1, CTT) 就呼叫 CloseCurrentLoop()，再改變 activeLoop。



2. 子 Loop (N1 in PO1) 只用一個 inPO1SubLoop 布林做判斷：

遇到 N1 時，若 activeLoop="PO1" → 進入子 Loop；若已經在子 Loop，就先把舊子 Loop結束。

其餘段 (REF, DTM, N3, N4 等) 就分配到 PO1N1Segments (若 inPO1SubLoop=True) 或直接給 PO1Segments (頂層)。



3. 程式更直覺：

大部分的「多重布林狀態」合併成一個 activeLoop + 一個 inPO1SubLoop，視覺上更好理解，程式碼行數也更少。



4. 可擴充：

若還有其他子 Loop（如 PO1 下的 PID 可能再有子段…），可以再用類似的方法加一個布林或在 ProcessPO1Loop 裡面判斷。

若要處理「多個 ST/交易」、「多個 GS/功能群」、「多個 ISA/互換」之間的配對，則要再把 [ISA], [GS], [ST] 收集起來後，一筆筆配合控制號 (ISA13、GS06、ST02) 進行驗證。





---

總結

上述程式已經盡量簡化為：

單一「活躍 Loop」(activeLoop)

單一「PO1 子狀態」(inPO1SubLoop)

一個共用的 CloseCurrentLoop() 來一次收尾。


讀起來邏輯更直接：

碰到新頂層段 → 關閉舊的 → 開啟新的 → 收集段 → 結束再處理。

在 PO1 中若見到 N1 → 進到子 Loop → 收集 → 結束時再處理。


如此即可大致應付「ISA/GS/ST + (N9、N1、PO1、CTT) + PO1 N1 Sub-Loop + REF/DTM/N3/N4…」的情境，而不會有過於冗長、重複的狀態判斷。


