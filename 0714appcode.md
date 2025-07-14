要將 GridView 的更新操作記錄到日誌（log）中，您可以在使用 `SqlDataSource` 的 `UpdateCommand` 時，利用 `SqlDataSource` 的 `Updated` 事件來捕捉更新操作，並將相關資訊寫入日誌。日誌可以儲存在資料庫、文字檔或其他儲存媒介中。以下以將日誌儲存到資料庫為例，逐步說明如何實現。

### 假設情境
- 您使用前述的 `SqlDataSource` 和 GridView 設定，包含 DropDownList 用於更新 `CategoryID`。
- 您希望每次更新操作完成後，將操作記錄到一個日誌表（例如 `UpdateLog`），記錄內容包括：
  - 更新時間
  - 操作者（例如當前使用者）
  - 更新前的 `CategoryID`
  - 更新後的 `CategoryID`
  - 更新的記錄 ID
- 日誌表結構假設如下：
  ```sql
  CREATE TABLE UpdateLog (
      LogID INT IDENTITY(1,1) PRIMARY KEY,
      RecordID INT, -- 被更新的記錄 ID
      OldCategoryID INT, -- 更新前的 CategoryID
      NewCategoryID INT, -- 更新後的 CategoryID
      UserName NVARCHAR(50), -- 操作者
      UpdateTime DATETIME -- 更新時間
  )
  ```

以下是具體步驟：

---

### 步驟 1：修改 SqlDataSource 的 Updated 事件
`SqlDataSource` 的 `Updated` 事件會在更新操作完成後觸發，您可以在此事件中記錄日誌。

1. 在 `.aspx` 中，為 `SqlDataSource` 新增 `OnUpdated` 事件：
   ```aspx
   <asp:SqlDataSource ID="SqlDataSource1" runat="server"
       ConnectionString="<%$ ConnectionStrings:YourConnectionString %>"
       SelectCommand="SELECT ID, CategoryID, CategoryName FROM YourTable JOIN Categories ON YourTable.CategoryID = Categories.CategoryID"
       UpdateCommand="UPDATE YourTable SET CategoryID = @CategoryID WHERE ID = @ID"
       OnUpdated="SqlDataSource1_Updated">
       <UpdateParameters>
           <asp:Parameter Name="CategoryID" Type="Int32" />
           <asp:Parameter Name="ID" Type="Int32" />
       </UpdateParameters>
   </asp:SqlDataSource>
   ```

2. 在 `.aspx.vb` 中，實現 `SqlDataSource1_Updated` 事件，並記錄日誌：
   ```vb
   Protected Sub SqlDataSource1_Updated(sender As Object, e As SqlDataSourceStatusEventArgs)
       If e.Exception Is Nothing AndAlso e.AffectedRows > 0 Then
           ' 獲取更新參數
           Dim newCategoryID As Integer = Convert.ToInt32(e.Command.Parameters("@CategoryID").Value)
           Dim recordID As Integer = Convert.ToInt32(e.Command.Parameters("@ID").Value)

           ' 獲取更新前的 CategoryID
           Dim oldCategoryID As Integer = GetOldCategoryID(recordID)

           ' 獲取當前使用者（假設使用 Windows 身份驗證或 Session）
           Dim userName As String = HttpContext.Current.User.Identity.Name
           If String.IsNullOrEmpty(userName) Then userName = "Unknown"

           ' 記錄日誌到資料庫
           LogUpdate(recordID, oldCategoryID, newCategoryID, userName)

           ' 顯示成功訊息（可選）
           lblError.Text = "更新成功，已記錄日誌"
       ElseIf e.Exception IsNot Nothing Then
           ' 處理錯誤
           lblError.Text = "更新失敗: " & e.Exception.Message
           e.ExceptionHandled = True
       End If
   End Sub
   ```

---

### 步驟 2：實現獲取更新前值的函數
由於 `SqlDataSource` 的 `Updated` 事件中無法直接獲取更新前的 `CategoryID`，您需要查詢資料庫來取得更新前的值。

