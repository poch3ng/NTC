如果你的 CustomerAccount 表目前沒有密碼修改時間 (modify_date) 的欄位，你需要新增一個 LastPasswordChange 欄位，並在每次密碼變更時更新這個欄位。


---

步驟

1. 新增 LastPasswordChange 欄位


2. 修改登入驗證，檢查密碼變更天數


3. 當用戶變更密碼時，更新 LastPasswordChange




---

1. SQL：新增 LastPasswordChange 欄位

如果 CustomerAccount 沒有記錄密碼修改時間，請先執行 SQL 新增一個日期欄位：

ALTER TABLE CustomerAccount
ADD LastPasswordChange DATETIME NULL;

並設定所有現有用戶的密碼修改時間為當前時間：

UPDATE CustomerAccount
SET LastPasswordChange = GETDATE();


---

2. VB.NET：登入時檢查密碼變更天數

修改登入檢查程式，若用戶超過 90 天沒換密碼，則強制跳轉到變更密碼頁面。

Imports System.Data.SqlClient

Protected Sub btnLogin_Click(sender As Object, e As EventArgs) Handles btnLogin.Click
    Dim username As String = txtUsername.Text
    Dim password As String = txtPassword.Text
    Dim connString As String = "Server=你的SQL伺服器;Database=你的資料庫;User Id=你的DB帳號;Password=你的DB密碼;"
    Dim daysSinceLastChange As Integer = 0

    Using conn As New SqlConnection(connString)
        conn.Open()

        ' 驗證用戶名和密碼
        Dim sqlLogin As String = "SELECT COUNT(*), ISNULL(DATEDIFF(DAY, LastPasswordChange, GETDATE()), 0) FROM CustomerAccount WHERE Username = @Username AND PasswordHash = HASHBYTES('SHA2_256', @Password);"
        Using cmdLogin As New SqlCommand(sqlLogin, conn)
            cmdLogin.Parameters.AddWithValue("@Username", username)
            cmdLogin.Parameters.AddWithValue("@Password", password)

            Using reader As SqlDataReader = cmdLogin.ExecuteReader()
                If reader.Read() Then
                    Dim count As Integer = reader.GetInt32(0)
                    daysSinceLastChange = reader.GetInt32(1)

                    If count = 0 Then
                        lblMessage.Text = "登入失敗，請檢查帳號或密碼！"
                        Exit Sub
                    End If
                End If
            End Using
        End Using
    End Using

    ' 如果超過 90 天，則要求修改密碼
    If daysSinceLastChange >= 90 Then
        Response.Redirect("ChangePassword.aspx?user=" & username)
    Else
        ' 正常登入
        Session("Username") = username
        Response.Redirect("Dashboard.aspx")
    End If
End Sub


---

3. VB.NET：用戶變更密碼時更新 LastPasswordChange

當用戶變更密碼後，將 LastPasswordChange 更新為當前時間。

Protected Sub btnChangePassword_Click(sender As Object, e As EventArgs) Handles btnChangePassword.Click
    Dim username As String = Request.QueryString("user")
    Dim newPassword As String = txtNewPassword.Text
    Dim connString As String = "Server=你的SQL伺服器;Database=你的資料庫;User Id=你的DB帳號;Password=你的DB密碼;"

    Using conn As New SqlConnection(connString)
        conn.Open()
        Dim sqlUpdate As String = "UPDATE CustomerAccount SET PasswordHash = HASHBYTES('SHA2_256', @NewPassword), LastPasswordChange = GETDATE() WHERE Username = @Username;"
        Using cmd As New SqlCommand(sqlUpdate, conn)
            cmd.Parameters.AddWithValue("@NewPassword", newPassword)
            cmd.Parameters.AddWithValue("@Username", username)
            cmd.ExecuteNonQuery()
        End Using
    End Using

    lblMessage.Text = "密碼修改成功，請重新登入！"
    Response.Redirect("Login.aspx")
End Sub


---

結果

用戶登入時會檢查 LastPasswordChange。

超過 90 天 沒改密碼 → 強制跳轉 ChangePassword.aspx 要求用戶更新密碼。

用戶修改密碼後，LastPasswordChange 更新為當前時間，下次再等 90 天才會要求更新。


這樣可以確保密碼定期更新，提升安全性！

