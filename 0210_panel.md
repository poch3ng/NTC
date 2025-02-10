是的，你可以直接 複製已經設計好的 Panel，並在 後端（VB.NET）動態產生新的 Panel，然後將它們加入 PlaceHolder 或其他容器內，無需手動重新建立所有 Label 和 TextBox。


---

方法 1：使用 Clone 複製 Panel

在 ASP.NET WebForms (aspx) 內，Panel 本身是 伺服器控制項，但 不能直接用 Clone() 複製。你需要 透過 LoadControl 動態載入 .ascx 控制項，或者 手動建立新的 Panel 並複製內部控制項。

範例

Protected Sub DuplicatePanel()
    ' 取得現有的 Panel
    Dim originalPanel As Panel = Panel1

    ' 建立新的 Panel 並複製屬性
    Dim newPanel As New Panel()
    newPanel.ID = "PanelCopy_" & DateTime.Now.Ticks
    newPanel.CssClass = originalPanel.CssClass

    ' 複製 Panel 內的所有控制項
    For Each ctrl As Control In originalPanel.Controls
        Dim newCtrl As Control = CloneControl(ctrl)
        If newCtrl IsNot Nothing Then
            newPanel.Controls.Add(newCtrl)
        End If
    Next

    ' 加入新 Panel 到 PlaceHolder 或其他容器
    PlaceHolder1.Controls.Add(newPanel)
End Sub

Private Function CloneControl(ctrl As Control) As Control
    If TypeOf ctrl Is TextBox Then
        Dim originalTextBox As TextBox = DirectCast(ctrl, TextBox)
        Dim newTextBox As New TextBox()
        newTextBox.ID = "txtCopy_" & DateTime.Now.Ticks
        newTextBox.Text = originalTextBox.Text
        newTextBox.CssClass = originalTextBox.CssClass
        Return newTextBox
    ElseIf TypeOf ctrl Is Label Then
        Dim originalLabel As Label = DirectCast(ctrl, Label)
        Dim newLabel As New Label()
        newLabel.ID = "lblCopy_" & DateTime.Now.Ticks
        newLabel.Text = originalLabel.Text
        newLabel.CssClass = originalLabel.CssClass
        Return newLabel
    ElseIf TypeOf ctrl Is DropDownList Then
        Dim originalDropDown As DropDownList = DirectCast(ctrl, DropDownList)
        Dim newDropDown As New DropDownList()
        newDropDown.ID = "ddlCopy_" & DateTime.Now.Ticks
        newDropDown.CssClass = originalDropDown.CssClass

        ' 複製選項
        For Each item As ListItem In originalDropDown.Items
            newDropDown.Items.Add(New ListItem(item.Text, item.Value))
        Next

        Return newDropDown
    End If

    Return Nothing ' 其他類型的控制項可視需求添加
End Function

說明

1. DuplicatePanel()：

取得已存在的 Panel

建立新 Panel，並設定相同的 CssClass

透過 CloneControl(ctrl) 複製內部所有控制項

新 Panel 加入 PlaceHolder1（或其他 div 容器）



2. CloneControl(ctrl)：

根據 TextBox、Label、DropDownList 等類型 複製控制項

確保 ID 唯一，避免 ID 重複錯誤





---

方法 2：用 .ascx UserControl 動態載入

如果你的 Panel 結構複雜（例如包含 多個輸入欄位、按鈕、JS 事件），可以把它 存成 UserControl (.ascx)，然後 動態載入多個 Panel。

步驟

1. 建立 UserControl.ascx（例如 MyPanel.ascx）



<asp:Panel ID="Panel1" runat="server" CssClass="panel">
    <asp:Label ID="lblTitle" runat="server" Text="標題"></asp:Label>
    <asp:TextBox ID="txtInput" runat="server"></asp:TextBox>
</asp:Panel>

2. 在 aspx.vb 內動態載入



Protected Sub AddNewPanel()
    Dim newPanel As Control = LoadControl("~/MyPanel.ascx")
    newPanel.ID = "MyPanel_" & DateTime.Now.Ticks
    PlaceHolder1.Controls.Add(newPanel)
End Sub

這樣，每次呼叫 AddNewPanel()，就會 動態新增一個相同的 Panel。


---

儲存複製的 Panel 資料到資料庫

不論用 方法 1 或 方法 2，最終你會有多個 Panel，每個 Panel 內有 不同的 TextBox。你可以用 迴圈遍歷所有 Panel，然後 儲存到 SQL：

Protected Sub SaveAllData()
    Dim connectionString As String = "Server=your_server;Database=your_db;User Id=your_user;Password=your_password;"
    Dim query As String = "INSERT INTO your_table (Column1, Column2) VALUES (@Column1, @Column2)"

    Using conn As New SqlConnection(connectionString)
        conn.Open()

        For Each panel As Control In PlaceHolder1.Controls
            If TypeOf panel Is Panel Then
                Dim txt1 As TextBox = DirectCast(panel.FindControl("txtInput"), TextBox)
                Dim txt2 As TextBox = DirectCast(panel.FindControl("txtOther"), TextBox)

                If txt1 IsNot Nothing AndAlso txt2 IsNot Nothing Then
                    Using cmd As New SqlCommand(query, conn)
                        cmd.Parameters.AddWithValue("@Column1", txt1.Text)
                        cmd.Parameters.AddWithValue("@Column2", txt2.Text)
                        cmd.ExecuteNonQuery()
                    End Using
                End If
            End If
        Next
    End Using

    MessageBox.Show("所有資料已儲存！")
End Sub


---

總結

1. 如果你的 Panel 結構簡單 → 方法 1 (CloneControl())


2. 如果你的 Panel 很複雜，有 JS 或事件 → 方法 2 (UserControl.ascx)


3. 遍歷 Panel 並儲存資料 → SaveAllData()



這樣，你就能 複製現有 Panel 並讓使用者填寫，最後存回 SQL！