```vb
Private Function GetOldCategoryID(recordID As Integer) As Integer
    Dim connectionString As String = ConfigurationManager.ConnectionStrings("YourConnectionString").ConnectionString
    Using conn As New SqlConnection(connectionString)
        Dim query As String = "SELECT CategoryID FROM YourTable WHERE ID = @ID"
        Dim cmd As New SqlCommand(query, conn)
        cmd.Parameters.AddWithValue("@ID", recordID)
        conn.Open()
        Dim result As Object = cmd.ExecuteScalar()
        conn.Close()
        Return If(result IsNot Nothing, Convert.ToInt32(result), 0)
    End Using
End Function
```

---

### 步驟 3：實現日誌記錄函數
撰寫一個函數將更新資訊寫入 `UpdateLog` 表：

```vb
Private Sub LogUpdate(recordID As Integer, oldCategoryID As Integer, newCategoryID As Integer, userName As String)
    Dim connectionString As String = ConfigurationManager.ConnectionStrings("YourConnectionString").ConnectionString
    Using conn As New SqlConnection(connectionString)
        Dim query As String = "INSERT INTO UpdateLog (RecordID, OldCategoryID, NewCategoryID, UserName, UpdateTime) " & _
                              "VALUES (@RecordID, @OldCategoryID, @NewCategoryID, @UserName, @UpdateTime)"
        Dim cmd As New SqlCommand(query, conn)
        cmd.Parameters.AddWithValue("@RecordID", recordID)
        cmd.Parameters.AddWithValue("@OldCategoryID", oldCategoryID)
        cmd.Parameters.AddWithValue("@NewCategoryID", newCategoryID)
        cmd.Parameters.AddWithValue("@UserName", userName)
        cmd.Parameters.AddWithValue("@UpdateTime", DateTime.Now)
        conn.Open()
        cmd.ExecuteNonQuery()
        conn.Close()
    End Using
End Sub
```

---

### 步驟 4：確保 Web.config 配置正確
確保 `Web.config` 中已定義連線字串：
```xml
<connectionStrings>
    <add name="YourConnectionString" connectionString="Server=您的伺服器;Database=您的資料庫;Trusted_Connection=True;" providerName="System.Data.SqlClient" />
</connectionStrings>
```

---

### 步驟 5：完整的 .aspx 和 .aspx.vb 程式碼
#### .aspx
```aspx
<asp:SqlDataSource ID="SqlDataSource1" runat="server"
    ConnectionString="<%$ ConnectionStrings:YourConnectionString %>"
    SelectCommand="SELECT ID, CategoryID, CategoryName FROM YourTable JOIN Categories ON YourTable.CategoryID = Categories.CategoryID"
    UpdateCommand="UPDATE YourTable SET CategoryID = @CategoryID WHERE ID = @ID"
    OnUpdated="SqlDataSource1_Updated">
    <UpdateParameters>
        <asp:Parameter Name="CategoryID" Type="Int32" />
        <asp:Parameter Name="ID" Type="Int32" />
    </UpdateParameters>
</asp:SqlDataSource>

<asp:SqlDataSource ID="SqlDataSourceCategories" runat="server"
    ConnectionString="<%$ ConnectionStrings:YourConnectionString %>"
    SelectCommand="SELECT CategoryID, CategoryName FROM Categories">
</asp:SqlDataSource>

<asp:GridView ID="GridView1" runat="server" 
    AutoGenerateColumns="False" 
    DataSourceID="SqlDataSource1" 
    DataKeyNames="ID"
    AutoGenerateEditButton="True">
    <Columns>
        <asp:BoundField DataField="ID" HeaderText="ID" ReadOnly="True" />
        <asp:TemplateField HeaderText="類別">
            <ItemTemplate>
                <asp:Label ID="lblCategory" runat="server" Text='<%# Eval("CategoryName") %>'></asp:Label>
            </ItemTemplate>
            <EditItemTemplate>
                <asp:DropDownList ID="ddlCategory" runat="server"
                    DataSourceID="SqlDataSourceCategories"
                    DataTextField="CategoryName"
                    DataValueField="CategoryID"
                    SelectedValue='<%# Bind("CategoryID") %>'>
                </asp:DropDownList>
            </EditItemTemplate>
        </asp:TemplateField>
    </Columns>
</asp:GridView>
<asp:Label ID="lblError" runat="server" ForeColor="Red"></asp:Label>
```

