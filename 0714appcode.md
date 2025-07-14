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

