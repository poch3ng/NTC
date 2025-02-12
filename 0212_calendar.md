✅ 目標

當 複製 PanelTemplate 時，確保 CalendarExtender（ccl: CalendarExtender）也成功複製 到新的 Panel。


---

🔹 問題分析

1. ASP.NET AJAX 控制項（如 CalendarExtender）綁定方式與一般控制項不同


2. CloneControl() 無法直接複製 CalendarExtender


3. **CalendarExtender 需要重新 動態建立 並 綁定到對應的 TextBox


4. PostBack 之後仍能保持 CalendarExtender 的功能




---

✅ 1️⃣ ClonePanel() - 重新建立 CalendarExtender

' ✅ 產生 Panel 並加入對應的 TextBox 與 CalendarExtender
Private Function ClonePanel(original As Panel, newID As String) As Panel
    Dim newPanel As New Panel()
    newPanel.ID = newID
    newPanel.CssClass = original.CssClass
    newPanel.BorderStyle = original.BorderStyle
    newPanel.BorderWidth = original.BorderWidth
    newPanel.Width = original.Width
    newPanel.Visible = False ' ✅ 預設隱藏

    ' ✅ 產生 TextBox 以存放日期
    Dim txtDate As New TextBox()
    txtDate.ID = "txt_Date_" & newID
    txtDate.Width = 150
    txtDate.CssClass = "form-control"

    ' ✅ 重新建立 CalendarExtender，並綁定到 TextBox
    Dim calExtender As New AjaxControlToolkit.CalendarExtender()
    calExtender.ID = "calExtender_" & newID
    calExtender.TargetControlID = txtDate.ID

    ' ✅ 新增 Label 和 TextBox
    newPanel.Controls.Add(New LiteralControl("<br/>選擇日期: "))
    newPanel.Controls.Add(txtDate)
    newPanel.Controls.Add(calExtender)
    newPanel.Controls.Add(New LiteralControl("<br/>"))

    ' ✅ 確保動態建立的 CalendarExtender 仍有效
    ScriptManager.GetCurrent(Page).RegisterExtenderControl(calExtender)

    ' ✅ 新增對應的 CheckBox
    Dim newCheckBox As New CheckBox()
    newCheckBox.ID = "chk_" & newID
    newCheckBox.Text = "填入數據 " & newID
    newCheckBox.AutoPostBack = True
    AddHandler newCheckBox.CheckedChanged, AddressOf CheckBox_Changed

    ' ✅ 記錄 CheckBox 與 Panel 的關聯
    ViewState(newCheckBox.ID) = newPanel.ID

    ' ✅ 在 PlaceHolder 加入 CheckBox 與 Panel
    PlaceHolder1.Controls.Add(newCheckBox)
    PlaceHolder1.Controls.Add(New LiteralControl("<br/>"))
    PlaceHolder1.Controls.Add(newPanel)

    Return newPanel
End Function


---

✅ 2️⃣ Page_Load() - 確保 ScriptManager 存在

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        PanelTemplate.Visible = False

        ' ✅ 確保 ScriptManager 存在
        If ScriptManager.GetCurrent(Page) Is Nothing Then
            Dim scriptManager As New ScriptManager()
            scriptManager.ID = "ScriptManager1"
            scriptManager.EnablePartialRendering = True
            Page.Form.Controls.AddAt(0, scriptManager)
        End If

        ' ✅ 初始化可刪除的 Panel 列表
        ViewState("DeletablePanels") = New List(Of String)

        ' ✅ 從 `dtsDetailItem` 取得 `po1`
        LoadPo1FromDataSet()

        ' ✅ 取得 `po1` 數量
        Dim po1Count As Integer = CType(ViewState("PanelCount"), Integer)

        ' ✅ 依照 `po1` 生成 `Panel`
        If po1Count > 0 Then
            GeneratePanels(po1Count) ' ✅ 這些 Panel 不可刪除
        End If
    Else
        ReloadPanels() ' ✅ PostBack 之後重新載入 `Panel` 和 `CheckBox`
    End If
End Sub


---

✅ 3️⃣ 測試方式

1. 點擊「複製」按鈕，允許新增 Panel


2. 確保 CalendarExtender 出現在新 Panel 的 TextBox 上


3. 點擊 TextBox，確保 CalendarExtender 顯示日期選擇器


4. PostBack 之後仍能正常運作




---

✅ 結論

✅ 允許 itemNo 拆分成多筆
✅ 動態 Panel 內的 TextBox 仍能綁定 CalendarExtender
✅ 確保 CalendarExtender 可運作，即使 PostBack 之後
✅ ScriptManager 仍然有效，確保 AJAX 控制項正常運行 🚀

