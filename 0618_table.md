了解，你希望在 Content Page 的表格中，將**直的奇數列**（即第1、3、5...列）設置背景色，以增強視覺效果，並確保與 Master Page 的顏色主題（深藍色 `#003366`、淺藍色 `#e3f2fd`、漸層色 `#b3e5fc` 和 `#e1f5fe`）一致。以下我會修改 Content Page 的 CSS，添加直的奇數列背景色，並保持表格、按鈕、下拉選單等其他元素的樣式與 Master Page 協調。同時，我會確保 CSS 與 Master Page 分離，並使用你偏好的外部 CSS 檔案方式（`content-styles.css`）。

### 設計目標
1. **直的奇數列背景色**：
   - 為表格的第1、3、5...列（`td:nth-child(odd)`）添加背景色，例如淺藍色（`#f5faff`），以區分列。
2. **顏色一致性**：
   - 使用與 Master Page 相同的顏色主題（`#003366`、`#e3f2fd`、`#90caf9`、`#f5faff`）。
3. **CSS 分離**：
   - 將表格、按鈕、下拉選單等樣式獨立於 `Content/content-styles.css`，與 Master Page 的 CSS 分開。
4. **其他元素**：
   - 保留按鈕（主要、次要、危險）、下拉選單、輸入框的樣式，確保一致性。
5. **響應式**：
   - 確保表格在小螢幕（手機、平板）上正常顯示，支援水平滾動。

### 修改 CSS
我會更新 `content-styles.css`，為表格的奇數列添加背景色，並保留其他元素的樣式。Master Page 的 CSS 保持不變，僅包含頁首、側邊欄、頁尾和 `TreeView` 的樣式。

#### 更新 Content Page CSS (Content/content-styles.css)
```css
/* content-styles.css */

/* 表格樣式 */
.table-container {
    width: 100%;
    overflow-x: auto;
    margin-bottom: 20px;
}

table {
    width: 100%;
    border-collapse: collapse;
    background-color: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    overflow: hidden;
}

th, td {
    padding: 12px 15px;
    text-align: left;
    border-bottom: 1px solid #e3f2fd;
}

th {
    background-color: #003366;
    color: white;
    font-weight: 600;
    text-transform: uppercase;
    font-size: 14px;
}

td {
    color: #333;
    font-size: 14px;
}

/* 直的奇數列背景色 */
td:nth-child(odd) {
    background-color: #f5faff; /* 淺藍色背景，與主題一致 */
}

/* 表格行 hover 效果 */
tr:hover {
    background-color: #e3f2fd; /* hover 時使用更深的淺藍色 */
}

/* 按鈕樣式 */
.btn {
    display: inline-block;
    padding: 8px 16px;
    border-radius: 4px;
    text-decoration: none;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s ease, transform 0.1s ease;
    border: none;
    outline: none;
}

.btn-primary {
    background-color: #003366;
    color: white;
}

.btn-primary:hover {
    background-color: #002244;
    transform: translateY(-1px);
}

.btn-primary:disabled, .btn-primary.disabled {
    background-color: #90caf9;
    cursor: not-allowed;
    opacity: 0.6;
}

.btn-secondary {
    background-color: #e3f2fd;
    color: #003366;
    border: 1px solid #003366;
}

.btn-secondary:hover {
    background-color: #cbe3ff;
    transform: translateY(-1px);
}

.btn-secondary:disabled, .btn-secondary.disabled {
    background-color: #f5faff;
    color: #666;
    cursor: not-allowed;
    border: 1px solid #ccc;
}

.btn-danger {
    background-color: #d32f2f;
    color: white;
}

.btn-danger:hover {
    background-color: #b71c1c;
    transform: translateY(-1px);
}

.btn-danger:disabled, .btn-danger.disabled {
    background-color: #ef9a9a;
    cursor: not-allowed;
    opacity: 0.6;
}

/* 下拉選單樣式 */
select, .dropdown-list {
    appearance: none;
    -webkit-appearance: none;
    -moz-appearance: none;
    background-color: #fff;
    border: 1px solid #e3f2fd;
    border-radius: 4px;
    padding: 8px 30px 8px 12px;
    font-size: 14px;
    color: #333;
    width: 100%;
    max-width: 300px;
    background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%23003366"><path d="M7 10l5 5 5-5z"/></svg>');
    background-repeat: no-repeat;
    background-position: right 10px center;
    background-size: 16px;
    transition: border-color 0.2s ease;
}

select:focus, .dropdown-list:focus {
    border-color: #003366;
    outline: none;
    box-shadow: 0 0 5px rgba(0, 51, 102, 0.3);
}

select:disabled, .dropdown-list:disabled {
    background-color: #f5faff;
    color: #666;
    cursor: not-allowed;
}

/* 輸入框和文字區域 */
input[type="text"], input[type="email"], input[type="number"], textarea {
    border: 1px solid #e3f2fd;
    border-radius: 4px;
    padding: 8px 12px;
    font-size: 14px;
    color: #333;
    width: 100%;
    max-width: 300px;
    transition: border-color 0.2s ease;
}

input:focus, textarea:focus {
    border-color: #003366;
    outline: none;
    box-shadow: 0 0 5px rgba(0, 51, 102, 0.3);
}

input:disabled, textarea:disabled {
    background-color: #f5faff;
    color: #666;
    cursor: not-allowed;
}

/* ASP.NET 控制項樣式 */
.aspNetDisabled {
    background-color: #f5faff !important;
    color: #666 !important;
    cursor: not-allowed !important;
    opacity: 0.6;
}

/* 響應式調整 */
@media (max-width: 768px) {
    table {
        font-size: 13px;
    }
    th, td {
        padding: 8px 10px;
    }
    .btn {
        padding: 6px 12px;
        font-size: 13px;
    }
    select, input[type="text"], input[type="email"], input[type="number"], textarea {
        font-size: 13px;
        padding: 6px 10px;
    }
}
```

