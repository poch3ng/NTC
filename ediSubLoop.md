在 EDI 實務上，Loop 結構往往會「巢狀 (Nested)」出現，例如：

主體是 PO1 Loop (Line Item)，

卻在 PO1 Loop 底下又有一個 N1 Loop (Party Identification) ，裡面包含 N3、N4 等段。


以下給出一個「巢狀 Loop」的解析範例，示範如何在 PO1 Loop 內，再偵測到 N1 Loop、並以類似的「狀態機 + 清單收集」機制進行處理。程式以 VB.NET 為例，重點在「偵測巢狀結構」的邏輯，供參考。


---

一、邏輯思路

1. 主程式先掃描，遇到 PO1 → 進入 PO1 Loop 狀態


2. 在 PO1 Loop 中：如果遇到 N1，就進入 N1 Sub-Loop 狀態


3. 在 N1 Sub-Loop 中：收集所有屬於該子 Loop 的段 (N1、N3、N4、REF…等)


4. 結束判斷：

假如又遇到下一個 N1 → 前一個 N1 子 Loop結束、開始新一個 N1 子 Loop

若遇到新的 PO1 或 CTT、SE 等 → 表示 N1 子 Loop 結束，同時也可能結束 PO1 Loop，或者繼續下一個 PO1




藉由「狀態機 (State Machine) + 巢狀控制」，即可解析多層次 Loop。


---

二、範例程式 (VB.NET)

以下程式只是一個示例框架，示範如何偵測、收集、與結束巢狀 Loop。實際業務上，你可在 ProcessPO1Loop() 或 ProcessN1SubLoop() 裡面再進一步拆分欄位並寫入資料庫。

Imports System.IO

