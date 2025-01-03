以下提供一個「單一程式」的完整示例程式碼，示範如何在 VB.NET 中以「狀態機 + 清單收集」的方式，同時處理 N9 Loop、N1 Loop、PO1 Loop（含巢狀 N1 Sub-Loop）、CTT Loop 等四種主要 Loop 結構。程式邏輯如下：

1. 頂層 (Top-Level) Loop：N9、N1、PO1、CTT

在掃描 EDI 段時，遇到 N9 → 關閉其他頂層 Loop → 開啟或延續 N9 Loop

遇到 N1 → 關閉其他頂層 Loop → 開啟或延續 N1 Loop

遇到 PO1 → 關閉其他頂層 Loop → 開啟 PO1 Loop

遇到 CTT → 關閉其他頂層 Loop → 開啟 CTT Loop

遇到 SE、GE、IEA → 代表交易即將結束，關閉所有已開啟的 Loop



2. PO1 巢狀 N1 Sub-Loop

在「PO1 Loop」內部，再偵測到 N1 時，開啟一個 N1 Sub-Loop，收集 N1、N3、N4、REF…

若再遇到下一個 N1，就結束上一個 N1 Sub-Loop，開始新的 N1 Sub-Loop

若遇到 PO1、CTT 或交易結尾，則結束所有 N1 Sub-Loop



3. 收集並處理

每個 Loop 以一個 List(Of String) 儲存相關段

結束時呼叫對應的 ProcessXXXLoop() 函式，解析欄位並（示範）印出結果

實務中，可在這些函式中進一步「寫入資料庫」、「執行業務邏輯」等




> 注意：以下程式僅示範核心解析概念，實際 EDI 需求可更複雜 (多層巢狀、其他段、重複出現順序不固定…)，請依實況調整。




---

完整程式碼示例

Imports System.IO