#### 修改說明
1. **奇數列背景色**：
   - 新增 `td:nth-child(odd) { background-color: #f5faff; }`，為第1、3、5...列的單元格設置淺藍色背景。
   - 移除之前的 `tr:nth-child(even)`（偶數行背景色），以避免與直的奇數列背景色混淆。
   - `tr:hover` 使用更深的淺藍色（`#e3f2fd`），確保 hover 效果覆蓋奇數列背景。
2. **顏色一致性**：
   - 奇數列背景色（`#f5faff`）與 Master Page 的側邊欄（`#e3f2fd`）和 hover 效果協調。
   - 表格標頭（`#003366`）與頁首一致，分隔線（`#e3f2fd`）與側邊欄背景匹配。
3. **其他元素**：
   - 按鈕（`.btn-primary`、`#003366`）、下拉選單（`.dropdown-list`）、輸入框的邊框和禁用狀態（`#f5faff`）保持與 Master Page 主題一致。
4. **響應式**：
   - 表格在小螢幕下縮減字體（13px）和內距（8px 10px），確保可讀性。

#### Content Page (SamplePage.aspx)
以下是 Content Page，引用外部 CSS 並展示表格（含奇數列背景）、按鈕、下拉選單和輸入框：

```html
<%@ Page Language="C#" MasterPageFile="~/Site.master" AutoEventWireup="true" CodeFile="SamplePage.aspx.cs" Inherits="SamplePage" %>

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <link href="Content/content-styles.css" rel="stylesheet" />

    <h2>資料管理</h2>
    <div class="table-container">
        <table>
            <thead>
                <tr>
                    <th>編號</th>
                    <th>名稱</th>
                    <th>狀態</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>001</td>
                    <td>項目 A</td>
                    <td>進行中</td>
                    <td>
                        <asp:Button ID="EditButton" runat="server" Text="編輯" CssClass="btn btn-primary" />
                        <asp:Button ID="DeleteButton" runat="server" Text="刪除" CssClass="btn btn-danger" />
                    </td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>項目 B</td>
                    <td>已完成</td>
                    <td>
                        <asp:Button ID="ViewButton" runat="server" Text="檢視" CssClass="btn btn-secondary" />
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

    <div style="margin-bottom: 20px;">
        <label for="SampleDropDown">選擇類別：</label>
        <asp:DropDownList ID="SampleDropDown" runat="server" CssClass="dropdown-list">
            <asp:ListItem Text="選項 1" Value="1" />
            <asp:ListItem Text="選項 2" Value="2" />
            <asp:ListItem Text="選項 3" Value="3" />
        </asp:DropDownList>
    </div>

    <div style="margin-bottom: 20px;">
        <label for="SampleTextBox">輸入名稱：</label>
        <asp:TextBox ID="SampleTextBox" runat="server" placeholder="輸入文字..." />
    </div>

    <asp:Button ID="SubmitButton" runat="server" Text="提交" CssClass="btn btn-primary" />
</asp:Content>
```

#### Master Page (Site.master)
Master Page 的程式碼保持不變，僅包含核心結構和 `TreeView` 樣式，已在先前提供。這裡確認其圖片路徑和 CSS 未包含表格相關樣式：

