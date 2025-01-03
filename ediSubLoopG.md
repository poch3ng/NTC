以下提供一個「改良版」的完整 VB.NET 範例，示範如何同時解析：

1. 頂層 (Top-Level) Loop：

N9 Loop

N1 Loop

PO1 Loop（裡面又可以有多個子段 PID、SAC 等）



2. PO1 巢狀 (Sub) Loop：

N1 Sub-Loop（含 N3、N4、REF…）

也就是當我們「已經在 PO1 Loop」時，如果再遇到 N1，就進入 PO1 內部的 N1 Loop，而不跳到頂層的 N1 Loop。



3. CTT Loop：總結行數/數量等。



> 重點修正：

當我們「已經在 PO1 Loop」時，遇到 N1 段，不再關閉 PO1 Loop 改開啟頂層 N1 Loop；而是「視為 PO1 Loop 裡的 N1 Sub-Loop」。

如果「不在 PO1 Loop」，遇到 N1 段才視為頂層 N1 Loop。




此方式能避免「碰到 N1 就直接切到頂層」，導致 PO1 Sub-Loop 解析失敗的情況。


---

完整程式碼

Imports System.IO

Module EDIParserAllLoops
    Sub Main()
        Dim ediFilePath As String = "C:\path\to\your\file.edi"
        Dim ediContent As String = File.ReadAllText(ediFilePath)

        ' 一般常見的分隔符號：段分隔 "~"，元素分隔 "*"
        Dim segmentDelimiter As String = "~"
        Dim elementDelimiter As Char = "*"c

        ' 先將 EDI 切割成 segment
        Dim segments As String() = ediContent.Split(
            New String() {segmentDelimiter},
            StringSplitOptions.RemoveEmptyEntries
        )

        '
        ' 頂層 Loop 狀態
        '
        Dim inN9Loop As Boolean = False
        Dim inTopN1Loop As Boolean = False    ' (頂層的 N1 Loop)
        Dim inPO1Loop As Boolean = False
        Dim inCTTLoop As Boolean = False

        '
        ' 各頂層 Loop 的收集區
        '
        Dim N9Segments As New List(Of String)
        Dim TopN1Segments As New List(Of String)
        Dim PO1Segments As New List(Of String)
        Dim CTTSegments As New List(Of String)

        '
        ' 迭代每個 Segment
        '
        For Each rawSeg As String In segments
            Dim seg As String = rawSeg.Trim()
            If seg = "" Then Continue For

            Dim elements As String() = seg.Split(elementDelimiter)
            Dim segID As String = elements(0).Trim()

            Select Case segID

                '==============================
                '   N9 (頂層 Loop)
                '==============================
                Case "N9"
                    ' 若已在其他頂層 Loop，先關閉
                    If inTopN1Loop Then
                        ProcessTopN1Loop(TopN1Segments)
                        TopN1Segments.Clear()
                        inTopN1Loop = False
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

                    ' 開啟 (或延續) N9 Loop
                    inN9Loop = True
                    N9Segments.Add(seg)

                '==============================
                '   N1 (可能是頂層，也可能是 PO1 裡的 Sub-Loop)
                '==============================
                Case "N1"
                    If inPO1Loop Then
                        ' -- 已經在 PO1 Loop 裡，遇到 N1 => PO1 Sub-Loop (N1)
                        PO1Segments.Add(seg)

                    Else
                        ' -- 不在 PO1 Loop => 頂層 N1 Loop
                        If inN9Loop Then
                            ProcessN9Loop(N9Segments)
                            N9Segments.Clear()
                            inN9Loop = False
                        End If
                        If inTopN1Loop Then
                            ' 如果先前已經在頂層 N1 Loop，再遇到 N1，代表要先關閉上一個 N1 Loop
                            ProcessTopN1Loop(TopN1Segments)
                            TopN1Segments.Clear()
                            inTopN1Loop = False
                        End If
                        If inCTTLoop Then
                            ProcessCTTLoop(CTTSegments)
                            CTTSegments.Clear()
                            inCTTLoop = False
                        End If

                        ' 開啟 (或延續) 頂層 N1 Loop
                        inTopN1Loop = True
                        TopN1Segments.Add(seg)
                    End If

                '==============================
                '   PO1 (頂層 Loop)
                '==============================
                Case "PO1"
                    ' 如果此時已在 N9、N1、CTT Loop，先關閉
                    If inN9Loop Then
                        ProcessN9Loop(N9Segments)
                        N9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inTopN1Loop Then
                        ProcessTopN1Loop(TopN1Segments)
                        TopN1Segments.Clear()
                        inTopN1Loop = False
                    End If
                    If inCTTLoop Then
                        ProcessCTTLoop(CTTSegments)
                        CTTSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 如果已在 PO1 Loop 中，再遇到新的 PO1 => 先關閉上一個
                    If inPO1Loop Then
                        ProcessPO1Loop(PO1Segments)
                        PO1Segments.Clear()
                        inPO1Loop = False
                    End If

                    ' 開啟 (或延續) PO1 Loop
                    inPO1Loop = True
                    PO1Segments.Add(seg)

                '==============================
                '   CTT (頂層 Loop)
                '==============================
                Case "CTT"
                    ' 關閉其他頂層
                    If inN9Loop Then
                        ProcessN9Loop(N9Segments)
                        N9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inTopN1Loop Then
                        ProcessTopN1Loop(TopN1Segments)
                        TopN1Segments.Clear()
                        inTopN1Loop = False
                    End If
                    If inPO1Loop Then
                        ProcessPO1Loop(PO1Segments)
                        PO1Segments.Clear()
                        inPO1Loop = False
                    End If

                    ' 如果已在 CTT，先關閉
                    If inCTTLoop Then
                        ProcessCTTLoop(CTTSegments)
                        CTTSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 開啟 CTT Loop
                    inCTTLoop = True
                    CTTSegments.Add(seg)

                '==============================
                '   SE / GE / IEA -> 結束
                '==============================
                Case "SE", "GE", "IEA"
                    ' 結束所有已開啟的頂層
                    If inN9Loop Then
                        ProcessN9Loop(N9Segments)
                        N9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inTopN1Loop Then
                        ProcessTopN1Loop(TopN1Segments)
                        TopN1Segments.Clear()
                        inTopN1Loop = False
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
                    Console.WriteLine($"Segment {segID} found. End of transaction/interchange.")

                '==============================
                '   其他 Segment
                '==============================
                Case Else
                    ' 若在 N9 Loop
                    If inN9Loop Then
                        N9Segments.Add(seg)

                    ' 若在頂層 N1 Loop
                    ElseIf inTopN1Loop Then
                        TopN1Segments.Add(seg)

                    ' 若在 PO1 Loop
                    ElseIf inPO1Loop Then
                        PO1Segments.Add(seg)

                    ' 若在 CTT Loop
                    ElseIf inCTTLoop Then
                        CTTSegments.Add(seg)

                    Else
                        ' 不在任何已知 Loop (例如 ISA, GS, ST… 或其他無法歸類段)
                        Console.WriteLine($"Outside any loop: {segID}")
                    End If

            End Select
        Next

        '
        ' 檔案結束後，若還有未關閉的 Loop，也要處理
        '
        If inN9Loop Then
            ProcessN9Loop(N9Segments)
        End If
        If inTopN1Loop Then
            ProcessTopN1Loop(TopN1Segments)
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

    '---------------------------------------------------------------
    '  以下為處理各「頂層」Loop 的函式
    '---------------------------------------------------------------
    Private Sub ProcessN9Loop(n9Segments As List(Of String))
        Console.WriteLine("=== [ProcessN9Loop] Start ===")
        For Each seg In n9Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N9"
                    Console.WriteLine($"  N9 Found. RefID={elements.ElementAtOrDefault(1)}, FreeForm={elements.ElementAtOrDefault(2)}")
                Case "REF"
                    Console.WriteLine($"  REF Found. Type={elements.ElementAtOrDefault(1)}, Value={elements.ElementAtOrDefault(2)}")
                Case "MSG"
                    Console.WriteLine($"  MSG Found. MsgText={elements.ElementAtOrDefault(1)}")
                Case Else
                    Console.WriteLine($"  => N9 other: {elements(0)}")
            End Select
        Next
        Console.WriteLine("=== [ProcessN9Loop] End ===")
    End Sub

    Private Sub ProcessTopN1Loop(n1Segments As List(Of String))
        Console.WriteLine("=== [ProcessTopN1Loop] Start ===")
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
                    Console.WriteLine($"  => N1 other: {elements(0)}")
            End Select
        Next
        Console.WriteLine("=== [ProcessTopN1Loop] End ===")
    End Sub

    Private Sub ProcessPO1Loop(po1Segments As List(Of String))
        Console.WriteLine("=== [ProcessPO1Loop] Start ===")

        '----------------------------------
        ' 內部還有 N1 Sub-Loop
        '----------------------------------
        Dim inPO1N1Loop As Boolean = False
        Dim po1N1Segments As New List(Of String)

        For Each seg In po1Segments
            Dim elements = seg.Split("*"c)
            Dim segID = elements(0)

            Select Case segID
                Case "PO1"
                    Console.WriteLine($"  PO1 Found. LineNo={elements.ElementAtOrDefault(1)}, Qty={elements.ElementAtOrDefault(2)}, Price={elements.ElementAtOrDefault(4)}")

                Case "N1"
                    ' 遇到新的 N1 -> 可能是 PO1 下的 N1 Sub-Loop
                    If inPO1N1Loop Then
                        ' 關閉上一個 Sub-Loop
                        ProcessPO1N1SubLoop(po1N1Segments)
                        po1N1Segments.Clear()
                        inPO1N1Loop = False
                    End If
                    ' 開啟新的 Sub-Loop
                    inPO1N1Loop = True
                    po1N1Segments.Add(seg)

                Case "N3", "N4", "REF"
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
                    If inPO1N1Loop Then
                        po1N1Segments.Add(seg)
                    Else
                        Console.WriteLine($"  => PO1 other: {segID}")
                    End If
            End Select
        Next

        ' 結尾如果還在 N1 Sub-Loop，收尾
        If inPO1N1Loop Then
            ProcessPO1N1SubLoop(po1N1Segments)
        End If

        Console.WriteLine("=== [ProcessPO1Loop] End ===")
    End Sub

    Private Sub ProcessPO1N1SubLoop(po1N1Segments As List(Of String))
        Console.WriteLine("    => [ProcessPO1N1SubLoop] Start")
        For Each seg In po1N1Segments
            Dim elements = seg.Split("*"c)
            Select Case elements(0)
                Case "N1"
                    Console.WriteLine($"       N1 Sub. Code={elements.ElementAtOrDefault(1)}, Name={elements.ElementAtOrDefault(2)}")
                Case "N3"
                    Console.WriteLine($"       N3 Sub. Address={elements.ElementAtOrDefault(1)}")
                Case "N4"
                    Console.WriteLine($"       N4 Sub. City/State/Zip={elements.ElementAtOrDefault(1)}/{elements.ElementAtOrDefault(2)}/{elements.ElementAtOrDefault(3)}")
                Case "REF"
                    Console.WriteLine($"       REF Sub. Type={elements.ElementAtOrDefault(1)}, Value={elements.ElementAtOrDefault(2)}")
                Case Else
                    Console.WriteLine($"       => N1 Sub-other: {elements(0)}")
            End Select
        Next
        Console.WriteLine("    => [ProcessPO1N1SubLoop] End")
    End Sub

    Private Sub ProcessCTTLoop(cttSegments As List(Of String))
        Console.WriteLine("=== [ProcessCTTLoop] Start ===")
        For Each seg In cttSegments
            Dim elements = seg.Split("*"c)
            If elements(0) = "CTT" Then
                Console.WriteLine($"  CTT Found. TotalLineItems={elements.ElementAtOrDefault(1)}, SumOfQty={elements.ElementAtOrDefault(2)}")
            Else
                Console.WriteLine($"  => CTT Other: {elements(0)}")
            End If
        Next
        Console.WriteLine("=== [ProcessCTTLoop] End ===")
    End Sub