Module EDIParserAllLoops
    Sub Main()
        ' 假設你的 EDI 檔案中同時會出現
        ' N9 Loop、N1 Loop、PO1 Loop（含巢狀 N1 Loop）、以及 CTT Loop
        Dim ediFilePath As String = "C:\path\to\your\file.edi"
        Dim ediContent As String = File.ReadAllText(ediFilePath)

        ' 常見做法：段分隔 "~"，元素分隔 "*"
        Dim segmentDelimiter As String = "~"
        Dim elementDelimiter As Char = "*"c

        ' 先將 EDI 切割成 segment
        Dim segments As String() = ediContent.Split(
            New String() {segmentDelimiter},
            StringSplitOptions.RemoveEmptyEntries
        )

        '------------------------------------------------
        ' 狀態旗標：是否位於各個頂層 Loop
        '------------------------------------------------
        Dim inN9Loop As Boolean = False
        Dim inN1Loop As Boolean = False
        Dim inPO1Loop As Boolean = False
        Dim inCTTLoop As Boolean = False

        '------------------------------------------------
        ' 暫存各個頂層 Loop 的段落
        '------------------------------------------------
        Dim N9Segments As New List(Of String)
        Dim N1Segments As New List(Of String)
        Dim PO1Segments As New List(Of String)
        Dim CTTSegments As New List(Of String)

        For Each rawSeg As String In segments
            Dim seg As String = rawSeg.Trim()
            If seg = "" Then Continue For

            Dim elements As String() = seg.Split(elementDelimiter)
            Dim segID As String = elements(0).Trim()

            Select Case segID

                '==============================
                '   N9 Loop
                '==============================
                Case "N9"
                    ' 先關閉所有其他頂層 Loop
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

                    ' 開始 (或延續) N9 Loop
                    inN9Loop = True
                    N9Segments.Add(seg)

                '==============================
                '   N1 Loop (頂層)
                '==============================
                Case "N1"
                    ' 關閉其他頂層 Loop (N9, PO1, CTT)
                    If inN9Loop Then
                        ProcessN9Loop(N9Segments)
                        N9Segments.Clear()
                        inN9Loop = False
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

                    ' 開始 (或延續) N1 Loop
                    inN1Loop = True
                    N1Segments.Add(seg)

                '==============================
                '   PO1 Loop (頂層)
                '==============================
                Case "PO1"
                    ' 關閉其他頂層 Loop (N9, N1, CTT)
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

                    ' 開始 (或延續) PO1 Loop
                    inPO1Loop = True
                    PO1Segments.Add(seg)

                '==============================
                '   CTT Loop
                '==============================
                Case "CTT"
                    ' 關閉其他頂層 Loop (N9, N1, PO1)
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

                    ' 開始 (或延續) CTT Loop
                    inCTTLoop = True
                    CTTSegments.Add(seg)

                '==============================
                '   交易尾端段 SE/GE/IEA
                '==============================
                Case "SE", "GE", "IEA"
                    ' 這些通常代表交易結束或整包交換結束
                    ' 先把所有已開啟的 Loop 做收尾
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

                    Console.WriteLine($"Segment {segID} found. End of transaction/Interchange.")

                '==============================
                '   其他 Segment
                '   (可能屬於某個頂層 Loop 中)
                '==============================
                Case Else
                    If inN9Loop Then
                        N9Segments.Add(seg)
                    ElseIf inN1Loop Then
                        N1Segments.Add(seg)
                    ElseIf inPO1Loop Then
                        PO1Segments.Add(seg)
                    ElseIf inCTTLoop Then
                        CTTSegments.Add(seg)
                    Else
                        ' 不在任何 Loop (可能是 ISA/GS/ST... 或其他非預期段)
                        Console.WriteLine($"Outside any loop: {segID}")
                    End If

            End Select
        Next

        '-------------------------------------------
        ' 最後如果還有開啟的 Loop，做收尾處理
        '-------------------------------------------
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

    '=========================================================
    '  以下為各頂層 Loop 的處理函式
    '=========================================================

    ''' <summary>
    ''' 處理 N9 Loop (可含多個 N9、MSG、REF…等段)
    ''' </summary>
    Private Sub ProcessN9Loop(n9Segments As List(Of String))
        Console.WriteLine("=== [ProcessN9Loop] Start ===")
        For Each seg In n9Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N9"
                    Console.WriteLine($"  N9 Found. RefID={elements.ElementAtOrDefault(1)}, FreeForm={elements.ElementAtOrDefault(2)}")
                Case "MSG"
                    Console.WriteLine($"  MSG Found. MsgText={elements.ElementAtOrDefault(1)}")
                Case "REF"
                    Console.WriteLine($"  REF Found. Type={elements.ElementAtOrDefault(1)}, Value={elements.ElementAtOrDefault(2)}")
                Case Else
                    Console.WriteLine($"  => N9 Other: {elements(0)}")
            End Select
        Next
        Console.WriteLine("=== [ProcessN9Loop] End ===")
    End Sub

    ''' <summary>
    ''' 處理頂層 N1 Loop (包含 N1, N3, N4, REF…)
    ''' </summary>
    Private Sub ProcessN1Loop(n1Segments As List(Of String))
        Console.WriteLine("=== [ProcessN1Loop] Start ===")
        For Each seg In n1Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N1"
                    Console.WriteLine($"  N1 Found. Code={elements.ElementAtOrDefault(1)}, Name={elements.ElementAtOrDefault(2)}")
                Case "N3"
                    Console.WriteLine($"  N3 Found. Address={elements.ElementAtOrDefault(1)}")
                Case "N4"
                    Console.WriteLine($"  N4 Found. City/State/Zip={elements.ElementAtOrDefault(1)}/{elements.ElementAtOrDefault(2)}/{elements.ElementAtOrDefault(3)}")
                Case "REF"
                    Console.WriteLine($"  REF Found. Type={elements.ElementAtOrDefault(1)}, Value={elements.ElementAtOrDefault(2)}")
                Case Else
                    Console.WriteLine($"  => N1 Other: {elements(0)}")
            End Select
        Next
        Console.WriteLine("=== [ProcessN1Loop] End ===")
    End Sub

    ''' <summary>
    ''' 處理 PO1 Loop (含 PO1, PID, SAC…等)
    ''' 裡面若偵測到 N1, N3, N4… 則進入 N1 Sub-Loop。
    ''' </summary>
    Private Sub ProcessPO1Loop(po1Segments As List(Of String))
        Console.WriteLine("=== [ProcessPO1Loop] Start ===")

        ' 巢狀 N1 子 Loop
        Dim inPO1N1Loop As Boolean = False
        Dim po1N1Segments As New List(Of String)

        For Each seg In po1Segments
            Dim elements = seg.Split("*"c)
            Dim segID = elements(0).Trim()

            Select Case segID

                Case "PO1"
                    Console.WriteLine($"  PO1 Found. LineNo={elements.ElementAtOrDefault(1)}, Qty={elements.ElementAtOrDefault(2)}, Price={elements.ElementAtOrDefault(4)}")

                Case "N1"
                    ' 若發現新的 N1 (Sub-Loop) → 先結束上一個
                    If inPO1N1Loop Then
                        ProcessPO1N1SubLoop(po1N1Segments)
                        po1N1Segments.Clear()
                        inPO1N1Loop = False
                    End If
                    ' 開始新的巢狀 N1 Sub-Loop
                    inPO1N1Loop = True
                    po1N1Segments.Add(seg)

                Case "N3", "N4", "REF"
                    ' 若處於 PO1N1Loop 狀態，就收集
                    If inPO1N1Loop Then
                        po1N1Segments.Add(seg)
                    Else
                        Console.WriteLine($"  (PO1-level) {segID} Found.")
                    End If

                Case "PID"
                    Console.WriteLine($"  PID Found. Description={elements.ElementAtOrDefault(4)}")

                Case "SAC"
                    Console.WriteLine($"  SAC Found. Code={elements.ElementAtOrDefault(1)}, Amt={elements.ElementAtOrDefault(5)}")

                Case Else
                    ' 其他段
                    If inPO1N1Loop Then
                        po1N1Segments.Add(seg)
                    Else
                        Console.WriteLine($"  => PO1 Other Segment: {segID}")
                    End If

            End Select
        Next

        ' 結束前，如還在巢狀 N1 Sub-Loop，處理收尾
        If inPO1N1Loop Then
            ProcessPO1N1SubLoop(po1N1Segments)
        End If

        Console.WriteLine("=== [ProcessPO1Loop] End ===")
    End Sub

    ''' <summary>
    ''' PO1 Loop 裏頭的 N1 Sub-Loop (N1, N3, N4, REF…)
    ''' </summary>
    Private Sub ProcessPO1N1SubLoop(po1N1Segments As List(Of String))
        Console.WriteLine("    => [ProcessPO1N1SubLoop] Start")
        For Each seg In po1N1Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N1"
                    Console.WriteLine($"       [N1-Sub] Code={elements.ElementAtOrDefault(1)}, Name={elements.ElementAtOrDefault(2)}")
                Case "N3"
                    Console.WriteLine($"       [N3-Sub] Address={elements.ElementAtOrDefault(1)}")
                Case "N4"
                    Console.WriteLine($"       [N4-Sub] City/State/Zip={elements.ElementAtOrDefault(1)}/{elements.ElementAtOrDefault(2)}/{elements.ElementAtOrDefault(3)}")
                Case "REF"
                    Console.WriteLine($"       [REF-Sub] Type={elements.ElementAtOrDefault(1)}, Value={elements.ElementAtOrDefault(2)}")
                Case Else
                    Console.WriteLine($"       => [N1-Sub Other] {elements(0)}")
            End Select
        Next
        Console.WriteLine("    => [ProcessPO1N1SubLoop] End")
    End Sub

    ''' <summary>
    ''' 處理 CTT Loop (常用於總結行數、總數量等)
    ''' </summary>
    Private Sub ProcessCTTLoop(cttSegments As List(Of String))
        Console.WriteLine("=== [ProcessCTTLoop] Start ===")
        For Each seg In cttSegments
            Dim elements = seg.Split("*"c)
            If elements(0) = "CTT" Then
                Console.WriteLine($"  CTT Found. TotalLineItems={elements.ElementAtOrDefault(1)}, SumOfQty={elements.ElementAtOrDefault(2)}")
            Else
                Console.WriteLine($"  => CTT Other Segment: {elements(0)}")
            End If
        Next
        Console.WriteLine("=== [ProcessCTTLoop] End ===")
    End Sub

