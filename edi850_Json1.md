下面提供一個「完整 VB.NET 範例」，示範如何在解析含有 N9 Loop、N1 Loop、PO1 Loop（含 PO1 內的 N1 Sub-Loop）與 CTT Loop 的 EDI 段後，將結果整理成 JSON。整體邏輯與先前相同，但這次我們會在解析過程中，將資料儲存在一組自定義的資料結構（class），最後用 System.Text.Json 序列化成 JSON 輸出。

> 注意：

1. 以下程式只是一個「示例框架」。若你的實際 EDI 場景更複雜（還有更多段、更多巢狀），請依需求擴充。


2. 若你使用的是 .NET Framework 4.x，想用內建 Newtonsoft.Json (或其他套件) 也行，程式碼相似，只需換序列化方式即可。這裡示範 .NET 5+/Core 常見的 System.Text.Json。


3. 由於每個 Loop 下的段(如 N1, N3, N4, REF…) 可能還有多種結構，以下為示意：我們將段拆解後，以「SegmentId + 一串欄位」的方式儲存；或者也可以改寫成更嚴謹的強型別屬性。






---

一、程式結構與資料類別

1. EdiResult

放整個 EDI 解析後的結果，包含多個 Loop 清單：N9Loops, TopN1Loops, PO1Loops, CTTLoops



2. Loop 資料類別

N9Loop, N1Loop, PO1Loop, CTTLoop

每個 Loop 下可能有多個段，每個段用 SegmentData 來記錄

PO1Loop 內還包含 List(Of PO1N1Loop)（也就是 PO1 裡的 N1 Sub-Loop）



3. SegmentData

只示範把段 ID（例如 "N1", "N3" ...）和各欄位存在 Fields 裡

你可以改成更細的屬性，如 RefID, Name, City，以符合實際需求



4. 程式流程

與先前範例邏輯相同：以「狀態機」來判斷當前是否在 N9、N1、PO1 或 CTT Loop；若遇到新的 Loop Segment，先關閉舊的，再開新的

針對 PO1 Loop，若再遇到 N1，則視為「PO1 內的 N1 Sub-Loop」

解析完成後，把資料都存在 ediResult 中

最後：用 JsonSerializer.Serialize(ediResult) 產生 JSON





---

二、完整程式碼範例

Imports System.IO
Imports System.Text.Json
Imports System.Text.Json.Serialization

