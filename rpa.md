在 UiPath 中使用 Invoke Code 活動時，確保 Imports System.Drawing 能夠正常使用圖像處理功能，有時可能需要手動在編碼區域中指定類的完整命名空間。如果 Image 模棱兩可，可能是由於 System.Web.UI.WebControls 或其他命名空間中的 Image 類造成的命名衝突。

以下是具體的解決方法：

方法 1：在代碼中指定完整的命名空間

當遇到 Image 模棱兩可的情況時，可以在代碼中明確指定 System.Drawing.Image，這樣可以避免命名衝突。

代碼範例

Imports System.Drawing

' 定義圖片路徑
Dim inputImagePath As String = "C:\Images\P3Chart.png" ' 原始圖片路徑
Dim outputImagePath As String = "C:\Images\P3Chart_cropped.png" ' 裁切後圖片的保存路徑

' 裁切範圍的設定（從 x=0 到 x=910 的區域，保留原始高度）
Dim cropWidth As Integer = 910
Dim cropHeight As Integer

' 加載原始圖片，使用 System.Drawing.Image 來避免模棱兩可
Using originalImage As System.Drawing.Image = System.Drawing.Image.FromFile(inputImagePath)
    ' 設置裁切高度為圖片的原始高度
    cropHeight = originalImage.Height

    ' 設置裁切區域
    Dim cropArea As New Rectangle(0, 0, cropWidth, cropHeight)

    ' 創建新的位圖，用於存儲裁切後的圖片
    Using croppedImage As New Bitmap(cropWidth, cropHeight)
        Using g As Graphics = Graphics.FromImage(croppedImage)
            ' 裁切圖片，僅保留指定範圍
            g.DrawImage(originalImage, New Rectangle(0, 0, cropWidth, cropHeight), cropArea, GraphicsUnit.Pixel)
        End Using

        ' 保存裁切後的圖片
        croppedImage.Save(outputImagePath)
    End Using
End Using

方法 2：在 Imports 中指定完整命名空間

在 Invoke Code 活動的 Imports 部分中，明確導入 System.Drawing，以確保正確使用 System.Drawing.Image 類：

1. 在 UiPath 的 Invoke Code 活動中，找到 Imports 屬性。


2. 在 Imports 中添加 System.Drawing，這樣可以使 Image 類默認為 System.Drawing.Image。


3. 如果仍有衝突，可以在代碼中使用 System.Drawing.Image。



方法 3：只使用 System.Drawing.Bitmap

如果您只需使用 Bitmap 類，可以直接用 Bitmap，並避免使用 Image。這樣可以避免命名衝突：

' 使用 Bitmap 類直接加載圖片
Using originalImage As New Bitmap(inputImagePath)
    ' 其他代碼與之前一致
End Using

這樣可以避免 Image 的模棱兩可問題，並使代碼更加簡潔。



如果您希望截取圖片 P3Chart.png 中寬度的 0 到 910 像素這一段（即對圖片進行裁切，而不是縮放），可以使用 UiPath 的 Invoke Code 活動，並編寫 VB.NET 代碼來裁切圖片。以下是具體操作步驟：

步驟 1：截圖並保存圖片

使用 Take Screenshot 和 Save Image 活動將網頁上的圖表保存為 P3Chart.png。

步驟 2：使用 Invoke Code 活動裁切圖片

在 Invoke Code 活動中使用 VB.NET 的 System.Drawing 命名空間來裁切圖片的指定區域。以下是完整的 VB.NET 代碼：

Imports System.Drawing

' 定義圖片路徑
Dim inputImagePath As String = "C:\Images\P3Chart.png" ' 原始圖片路徑
Dim outputImagePath As String = "C:\Images\P3Chart_cropped.png" ' 裁切後圖片的保存路徑

' 裁切範圍的設定（從 x=0 到 x=910 的區域，保留原始高度）
Dim cropWidth As Integer = 910
Dim cropHeight As Integer

' 加載原始圖片
Using originalImage As Image = Image.FromFile(inputImagePath)
    ' 設置裁切高度為圖片的原始高度
    cropHeight = originalImage.Height

    ' 設置裁切區域
    Dim cropArea As New Rectangle(0, 0, cropWidth, cropHeight)

    ' 創建新的位圖，用於存儲裁切後的圖片
    Using croppedImage As New Bitmap(cropWidth, cropHeight)
        Using g As Graphics = Graphics.FromImage(croppedImage)
            ' 裁切圖片，僅保留指定範圍
            g.DrawImage(originalImage, New Rectangle(0, 0, cropWidth, cropHeight), cropArea, GraphicsUnit.Pixel)
        End Using

        ' 保存裁切後的圖片
        croppedImage.Save(outputImagePath)
    End Using
End Using

代碼說明

1. inputImagePath 和 outputImagePath：分別是原始圖片路徑和裁切後圖片的保存路徑。請根據您的文件夾路徑修改這些值。