```html
<!DOCTYPE html>
<html>
<head runat="server">
    <title>南亞科技系統</title>
    <link href="Content/bootstrap.css" rel="stylesheet" />
    <script src="Scripts/bootstrap.bundle.js"></script>
    <style>
        body, html {
            height: 100%;
            margin: 0;
            font-family: "Segoe UI", "微軟正黑體", sans-serif;
        }

        .header {
            background-color: #003366;
            color: white;
            padding: 10px 20px;
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
        }

        .header img {
            height: 40px;
            position: absolute;
            left: 20px;
            top: 50%;
            transform: translateY(-50%);
        }

        .header h4 {
            margin: 0;
        }

        .sidebar {
            background-color: #e3f2fd;
            width: 220px;
            padding: 20px;
            overflow-y: auto;
        }

        .treeview-style {
            font-size: 15px;
            color: #003366;
        }

        .treeview-style ul {
            list-style: none;
            padding-left: 0;
        }

        .treeview-style li {
            margin-bottom: 8px;
            position: relative;
        }

        .treeview-style img[src*="plus.gif"],
        .treeview-style img[src*="minus.gif"] {
            display: none;
        }

        .treeview-style a {
            color: #003366;
            padding: 8px 12px;
            display: block;
            border-radius: 4px;
            text-decoration: none;
            transition: background-color 0.2s ease, color 0.2s ease;
            position: relative;
            padding-left: 30px;
        }

        .treeview-style a:hover {
            background-color: #cbe3ff;
            color: #002244;
        }

        .treeview-style a.selected {
            background-color: #90caf9;
            color: #003366;
            font-weight: bold;
        }

        .treeview-style ul ul {
            padding-left: 20px;
        }

        .treeview-style .AspNet-TreeView-Expand::before,
        .treeview-style .AspNet-TreeView-Collapse::before {
            position: absolute;
            left: 10px;
            font-size: 12px;
            color: #003366;
            top: 50%;
            transform: translateY(-50%);
        }

        .treeview-style .AspNet-TreeView-Expand::before {
            content: '\25B6';
        }

        .treeview-style .AspNet-TreeView-Collapse::before {
            content: '\25BC';
        }

        .treeview-style .AspNet-TreeView-Leaf > a::before {
            content: '\2022';
            position: absolute;
            left: 10px;
            font-size: 12px;
            color: #003366;
            top: 50%;
            transform: translateY(-50%);
        }

        .treeview-style ul ul li::before {
            content: '';
            position: absolute;
            left: 5px;
            top: -10px;
            height: calc(100% + 10px);
            width: 1px;
            background-color: #90caf9;
        }

        .treeview-style ul ul li::after {
            content: '';
            position: absolute;
            left: 5px;
            top: 12px;
            width: 10px;
            height: 1px;
            background-color: #90caf9;
        }

        .content {
            flex-grow: 1;
            padding: 20px;
            background-color: #fff;
        }

        .footer {
            background: linear-gradient(to right, #b3e5fc, #e1f5fe);
            padding: 10px 20px;
            text-align: center;
            color: #333;
            font-size: 14px;
        }

        .layout {
            display: flex;
            flex-direction: column;
            height: 100%;
        }

        .main-area {
            display: flex;
            flex-grow: 1;
            overflow: hidden;
        }
    </style>
</head>
<body>
    <form id="form1" runat="server" class="layout">
        <!-- 頂部 -->
        <div class="header">
            <img src="~/images/nanya.png" runat="server" alt="南亞科技" />
            <h4>南亞科技內部系統</h4>
        </div>

        <!-- 中間主要區域 -->
        <div class="main-area">
            <!-- 左側選單 -->
            <div class="sidebar">
                <asp:TreeView ID="TreeView1" runat="server" CssClass="treeview-style" ShowExpandCollapse="True" ExpandDepth="1" />
            </div>

            <!-- 右側主內容 -->
            <div class="content">
                <asp:ContentPlaceHolder ID="MainContent" runat="server" />
            </div>
        </div>

        <!-- 底部 -->
        <div class="footer">
            © 2025 南亞科技股份有限公司. 保留所有權利。
        </div>
    </form>
</body>
</html>
```

### 檔案結構
確保專案目錄如下：
```
YourProject/
├── Site.master
├── Content/
│   ├── bootstrap.css
│   ├── content-styles.css
├── Scripts/
│   ├── bootstrap.bundle.js
├── images/
│   ├── nanya.png
├── SamplePage.aspx
├── SamplePage.aspx.cs
```

### 說明
1. **奇數列背景色**：
   - 使用 `td:nth-child(odd)` 為表格的第1、3、5...列設置背景色 `#f5faff`，與 Master Page 的側邊欄（`#e3f2fd`）和 hover 效果（`#cbe3ff`）協調。
   - 移除 `tr:nth-child(even)`，以避免行背景與列背景衝突。
   - `tr:hover` 使用 `#e3f2fd` 確保 hover 效果覆蓋奇數列背景。
2. **CSS 分離**：
   - Master Page 的 CSS 只包含頁首、側邊欄、頁尾和 `TreeView` 樣式。
   - Content Page 的樣式獨立於 `Content/content-styles.css`，通過 `<link href="Content/content-styles.css" rel="stylesheet" />` 引用。
3. **顏色一致性**：
   - 表格標頭（`#003366`）、分隔線（`#e3f2fd`）、奇數列（`#f5faff`）與 Master Page 的主題一致。
   - 按鈕（`.btn-primary` 使用 `#003366`）、下拉選單和輸入框的邊框（`#e3f2fd`）與側邊欄和頁首協調。