Module EDIParserToJson

    Sub Main()
        ' 假設你的檔案含有 N9 Loop、N1 Loop、PO1 Loop (含 N1 Sub-Loop)、CTT Loop
        Dim ediFilePath As String = "C:\path\to\your\file.edi"
        Dim ediContent As String = File.ReadAllText(ediFilePath)

        ' 分隔符號：段用 "~"，欄位用 "*"
        Dim segmentDelimiter As String = "~"
        Dim elementDelimiter As Char = "*"c

        ' 切割成 segment
        Dim segments As String() = ediContent.Split(
            New String() {segmentDelimiter},
            StringSplitOptions.RemoveEmptyEntries
        )

        ' 建立一個最終結果的容器
        Dim ediResult As New EdiResult()

        ' 狀態：是否在頂層的 N9 / N1 / PO1 / CTT Loop
        Dim inN9Loop As Boolean = False
        Dim inTopN1Loop As Boolean = False
        Dim inPO1Loop As Boolean = False
        Dim inCTTLoop As Boolean = False

        ' 暫存頂層 Loop 的段落
        Dim n9Segments As New List(Of String)
        Dim topN1Segments As New List(Of String)
        Dim po1Segments As New List(Of String)
        Dim cttSegments As New List(Of String)

        ' 逐段掃描
        For Each rawSeg As String In segments
            Dim seg As String = rawSeg.Trim()
            If seg = "" Then Continue For

            Dim elements = seg.Split(elementDelimiter)
            Dim segID As String = elements(0).Trim()

            Select Case segID

                '=============================
                '   N9 頂層 Loop
                '=============================
                Case "N9"
                    ' 關閉其他頂層 Loop
                    If inTopN1Loop Then
                        ProcessTopN1Loop(topN1Segments, ediResult)
                        topN1Segments.Clear()
                        inTopN1Loop = False
                    End If
                    If inPO1Loop Then
                        ProcessPO1Loop(po1Segments, ediResult)
                        po1Segments.Clear()
                        inPO1Loop = False
                    End If
                    If inCTTLoop Then
                        ProcessCTTLoop(cttSegments, ediResult)
                        cttSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 若之前已在 N9 Loop，則先關閉
                    If inN9Loop Then
                        ProcessN9Loop(n9Segments, ediResult)
                        n9Segments.Clear()
                        inN9Loop = False
                    End If

                    ' 開啟新一個 N9 Loop
                    inN9Loop = True
                    n9Segments.Add(seg)

                '=============================
                '   N1 (頂層 or PO1 裡)
                '=============================
                Case "N1"
                    If inPO1Loop Then
                        ' => PO1 裡的 N1 Sub-Loop
                        po1Segments.Add(seg)
                    Else
                        ' => 頂層 N1
                        If inN9Loop Then
                            ProcessN9Loop(n9Segments, ediResult)
                            n9Segments.Clear()
                            inN9Loop = False
                        End If
                        If inTopN1Loop Then
                            ProcessTopN1Loop(topN1Segments, ediResult)
                            topN1Segments.Clear()
                            inTopN1Loop = False
                        End If
                        If inCTTLoop Then
                            ProcessCTTLoop(cttSegments, ediResult)
                            cttSegments.Clear()
                            inCTTLoop = False
                        End If

                        ' 開新的頂層 N1
                        inTopN1Loop = True
                        topN1Segments.Add(seg)
                    End If

                '=============================
                '   PO1 (頂層 Loop)
                '=============================
                Case "PO1"
                    ' 若已在 N9、N1、CTT，先關閉
                    If inN9Loop Then
                        ProcessN9Loop(n9Segments, ediResult)
                        n9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inTopN1Loop Then
                        ProcessTopN1Loop(topN1Segments, ediResult)
                        topN1Segments.Clear()
                        inTopN1Loop = False
                    End If
                    If inCTTLoop Then
                        ProcessCTTLoop(cttSegments, ediResult)
                        cttSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 若已在 PO1 Loop，再遇到新的 PO1，先關閉上一個
                    If inPO1Loop Then
                        ProcessPO1Loop(po1Segments, ediResult)
                        po1Segments.Clear()
                        inPO1Loop = False
                    End If

                    ' 開啟新 PO1
                    inPO1Loop = True
                    po1Segments.Add(seg)

                '=============================
                '   CTT (頂層 Loop)
                '=============================
                Case "CTT"
                    If inN9Loop Then
                        ProcessN9Loop(n9Segments, ediResult)
                        n9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inTopN1Loop Then
                        ProcessTopN1Loop(topN1Segments, ediResult)
                        topN1Segments.Clear()
                        inTopN1Loop = False
                    End If
                    If inPO1Loop Then
                        ProcessPO1Loop(po1Segments, ediResult)
                        po1Segments.Clear()
                        inPO1Loop = False
                    End If

                    ' 若已在 CTT，再來一個 CTT 就要先關閉
                    If inCTTLoop Then
                        ProcessCTTLoop(cttSegments, ediResult)
                        cttSegments.Clear()
                        inCTTLoop = False
                    End If

                    ' 新開 CTT
                    inCTTLoop = True
                    cttSegments.Add(seg)

                '=============================
                '   SE, GE, IEA -> 結束
                '=============================
                Case "SE", "GE", "IEA"
                    If inN9Loop Then
                        ProcessN9Loop(n9Segments, ediResult)
                        n9Segments.Clear()
                        inN9Loop = False
                    End If
                    If inTopN1Loop Then
                        ProcessTopN1Loop(topN1Segments, ediResult)
                        topN1Segments.Clear()
                        inTopN1Loop = False
                    End If
                    If inPO1Loop Then
                        ProcessPO1Loop(po1Segments, ediResult)
                        po1Segments.Clear()
                        inPO1Loop = False
                    End If
                    If inCTTLoop Then
                        ProcessCTTLoop(cttSegments, ediResult)
                        cttSegments.Clear()
                        inCTTLoop = False
                    End If

                    Console.WriteLine($"Segment {segID} => End of transaction/interchange.")

                '=============================
                '   其它段
                '=============================
                Case Else
                    If inN9Loop Then
                        n9Segments.Add(seg)
                    ElseIf inTopN1Loop Then
                        topN1Segments.Add(seg)
                    ElseIf inPO1Loop Then
                        po1Segments.Add(seg)
                    ElseIf inCTTLoop Then
                        cttSegments.Add(seg)
                    Else
                        ' 不在任何 Loop，可能是 ISA, GS, ST...
                        Console.WriteLine($"Outside any loop: {segID}")
                    End If
            End Select
        Next

        ' 結束後, 若還有未關閉的 Loop, 處理收尾
        If inN9Loop Then
            ProcessN9Loop(n9Segments, ediResult)
        End If
        If inTopN1Loop Then
            ProcessTopN1Loop(topN1Segments, ediResult)
        End If
        If inPO1Loop Then
            ProcessPO1Loop(po1Segments, ediResult)
        End If
        If inCTTLoop Then
            ProcessCTTLoop(cttSegments, ediResult)
        End If

        '== 產生 JSON ==
        Dim options As New JsonSerializerOptions() With {
            .WriteIndented = True ' 格式化輸出 (縮排)
        }

        Dim jsonOutput As String = JsonSerializer.Serialize(ediResult, options)

        Console.WriteLine("=== EDI Parsing Complete, JSON Result Below ===")
        Console.WriteLine(jsonOutput)
        Console.ReadLine()
    End Sub

    '------------------------------------------------------
    '   各頂層 Loop 的處理，建立對應的物件後加入 ediResult
    '------------------------------------------------------
    Private Sub ProcessN9Loop(n9Segments As List(Of String), result As EdiResult)
        ' 建立一個 N9Loop 物件
        Dim loopObj As New N9Loop()

        ' 解析每段
        For Each seg In n9Segments
            loopObj.Segments.Add(MakeSegmentData(seg))
        Next

        ' 加入結果
        result.N9Loops.Add(loopObj)
    End Sub

    Private Sub ProcessTopN1Loop(n1Segments As List(Of String), result As EdiResult)
        Dim loopObj As New N1Loop()

        For Each seg In n1Segments
            loopObj.Segments.Add(MakeSegmentData(seg))
        Next

        result.TopN1Loops.Add(loopObj)
    End Sub

    Private Sub ProcessPO1Loop(po1Segments As List(Of String), result As EdiResult)
        ' 建立 PO1Loop 物件
        Dim po1Obj As New PO1Loop()

        ' 巢狀 N1 Sub-Loop 狀態
        Dim inSubN1Loop As Boolean = False
        Dim subN1Segments As New List(Of String)

        For Each seg In po1Segments
            Dim segID = seg.Split("*"c)(0).Trim()

            If segID = "PO1" Then
                ' 直接把此段記錄到 PO1Loop 裡的 Segments
                po1Obj.Segments.Add(MakeSegmentData(seg))

            ElseIf segID = "N1" Then
                ' 遇到新的 N1 => 結束上一個 Sub-Loop
                If inSubN1Loop Then
                    Dim subLoopObj As New PO1N1Loop()
                    For Each s In subN1Segments
                        subLoopObj.Segments.Add(MakeSegmentData(s))
                    Next
                    po1Obj.N1SubLoops.Add(subLoopObj)

                    subN1Segments.Clear()
                    inSubN1Loop = False
                End If
                ' 開啟新的 Sub-Loop
                inSubN1Loop = True
                subN1Segments.Add(seg)

            ElseIf segID = "N3" Or segID = "N4" Or segID = "REF" Then
                ' 這些屬於 N1 Sub-Loop
                If inSubN1Loop Then
                    subN1Segments.Add(seg)
                Else
                    ' 否則就直接歸到 PO1Loop (有些規範可能不一定在 Sub-Loop)
                    po1Obj.Segments.Add(MakeSegmentData(seg))
                End If

            Else
                ' 其他段, 如 PID, SAC, 或任何
                If inSubN1Loop Then
                    subN1Segments.Add(seg)
                Else
                    po1Obj.Segments.Add(MakeSegmentData(seg))
                End If
            End If
        Next

        ' 最後若還在 N1 Sub-Loop，收尾
        If inSubN1Loop Then
            Dim subLoopObj As New PO1N1Loop()
            For Each s In subN1Segments
                subLoopObj.Segments.Add(MakeSegmentData(s))
            Next
            po1Obj.N1SubLoops.Add(subLoopObj)
        End If

        result.PO1Loops.Add(po1Obj)
    End Sub

    Private Sub ProcessCTTLoop(cttSegments As List(Of String), result As EdiResult)
        Dim loopObj As New CTTLoop()

        For Each seg In cttSegments
            loopObj.Segments.Add(MakeSegmentData(seg))
        Next

        result.CTTLoops.Add(loopObj)
    End Sub

    '---------------------------------------------
    ' 建立 SegmentData：SegmentId + 各欄位
    '---------------------------------------------
    Private Function MakeSegmentData(rawSegment As String) As SegmentData
        Dim fields = rawSegment.Split("*"c)
        Dim segData As New SegmentData With {
            .SegmentId = fields(0),
            .Fields = fields.Skip(1).ToList()
        }
        Return segData
    End Function

