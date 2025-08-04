以下是如何在 VB.NET 的 WebForm 中建立一個函式，用來查詢資料庫並回傳特定資料的 ID。如果查不到資料，函式會回傳一個適當的值（例如 -1 或 0）。我會提供兩種實現方式：使用 ADO.NET 和 Entity Framework。

### 1. 使用 ADO.NET 的範例
以下是一個函式，根據指定的條件（例如 `Name`）查詢資料庫並回傳 `ID`：

```vb
Imports System.Data.SqlClient

Partial Class YourWebForm
    Inherits System.Web.UI.Page

    ' 函式：根據 Name 查詢 ID
    Private Function GetIdByName(searchName As String) As Integer
        ' 資料庫連線字串
        Dim connectionString As String = "Data Source=YourServer;Initial Catalog=YourDatabase;Integrated Security=True"
        Dim query As String = "SELECT ID FROM Users WHERE Name = @Name"
        Dim userId As Integer = -1 ' 若查不到，回傳 -1

        Try
            Using conn As New SqlConnection(connectionString)
                Using cmd As New SqlCommand(query, conn)
                    ' 加入參數以避免 SQL Injection
                    cmd.Parameters.AddWithValue("@Name", searchName)
                    conn.Open()

                    ' 執行查詢並取得結果
                    Dim result As Object = cmd.ExecuteScalar()
                    If result IsNot Nothing Then
                        userId = Convert.ToInt32(result)
                    End If
                End Using
            End Using
        Catch ex As Exception
            ' 可選擇記錄錯誤日誌
            System.Diagnostics.Debug.WriteLine("查詢錯誤: " & ex.Message)
        End Try

        Return userId
    End Function

    Protected Sub Page_Load(sender As Object, e As EventArgs) Handles Me.Load
        Dim searchName As String = "John"
        Dim id As Integer = GetIdByName(searchName)

        If id > 0 Then
            Response.Write("找到的 ID 是: " & id)
        Else
            Response.Write("查無符合條件的資料，沒關係！")
        End If
    End Sub
End Class
```

#### 說明
- **函式定義**：`GetIdByName` 接受一個 `searchName` 參數，回傳 `Integer` 類型的 `ID`。
- **回傳值**：
  - 如果查到資料，回傳對應的 `ID`。
  - 如果查不到資料，回傳 -1（你可以根據需求改為其他值，例如 0）。
- **安全性**：使用參數化查詢（`@Name`）防止 SQL Injection。
- **錯誤處理**：使用 `Try-Catch` 捕捉可能的錯誤，確保函式不會因為資料庫問題而崩潰。

### 2. 使用 Entity Framework 的範例
如果你的專案使用 Entity Framework，函式可以更簡潔：

```vb
Imports YourProjectName.Models ' 假設這是你的 EF 模型命名空間

Partial Class YourWebForm
    Inherits System.Web.UI.Page

    ' 函式：根據 Name 查詢 ID
    Private Function GetIdByName(searchName As String) As Integer
        Dim userId As Integer = -1 ' 若查不到，回傳 -1

        Try
            Using db As New YourDbContext() ' 你的 EF DbContext
                Dim user = db.Users.FirstOrDefault(Function(u) u.Name = searchName)
                If user IsNot Nothing Then
                    userId = user.ID
                End If
            End Using
        Catch ex As Exception
            ' 可選擇記錄錯誤日誌
            System.Diagnostics.Debug.WriteLine("查詢錯誤: " & ex.Message)
        End Try

        Return userId
    End Function

    Protected Sub Page_Load(sender As Object, e As EventArgs) Handles Me.Load
        Dim searchName As String = "John"
        Dim id As Integer = GetIdByName(searchName)

        If id > 0 Then
            Response.Write("找到的 ID 是: " & id)
        Else
            Response.Write("查無符合條件的資料，沒關係！")
        End If
    End Sub
End Class
```

#### 說明
- **函式定義**：同樣接受 `searchName` 參數，回傳 `ID`。
- **查詢方式**：使用 LINQ 的 `FirstOrDefault` 方法查詢資料。
- **回傳值**：查到資料回傳 `ID`，查不到回傳 -1。
- **錯誤處理**：捕捉可能的資料庫錯誤，確保函式穩定性。

### 使用方式
你可以在任何需要的地方呼叫 `GetIdByName` 函式，例如：
- 在按鈕點擊事件中：
  ```vb
  Protected Sub btnSearch_Click(sender As Object, e As EventArgs) Handles btnSearch.Click
      Dim searchName As String = txtSearch.Text.Trim()
      Dim id As Integer = GetIdByName(searchName)
      lblResult.Text = If(id > 0, "找到的 ID 是: " & id, "查無符合條件的資料，沒關係！")
  End Sub
  ```

### 注意事項
1. **連線字串**：
   - 將連線字串儲存在 `Web.config` 中，並使用以下方式讀取：
     ```vb
     Dim connectionString As String = ConfigurationManager.ConnectionStrings("YourConnectionStringName").ConnectionString
     ```

2. **輸入驗證**：
   - 如果 `searchName` 來自使用者輸入，建議驗證：
     ```vb
     If String.IsNullOrEmpty(searchName) Then
         Return -1 ' 或其他預設值
     End If
     ```

3. **回傳值**：
   - 你可以根據需求調整回傳值，例如查不到資料時回傳 0 或 `Nothing`（改為 `Nullable(Of Integer)`）：
     ```vb
     Private Function GetIdByName(searchName As String) As Integer?
         ' ... 查詢邏輯 ...
         Return If(user IsNot Nothing, user.ID, Nothing)
     End Function
     ```

4. **日誌記錄**：
   - 若查無資料或發生錯誤，可記錄日誌以便除錯：
     ```vb
     System.Diagnostics.Debug.WriteLine("查詢 Name = " & searchName & " 無結果")
     ```

如果你需要針對特定資料表、條件，或其他資料庫（例如 MySQL、Oracle）進行調整，請提供更多細節，我可以幫你客製化程式碼！