4. **其他元素**：
   - 按鈕（主要、次要、危險）、下拉選單（`<asp:DropDownList>`）、輸入框（`<asp:TextBox>`）的樣式保持不變，支援 hover、focus 和 disabled 狀態。
5. **響應式**：
   - 表格在小螢幕下（`@media (max-width: 768px)`）縮減字體和內距，`.table-container` 支援水平滾動。

### 注意事項
1. **圖片路徑**：
   - 確認 `~/images/nanya.png` 已放置在專案的 `/images` 資料夾，並檢查 `<img src="~/images/nanya.png" runat="server" />` 是否正確解析。
   - 如果圖片無法顯示，檢查瀏覽器開發者工具中的 `<img>` 的 `src` 值（應為 `/YourApp/images/nanya.png`）。
2. **CSS 引用**：
   - 確保 `Content/content-styles.css` 已正確放置在 `/Content` 資料夾，並在 Content Page 中引用。
   - 如果多個 Content Page 需要相同樣式，確保每頁都包含 `<link href="Content/content-styles.css" rel="stylesheet" />`。
3. **Bootstrap 衝突**：
   - CSS 使用具體選擇器（`.btn-primary`、`.dropdown-list`）以避免 Bootstrap 樣式覆蓋。
   - 如果發現樣式異常（例如按鈕或表格樣式被覆蓋），請檢查瀏覽器開發者工具並分享問題細節。
4. **ASP.NET 控制項**：
   - `<asp:Button>`、`<asp:DropDownList>`、`<asp:TextBox>` 的樣式通過 `.btn`、`.dropdown-list` 和 `input` 選擇器應用。
   - 如果使用其他控制項（例如 `<asp:GridView>`、`<asp:CheckBox>`），請提供範例，我可添加對應樣式。
5. **奇數列效果**：
   - 奇數列背景色（`#f5faff`）較淺，若覺得不明顯，可改為更深的顏色（如 `#e3f2fd` 或 `#cbe3ff`），請告知需求。

### 下一步
1. **測試**：
   - 將更新的 `Site.master`、 `Content/content-styles.css` 和 `SamplePage.aspx` 加入專案。
   - 檢查表格的奇數列是否顯示背景色（`#f5faff`），以及按鈕、下拉選單、輸入框是否與 Master Page 主題一致。
   - 確認圖片 `~/images/nanya.png` 在所有頁面正常顯示。
2. **回饋**：
   - 如果奇數列背景色需要調整（例如顏色、透明度），請提供具體要求。
   - 如果需要其他控制項（例如 `<asp:GridView>`、檢查框）或進階功能（例如表格排序），請告知。
3. **除錯**：
   - 如果表格樣式、圖片或其他元素未正確顯示，請使用瀏覽器開發者工具（F12）檢查 HTML 和 CSS，分享問題細節（例如類名、錯誤訊息）。
4. **進階需求**：
   - 如果需要字體圖標（Font Awesome）、動畫效果或暗色模式，請告訴我，我可提供對應程式碼。

請試用以上程式碼，並告訴我結果或需要微調的地方！如果有其他控制項或設計需求，請提供細節，我會立即協助。

了解，你希望將 Content Page 中表格、按鈕、下拉選單等元素的 CSS 樣式與 Master Page 的 CSS 分開，以提高程式碼模組化和可維護性。以下我會將原本整合在 Master Page 的 CSS 分為兩部分：

1. **Master Page CSS**：保留與頁首（header）、側邊欄（sidebar）、頁尾（footer）和 `TreeView` 相關的樣式。
2. **Content Page CSS**：將表格、按鈕、下拉選單、輸入框等元素的樣式獨立出來，放入 Content Page 或外部 CSS 檔案。

我會提供：
- 更新後的 Master Page（僅包含核心樣式）。
- 獨立的 Content Page，包含表格、按鈕、下拉選單等樣式。
- 將 Content Page 樣式放入外部 CSS 檔案的選項。
- 確保顏色主題（深藍色 `#003366`、淺藍色 `#e3f2fd`、漸層色 `#b3e5fc` 和 `#e1f5fe`）一致。

---

### 解決方案

#### 1. Master Page CSS（核心樣式）
Master Page 的 CSS 只保留與頁面結構（header、sidebar、footer、TreeView）相關的樣式，去除表格、按鈕、下拉選單等內容相關的樣式。