End Module


---

說明與重點

1. 區分「頂層 N1」與「PO1 內的 N1 Sub-Loop」

若 inPO1Loop = True，遇到 N1，程式直接把此段加到 PO1Segments，由 ProcessPO1Loop() 內部的程式來判斷為 N1 Sub-Loop。

若 inPO1Loop = False，才會被當作「頂層 N1 Loop」。



2. 頂層 Loop 切換

N9 / N1 / PO1 / CTT 之間，如果遇到下一個頂層 Segment，就先結束前一個 Loop，呼叫對應的 ProcessXxxLoop()。

例如：先前在 N9 Loop 狀態，現在又讀到 N1（而且不在 PO1 Loop 下），就要先 ProcessN9Loop(...)，再進入 N1 Loop。



3. PO1 Loop 結構

在 ProcessPO1Loop() 內部，再用 inPO1N1Loop 來處理「PO1 中的 N1 Sub-Loop」。

每次遇到 N1 (在 PO1 Loop 裡) 就開啟/關閉 PO1N1SubLoop，收集 N1、N3、N4、REF…



4. 為何避免「N1 就直接切換到頂層」的問題？

在上層的 Select Case segID = "N1" 裡，多加了判斷：

If inPO1Loop Then
    ' => PO1 裏面的 N1 Sub-Loop
    PO1Segments.Add(seg)