#### .aspx.vb
```vb
Protected Sub SqlDataSource1_Updated(sender As Object, e As SqlDataSourceStatusEventArgs)
    If e.Exception Is Nothing AndAlso e.AffectedRows > 0 Then
        ' 獲取更新參數
        Dim newCategoryID As Integer = Convert.ToInt32(e.Command.Parameters("@CategoryID").Value)
        Dim recordID As Integer = Convert.ToInt32(e.Command.Parameters("@ID").Value)

        ' 獲取更新前的 CategoryID
        Dim oldCategoryID As Integer = GetOldCategoryID(recordID)

        ' 獲取當前使用者
        Dim userName As String = HttpContext.Current.User.Identity.Name
        If String.IsNullOrEmpty(userName) Then userName = "Unknown"

        ' 記錄日誌
        LogUpdate(recordID, oldCategoryID, newCategoryID, userName)

        lblError.Text = "更新成功，已記錄日誌"
    ElseIf e.Exception IsNot Nothing Then
        lblError.Text = "更新失敗: " & e.Exception.Message
        e.ExceptionHandled = True
    End If
End Sub

Private Function GetOldCategoryID(recordID As Integer) As Integer
    Dim connectionString As String = ConfigurationManager.ConnectionStrings("YourConnectionString").ConnectionString
    Using conn As New SqlConnection(connectionString)
        Dim query As String = "SELECT CategoryID FROM YourTable WHERE ID = @ID"
        Dim cmd As New SqlCommand(query, conn)
        cmd.Parameters.AddWithValue("@ID", recordID)
        conn.Open()
        Dim result As Object = cmd.ExecuteScalar()
        conn.Close()
        Return If(result IsNot Nothing, Convert.ToInt32(result), 0)
    End Using
End Function

Private Sub LogUpdate(recordID As Integer, oldCategoryID As Integer, newCategoryID As Integer, userName As String)
    Dim connectionString As String = ConfigurationManager.ConnectionStrings("YourConnectionString").ConnectionString
    Using conn As New SqlConnection(connectionString)
        Dim query As String = "INSERT INTO UpdateLog (RecordID, OldCategoryID, NewCategoryID, UserName, UpdateTime) " & _
                              "VALUES (@RecordID, @OldCategoryID, @NewCategoryID, @UserName, @UpdateTime)"
        Dim cmd As New SqlCommand(query, conn)
        cmd.Parameters.AddWithValue("@RecordID", recordID)
        cmd.Parameters.AddWithValue("@OldCategoryID", oldCategoryID)
        cmd.Parameters.AddWithValue("@NewCategoryID", newCategoryID)
        cmd.Parameters.AddWithValue("@UserName", userName)
        cmd.Parameters.AddWithValue("@UpdateTime", DateTime.Now)
        conn.Open()
        cmd.ExecuteNonQuery()
        conn.Close()
    End Using
End Sub
```

---

### 替代方案：將日誌寫入檔案
如果您不想將日誌儲存在資料庫，而是寫入文字檔案，可以修改 `LogUpdate` 函數如下：

```vb
Private Sub LogUpdate(recordID As Integer, oldCategoryID As Integer, newCategoryID As Integer, userName As String)
    Dim logMessage As String = String.Format("[{0}] User: {1}, RecordID: {2}, OldCategoryID: {3}, NewCategoryID: {4}{5}",
                                            DateTime.Now.ToString("yyyy-MM-dd HH:mm:ss"),
                                            userName,
                                            recordID,
                                            oldCategoryID,
                                            newCategoryID,
                                            Environment.NewLine)
    Dim logFilePath As String = Server.MapPath("~/App_Data/UpdateLog.txt")
    File.AppendAllText(logFilePath, logMessage)
End Sub
```
- 確保 `~/App_Data/` 資料夾存在且應用程式有寫入權限。
- 日誌將以純文字格式儲存在 `UpdateLog.txt` 中。

---

