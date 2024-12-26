以下列出幾種 UiPath 在遠端電腦執行後，進行截圖的常見方法，並說明各方法的限制與可能成本。為方便閱讀，先大略歸納，再逐項簡述。


---

一、使用 Remote Desktop + UiPath Remote Runtime

1. 操作方式

在遠端電腦上安裝 UiPath Remote Runtime。

本機端則安裝 UiPath Remote Desktop Extension。

透過「Take Screenshot」或其他影像操作 Activity，直接抓取遠端畫面。



2. 優點

可以擁有跟本機類似的選擇器 (Selector) 與螢幕截圖能力。

影像偵測與互動效率較高。



3. 限制

需要在遠端機器上安裝 Remote Runtime，若無法在遠端安裝軟體，則無法使用。

遠端桌面環境需有足夠權限、且要設定網路連線等。



4. 費用

若已擁有 UiPath Studio/Robot Enterprise 許可，UiPath Remote Runtime 通常已包含在授權內。

若為 Community 版本，則依照 Community 授權條款使用，無額外費用。

大型企業環境需購買 Enterprise 套件授權 (以 UiPath 官方定價為準)。





---

二、使用 Citrix / VDI 環境下的 Image Automation (傳統影像自動化)

1. 操作方式

不在遠端電腦上安裝任何 UiPath 執行元件，而是直接透過 Citrix 或 VDI 串流到本機。

使用 UiPath 的「Image」類活動 (如「Click Image」、「Take Screenshot」、「Text OCR」等) 針對螢幕影像進行辨識與截圖。



2. 優點

適用於無法或不便安裝 UiPath Remote Runtime 的環境。

基礎需求只要能遠端顯示畫面即可。



3. 限制

影像操作速度較慢，也容易受螢幕顯示解析度與網路品質影響。

無法直接抓取標準 UI 元件，純影像比對可能出現失誤。



4. 費用

若是 Community 版本，可直接使用 Image Automation。

若為 Enterprise 版本，一樣包含在 UiPath Studio/Robot 授權內。

由於此方式不需安裝 Remote Runtime，因此沒有額外 Runtime 授權的成本。





---

三、使用 Computer Vision (CV)

1. 操作方式

在本機 UiPath 流程中，啟用 UiPath Computer Vision，針對遠端桌面影像進行電腦視覺辨識並截圖。

適用於各種 Citrix、VDI 或 RDP 情境下，只要能取得遠端畫面。



2. 優點

透過電腦視覺比單純的 Image Automation 準確度更高。

部分 UI 介面能被 Computer Vision 自動標記。



3. 限制

電腦視覺的準確度仍依賴螢幕解析度、網路清晰度等因素。

特殊字型或畫面變化較大時，需要進行細部調整。



4. 費用

在 Community 版中，Computer Vision API 有使用次數限制。若超出額度，需購買額外授權。

在 Enterprise 版中，會依照企業授權協議計費。





---

四、在遠端電腦上直接執行 UiPath Robot (前端/後端機器)

1. 操作方式

於遠端電腦本身安裝 UiPath Robot/Agent/Studio。

在遠端電腦直接執行流程，可在同機內「Take Screenshot」並將檔案存到指定位置 (雲端或其他共用資料夾)。



2. 優點

可以完整存取該遠端電腦的所有檔案/系統資源。

截圖過程中，不受網路速度影響畫面清晰度。



3. 限制

必須能在遠端安裝並執行 UiPath。若 IT 安全性政策限制，則無法採用。

需要設置管道讓截圖檔案可傳回本機或存放處。



4. 費用

遠端電腦需要有 UiPath 授權 (Robot/Studio) 或使用 Orchestrator 授權做派工。

若僅有一份 Studio/Robot 授權，有可能無法同時裝在多台機器，需要額外購買授權。





---

五、間接截圖 (非 UiPath 內建功能)

1. 操作方式

在遠端電腦上使用其他螢幕截圖工具 (Windows Snipping Tool、第三方程式等)。

再透過 UiPath 執行該截圖工具、並將結果傳回本地或雲端。

也可直接呼叫 Windows CMD/PowerShell + PrintScreen 指令，將螢幕圖存在遠端再傳回。



2. 優點

若本身遠端已經內建或慣用某截圖工具，可沿用舊流程。



3. 限制

與 UiPath 整合度較低，需要自訂額外流程或指令。

工具若不支援參數化或無法安裝，會造成自動化困難。



4. 費用

截圖工具多為免費或系統附帶，成本取決於該工具授權。

不需額外購買 UiPath 元件授權，但需投時間成本整合。





---

小結

最完整做法：安裝 UiPath Remote Runtime (方法一) 或在遠端電腦裝 UiPath Robot (方法四)。

不方便安裝 Runtime：可用 Citrix/Image Automation (方法二) 或 Computer Vision (方法三)。

無法使用 UiPath 內建截圖：可考慮間接截圖方式 (方法五)。


費用部分主要分為「UiPath Remote Runtime / Robot 授權成本」與「Computer Vision API 使用成本」，若已擁有 Enterprise 授權或在 Community 版限制內，則多數功能可直接使用。

以上為 UiPath 在操作遠端電腦後截圖的各種可行方法及其限制與費用。可根據遠端環境限制、公司預算與既有系統狀況，選擇最適合的解決方案。

