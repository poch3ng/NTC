✅ 目標

1. 允許使用者新增相同 itemNo 的 Panel（允許拆單）


2. 允許使用者手動刪除後來新增的 Panel


3. 原始 GridView 綁定的 Panel 不可刪除


4. 確保 CheckBox 仍能控制 TextBox 內容


5. PostBack 之後仍能保持 Panel 和 TextBox 狀態




---

🔹 解決方案

1. 在動態 Panel 內新增「刪除」按鈕

只有 後來新增的 Panel 會有刪除按鈕

按下按鈕後 移除該 Panel



2. 動態新增 Panel 並記錄是否可刪除

從 GridView 綁定的 Panel 設定為不可刪除

使用 ViewState("DeletablePanels") 記錄可刪除的 Panel



3. 在 PostBack 之後仍能正確還原 Panel

ReloadPanels() 確保後來新增的 Panel 仍然存在





---

✅ 1️⃣ ClonePanelWithModification() - 允許新增相同 itemNo

' ✅ 複製 Panel，並允許使用者修改數據與刪除
Private Function ClonePanelWithModification(originalPanel As Panel) As Panel
    ' 取得新 Panel ID
    Dim newID As String = "Panel_" & (CInt(ViewState("PanelCount")) + 1)
    ViewState("PanelCount") = CInt(ViewState("PanelCount")) + 1

    ' ✅ 產生新 Panel
    Dim newPanel As Panel = ClonePanel(originalPanel, newID)

    ' ✅ 讓 Quantity 可手動輸入
    Dim txtQuantity As TextBox = CType(RecursiveFindControl(newPanel, "txt_Quantity_" & newID), TextBox)
    If txtQuantity IsNot Nothing Then
        txtQuantity.Text = "" ' 清空數量，讓使用者自行輸入
        txtQuantity.Enabled = True
    End If

    ' ✅ 新增「刪除」按鈕
    Dim btnDelete As New Button()
    btnDelete.Text = "刪除"
    btnDelete.CommandArgument = newID
    AddHandler btnDelete.Click, AddressOf RemovePanel

    newPanel.Controls.Add(btnDelete)

    ' ✅ 記錄可刪除的 Panel
    Dim deletablePanels As List(Of String) = CType(ViewState("DeletablePanels"), List(Of String))
    deletablePanels.Add(newID)
    ViewState("DeletablePanels") = deletablePanels

    ' ✅ 新增 Panel 到 PlaceHolder
    PlaceHolder1.Controls.Add(New LiteralControl("<br/>"))
    PlaceHolder1.Controls.Add(newPanel)

    Return newPanel
End Function


---

✅ 2️⃣ RemovePanel() - 移除後來新增的 Panel

' ✅ 移除後來新增的 Panel
Protected Sub RemovePanel(sender As Object, e As EventArgs)
    Dim btn As Button = CType(sender, Button)
    Dim panelID As String = btn.CommandArgument ' 取得要刪除的 Panel ID

    ' ✅ 確保 Panel 在可刪除列表內
    Dim deletablePanels As List(Of String) = CType(ViewState("DeletablePanels"), List(Of String))
    If deletablePanels.Contains(panelID) Then
        ' ✅ 從 ViewState 移除該 Panel
        deletablePanels.Remove(panelID)
        ViewState("DeletablePanels") = deletablePanels

        ' ✅ 移除 Panel
        Dim panelToRemove As Panel = CType(RecursiveFindControl(Me, panelID), Panel)
        If panelToRemove IsNot Nothing Then
            PlaceHolder1.Controls.Remove(panelToRemove)
        End If
    End If
End Sub


---

✅ 3️⃣ AddDuplicatePanel() - 允許新增相同 itemNo

' ✅ 點擊「複製」按鈕時，新增相同的 `itemNo`
Protected Sub AddDuplicatePanel(sender As Object, e As EventArgs)
    Dim btn As Button = CType(sender, Button)
    Dim panelID As String = btn.CommandArgument ' 取得對應 Panel ID

    ' ✅ 取得原始 Panel
    Dim originalPanel As Panel = CType(RecursiveFindControl(Me, panelID), Panel)
    If originalPanel IsNot Nothing Then
        ClonePanelWithModification(originalPanel) ' ✅ 新增相同 `itemNo` 的 Panel
    End If
End Sub


---

✅ 4️⃣ Page_Load() - 確保 Panel 仍然存在

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        PanelTemplate.Visible = False

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

✅ 5️⃣ ReloadPanels() - 重新載入可刪除的 Panel

' ✅ 重新載入可刪除的 `Panel`
Private Sub ReloadPanels()
    ' ✅ 重新載入原始 Panel
    Dim po1Count As Integer = CType(ViewState("PanelCount"), Integer)
    For i As Integer = 1 To po1Count
        Dim newPanel As Panel = ClonePanel(PanelTemplate, "Panel_" & i)
        PlaceHolder1.Controls.Add(newPanel)
    Next

    ' ✅ 重新載入可刪除的 Panel
    Dim deletablePanels As List(Of String) = CType(ViewState("DeletablePanels"), List(Of String))
    For Each panelID In deletablePanels
        Dim newPanel As Panel = ClonePanelWithModification(PanelTemplate)
        PlaceHolder1.Controls.Add(newPanel)
    Next
End Sub


---

✅ 測試方式

1. 點擊「複製」按鈕，允許新增相同 itemNo


2. 允許修改 Quantity


3. 確保 CheckBox 仍能控制 TextBox 的填入


4. 允許刪除後來新增的 Panel，但原始 Panel 不可刪除


5. PostBack 之後仍能保持 Panel 和 TextBox 狀態




---

✅ 結論

✅ 允許 itemNo 拆分成多筆
✅ 允許 Quantity 可自行輸入
✅ CheckBox 仍可控制 TextBox
✅ 可刪除後來新增的 Panel，但原始 Panel 不可刪除
✅ PostBack 之後仍保持狀態 🚀