**Master Page (Site.master)**：
```html
<!DOCTYPE html>
<html>
<head runat="server">
    <title>南亞科技系統</title>
    <link href="Content/bootstrap.css" rel="stylesheet" />
    <script src="Scripts/bootstrap.bundle.js"></script>
    <style>
        body, html {
            height: 100%;
            margin: 0;
            font-family: "Segoe UI", "微軟正黑體", sans-serif;
        }

        .header {
            background-color: #003366;
            color: white;
            padding: 10px 20px;
            display: flex;
            align-items: center;
            justify-content: center;
            position: relative;
        }

        .header img {
            height: 40px;
            position: absolute;
            left: 20px;
            top: 50%;
            transform: translateY(-50%);
        }

        .header h4 {
            margin: 0;
        }

        .sidebar {
            background-color: #e3f2fd;
            width: 220px;
            padding: 20px;
            overflow-y: auto;
        }

        .treeview-style {
            font-size: 15px;
            color: #003366;
        }

        .treeview-style ul {
            list-style: none;
            padding-left: 0;
        }

        .treeview-style li {
            margin-bottom: 8px;
            position: relative;
        }

        .treeview-style img[src*="plus.gif"],
        .treeview-style img[src*="minus.gif"] {
            display: none;
        }

        .treeview-style a {
            color: #003366;
            padding: 8px 12px;
            display: block;
            border-radius: 4px;
            text-decoration: none;
            transition: background-color 0.2s ease, color 0.2s ease;
            position: relative;
            padding-left: 30px;
        }

        .treeview-style a:hover {
            background-color: #cbe3ff;
            color: #002244;
        }

        .treeview-style a.selected {
            background-color: #90caf9;
            color: #003366;
            font-weight: bold;
        }

        .treeview-style ul ul {
            padding-left: 20px;
        }

        .treeview-style .AspNet-TreeView-Expand::before,
        .treeview-style .AspNet-TreeView-Collapse::before {
            position: absolute;
            left: 10px;
            font-size: 12px;
            color: #003366;
            top: 50%;
            transform: translateY(-50%);
        }

        .treeview-style .AspNet-TreeView-Expand::before {
            content: '\25B6';
        }

        .treeview-style .AspNet-TreeView-Collapse::before {
            content: '\25BC';
        }

        .treeview-style .AspNet-TreeView-Leaf > a::before {
            content: '\2022';
            position: absolute;
            left: 10px;
            font-size: 12px;
            color: #003366;
            top: 50%;
            transform: translateY(-50%);
        }

        .treeview-style ul ul li::before {
            content: '';
            position: absolute;
            left: 5px;
            top: -10px;
            height: calc(100% + 10px);
            width: 1px;
            background-color: #90caf9;
        }

        .treeview-style ul ul li::after {
            content: '';
            position: absolute;
            left: 5px;
            top: 12px;
            width: 10px;
            height: 1px;
            background-color: #90caf9;
        }

        .content {
            flex-grow: 1;
            padding: 20px;
            background-color: #fff;
        }

        .footer {
            background: linear-gradient(to right, #b3e5fc, #e1f5fe);
            padding: 10px 20px;
            text-align: center;
            color: #333;
            font-size: 14px;
        }

        .layout {
            display: flex;
            flex-direction: column;
            height: 100%;
        }

        .main-area {
            display: flex;
            flex-grow: 1;
            overflow: hidden;
        }
    </style>
</head>
<body>
    <form id="form1" runat="server" class="layout">
        <!-- 頂部 -->
        <div class="header">
            <img src="~/images/nanya.png" runat="server" alt="南亞科技" />
            <h4>南亞科技內部系統</h4>
        </div>

        <!-- 中間主要區域 -->
        <div class="main-area">
            <!-- 左側選單 -->
            <div class="sidebar">
                <asp:TreeView ID="TreeView1" runat="server" CssClass="treeview-style" ShowExpandCollapse="True" ExpandDepth="1" />
            </div>

            <!-- 右側主內容 -->
            <div class="content">
                <asp:ContentPlaceHolder ID="MainContent" runat="server" />
            </div>
        </div>

        <!-- 底部 -->
        <div class="footer">
            © 2025 南亞科技股份有限公司. 保留所有權利。
        </div>
    </form>
</body>
</html>
```

#### 2. Content Page CSS（獨立樣式）
Content Page 的 CSS 包含表格、按鈕、下拉選單、輸入框等元素的樣式，與 Master Page 的顏色主題保持一致。以下有兩種實現方式：

##### 選項 1：內嵌於 Content Page
將 CSS 直接寫在 Content Page 的 `<style>` 區塊中，適用於單一頁面或快速測試。

