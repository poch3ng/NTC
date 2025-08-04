Imports System.Data.SqlClient

Partial Class YourWebForm
    Inherits System.Web.UI.Page

    Protected Sub Page_Load(sender As Object, e As EventArgs) Handles Me.Load
        ' 資料庫連線字串
        Dim connectionString As String = "Data Source=YourServer;Initial Catalog=YourDatabase;Integrated Security=True"

        ' 要查詢的條件，例如 Name
        Dim searchName As String = "John" ' 假設你想找 Name = "John" 的 ID

        ' SQL 查詢語句
        Dim query As String = "SELECT ID FROM Users WHERE Name = @Name"

        Try
            ' 建立資料庫連線
            Using conn As New SqlConnection(connectionString)
                ' 建立 SQL 命令
                Using cmd As New SqlCommand(query, conn)
                    ' 加入參數以避免 SQL Injection
                    cmd.Parameters.AddWithValue("@Name", searchName)

                    ' 開啟連線
                    conn.Open()

                    ' 執行查詢並取得結果
                    Dim result As Object = cmd.ExecuteScalar()

                    If result IsNot Nothing Then
                        ' 取得 ID
                        Dim userId As Integer = Convert.ToInt32(result)
                        Response.Write("找到的 ID 是: " & userId)
                    Else
                        Response.Write("找不到符合條件的資料")
                    End If
                End Using
            End Using
        Catch ex As Exception
            Response.Write("錯誤: " & ex.Message)
        End Try
    End Sub
End Class