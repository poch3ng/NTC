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