Module EDIParserNestedLoop
    Sub Main()
        ' 假設檔案裡含有 PO1 Loop，且 PO1 Loop 下還有多個 N1 (包含 N3, N4...)
        Dim ediFilePath As String = "C:\path\to\your\file.edi"
        Dim ediContent As String = File.ReadAllText(ediFilePath)

        ' 常見段分隔 "~"，元素分隔 "*"
        Dim segmentDelimiter As String = "~"
        Dim elementDelimiter As Char = "*"c

        ' 先切割成段
        Dim segments As String() = ediContent.Split(New String() {segmentDelimiter}, StringSplitOptions.RemoveEmptyEntries)

        ' 狀態：是否在 PO1 Loop
        Dim inPO1Loop As Boolean = False
        ' 用來收集當前這個 PO1 Loop 內所有相關段 (包含 PO1、PID、N1、N3、N4、REF…)
        Dim PO1Segments As New List(Of String)

        For Each rawSeg In segments
            Dim segment As String = rawSeg.Trim()
            If segment = "" Then Continue For

            Dim elements = segment.Split(elementDelimiter)
            Dim segID As String = elements(0).Trim()

            Select Case segID
                '=====================================
                '   主 Loop: PO1
                '=====================================
                Case "PO1"
                    ' 遇到新的 PO1 → 代表上一個 PO1 Loop 結束（若 inPO1Loop=true）
                    If inPO1Loop Then
                        ' 處理前一組 PO1 Loop
                        ProcessPO1Loop(PO1Segments)
                        PO1Segments.Clear()
                    End If

                    ' 新一個 PO1 Loop 開始
                    inPO1Loop = True
                    PO1Segments.Add(segment)

                Case "CTT", "SE", "GE", "IEA"
                    ' 代表所有 PO1 都應該結束
                    If inPO1Loop Then
                        ProcessPO1Loop(PO1Segments)
                        PO1Segments.Clear()
                        inPO1Loop = False
                    End If

                    ' 也可在這裡處理 CTT、SE…(如同先前範例)
                    Console.WriteLine($"Segment {segID} found. End of transaction or summary.")

                Case Else
                    ' 不是 PO1、CTT、SE 等 → 可能是：
                    '   - PID、SAC… (屬於 PO1 Loop 下的段)
                    '   - N1, N3, N4… (屬於 PO1 下的子 Loop)
                    ' 若目前正在 PO1 Loop，則把這些段收集起來
                    If inPO1Loop Then
                        PO1Segments.Add(segment)
                    Else
                        ' 不在 PO1 Loop 的段，就看是否要另外處理
                        Console.WriteLine($"Other segment outside PO1: {segID}")
                    End If
            End Select
        Next

        ' 檔案結尾，如尚未處理最後一組 PO1 Loop
        If inPO1Loop Then
            ProcessPO1Loop(PO1Segments)
        End If

        Console.WriteLine("=== Parsing Complete ===")
        Console.ReadLine()
    End Sub

    ''' <summary>
    ''' 在此函式裡，我們再進行「PO1 Loop」的二次解析，
    ''' 包含檢查其中是否有 N1...N3...N4...子段 (N1 Sub-Loop)。
    ''' </summary>
    Private Sub ProcessPO1Loop(PO1Segments As List(Of String))
        Console.WriteLine("=== Start Process PO1 Loop ===")

        ' 在此再建立一個機制：偵測 N1 Sub-Loop
        Dim inN1SubLoop As Boolean = False
        Dim N1SubSegments As New List(Of String)

        For Each seg In PO1Segments
            Dim elements = seg.Split("*"c)
            Dim segID = elements(0)

            Select Case segID
                Case "PO1"
                    Console.WriteLine($"  [PO1] LineNo={elements(1)}, Qty={elements(2)}, Price={elements(4)}")

                Case "N1"
                    ' 如果已經在上一個 N1 Sub-Loop，先結束它
                    If inN1SubLoop Then
                        ProcessN1SubLoop(N1SubSegments)
                        N1SubSegments.Clear()
                        inN1SubLoop = False
                    End If
                    ' 開始新的 N1 Sub-Loop
                    inN1SubLoop = True
                    N1SubSegments.Add(seg)

                Case "N3", "N4", "REF"
                    ' 這些段若隸屬於 N1 Sub-Loop，就加到 N1SubSegments
                    If inN1SubLoop Then
                        N1SubSegments.Add(seg)
                    Else
                        ' 也可能是 PO1 Loop 下，但不在 N1 Sub-Loop 的其他段
                        Console.WriteLine($"  [PO1-level] {segID}")
                    End If

                Case "PID", "SAC"
                    ' 常見 PO1 子段
                    Console.WriteLine($"  [PO1] {segID} found.")

                Case Else
                    ' 其他未列出的 segment
                    If inN1SubLoop Then
                        ' 若仍在 N1 Sub-Loop，就收集
                        N1SubSegments.Add(seg)
                    Else
                        ' 否則就是 PO1 loop 下的其他段
                        Console.WriteLine($"  [PO1-other] {segID}")
                    End If
            End Select
        Next

        ' PO1 結束前，若還在 N1 Sub-Loop，要做最後的處理
        If inN1SubLoop Then
            ProcessN1SubLoop(N1SubSegments)
        End If

        Console.WriteLine("=== End Process PO1 Loop ===")
    End Sub

    ''' <summary>
    ''' N1 Sub-Loop 處理：把 N1, N3, N4, REF…等段做解析
    ''' </summary>
    Private Sub ProcessN1SubLoop(N1Segments As List(Of String))
        Console.WriteLine("    => Processing N1 Sub-Loop ...")
        For Each seg In N1Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N1"
                    Console.WriteLine($"       [N1] Code={elements(1)}, Name={elements(2)}")
                Case "N3"
                    Console.WriteLine($"       [N3] Address={elements(1)}")
                Case "N4"
                    Console.WriteLine($"       [N4] City/State/Zip={elements(1)}/{elements(2)}/{elements(3)}")
                Case "REF"
                    Console.WriteLine($"       [REF] Type={elements(1)}, Value={elements(2)}")
                Case Else
                    Console.WriteLine($"       [N1-other] {elements(0)}")
            End Select
        Next
        Console.WriteLine("    => N1 Sub-Loop Complete.")
    End Sub
End Module

範例解說

1. Main() 先掃描段

偵測 PO1 → 若已在舊的 PO1 Loop，就先結束並呼叫 ProcessPO1Loop(PO1Segments)

遇到 CTT、SE → 結束所有 PO1 Loop

其餘段若在 inPO1Loop = True 狀態下，就加入 PO1Segments



2. ProcessPO1Loop()

針對該 PO1 Loop 裡的段，再次掃描

偵測到 N1 就進入 (或重新進入) N1 Sub-Loop

偵測到 N3, N4, REF 等段時，如果 inN1SubLoop=True，就加到該 Sub-Loop；否則歸類為 PO1 其他段

最後結束時，若還在 inN1SubLoop，要呼叫 ProcessN1SubLoop() 把殘餘段一起處理



3. ProcessN1SubLoop()

這裡對 N1, N3, N4, REF 等段做進一步欄位拆解，實務中就能寫入 DB




整體就是「一層層解析」，PO1 Loop 在外層，N1(N3,N4,REF) Sub-Loop 在內層，每當遇到新的 PO1 或交易結束，就結束前一個 PO1 Loop；每當遇到新的 N1，就結束前一個 N1 Sub-Loop。這種做法能彈性處理巢狀 EDI 結構。


---

三、小結

1. 巢狀 Loop 處理思路不變：

判斷「哪個 Segment 代表開啟 Sub-Loop」 → 開始收集子段

遇到「下一個相同 Segment (或其它終止條件)」 → 結束 Sub-Loop，呼叫對應函式處理



2. 巢狀解析 常見範例：

PO1 Loop → 包含 PID、SAC…

PO1 Loop → 再包含 N1 Loop (N3, N4…)

N1 Loop → 可能還包含 REF、PER、其他延伸段



3. 程式結構建議：

可把「每個 Loop 的分析邏輯」獨立成函式 (如 ProcessPO1Loop)，避免程式過於冗長

內層 Loop (如 N1SubLoop) 再用相同的「狀態機 + 清單收集」方式拆解




只要掌握上述原則，就能應付「PO1 中也會包含 N1、N3、N4」的巢狀結構，把 EDI 資料解析到你需要的深度並寫進資料庫。祝開發順利!