2. cropWidth 和 cropHeight：cropWidth 設置為 910，cropHeight 設置為原始圖片的高度。這樣可以確保裁切區域僅保留圖片的左側 0 到 910 像素。


3. cropArea：使用 Rectangle 類來指定裁切區域，從 (0, 0) 開始，寬度為 910，並保留整個高度。


4. 使用 Graphics 物件將圖片的指定區域繪製到新圖片上，並保存裁切後的圖片。



操作結果

執行這段代碼後，您將得到一張寬度為 910 像素的裁切圖片，從原始圖片的左側開始，保留圖片的整個高度。裁切後的圖片將保存為 P3Chart_cropped.png，存儲在您指定的路徑下。



您可以將所有步驟都寫入一段 JavaScript 代碼，並使用 Inject JS Script 在 UiPath 中一次性執行，這樣就能在 JavaScript 中獲取視窗大小、目標元素大小，計算所需的縮放比例並直接應用到網頁上。以下是完整的 JavaScript 代碼：

完整的 JavaScript 代碼範例

這段代碼將根據目標元素的大小與視窗大小進行比較，並自動縮放頁面，以便目標元素在視窗內完全可見。

// 設定目標元素的 ID
var elementId = "targetElement";

// 獲取視窗大小
var windowWidth = window.innerWidth;
var windowHeight = window.innerHeight;

// 獲取目標元素
var element = document.getElementById(elementId);

// 確認元素存在
if (element) {
    // 獲取元素的寬高
    var elementWidth = element.offsetWidth;
    var elementHeight = element.offsetHeight;

    // 計算所需的縮放比例
    var scaleWidth = windowWidth / elementWidth;
    var scaleHeight = windowHeight / elementHeight;
    var scale = Math.min(scaleWidth, scaleHeight);

    // 限制縮放比例最大為 1（即 100%）
    if (scale > 1) {
        scale = 1;
    }

    // 應用縮放比例到頁面
    document.body.style.zoom = scale;
    
    // 回傳縮放比例，方便 UiPath 檢查結果
    return scale;
} else {
    // 如果元素不存在，回傳錯誤訊息
    return "Element not found";
}

說明

1. 目標元素 ID：修改 elementId 變數的值為您目標元素的 ID，例如 "targetElement"。


2. 視窗大小：使用 window.innerWidth 和 window.innerHeight 獲取當前視窗的寬度和高度。


3. 元素大小：使用 element.offsetWidth 和 element.offsetHeight 獲取元素的寬高。


4. 計算縮放比例：比較視窗和元素大小的比例，選擇最小的比例作為縮放比例，確保元素完全顯示在視窗中。


5. 應用縮放：將計算出的 scale 應用到 document.body.style.zoom，縮放整個頁面。


6. 回傳結果：如果元素找到並成功縮放，返回縮放比例 scale；如果找不到元素，返回 "Element not found" 以便 UiPath 判斷是否成功。



使用方法

1. 在 UiPath 中拖入 Inject JS Script 活動。


2. 將以上 JavaScript 代碼貼入 ScriptCode 欄位。


3. 設置一個 Output 變數（例如 zoomScale）來接收返回值。

若返回值為縮放比例數字，表示成功。

若返回 "Element not found"，則表示目標元素不存在。




這段代碼能夠在 UiPath 中一次性完成所有操作，根據視窗大小自動調整目標元素的顯示比例。



要實現根據網頁上某個元素的大小自動調整縮放比例，可以使用 UiPath 結合 JavaScript 來獲取元素的大小，並自動縮小或調整網頁顯示比例。這裡是一個分步指南來完成這個需求：

方法概述

1. 檢查元素的大小：使用 Inject JS Script 活動來獲取網頁中某個元素的寬高。


2. 計算縮放比例：根據獲取的元素大小和螢幕顯示區域大小計算合適的縮放比例。


3. 自動調整縮放：如果元素超出螢幕顯示區域，就調整網頁縮放比例至合適大小。



詳細步驟

步驟 1：使用 Inject JS Script 獲取元素大小

1. 在 UiPath 中添加 Inject JS Script 活動。


2. 編寫 JavaScript 代碼來獲取指定元素的大小，並將其返回給 UiPath。


3. 在 UiPath 中創建變數來接收元素的寬度和高度。



JavaScript 代碼範例

假設目標元素的 id 為 "targetElement"，以下 JavaScript 代碼可以獲取元素的寬度和高度：

// 獲取目標元素
var element = document.getElementById("targetElement");

// 獲取元素的寬高
var width = element.offsetWidth;
var height = element.offsetHeight;

// 返回寬度和高度
return { width: width, height: height };

將這段代碼放入 Inject JS Script 的 ScriptCode 屬性中，並設置輸出引數（例如 elementSize）來接收這個返回值。

步驟 2：計算適當的縮放比例

1. 使用 Invoke Code 活動，將 JavaScript 返回的元素大小和螢幕顯示區域的大小進行比較。