**Content Page (SamplePage.aspx)**：
```html
<%@ Page Language="C#" MasterPageFile="~/Site.master" AutoEventWireup="true" CodeFile="SamplePage.aspx.cs" Inherits="SamplePage" %>

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <style>
        /* 表格樣式 */
        .table-container {
            width: 100%;
            overflow-x: auto;
            margin-bottom: 20px;
        }

        table {
            width: 100%;
            border-collapse: collapse;
            background-color: #fff;
            box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
            border-radius: 8px;
            overflow: hidden;
        }

        th, td {
            padding: 12px 15px;
            text-align: left;
            border-bottom: 1px solid #e3f2fd;
        }

        th {
            background-color: #003366;
            color: white;
            font-weight: 600;
            text-transform: uppercase;
            font-size: 14px;
        }

        td {
            color: #333;
            font-size: 14px;
        }

        tr:hover {
            background-color: #f5faff;
        }

        tr:nth-child(even) {
            background-color: #fafafa;
        }

        /* 按鈕樣式 */
        .btn {
            display: inline-block;
            padding: 8px 16px;
            border-radius: 4px;
            text-decoration: none;
            font-size: 14px;
            font-weight: 500;
            cursor: pointer;
            transition: background-color 0.2s ease, transform 0.1s ease;
            border: none;
            outline: none;
        }

        .btn-primary {
            background-color: #003366;
            color: white;
        }

        .btn-primary:hover {
            background-color: #002244;
            transform: translateY(-1px);
        }

        .btn-primary:disabled, .btn-primary.disabled {
            background-color: #90caf9;
            cursor: not-allowed;
            opacity: 0.6;
        }

        .btn-secondary {
            background-color: #e3f2fd;
            color: #003366;
            border: 1px solid #003366;
        }

        .btn-secondary:hover {
            background-color: #cbe3ff;
            transform: translateY(-1px);
        }

        .btn-secondary:disabled, .btn-secondary.disabled {
            background-color: #f5faff;
            color: #666;
            cursor: not-allowed;
            border: 1px solid #ccc;
        }

        .btn-danger {
            background-color: #d32f2f;
            color: white;
        }

        .btn-danger:hover {
            background-color: #b71c1c;
            transform: translateY(-1px);
        }

        .btn-danger:disabled, .btn-danger.disabled {
            background-color: #ef9a9a;
            cursor: not-allowed;
            opacity: 0.6;
        }

        /* 下拉選單樣式 */
        select, .dropdown-list {
            appearance: none;
            -webkit-appearance: none;
            -moz-appearance: none;
            background-color: #fff;
            border: 1px solid #e3f2fd;
            border-radius: 4px;
            padding: 8px 30px 8px 12px;
            font-size: 14px;
            color: #333;
            width: 100%;
            max-width: 300px;
            background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%23003366"><path d="M7 10l5 5 5-5z"/></svg>');
            background-repeat: no-repeat;
            background-position: right 10px center;
            background-size: 16px;
            transition: border-color 0.2s ease;
        }

        select:focus, .dropdown-list:focus {
            border-color: #003366;
            outline: none;
            box-shadow: 0 0 5px rgba(0, 51, 102, 0.3);
        }

        select:disabled, .dropdown-list:disabled {
            background-color: #f5faff;
            color: #666;
            cursor: not-allowed;
        }

        /* 輸入框和文字區域 */
        input[type="text"], input[type="email"], input[type="number"], textarea {
            border: 1px solid #e3f2fd;
            border-radius: 4px;
            padding: 8px 12px;
            font-size: 14px;
            color: #333;
            width: 100%;
            max-width: 300px;
            transition: border-color 0.2s ease;
        }

        input:focus, textarea:focus {
            border-color: #003366;
            outline: none;
            box-shadow: 0 0 5px rgba(0, 51, 102, 0.3);
        }

        input:disabled, textarea:disabled {
            background-color: #f5faff;
            color: #666;
            cursor: not-allowed;
        }

        /* ASP.NET 控制項樣式 */
        .aspNetDisabled {
            background-color: #f5faff !important;
            color: #666 !important;
            cursor: not-allowed !important;
            opacity: 0.6;
        }

        /* 響應式調整 */
        @media (max-width: 768px) {
            table {
                font-size: 13px;
            }
            th, td {
                padding: 8px 10px;
            }
            .btn {
                padding: 6px 12px;
                font-size: 13px;
            }
            select, input[type="text"], input[type="email"], input[type="number"], textarea {
                font-size: 13px;
                padding: 6px 10px;
            }
        }
    </style>

    <h2>資料管理</h2>
    <div class="table-container">
        <table>
            <thead>
                <tr>
                    <th>編號</th>
                    <th>名稱</th>
                    <th>狀態</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>001</td>
                    <td>項目 A</td>
                    <td>進行中</td>
                    <td>
                        <asp:Button ID="EditButton" runat="server" Text="編輯" CssClass="btn btn-primary" />
                        <asp:Button ID="DeleteButton" runat="server" Text="刪除" CssClass="btn btn-danger" />
                    </td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>項目 B</td>
                    <td>已完成</td>
                    <td>
                        <asp:Button ID="ViewButton" runat="server" Text="檢視" CssClass="btn btn-secondary" />
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

    <div style="margin-bottom: 20px;">
        <label for="SampleDropDown">選擇類別：</label>
        <asp:DropDownList ID="SampleDropDown" runat="server" CssClass="dropdown-list">
            <asp:ListItem Text="選項 1" Value="1" />
            <asp:ListItem Text="選項 2" Value="2" />
            <asp:ListItem Text="選項 3" Value="3" />
        </asp:DropDownList>
    </div>

    <div style="margin-bottom: 20px;">
        <label for="SampleTextBox">輸入名稱：</label>
        <asp:TextBox ID="SampleTextBox" runat="server" placeholder="輸入文字..." />
    </div>

    <asp:Button ID="SubmitButton" runat="server" Text="提交" CssClass="btn btn-primary" />
</asp:Content>
```

