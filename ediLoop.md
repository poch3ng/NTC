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