End Module

'============================================================
'   資料結構 (class) 區
'============================================================

''' <summary>
''' 解析後的整份 EDI 結果
''' </summary>
Public Class EdiResult
    Public Property N9Loops As New List(Of N9Loop)
    Public Property TopN1Loops As New List(Of N1Loop)
    Public Property PO1Loops As New List(Of PO1Loop)
    Public Property CTTLoops As New List(Of CTTLoop)
End Class

''' <summary>
''' 通用：用來存每個 Segment 的資料
''' SegmentId = "N1", "PO1", "REF"...
''' Fields = 其餘欄位
''' </summary>
Public Class SegmentData
    Public Property SegmentId As String
    Public Property Fields As List(Of String)
End Class

'-------------------- 各 Loop 的 class -----------------------

Public Class N9Loop
    Public Property Segments As New List(Of SegmentData)
End Class

Public Class N1Loop
    Public Property Segments As New List(Of SegmentData)
End Class

Public Class PO1Loop
    ' PO1 Loop 本身的 Segment (PO1, PID, SAC…)
    Public Property Segments As New List(Of SegmentData)
    ' 在 PO1 下的 N1 Sub-Loop
    Public Property N1SubLoops As New List(Of PO1N1Loop)
End Class

''' <summary>
''' PO1 裡的 N1 Loop (N1, N3, N4, REF…)
''' </summary>
Public Class PO1N1Loop
    Public Property Segments As New List(Of SegmentData)
End Class

Public Class CTTLoop
    Public Property Segments As New List(Of SegmentData)
End Class


---

三、執行後的 JSON 範例

假設 EDI 內容大致如下(僅示意)：

N9*AB*123~
N9*XY*456~
N1*BT*ABC COMPANY~
PO1*1*10*EA*5.00*…
N1*ST*SHIPTO NAME~
N3*SHIPTO ADDR 1~
N4*CITY*ST*ZIP~
PO1*2*20*EA*10.00*…
CTT*2~
SE*…

程式結束後，在 Console 中印出的 JSON（略）可能會像：

{
  "N9Loops": [
    {
      "Segments": [
        {
          "SegmentId": "N9",
          "Fields": [ "AB", "123" ]
        },
        {
          "SegmentId": "N9",
          "Fields": [ "XY", "456" ]
        }
      ]
    }
  ],
  "TopN1Loops": [],
  "PO1Loops": [
    {
      "Segments": [
        {
          "SegmentId": "PO1",
          "Fields": [ "1", "10", "EA", "5.00", "…" ]
        },
        {
          "SegmentId": "PO1",
          "Fields": [ "2", "20", "EA", "10.00", "…" ]
        }
      ],
      "N1SubLoops": [
        {
          "Segments": [
            {
              "SegmentId": "N1",
              "Fields": [ "ST", "SHIPTO NAME" ]
            },
            {
              "SegmentId": "N3",
              "Fields": [ "SHIPTO ADDR 1" ]
            },
            {
              "SegmentId": "N4",
              "Fields": [ "CITY", "ST", "ZIP" ]
            }
          ]
        }
      ]
    }
  ],
  "CTTLoops": [
    {
      "Segments": [
        {
          "SegmentId": "CTT",
          "Fields": [ "2" ]
        }
      ]
    }
  ]
}

這就是最終的 JSON 結構，每個 Loop 的段都收集完好。實務上，你可以在 MakeSegmentData() 或各 ProcessXxxLoop() 裡，把欄位再拆解成更具意義的欄位名稱（如 PoLineNumber, Quantity, Price）並存成強型別屬性，而不只是簡單的 SegmentId 和 Fields。


---

總結

1. 「狀態機 + 清單收集 + 巢狀判斷」 的解析邏輯，搭配一份 EdiResult 資料結構，即可將結果整理為程式物件。


2. 最後使用 JsonSerializer（或其他 JSON 套件）即可輸出 JSON。


3. 如需更細的業務欄位(如 PO Number、Address、City…)，可在 ProcessPO1Loop() 等函式內更精細解析，再對應至物件屬性。



此範例示範了「完整頂層 + Sub-Loop」解析，並轉成 JSON 的流程，供你參考或直接改造使用。祝開發順利!

