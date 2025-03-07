在 VB WebForm (ASP.NET Web Forms) 中，你可以使用 Session 變數來記錄使用者登入錯誤的次數，當錯誤次數達到 4 次時，顯示警告訊息。以下是具體步驟：


---

步驟 1：在登入按鈕的 Click 事件處理邏輯

Protected Sub btnLogin_Click(ByVal sender As Object, ByVal e As EventArgs) Handles btnLogin.Click
    ' 假設這是驗證帳號密碼的方式
    Dim username As String = txtUsername.Text.Trim()
    Dim password As String = txtPassword.Text.Trim()
    
    ' 假設驗證帳密的方法
    If ValidateUser(username, password) Then
        ' 登入成功，清除錯誤次數
        Session("LoginAttempts") = 0
        Response.Redirect("Home.aspx") ' 導向首頁
    Else
        ' 取得目前錯誤次數
        Dim attempts As Integer = 0
        If Session("LoginAttempts") IsNot Nothing Then
            attempts = Convert.ToInt32(Session("LoginAttempts"))
        End If
        
        ' 增加錯誤次數
        attempts += 1
        Session("LoginAttempts") = attempts

        ' 檢查是否達到 4 次
        If attempts >= 4 Then
            lblMessage.Text = "您已經錯誤 4 次，請稍後再試。"
            btnLogin.Enabled = False ' 禁用登入按鈕
        Else
            lblMessage.Text = "登入失敗，您還可以嘗試 " & (4 - attempts) & " 次。"
        End If
    End If
End Sub


---

步驟 2：定義驗證帳號密碼的方法

Private Function ValidateUser(username As String, password As String) As Boolean
    ' 這裡可以改為你的驗證邏輯，例如查詢資料庫
    Return username = "admin" And password = "1234"
End Function


---

步驟 3：讓使用者重新嘗試

如果你希望使用者等一段時間後可以重新嘗試登入，可以在 Page_Load 事件中增加時間檢查：

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        If Session("LoginAttempts") IsNot Nothing AndAlso Convert.ToInt32(Session("LoginAttempts")) >= 4 Then
            lblMessage.Text = "請稍後再試，或聯繫管理員解鎖帳戶。"
            btnLogin.Enabled = False
        End If
    End If
End Sub


---

補充

1. 可以設定冷卻時間

例如使用 Session("LockTime") = DateTime.Now.AddMinutes(5) 來記錄鎖定時間，然後在登入時檢查時間是否已過。



2. 可以將錯誤次數存入資料庫

如果想要長期記錄錯誤次數，可以將其儲存在資料庫中，並根據用戶帳號來管理錯誤次數。




這樣，當使用者連續登入錯誤 4 次時，系統會顯示提醒，並可以選擇鎖定按鈕或執行其他處理。

