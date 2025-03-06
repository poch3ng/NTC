如果你的 Label 是在 LoginView 的 Login 控制項內，你需要先 找到 Login 控制項，再用 FindControl() 找到 Label 並設定它的值。


---

方法 1：在 Login1_LoggedIn 事件中設定 Label

當使用者成功登入後，Login 控制項會觸發 LoggedIn 事件，你可以在這裡設定 Label 的值：

Protected Sub Login1_LoggedIn(ByVal sender As Object, ByVal e As EventArgs)
    Dim loginControl As Login = CType(LoginView1.FindControl("Login1"), Login)
    If loginControl IsNot Nothing Then
        Dim lbl As Label = CType(loginControl.FindControl("Label1"), Label)
        If lbl IsNot Nothing Then
            lbl.Text = "登入成功！歡迎, " & User.Identity.Name
        End If
    End If
End Sub

對應的 .aspx

<asp:LoginView ID="LoginView1" runat="server">
    <AnonymousTemplate>
        <asp:Login ID="Login1" runat="server" OnLoggedIn="Login1_LoggedIn">
            <LayoutTemplate>
                <asp:Label ID="Label1" runat="server" ForeColor="Red"></asp:Label>
                <asp:TextBox ID="UserName" runat="server"></asp:TextBox>
                <asp:TextBox ID="Password" runat="server" TextMode="Password"></asp:TextBox>
                <asp:Button ID="LoginButton" runat="server" Text="登入" />
            </LayoutTemplate>
        </asp:Login>
    </AnonymousTemplate>
</asp:LoginView>

說明

1. Login1_LoggedIn 事件在使用者成功登入後執行。


2. 先 找到 Login 控制項 (Login1)。


3. 再 用 FindControl("Label1") 找到 Label 並設定值。




---

方法 2：在 Page_Load 中檢查並設定

如果 Label 的值應該在頁面載入時設定，而不等使用者登入後才顯示，可以在 Page_Load 設定：

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        Dim loginControl As Login = CType(LoginView1.FindControl("Login1"), Login)
        If loginControl IsNot Nothing Then
            Dim lbl As Label = CType(loginControl.FindControl("Label1"), Label)
            If lbl IsNot Nothing Then
                lbl.Text = "請輸入您的帳號和密碼"
            End If
        End If
    End If
End Sub

適用情境

這種方法適用於 在登入畫面顯示初始訊息，但不會影響登入後的狀態。



---

方法 3：如果 Label 在 LoginTemplate 內

如果你的 Login 控制項是使用 LoginTemplate，你可以這樣找：

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    Dim loginControl As Control = LoginView1.FindControl("Login1")
    If loginControl IsNot Nothing Then
        Dim loginTemplate As Control = loginControl.FindControl("LoginTemplate")
        If loginTemplate IsNot Nothing Then
            Dim lbl As Label = CType(loginTemplate.FindControl("Label1"), Label)
            If lbl IsNot Nothing Then
                lbl.Text = "請輸入您的登入資訊"
            End If
        End If
    End If
End Sub


---

結論

如果你要在 登入後才顯示 訊息，方法 1（Login1_LoggedIn）最合適！