End Module


---

程式重點說明

1. 頂層 Loop 切換 (N9, N1, PO1, CTT)

以四個布林變數 inN9Loop, inN1Loop, inPO1Loop, inCTTLoop 來代表是否正在該 Loop 中。

每當遇到另一個頂層 Loop 的起始段 (N9、N1、PO1、CTT)，便先關閉目前所有開啟的頂層 Loop，執行對應的 ProcessXxxLoop()，再清空暫存並開啟新 Loop。



2. PO1 中的 N1 子 Loop

在 ProcessPO1Loop() 內，再用額外的布林 inPO1N1Loop 來判斷是否正在 PO1 下的 N1 Sub-Loop。

如果偵測到新的 N1，代表要開啟或切換到另一個子 Loop → 關閉上一個並呼叫 ProcessPO1N1SubLoop()。

對應的 N3, N4, REF 等都歸屬在這個子 Loop 中一起收集。



3. 結束判斷

遇到 SE, GE, IEA (交易或交換結束) 時，關閉所有已開啟的 Loop。

最後 For Each 結束時，也要檢查是否還有未結束的 Loop（例如檔案沒有 SE 但已到終點），一併呼叫 ProcessXxxLoop() 收尾。



4. 實務中改寫

這份程式碼主要示範「多個頂層 Loop + 巢狀 N1 Sub-Loop」的解析架構。

實際 EDI 可能會有更多子段、更多層巢狀，或不固定出現順序，要做更複雜的判斷。

可以在 ProcessN9Loop(), ProcessN1Loop(), ProcessPO1Loop()… 等函式中，將拆解好的欄位直接寫入資料庫。

若要處理多筆 N9 或 N1 重複出現，也請確保程式能在頂層或巢狀結構內，正確「一次結束上一個 Loop，再開新的 Loop」。





---

結論

以上範例程式展示了「頂層多 Loop + PO1 巢狀 N1 Loop」的 VB.NET 解析流程。

核心理念是「狀態機 (State Machine)」+「清單收集」+「遇到新 Loop 或結束段時進行處理」。

你可以將這段程式做為基礎，針對自己的 EDI 850 (或其他交易) 規範做修改，並在各 ProcessXXXLoop() 函式裡，完成儲存資料庫或商業邏輯。


祝開發順利!



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