2. 如果元素寬度或高度超過螢幕顯示區域，計算適合的縮放比例。


3. 縮放比例可以這樣計算：

Dim screenWidth As Integer = 1920 ' 螢幕寬度（根據您的螢幕調整）
Dim screenHeight As Integer = 1080 ' 螢幕高度（根據您的螢幕調整）
Dim elementWidth As Integer = elementSize("width")
Dim elementHeight As Integer = elementSize("height")
Dim scale As Double = Math.Min(screenWidth / elementWidth, screenHeight / elementHeight)

' 限制縮放比例
If scale > 1 Then
    scale = 1
End If



步驟 3：根據計算結果調整網頁縮放

1. 使用 Inject JS Script 將縮放比例應用到網頁。


2. 使用以下 JavaScript 代碼來設置縮放比例：

document.body.style.zoom = "<縮放比例>";


3. 將縮放比例以引數傳遞至 Inject JS Script 中，並替換 "<縮放比例>" 為計算出的比例值。



整體流程

1. Step 1：使用 Inject JS Script 獲取元素大小，儲存在變數 elementSize 中。


2. Step 2：在 Invoke Code 活動中計算縮放比例 scale。


3. Step 3：再次使用 Inject JS Script，將 scale 應用到網頁縮放。



這樣，您可以根據元素大小自動縮放網頁顯示比例，確保元素在螢幕上顯示完整而不超出邊界。



以下是標準的 UiPath 專案資料夾結構，您可以依照這個結構自己建立：

UiPath_Project_Template
├── Main
│   └── Main.xaml
├── Workflows
├── Data
│   ├── Input
│   └── Output
├── Screenshots
├── Logs
├── Exceptions
├── Config
│   └── Config.xlsx
├── Libraries
└── Temp

資料夾說明

Main：主流程的 .xaml 檔案（例如 Main.xaml），是專案的進入點。

Workflows：放置子流程或模組化的工作流程。

Data

Input：放置輸入資料檔案（如 Excel、CSV 等）。

Output：放置輸出結果檔案。


Screenshots：存放截圖或錯誤紀錄的影像。

Logs：自定義日誌檔案，追蹤執行狀況。

Exceptions：記錄例外錯誤的檔案。

Config：專案的設定檔案，如 Config.xlsx。

Libraries：自定義函式庫，用於存放可重複使用的元件。

Temp：暫存資料夾，用於存放臨時檔案，流程結束後可以刪除。


依照這個結構建立您的專案資料夾，有助於維持專案的條理性與易用性。



為了讓 Series.XValues 動態根據月份的天數調整資料數量，您可以在程式中先檢查當前月份的天數，然後根據該天數動態設定對應的 XValues 和資料範圍。這樣，您就不需要額外添加日期或數據。

以下是這種方式的實現程式碼，這段程式碼會根據當月天數自動調整 XValues 和資料的範圍：

Sub AdjustSeriesXValuesAndValues()
    Dim ws As Worksheet
    Dim chartObj As ChartObject
    Dim srs As Series
    Dim lastDay As Integer
    Dim endColumn As String
    Dim xValuesRange As Range
    Dim valuesRange As Range

    ' 設定工作表與圖表
    Set ws = ThisWorkbook.Sheets("Sheet1") ' 請更改為你的工作表名稱
    Set chartObj = ws.ChartObjects("Chart1") ' 請更改為你的圖表名稱
    Set srs = chartObj.Chart.SeriesCollection(1) ' 假設是第一個系列

    ' 取得當前月份的最後一天
    lastDay = Day(DateSerial(Year(Date), Month(Date) + 1, 0))

    ' 根據當月天數計算結束欄位 (例如從D1開始)
    endColumn = Split(ws.Cells(1, 4 + lastDay - 1).Address, "$")(1)

    ' 設定 XValues 的範圍為當月的天數 (例如 D1 到對應的結束欄位)
    Set xValuesRange = ws.Range("D1:" & endColumn & "1")
    srs.XValues = xValuesRange

    ' 設定資料的範圍，與 XValues 範圍相同 (假設資料在 D2 開始對應日期)
    Set valuesRange = ws.Range("D2:" & endColumn & "2")
    srs.Values = valuesRange

    MsgBox "XValues 和資料範圍已根據當月天數自動調整至 " & endColumn & "！"
End Sub

說明：

1. lastDay 根據當前月份自動取得最後一天的日期，這樣可以適應每個月不同的天數。


2. endColumn 動態計算 X 軸的範圍結束位置，根據天數自動調整結束欄位。


3. xValuesRange 和 valuesRange 都根據 endColumn 自動擴展或縮短，確保 XValues 和資料的長度一致，避免錯誤。


4. srs.Values 將資料範圍設為與 XValues 相同的天數範圍。



這樣的設定可以保證程式每月自動調整，不需要手動修改範圍。



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

