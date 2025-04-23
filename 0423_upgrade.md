若你要將 Visual Basic (VB) 的 Website 專案升級為 Web Application 專案，可以依下列步驟操作：


---

升級步驟：VB Website → Web Application

1. 備份原始專案

務必先將整個網站專案資料夾備份，避免升級中遺失原始檔案。



---

2. 建立新的 Web Application 專案

在 Visual Studio 中：

File → New → Project → 選擇 ASP.NET Web Application (.NET Framework)（不是 ASP.NET Core）

選擇 Empty + Web Forms 或依需求加入 Web Forms/MVC 等功能。

設定語言為 Visual Basic。




---

3. 將 Website 檔案搬入 Web Application 專案

將以下類型的檔案搬入新專案：

.aspx, .ascx, .master, .vb, .css, .js, .html, images 資料夾等


注意：不要搬 App_Code、App_Data、Bin 資料夾內容（等下會手動處理）



---

4. 將 App_Code 中的類別轉為編譯類別

在 Web Application 專案中，建立一個新的資料夾（例如 Code）

把 App_Code 中的類別移入這個資料夾

確保這些類別有正確的 Namespace 與 Class 結構



---

5. 修正每個 .aspx 頁面

每個 .aspx 檔案的 @Page 指示詞要加上 CodeBehind 屬性

<%@ Page Language="VB" AutoEventWireup="false" CodeBehind="Default.aspx.vb" Inherits="YourNamespace.Default" %>

確認 Inherits 指向正確的類別名稱（含命名空間）



---

6. 將 Bin 資料夾中 DLL 加入參考

右鍵新專案 → Add Reference... → 加入你原本使用的 DLL（如果有）



---

7. 建立 Global.asax（如果需要）

如果原本沒有 Global.asax，可在 Web Application 專案新增並初始化全域設定。



---

8. 編譯專案

使用 Build → Build Solution 檢查是否有錯誤

修正語法、命名空間錯誤（通常是舊 Website 專案沒有明確 namespace，Web Application 需要明確指定）



---

9. 測試功能

在本機執行網站進行驗證

測試頁面導航、功能邏輯是否一致



---

需要的話我可以幫你提供轉換工具或寫一段 script 協助搬移與調整命名空間，你需要嗎？



以下是將 VB Web Site 專案 轉換為 VB Web Application 專案 的完整步驟：


---

一、建立新的 Web Application 專案

1. 開啟 Visual Studio


2. 點選 檔案 > 新增 > 專案


3. 選擇 ASP.NET Web Application (.NET Framework)


4. 命名專案與指定路徑


5. 選擇範本為：Empty + Web Forms（空白 + Web 表單）


6. 點選「確定」建立




---

二、複製原始檔案

1. 開啟你的原始 VB Web Site 專案資料夾


2. 將以下內容「全部複製」到新 Web Application 專案中：

.aspx, .aspx.vb, .ascx, .ascx.vb, .master, .web.config 等



3. 在 Visual Studio 的 方案總管 中：

點選「顯示所有檔案」

將灰色的檔案右鍵 → Include in Project





---

三、修正語法與設定

Web Site → Web Application 需明確指定 CodeBehind 與 Inherits，檢查以下內容：

1. 頁面語法修正

原本 Web Site：

<%@ Page Language="VB" AutoEventWireup="false" CodeFile="Default.aspx.vb" Inherits="_Default" %>

改為 Web Application：

<%@ Page Language="VB" AutoEventWireup="false" CodeBehind="Default.aspx.vb" Inherits="YourProjectName._Default" %>

2. Code-behind 類別命名空間

檢查 .vb 中是否有：

Namespace YourProjectName
    Public Partial Class _Default
        Inherits System.Web.UI.Page
    End Class
End Namespace

或補上正確命名空間。


---

四、建置與修正錯誤

1. 按 Ctrl + Shift + B 建置專案


2. 修正錯誤（通常是命名空間、未宣告變數、重複名稱等）




---

五、加入至 DevOps Repo 並設置 CI

1. 將 .vbproj 專案提交至 Git 倉庫


2. 在 Azure DevOps / GitLab / GitHub Actions 中設定：

使用 MSBuild 針對 .vbproj 建置




範例 YAML（Azure Pipelines）：

- task: VSBuild@1
  inputs:
    solution: '**/*.vbproj'
    msbuildArgs: '/p:DeployOnBuild=true /p:Configuration=Release'


---

需要我幫你生成轉換後的 .vbproj 檔範本嗎？還是你可以提供你專案的結構，我直接幫你轉？

