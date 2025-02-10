你想根據 po1 的數量 自動生成相同數量的 Panel，這可以透過 Request 變數或參數傳遞 po1 值，然後在後端根據這個數值 動態新增 Panel。


---

✅ 解決方案

1. 從 Request 取得 po1 的數值


2. 根據 po1 產生對應數量的 Panel


3. 確保 PostBack 後 Panel 不會消失


4. 暫存 Panel 內的資料（可選）




---

🔹 程式碼

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        ' 從 QueryString 或 Request 取得 po1 數量
        Dim po1Count As Integer = If(Request.QueryString("po1") IsNot Nothing, Convert.ToInt32(Request.QueryString("po1")), 0)

        ' 紀錄 Panel 數量
        ViewState("PanelCount") = po1Count

        ' 生成對應數量的 Panel
        GeneratePanels(po1Count)
    Else
        ' PostBack 時，重新載入 Panel
        ReloadPanels()
    End If
End Sub

' 產生 Panel
Private Sub GeneratePanels(po1Count As Integer)
    Dim panelYOffset As Integer = 10 ' Panel 間距

    For i As Integer = 1 To po1Count
        Dim newPanel As New Panel()
        newPanel.ID = "Panel_" & i
        newPanel.CssClass = "panel"
        newPanel.BorderStyle = BorderStyle.Solid
        newPanel.BorderWidth = Unit.Pixel(1)
        newPanel.Width = Unit.Percentage(100)
        newPanel.Style("margin-bottom") = "10px"

        ' 生成 Label 與 TextBox
        Dim lbl As New Label()
        lbl.Text = "Panel " & i
        lbl.ID = "lbl_" & i
        lbl.CssClass = "panel-title"
        newPanel.Controls.Add(lbl)

        Dim txt As New TextBox()
        txt.ID = "txt_" & i
        txt.CssClass = "panel-input"
        newPanel.Controls.Add(New LiteralControl("<br/>")) ' 換行
        newPanel.Controls.Add(txt)

        ' 將 Panel 加入 PlaceHolder
        PlaceHolder1.Controls.Add(newPanel)
    Next
End Sub

' PostBack 時還原 Panel
Private Sub ReloadPanels()
    Dim po1Count As Integer = If(ViewState("PanelCount") IsNot Nothing, CInt(ViewState("PanelCount")), 0)
    GeneratePanels(po1Count)
End Sub


---

🔹 說明

1. 從 URL 讀取 po1

透過 Request.QueryString("po1") 取得 po1 的數值

po1 數值代表要 生成的 Panel 數量

例如：http://yourdomain.com/page.aspx?po1=3 → 會生成 3 個 Panel



2. GeneratePanels(po1Count As Integer)

根據 po1 數值動態產生 Panel

每個 Panel 內有一個 Label 和 TextBox

Panel 間距 margin-bottom: 10px



3. 確保 PostBack 時不消失

ViewState("PanelCount") 記錄 po1 值

Page_Load() 檢查 IsPostBack，如果是 PostBack，就 重新載入 Panel





---

🔹 測試方式

1. 開啟網頁時傳入 po1
網址示例：

http://yourdomain.com/page.aspx?po1=5

結果：

會自動生成 5 個 Panel

每個 Panel 內有 Label 和 TextBox



2. 頁面重新整理後 Panel 不會消失

因為 ViewState 記錄了 po1 的數量，PostBack 會自動重建 Panel。





---

✅ 進階功能：儲存 Panel 內的輸入資料

如果你希望 暫存使用者填入的數據，可以使用 Session。

📌 修改 Page_Load()

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        Dim po1Count As Integer = If(Request.QueryString("po1") IsNot Nothing, Convert.ToInt32(Request.QueryString("po1")), 0)
        ViewState("PanelCount") = po1Count
        GeneratePanels(po1Count)
    Else
        ReloadPanels()
        RestorePanelData()
    End If
End Sub

📌 儲存使用者輸入的資料

Protected Sub SavePanelData()
    Dim panelData As New Dictionary(Of String, String)

    For Each panel As Control In PlaceHolder1.Controls
        If TypeOf panel Is Panel Then
            Dim panelId As String = panel.ID

            For Each ctrl As Control In panel.Controls
                If TypeOf ctrl Is TextBox Then
                    Dim txt As TextBox = DirectCast(ctrl, TextBox)
                    panelData(panelId & "_" & txt.ID) = txt.Text
                End If
            Next
        End If
    Next

    Session("PanelData") = panelData ' 暫存到 Session
End Sub

📌 還原輸入資料

Private Sub RestorePanelData()
    If Session("PanelData") IsNot Nothing Then
        Dim panelData As Dictionary(Of String, String) = DirectCast(Session("PanelData"), Dictionary(Of String, String))

        For Each panel As Control In PlaceHolder1.Controls
            If TypeOf panel Is Panel Then
                Dim panelId As String = panel.ID

                For Each ctrl As Control In panel.Controls
                    If TypeOf ctrl Is TextBox Then
                        Dim txt As TextBox = DirectCast(ctrl, TextBox)
                        Dim key As String = panelId & "_" & txt.ID
                        If panelData.ContainsKey(key) Then
                            txt.Text = panelData(key) ' 還原 TextBox 內容
                        End If
                    End If
                Next
            End If
        Next
    End If
End Sub


---

🔹 總結

✅ 根據 po1 數量動態產生 Panel
✅ 確保 PostBack 時 Panel 不消失
✅ 可以記住並還原 TextBox 內的數據（選擇性儲存）
✅ 未來可擴展：將 Session 內的資料存入 SQL

這樣你的 Panel 就可以根據 po1 數值 自動生成相同數量的 Panel，並確保 PostBack 不會影響它們！



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