### 注意事項
1. **更新前的值**：因為 `SqlDataSource` 的 `UpdateCommand` 不會自動提供更新前的值，您需要額外查詢資料庫（如 `GetOldCategoryID`）。如果您的資料表很大，考慮在更新前快取原始資料以提高效能。
2. **使用者身份**：範例中使用 `HttpContext.Current.User.Identity.Name` 獲取使用者名稱。如果您的應用程式使用自訂身份驗證（例如 Forms Authentication），可能需要從 `Session` 或其他地方獲取使用者資訊。
3. **錯誤處理**：在 `LogUpdate` 和 `GetOldCategoryID` 中加入 Try-Catch 塊，以處理可能的資料庫連線錯誤：
   ```vb
   Try
       conn.Open()
       cmd.ExecuteNonQuery()
   Catch ex As Exception
       lblError.Text = "記錄日誌失敗: " & ex.Message
   Finally
       conn.Close()
   End Try
   ```
4. **日誌表結構**：根據需求，您可以擴展 `UpdateLog` 表，加入更多欄位（例如操作類型、IP 地址等）。
5. **安全性**：確保參數化查詢（`@RecordID` 等）用於所有資料庫操作，以防止 SQL 注入。

---

### 測試與驗證
1. 運行 WebForm，點擊「編輯」並選擇新的 `CategoryID`，然後點擊「更新」。
2. 檢查 `UpdateLog` 表，確認是否正確記錄了 `RecordID`、更新前後的 `CategoryID`、使用者名稱和時間。
3. 如果使用檔案日誌，檢查 `~/App_Data/UpdateLog.txt` 是否正確寫入記錄。
4. 模擬更新失敗（例如斷開資料庫連線），確認錯誤訊息是否正確顯示。

---

### 總結
- 使用 `SqlDataSource` 的 `Updated` 事件來捕捉更新操作，並記錄日誌。
- 透過查詢資料庫獲取更新前的 `CategoryID`，並將日誌寫入資料庫或檔案。
- 確保錯誤處理和安全性措施完善。

如果您有其他需求（例如記錄更多欄位、不同日誌格式等），請提供更多細節，我可以進一步調整程式碼！

是的，您可以直接使用 GridView 的 `UpdateCommand`（搭配資料來源控制項，例如 `SqlDataSource`）來處理更新操作，這樣可以簡化後端程式碼，無需手動撰寫 `RowUpdating` 事件中的資料庫更新邏輯。以下是逐步說明如何將您的 BoundField 改為 DropDownList，並使用 `SqlDataSource` 的 `UpdateCommand` 來處理更新。

### 假設情境
- 您有一個 GridView，使用 BoundField 顯示某欄位（例如 "Category"）。
- 您希望將其改為 DropDownList，並使用 `SqlDataSource` 的 `UpdateCommand` 處理資料更新。
- 資料表結構假設如下：
  - `YourTable`：包含欄位 `ID`（主鍵）、`CategoryID`（外鍵，指向類別表）。
  - `Categories`：包含欄位 `CategoryID` 和 `CategoryName`。

以下是具體步驟：

---

### 步驟 1：設置 SqlDataSource
1. 在您的 `.aspx` 頁面中，新增一個 `SqlDataSource` 控制項，用於 GridView 的資料繫結和更新。
   ```aspx
   <asp:SqlDataSource ID="SqlDataSource1" runat="server"
       ConnectionString="<%$ ConnectionStrings:YourConnectionString %>"
       SelectCommand="SELECT ID, CategoryID, CategoryName FROM YourTable JOIN Categories ON YourTable.CategoryID = Categories.CategoryID"
       UpdateCommand="UPDATE YourTable SET CategoryID = @CategoryID WHERE ID = @ID">
       <UpdateParameters>
           <asp:Parameter Name="CategoryID" Type="Int32" />
           <asp:Parameter Name="ID" Type="Int32" />
       </UpdateParameters>
   </asp:SqlDataSource>
   ```
   - `ConnectionString`：需在 `Web.config` 中定義您的資料庫連線字串。
   - `SelectCommand`：查詢 GridView 顯示的資料，包含 `CategoryName` 用於顯示。
   - `UpdateCommand`：定義更新 `YourTable` 的 SQL 語句，`@CategoryID` 將從 DropDownList 獲取，`@ID` 從 GridView 的 `DataKeyNames` 獲取。

