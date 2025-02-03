以下提供一個美化範例，使用內嵌 CSS 來調整版面，方便你直接放在 Master Page 的 Content 區塊中：

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <style type="text/css">
        .password-panel {
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.1);
            background-color: #f9f9f9;
        }
        .password-panel h2 {
            text-align: center;
            margin-bottom: 20px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        .form-group label {
            display: block;
            font-weight: bold;
            margin-bottom: 5px;
        }
        .form-group input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        .btn-custom {
            width: 100%;
            padding: 10px;
            background-color: #007bff;
            border: none;
            color: #fff;
            border-radius: 4px;
            cursor: pointer;
        }
        .btn-custom:hover {
            background-color: #0056b3;
        }
        .message {
            text-align: center;
            margin-bottom: 15px;
        }
    </style>

    <asp:Panel ID="pnlPassword" runat="server" CssClass="password-panel">
        <h2>修改密碼</h2>
        <asp:Label ID="lblMessage" runat="server" CssClass="message" ForeColor="Red"></asp:Label>
        
        <div class="form-group">
            <label for="txtOldPassword">舊密碼</label>
            <asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server" ControlToValidate="txtOldPassword" ErrorMessage="請輸入舊密碼" Display="Dynamic" />
        </div>
        
        <div class="form-group">
            <label for="txtNewPassword">新密碼</label>
            <asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server" ControlToValidate="txtNewPassword" ErrorMessage="請輸入新密碼" Display="Dynamic" />
        </div>
        
        <div class="form-group">
            <label for="txtConfirmPassword">確認新密碼</label>
            <asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server" ControlToValidate="txtConfirmPassword" ErrorMessage="請確認新密碼" Display="Dynamic" />
        </div>
        
        <asp:Button ID="btnChange" runat="server" Text="修改密碼" OnClick="btnChange_Click" CssClass="btn-custom" />
    </asp:Panel>
</asp:Content>

簡要說明：

1. CSS 區塊
調整了整個 Panel 的寬度、邊距與邊框，增加陰影效果與圓角，讓版面看起來更柔和。


2. Panel 與標題
將表單內容放入 <asp:Panel>，並以 <h2> 呈現頁面標題。


3. 表單區塊
每個欄位用 .form-group 分組，包含 Label、TextBox 及驗證器，確保版面整齊。


4. 按鈕樣式
使用 .btn-custom 樣式讓按鈕更具現代感與一致的風格。



依需求調整樣式與顏色，即可達到更美觀的效果。若網站已使用 Bootstrap 等框架，也可整合框架樣式。



以下提供一個基本範例，供你參考：


---

步驟 1：建立頁面與控件

1. 新增一個 ASP.NET Web Form（例如 ChangePassword.aspx）。


2. 在頁面上放入三個 TextBox（舊密碼、新密碼、確認新密碼）、一個 Button 及相應的驗證控件。



ChangePassword.aspx 範例：

<%@ Page Language="VB" AutoEventWireup="false" CodeBehind="ChangePassword.aspx.vb" Inherits="YourNamespace.ChangePassword" %>
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>修改密碼</title>
</head>
<body>
    <form id="form1" runat="server">
        <asp:Label ID="lblMessage" runat="server" ForeColor="Red"></asp:Label><br /><br />
        舊密碼：<asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password"></asp:TextBox><br /><br />
        新密碼：<asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password"></asp:TextBox><br /><br />
        確認新密碼：<asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password"></asp:TextBox><br /><br />
        <asp:Button ID="btnChange" runat="server" Text="修改密碼" OnClick="btnChange_Click" /><br /><br />
        <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server" ControlToValidate="txtOldPassword" ErrorMessage="請輸入舊密碼" Display="Dynamic" /><br />
        <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server" ControlToValidate="txtNewPassword" ErrorMessage="請輸入新密碼" Display="Dynamic" /><br />
        <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server" ControlToValidate="txtConfirmPassword" ErrorMessage="請確認新密碼" Display="Dynamic" />
    </form>
</body>
</html>


---

步驟 2：實作程式邏輯

在 CodeBehind（ChangePassword.aspx.vb）中，撰寫按鈕點擊事件來執行下列步驟：

1. 檢查新密碼與確認密碼是否一致。


2. 驗證使用者輸入的舊密碼（依照你目前的密碼驗證機制，建議使用加密比對）。


3. 更新資料庫中的密碼（記得以 Hash 處理）。



ChangePassword.aspx.vb 範例：

Partial Public Class ChangePassword
    Inherits System.Web.UI.Page

    Protected Sub btnChange_Click(sender As Object, e As EventArgs) Handles btnChange.Click
        Dim oldPwd As String = txtOldPassword.Text.Trim()
        Dim newPwd As String = txtNewPassword.Text.Trim()
        Dim confirmPwd As String = txtConfirmPassword.Text.Trim()

        ' 檢查新密碼是否與確認密碼一致
        If newPwd <> confirmPwd Then
            lblMessage.Text = "新密碼與確認密碼不一致。"
            Return
        End If

        ' 驗證舊密碼（請根據實際情況取出使用者的密碼進行比對，建議比對加密後的值）
        If Not ValidateOldPassword(oldPwd) Then
            lblMessage.Text = "舊密碼錯誤。"
            Return
        End If

        ' 更新密碼（更新前請先對新密碼做 Hash 處理）
        If UpdatePassword(newPwd) Then
            lblMessage.ForeColor = Drawing.Color.Green
            lblMessage.Text = "密碼修改成功！"
        Else
            lblMessage.Text = "密碼修改失敗，請稍後再試。"
        End If
    End Sub

    ' 根據實際邏輯實作驗證舊密碼
    Private Function ValidateOldPassword(ByVal oldPwd As String) As Boolean
        ' 範例：從資料庫取出使用者的密碼 Hash，與 oldPwd 的 Hash 比較
        ' 請替換成你自己的驗證邏輯
        Return True
    End Function

    ' 根據實際邏輯實作更新密碼
    Private Function UpdatePassword(ByVal newPwd As String) As Boolean
        ' 範例：對 newPwd 做 Hash 處理後，更新資料庫
        ' 請替換成你自己的資料庫操作邏輯
        Return True
    End Function
End Class


---

注意事項

安全性：密碼應以 Hash 方式儲存，避免明碼存取。驗證時請採用相同的加密機制。

驗證：可搭配 ASP.NET 提供的驗證控件與 Membership 功能，簡化安全與驗證流程。

例外處理：根據實際需求補上例外處理機制，確保使用者體驗。



---

以上步驟與範例提供了一個簡單的 VB WebForm 密碼修改頁面實作範例，可根據你專案的實際情況進行調整。

