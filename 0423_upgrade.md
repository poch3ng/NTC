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