2. 為 DropDownList 新增另一個 `SqlDataSource`，用於提供類別選項：
   ```aspx
   <asp:SqlDataSource ID="SqlDataSourceCategories" runat="server"
       ConnectionString="<%$ ConnectionStrings:YourConnectionString %>"
       SelectCommand="SELECT CategoryID, CategoryName FROM Categories">
   </asp:SqlDataSource>
   ```

---

### 步驟 2：將 BoundField 改為 TemplateField
1. 找到 GridView 中的 `<asp:BoundField>`，例如：
   ```aspx
   <asp:BoundField DataField="CategoryName" HeaderText="類別" />
   ```
2. 將其替換為 `<asp:TemplateField>`，並設置顯示模式和編輯模式的模板：
   ```aspx
   <asp:TemplateField HeaderText="類別">
       <ItemTemplate>
           <asp:Label ID="lblCategory" runat="server" Text='<%# Eval("CategoryName") %>'></asp:Label>
       </ItemTemplate>
       <EditItemTemplate>
           <asp:DropDownList ID="ddlCategory" runat="server"
               DataSourceID="SqlDataSourceCategories"
               DataTextField="CategoryName"
               DataValueField="CategoryID"
               SelectedValue='<%# Bind("CategoryID") %>'>
           </asp:DropDownList>
       </EditItemTemplate>
   </asp:TemplateField>
   ```
   - `ItemTemplate`：顯示模式下使用 Label 顯示 `CategoryName`。
   - `EditItemTemplate`：編輯模式下使用 DropDownList，繫結到 `SqlDataSourceCategories`。
   - `SelectedValue='<%# Bind("CategoryID") %>'`：將 DropDownList 的選擇值與資料來源的 `CategoryID` 雙向繫結，確保編輯時顯示當前值，並在更新時傳遞選擇值。

---

### 步驟 3：配置 GridView
確保 GridView 連接到 `SqlDataSource1`，並設置 `DataKeyNames` 為主鍵：
```aspx
<asp:GridView ID="GridView1" runat="server" 
    AutoGenerateColumns="False" 
    DataSourceID="SqlDataSource1" 
    DataKeyNames="ID"
    AutoGenerateEditButton="True">
    <Columns>
        <asp:BoundField DataField="ID" HeaderText="ID" ReadOnly="True" />
        <asp:TemplateField HeaderText="類別">
            <ItemTemplate>
                <asp:Label ID="lblCategory" runat="server" Text='<%# Eval("CategoryName") %>'></asp:Label>
            </ItemTemplate>
            <EditItemTemplate>
                <asp:DropDownList ID="ddlCategory" runat="server"
                    DataSourceID="SqlDataSourceCategories"
                    DataTextField="CategoryName"
                    DataValueField="CategoryID"
                    SelectedValue='<%# Bind("CategoryID") %>'>
                </asp:DropDownList>
            </EditItemTemplate>
        </asp:TemplateField>
    </Columns>
</asp:GridView>
```
- `DataSourceID="SqlDataSource1"`：將 GridView 繫結到 `SqlDataSource1`。
- `DataKeyNames="ID"`：確保 GridView 在更新時能傳遞主鍵值。
- `AutoGenerateEditButton="True"`：自動生成「編輯」和「更新/取消」按鈕。

---

### 步驟 4：Web.config 中的連線字串
確保 `Web.config` 中定義了資料庫連線字串，例如：
```xml
<connectionStrings>
    <add name="YourConnectionString" connectionString="Server=您的伺服器;Database=您的資料庫;Trusted_Connection=True;" providerName="System.Data.SqlClient" />
</connectionStrings>
```

---

### 步驟 5：測試與驗證
1. 運行 WebForm，確認 GridView 正確顯示資料。
2. 點擊「編輯」按鈕，確認 DropDownList 顯示所有類別選項，且當前行的 `CategoryID` 被正確選中。
3. 選擇一個新類別，點擊「更新」，確認資料庫中的 `CategoryID` 已更新。
4. 點擊「取消」，確認 GridView 恢復顯示模式。

