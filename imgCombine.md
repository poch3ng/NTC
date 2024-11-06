在 UiPath 中，您可以使用 VB.NET 代碼來合併和裁切圖片。以下是使用 VB.NET 和 System.Drawing 命名空間進行圖片合併和裁切的範例。這樣可以避免依賴外部工具，直接在 UiPath 中完成操作。

步驟概述

1. 在 UiPath 中截取兩張圖片，並保存到指定路徑。


2. 使用 VB.NET 代碼合併圖片並裁切，將合併後的結果保存為新圖片。



詳細步驟

步驟 1：在 UiPath 中截取圖片

1. 使用 Take Screenshot 活動分別截取需要的兩個網頁區域。


2. 使用 Save Image 活動將這兩張截圖保存到指定路徑，例如：

screenshot1.png

screenshot2.png




步驟 2：使用 VB.NET 代碼合併和裁切圖片

可以在 UiPath 中使用 Invoke Code 活動，直接編寫 VB.NET 代碼來合併和裁切圖片。

VB.NET 代碼範例

將以下代碼複製到 Invoke Code 活動中，並配置參數以傳遞圖片路徑和裁切區域。

' 必要的命名空間
Imports System.Drawing

' 定義輸入圖片路徑和輸出路徑
Dim img1Path As String = "screenshot1.png"
Dim img2Path As String = "screenshot2.png"
Dim outputPath As String = "final_output.png"

' 加載兩張圖片
Dim img1 As Image = Image.FromFile(img1Path)
Dim img2 As Image = Image.FromFile(img2Path)

' 計算合併後的圖片大小
Dim width As Integer = Math.Max(img1.Width, img2.Width)
Dim height As Integer = img1.Height + img2.Height

' 創建新畫布
Using mergedImage As New Bitmap(width, height)
    Using g As Graphics = Graphics.FromImage(mergedImage)
        ' 將第一張圖片繪製到畫布的頂部
        g.DrawImage(img1, 0, 0)
        ' 將第二張圖片繪製到畫布的底部
        g.DrawImage(img2, 0, img1.Height)
    End Using

    ' 裁切區域設定 (x, y, width, height)
    Dim cropArea As New Rectangle(0, 0, width, 500) ' 此處設定裁切的區域大小

    ' 執行裁切
    Using croppedImage As Bitmap = mergedImage.Clone(cropArea, mergedImage.PixelFormat)
        ' 保存最終圖片
        croppedImage.Save(outputPath, System.Drawing.Imaging.ImageFormat.Png)
    End Using
End Using

' 釋放圖片資源
img1.Dispose()
img2.Dispose()

參數說明

img1Path 和 img2Path：這是您之前保存的兩張截圖的路徑。

outputPath：合併並裁切後的最終輸出圖片路徑。

cropArea：裁切區域，您可以根據需求修改此區域大小 (x, y, width, height)。


注意事項

確保圖片路徑正確且圖片已保存至指定位置。

如果截圖尺寸不固定，可以動態設置裁切區域的大小（如透過引數傳入裁切範圍）。


小結

這段 VB.NET 代碼能夠在 UiPath 中實現圖片的合併和裁切操作，不依賴外部工具，且方便調整裁切區域。