##### 選項 2：外部 CSS 檔案
將 Content Page 的樣式提取到外部 CSS 檔案（例如 `Content/content-styles.css`），然後在 Content Page 或 Master Page 中引用，適合多頁面共用樣式。

1. **創建 CSS 檔案 (Content/content-styles.css)**：
```css
/* content-styles.css */
.table-container {
    width: 100%;
    overflow-x: auto;
    margin-bottom: 20px;
}

table {
    width: 100%;
    border-collapse: collapse;
    background-color: #fff;
    box-shadow: 0 2px 4px rgba(0, 0, 0, 0.1);
    border-radius: 8px;
    overflow: hidden;
}

th, td {
    padding: 12px 15px;
    text-align: left;
    border-bottom: 1px solid #e3f2fd;
}

th {
    background-color: #003366;
    color: white;
    font-weight: 600;
    text-transform: uppercase;
    font-size: 14px;
}

td {
    color: #333;
    font-size: 14px;
}

tr:hover {
    background-color: #f5faff;
}

tr:nth-child(even) {
    background-color: #fafafa;
}

.btn {
    display: inline-block;
    padding: 8px 16px;
    border-radius: 4px;
    text-decoration: none;
    font-size: 14px;
    font-weight: 500;
    cursor: pointer;
    transition: background-color 0.2s ease, transform 0.1s ease;
    border: none;
    outline: none;
}

.btn-primary {
    background-color: #003366;
    color: white;
}

.btn-primary:hover {
    background-color: #002244;
    transform: translateY(-1px);
}

.btn-primary:disabled, .btn-primary.disabled {
    background-color: #90caf9;
    cursor: not-allowed;
    opacity: 0.6;
}

.btn-secondary {
    background-color: #e3f2fd;
    color: #003366;
    border: 1px solid #003366;
}

.btn-secondary:hover {
    background-color: #cbe3ff;
    transform: translateY(-1px);
}

.btn-secondary:disabled, .btn-secondary.disabled {
    background-color: #f5faff;
    color: #666;
    cursor: not-allowed;
    border: 1px solid #ccc;
}

.btn-danger {
    background-color: #d32f2f;
    color: white;
}

.btn-danger:hover {
    background-color: #b71c1c;
    transform: translateY(-1px);
}

.btn-danger:disabled, .btn-danger.disabled {
    background-color: #ef9a9a;
    cursor: not-allowed;
    opacity: 0.6;
}

select, .dropdown-list {
    appearance: none;
    -webkit-appearance: none;
    -moz-appearance: none;
    background-color: #fff;
    border: 1px solid #e3f2fd;
    border-radius: 4px;
    padding: 8px 30px 8px 12px;
    font-size: 14px;
    color: #333;
    width: 100%;
    max-width: 300px;
    background-image: url('data:image/svg+xml;utf8,<svg xmlns="http://www.w3.org/2000/svg" viewBox="0 0 24 24" fill="%23003366"><path d="M7 10l5 5 5-5z"/></svg>');
    background-repeat: no-repeat;
    background-position: right 10px center;
    background-size: 16px;
    transition: border-color 0.2s ease;
}

select:focus, .dropdown-list:focus {
    border-color: #003366;
    outline: none;
    box-shadow: 0 0 5px rgba(0, 51, 102, 0.3);
}

select:disabled, .dropdown-list:disabled {
    background-color: #f5faff;
    color: #666;
    cursor: not-allowed;
}

input[type="text"], input[type="email"], input[type="number"], textarea {
    border: 1px solid #e3f2fd;
    border-radius: 4px;
    padding: 8px 12px;
    font-size: 14px;
    color: #333;
    width: 100%;
    max-width: 300px;
    transition: border-color 0.2s ease;
}

input:focus, textarea:focus {
    border-color: #003366;
    outline: none;
    box-shadow: 0 0 5px rgba(0, 51, 102, 0.3);
}

input:disabled, textarea:disabled {
    background-color: #f5faff;
    color: #666;
    cursor: not-allowed;
}

.aspNetDisabled {
    background-color: #f5faff !important;
    color: #666 !important;
    cursor: not-allowed !important;
    opacity: 0.6;
}

@media (max-width: 768px) {
    table {
        font-size: 13px;
    }
    th, td {
        padding: 8px 10px;
    }
    .btn {
        padding: 6px 12px;
        font-size: 13px;
    }
    select, input[type="text"], input[type="email"], input[type="number"], textarea {
        font-size: 13px;
        padding: 6px 10px;
    }
}
```