---

### 為什麼使用 UpdateCommand？
- **簡化程式碼**：使用 `SqlDataSource` 的 `UpdateCommand`，無需在 `.aspx.vb` 中手動撰寫資料庫更新邏輯。
- **自動參數處理**：`<%# Bind("CategoryID") %>` 會自動將 DropDownList 的選擇值傳遞給 `UpdateCommand` 的 `@CategoryID` 參數。
- **快速開發**：適合簡單的 CRUD 操作，減少後端程式碼的維護。

---

### 注意事項
- **資料一致性**：確保 `Categories` 表格中的 `CategoryID` 與 `YourTable` 的 `CategoryID` 一致，否則可能出現選擇值無效的錯誤。
- **效能**：如果 `Categories` 表格資料量大或不常變動，可以考慮啟用 `SqlDataSource` 的快取功能（例如 `EnableCaching="True"`）。
- **錯誤處理**：您可以在 `SqlDataSource` 的 `Updated` 事件中加入錯誤處理邏輯，例如：
  ```aspx
  <asp:SqlDataSource ID="SqlDataSource1" runat="server" OnUpdated="SqlDataSource1_Updated" ...>
  ```
  ```vb
  Protected Sub SqlDataSource1_Updated(sender As Object, e As SqlDataSourceStatusEventArgs)
      If e.Exception IsNot Nothing Then
          ' 處理錯誤
          lblError.Text = "更新失敗: " & e.Exception.Message
          e.ExceptionHandled = True
      Else
          lblError.Text = "更新成功"
      End If
  End Sub
  ```
- **安全性**：使用參數化查詢（`@CategoryID`、`@ID`）可防止 SQL 注入攻擊。

---

### 完整範例程式碼
#### .aspx
```aspx
<asp:SqlDataSource ID="SqlDataSource1" runat="server"
    ConnectionString="<%$ ConnectionStrings:YourConnectionString %>"
    SelectCommand="SELECT ID, CategoryID, CategoryName FROM YourTable JOIN Categories ON YourTable.CategoryID = Categories.CategoryID"
    UpdateCommand="UPDATE YourTable SET CategoryID = @CategoryID WHERE ID = @ID">
    <UpdateParameters>
        <asp:Parameter Name="CategoryID" Type="Int32" />
        <asp:Parameter Name="ID" Type="Int32" />
    </UpdateParameters>
</asp:SqlDataSource>

<asp:SqlDataSource ID="SqlDataSourceCategories" runat="server"
    ConnectionString="<%$ ConnectionStrings:YourConnectionString %>"
    SelectCommand="SELECT CategoryID, CategoryName FROM Categories">
</asp:SqlDataSource>

<asp:GridView ID="GridView1" runat="server" 
    AutoGenerateColumns="False" 
    DataSourceID="SqlDataSource1" 
    DataKeyNames="ID"
    AutoGenerateEditButton="True">
    <Columns>
        <asp:BoundField DataField="ID" HeaderText="ID" ReadOnly="True" />
        <asp:TemplateField HeaderText="類別">
            <ItemTemplate>
                <asp:Label ID="lblCategory" runat="server" Text='<%# Eval("CategoryName") %>'></asp:Label>
            </ItemTemplate>
            <EditItemTemplate>
                <asp:DropDownList ID="ddlCategory" runat="server"
                    DataSourceID="SqlDataSourceCategories"
                    DataTextField="CategoryName"
                    DataValueField="CategoryID"
                    SelectedValue='<%# Bind("CategoryID") %>'>
                </asp:DropDownList>
            </EditItemTemplate>
        </asp:TemplateField>
    </Columns>
</asp:GridView>
<asp:Label ID="lblError" runat="server" ForeColor="Red"></asp:Label>
```

#### .aspx.vb（可選）
如果需要錯誤處理，新增以下程式碼：
```vb
Protected Sub SqlDataSource1_Updated(sender As Object, e As SqlDataSourceStatusEventArgs)
    If e.Exception IsNot Nothing Then
        lblError.Text = "更新失敗: " & e.Exception.Message
        e.ExceptionHandled = True
    Else
        lblError.Text = "更新成功"
    End If
End Sub
```