Else
    ' => 頂層 N1
End If

這樣即可避免邏輯錯誤，保證當我們「在 PO1 Loop」時，N1 不會跑到頂層去。



5. 結束判斷

遇到 SE, GE, IEA → 代表交易結束，先把所有頂層 Loop 收尾。

走完 For Each 後，如果還有 inXXLoop = True，也要收尾。



6. 實務中可再微調

依照實際的 EDI 850 結構，可能還有更多段 (MSG, PER, SAC, TAX, etc.) 或更多層巢狀 (PO1 裡再包 PID / N1 / 其他子 Loop…)。

可以在各 ProcessXxxLoop 裡再細分欄位、寫入資料庫。

如果頂層 N1 與 PO1 子 N1 會出現不同的 qualifier、segment 結構，也可依實際需求調整。





---

結論

這份程式碼重點在「偵測是否在 PO1 Loop 中」來區分 N1 是屬於「PO1 巢狀 N1」還是「頂層 N1」。

當 inPO1Loop = True 時，碰到 N1 一律歸屬於 PO1 子 Loop；只有在不處於 PO1 Loop 時，才進入頂層 N1 Loop。

如此即可避免「只要一遇到 N1 就切換到頂層」的邏輯問題，正確實現 PO1 -> N1 Sub-Loop。


你可把此程式當作範本，再依實際規範（哪段屬於誰的子段）做客製化調整。這樣就能完整處理包含 N9、N1、PO1（含其內部 N1 Sub-Loop）以及 CTT 的情形。祝開發順利!