2. **修改 Content Page 以引用外部 CSS**：
```html
<%@ Page Language="C#" MasterPageFile="~/Site.master" AutoEventWireup="true" CodeFile="SamplePage.aspx.cs" Inherits="SamplePage" %>

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <link href="Content/content-styles.css" rel="stylesheet" />

    <h2>資料管理</h2>
    <div class="table-container">
        <table>
            <thead>
                <tr>
                    <th>編號</th>
                    <th>名稱</th>
                    <th>狀態</th>
                    <th>操作</th>
                </tr>
            </thead>
            <tbody>
                <tr>
                    <td>001</td>
                    <td>項目 A</td>
                    <td>進行中</td>
                    <td>
                        <asp:Button ID="EditButton" runat="server" Text="編輯" CssClass="btn btn-primary" />
                        <asp:Button ID="DeleteButton" runat="server" Text="刪除" CssClass="btn btn-danger" />
                    </td>
                </tr>
                <tr>
                    <td>002</td>
                    <td>項目 B</td>
                    <td>已完成</td>
                    <td>
                        <asp:Button ID="ViewButton" runat="server" Text="檢視" CssClass="btn btn-secondary" />
                    </td>
                </tr>
            </tbody>
        </table>
    </div>

    <div style="margin-bottom: 20px;">
        <label for="SampleDropDown">選擇類別：</label>
        <asp:DropDownList ID="SampleDropDown" runat="server" CssClass="dropdown-list">
            <asp:ListItem Text="選項 1" Value="1" />
            <asp:ListItem Text="選項 2" Value="2" />
            <asp:ListItem Text="選項 3" Value="3" />
        </asp:DropDownList>
    </div>

    <div style="margin-bottom: 20px;">
        <label for="SampleTextBox">輸入名稱：</label>
        <asp:TextBox ID="SampleTextBox" runat="server" placeholder="輸入文字..." />
    </div>

    <asp:Button ID="SubmitButton" runat="server" Text="提交" CssClass="btn btn-primary" />
</asp:Content>
```

3. **檔案結構**：
確保專案目錄如下：
```
YourProject/
├── Site.master
├── Content/
│   ├── bootstrap.css
│   ├── content-styles.css
├── Scripts/
│   ├── bootstrap.bundle.js
├── images/
│   ├── nanya.png
├── SamplePage.aspx
├── SamplePage.aspx.cs
```

### 說明
1. **CSS 分離**：
   - Master Page 只包含頁面結構（header、sidebar、footer、TreeView）的樣式，位於 `<style>` 內。
   - Content Page 的樣式（表格、按鈕、下拉選單、輸入框）可選擇內嵌（選項 1）或外部檔案（選項 2）。
2. **顏色一致性**：
   - 表格標頭（`#003366`）、分隔線（`#e3f2fd`）、hover 背景（`#f5faff`）與 Master Page 的側邊欄和頁首一致。
   - 按鈕（`.btn-primary`、`#003366`）、下拉選單（`#e3f2fd` 邊框）和輸入框使用相同主題色。
3. **外部 CSS 優勢**：
   - 選項 2（`content-styles.css`）適合多頁面共用樣式，減少重複程式碼。
   - 在所有 Content Page 的 `<asp:Content>` 中引用 `<link href="Content/content-styles.css" rel="stylesheet" />`。
4. **圖片路徑**：
   - Master Page 的 `<img src="~/images/nanya.png" runat="server" />` 確保圖片在所有頁面正確解析。
   - 確認 `nanya.png` 已放置在 `/images` 資料夾。

### 注意事項
1. **圖片路徑**：
   - 確保 `/images/nanya.png` 存在於專案根目錄的 `images` 資料夾。
   - 如果圖片仍無法顯示，檢查瀏覽器開發者工具（F12）中的 `<img>` 的 `src` 值，並確認 IIS 靜態檔案設定。
2. **Bootstrap 衝突**：
   - CSS 使用具體選擇器（`.btn-primary`、`.dropdown-list`）以避免 Bootstrap 樣式覆蓋。
   - 如果發現樣式異常，請檢查是否有 Bootstrap 類（例如 `btn`）衝突，我可提高選擇器優先級。
3. **ASP.NET 控制項**：
   - `<asp:Button>`、`<asp:DropDownList>`、`<asp:TextBox>` 的樣式通過 `.btn`、`.dropdown-list` 和 `input` 選擇器應用。
   - 如果使用其他控制項（例如 `<asp:GridView>`、`<asp:CheckBox>`），請提供範例，我可添加對應樣式。
4. **響應式**：
   - 表格和元素在小螢幕（`max-width: 768px`）下縮減字體和內距，確保手機端體驗。
5. **外部 CSS 部署**：
   - 如果使用選項 2，確保 `Content/content-styles.c