---

### 總結
- 使用 `SqlDataSource` 的 `UpdateCommand` 可以大幅簡化更新邏輯，無需手動處理 `RowUpdating` 事件。
- 將 BoundField 替換為 TemplateField，並在 `EditItemTemplate` 中使用 DropDownList，透過 `<%# Bind("CategoryID") %>` 實現雙向繫結。
- 確保 `SqlDataSource` 的參數和資料來源正確配置。

如果您有其他問題（例如多欄位更新、複雜資料結構等），請提供更多細節，我可以進一步協助！

了解，你要將 GridView 中的欄位 AppCode 改為編輯時顯示下拉選單，且內容從資料表載入。以下是 完整對應你需求的範例，請依步驟進行：


---

🧩 假設情境

GridView 資料表中有欄位：AppCode（原本是 BoundField）

下拉選單的內容來自資料表 AppTable

顯示欄位：AppName

值欄位：AppCode


連線字串名稱為：CRM



---

✅ 第 1 步：ASPX 改 BoundField 為 TemplateField

<asp:TemplateField HeaderText="應用系統">
    <ItemTemplate>
        <%# Eval("AppCode") %>
    </ItemTemplate>
    <EditItemTemplate>
        <asp:DropDownList ID="ddlAppCode" runat="server" />
    </EditItemTemplate>
</asp:TemplateField>


---

✅ 第 2 步：RowDataBound 載入下拉選單內容

Protected Sub GridView1_RowDataBound(sender As Object, e As GridViewRowEventArgs) Handles GridView1.RowDataBound
    If e.Row.RowType = DataControlRowType.DataRow AndAlso e.Row.RowState.HasFlag(DataControlRowState.Edit) Then
        Dim ddl As DropDownList = CType(e.Row.FindControl("ddlAppCode"), DropDownList)
        If ddl IsNot Nothing Then
            Dim connStr As String = ConfigurationManager.ConnectionStrings("CRM").ConnectionString
            Using conn As New SqlConnection(connStr)
                Dim cmd As New SqlCommand("SELECT AppCode, AppName FROM AppTable ORDER BY AppName", conn)
                conn.Open()
                Dim dt As New DataTable()
                dt.Load(cmd.ExecuteReader())

                ddl.DataSource = dt
                ddl.DataTextField = "AppName"
                ddl.DataValueField = "AppCode"
                ddl.DataBind()
            End Using

            ' 設定預設值為目前列的 AppCode
            Dim currentValue As String = DataBinder.Eval(e.Row.DataItem, "AppCode").ToString()
            ddl.SelectedValue = currentValue
        End If
    End If
End Sub


---

✅ 第 3 步：RowUpdating 中取出選取值

Protected Sub GridView1_RowUpdating(sender As Object, e As GridViewUpdateEventArgs)
    Dim row As GridViewRow = GridView1.Rows(e.RowIndex)
    Dim ddl As DropDownList = CType(row.FindControl("ddlAppCode"), DropDownList)
    Dim selectedAppCode As String = ddl.SelectedValue

    Dim key As String = GridView1.DataKeys(e.RowIndex).Value.ToString() ' 主鍵欄位
    Dim connStr As String = ConfigurationManager.ConnectionStrings("CRM").ConnectionString
    Using conn As New SqlConnection(connStr)
        Dim cmd As New SqlCommand("UPDATE 你的資料表 SET AppCode = @AppCode WHERE 主鍵欄位 = @ID", conn)
        cmd.Parameters.AddWithValue("@AppCode", selectedAppCode)
        cmd.Parameters.AddWithValue("@ID", key)
        conn.Open()
        cmd.ExecuteNonQuery()
    End Using

    GridView1.EditIndex = -1
    BindGrid()
End Sub


---

✅ 最後補充設定

GridView 要設定主鍵欄位：

<asp:GridView ID="GridView1" runat="server" DataKeyNames="主鍵欄位" ... >


---

如你提供：

AppTable 的實際欄位名稱

GridView 的資料來源與主鍵欄位名稱


我可以幫你完全客製成你系統在用的格式。是否需要？

