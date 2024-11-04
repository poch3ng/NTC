如果日期是從 D1 到 AH1（例如十月有 31 天），那麼在不同月份根據當月天數， XValues 的範圍就需要從 D1 開始，並延伸到相對應的欄位。

可以用以下的 VBA 程式碼來動態設定橫向日期範圍：

Sub AdjustSeriesXValuesForMonth()
    Dim ws As Worksheet
    Dim chartObj As ChartObject
    Dim srs As Series
    Dim lastDay As Integer
    Dim endColumn As String
    Dim xValuesRange As Range

    ' 設定工作表與圖表
    Set ws = ThisWorkbook.Sheets("Sheet1") ' 更改為你的工作表名稱
    Set chartObj = ws.ChartObjects("Chart1") ' 更改為你的圖表名稱
    Set srs = chartObj.Chart.SeriesCollection(1) ' 假設是第一個系列

    ' 取得當前月份的最後一天
    lastDay = Day(DateSerial(Year(Date), Month(Date) + 1, 0))

    ' 根據當月天數計算結束欄位
    endColumn = Split(ws.Cells(1, 4 + lastDay - 1).Address, "$")(1)

    ' 設定XValues的範圍，從D1開始到計算出的結束欄位
    Set xValuesRange = ws.Range("D1:" & endColumn & "1")
    srs.XValues = xValuesRange

    MsgBox "XValues已根據當月天數自動調整至 " & endColumn & "1！"
End Sub

說明：

1. lastDay 計算當月的天數，例如，十月會是 31 天，十一月會是 30 天。


2. endColumn 根據 lastDay 計算出 XValues 的結束欄位。這裡透過 Cells(1, 4 + lastDay - 1) 來動態定位欄位，4 代表 D 欄，依據天數調整到該月份的最後一天的欄位。


3. xValuesRange 設定為從 D1 到計算出的 endColumn，這樣在月份變動時，XValues 的範圍會自動調整。



此程式碼可以根據每月天數自動更新 XValues，避免月份變化導致的錯誤。



如果日期是從 D1, E1 等橫向排列（例如一行中的不同欄位代表每天的日期），可以稍微調整程式碼，讓它根據當前月份的天數動態調整 XValues 的橫向範圍。以下是針對橫向日期的範例程式碼：

Sub AdjustSeriesXValuesHorizontal()
    Dim ws As Worksheet
    Dim chartObj As ChartObject
    Dim srs As Series
    Dim lastDay As Integer
    Dim xValuesRange As Range

    ' 設定工作表與圖表
    Set ws = ThisWorkbook.Sheets("Sheet1") ' 更改為你的工作表名稱
    Set chartObj = ws.ChartObjects("Chart1") ' 更改為你的圖表名稱
    Set srs = chartObj.Chart.SeriesCollection(1) ' 假設是第一個系列

    ' 取得當前月份的最後一天
    lastDay = Day(DateSerial(Year(Date), Month(Date) + 1, 0))

    ' 設定XValues的範圍，根據當月天數橫向調整
    Set xValuesRange = ws.Range("D1").Resize(1, lastDay) ' 假設日期從D1橫向排列
    srs.XValues = xValuesRange

    MsgBox "XValues已根據當月天數自動調整！"
End Sub

說明：

1. lastDay 計算當月的天數，以便設定範圍。


2. xValuesRange 使用 Range("D1").Resize(1, lastDay) 動態擴展範圍，從 D1 開始並橫向擴展至當月的最後一天，這樣無論月份天數變動，XValues 都會自動調整範圍。



這樣可以針對橫向排列的日期自動適應每月的不同天數。



若使用VBA來讀取 SeriesCollection.XValues，且不同月份的天數造成XValues的長度不一致，可能導致程式錯誤。解決方法之一是動態調整 XValues 的範圍，讓其自動適應每個月份的長度。

以下是處理這種情況的步驟：

1. 取得XValues的日期範圍：動態計算當前月份的天數，並根據這個範圍設定XValues。


2. 計算當月的天數：可以使用 VBA 來取得當前月份的最後一天，以確保 XValues 覆蓋正確的天數。


3. 設定XValues範圍：根據所計算的天數，重新設定 SeriesCollection.XValues 的範圍。



以下是範例程式碼：

Sub AdjustSeriesXValues()
    Dim ws As Worksheet
    Dim chartObj As ChartObject
    Dim srs As Series
    Dim lastDay As Integer
    Dim startDate As Date
    Dim endDate As Date
    Dim xValuesRange As Range

    ' 設定工作表與圖表
    Set ws = ThisWorkbook.Sheets("Sheet1") ' 請更改為你的工作表名稱
    Set chartObj = ws.ChartObjects("Chart1") ' 請更改為你的圖表名稱
    Set srs = chartObj.Chart.SeriesCollection(1) ' 假設第一個系列

    ' 取得當前月份的第一天和最後一天
    startDate = DateSerial(Year(Date), Month(Date), 1)
    lastDay = Day(DateSerial(Year(Date), Month(Date) + 1, 0))
    endDate = DateSerial(Year(Date), Month(Date), lastDay)

    ' 設定XValues的範圍，根據當月的天數
    Set xValuesRange = ws.Range("A2:A" & lastDay + 1) ' 假設日期從A2開始
    srs.XValues = xValuesRange

    MsgBox "XValues已根據當月天數自動調整！"
End Sub

說明：

1. startDate 和 endDate 用來確定當月的範圍。


2. xValuesRange 根據 lastDay 動態調整，這樣無論月份有多少天，都能正確設定 XValues。



這樣可以讓 XValues 每月自動調整，以避免月份天數不同造成的程式錯誤。

