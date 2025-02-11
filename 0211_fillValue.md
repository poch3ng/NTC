✅ 目標

當 勾選 CheckBox 時，對應的 Panel 內的 TextBox 自動填入 po1 資料，且這些 po1 資料直接來自 已綁定 GridView 的 DataSet (dtsDetailItem)，而不是 GridView.Rows。


---

🔹 解決方案

1. 從 dtsDetailItem 取得 po1 資料

GridView.DataSource 來自 dtsDetailItem

直接從 ViewState("dtsDetailItem") 取得 DataTable

無需遍歷 GridView.Rows，提高效能



2. 當 CheckBox 勾選時，填入對應 po1 值

綁定 CheckBox.CheckedChanged 事件

透過 ViewState("dtsDetailItem") 取得對應 DataRow

透過 RecursiveFindControl() 找到 Panel 內的 TextBox

填入 TextBox 值



3. PostBack 之後保持 TextBox 內容

ViewState 保持 dtsDetailItem

ReloadPanels() 讓 Panel 狀態不變





---

✅ 1️⃣ 儲存 dtsDetailItem 至 ViewState

' ✅ 儲存 dtsDetailItem 到 ViewState
Private Sub LoadPo1FromDataSet()
    If Session("dtsDetailItem") IsNot Nothing Then
        Dim dtsDetailItem As DataSet = CType(Session("dtsDetailItem"), DataSet)

        ' ✅ 確保 DataSet 內有資料表
        If dtsDetailItem.Tables.Count > 0 Then
            ViewState("dtsDetailItem") = dtsDetailItem.Tables(0) ' 只取第一個 DataTable
            ViewState("PanelCount") = dtsDetailItem.Tables(0).Rows.Count
        End If
    End If
End Sub


---

✅ 2️⃣ CheckBox_Changed() - 勾選時填入對應 po1 值

' ✅ 當 CheckBox 勾選時，自動填入 Panel 內的 TextBox
Protected Sub CheckBox_Changed(sender As Object, e As EventArgs)
    Dim chk As CheckBox = CType(sender, CheckBox)
    If ViewState(chk.ID) IsNot Nothing Then
        Dim panelID As String = ViewState(chk.ID).ToString()

        ' ✅ 找到對應的 Panel
        Dim targetPanel As Panel = CType(RecursiveFindControl(Me, panelID), Panel)
        If targetPanel IsNot Nothing Then
            ' ✅ 取得 `dtsDetailItem` DataTable
            Dim dtPo1 As DataTable = CType(ViewState("dtsDetailItem"), DataTable)
            Dim panelIndex As Integer = CInt(panelID.Replace("Panel_", "")) - 1 ' 取得 Panel 的索引

            ' ✅ 取得對應的 DataRow
            If panelIndex < dtPo1.Rows.Count Then
                Dim po1Row As DataRow = dtPo1.Rows(panelIndex)

                ' ✅ 找到 Panel 內的 TextBox 並填入對應的 po1 值
                Dim txtID As TextBox = CType(RecursiveFindControl(targetPanel, "txt_ID_" & panelID), TextBox)
                Dim txtPrice As TextBox = CType(RecursiveFindControl(targetPanel, "txt_Price_" & panelID), TextBox)
                Dim txtQuantity As TextBox = CType(RecursiveFindControl(targetPanel, "txt_Quantity_" & panelID), TextBox)

                If chk.Checked Then
                    ' ✅ 填入對應 po1 值
                    If txtID IsNot Nothing Then txtID.Text = po1Row("ID").ToString()
                    If txtPrice IsNot Nothing Then txtPrice.Text = po1Row("Price").ToString()
                    If txtQuantity IsNot Nothing Then txtQuantity.Text = po1Row("Quantity").ToString()
                Else
                    ' ✅ 取消勾選時清空 TextBox
                    If txtID IsNot Nothing Then txtID.Text = ""
                    If txtPrice IsNot Nothing Then txtPrice.Text = ""
                    If txtQuantity IsNot Nothing Then txtQuantity.Text = ""
                End If
            End If
        End If
    End If
End Sub


---

✅ 3️⃣ Page_Load() - 確保 dtsDetailItem 正確載入

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        PanelTemplate.Visible = False

        ' ✅ 從 `dtsDetailItem` 取得 `po1`
        LoadPo1FromDataSet()

        ' ✅ 取得 `po1` 數量
        Dim po1Count As Integer = CType(ViewState("PanelCount"), Integer)

        ' ✅ 依照 `po1` 生成 `Panel`
        If po1Count > 0 Then
            GeneratePanels(po1Count)
        End If
    Else
        ReloadPanels() ' ✅ PostBack 之後重新載入 `Panel` 和 `CheckBox`
    End If
End Sub


---

✅ 4️⃣ ClonePanel() - 產生 Panel 並新增 TextBox

' ✅ 產生 Panel 並加入對應的 TextBox
Private Function ClonePanel(original As Panel, newID As String) As Panel
    Dim newPanel As New Panel()
    newPanel.ID = newID
    newPanel.CssClass = original.CssClass
    newPanel.BorderStyle = original.BorderStyle
    newPanel.BorderWidth = original.BorderWidth
    newPanel.Width = original.Width
    newPanel.Visible = False ' ✅ 預設隱藏

    ' ✅ 產生 TextBox 以存放 po1 值
    Dim txtID As New TextBox()
    txtID.ID = "txt_ID_" & newID
    txtID.Width = 150
    txtID.CssClass = "form-control"

    Dim txtPrice As New TextBox()
    txtPrice.ID = "txt_Price_" & newID
    txtPrice.Width = 150
    txtPrice.CssClass = "form-control"

    Dim txtQuantity As New TextBox()
    txtQuantity.ID = "txt_Quantity_" & newID
    txtQuantity.Width = 150
    txtQuantity.CssClass = "form-control"

    ' ✅ 新增 Label 和 TextBox
    newPanel.Controls.Add(New LiteralControl("<br/>ID: "))
    newPanel.Controls.Add(txtID)
    newPanel.Controls.Add(New LiteralControl("<br/>Price: "))
    newPanel.Controls.Add(txtPrice)
    newPanel.Controls.Add(New LiteralControl("<br/>Quantity: "))
    newPanel.Controls.Add(txtQuantity)
    newPanel.Controls.Add(New LiteralControl("<br/>"))

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

✅ 測試方式

1. GridView 綁定 dtsDetailItem


2. 從 dtsDetailItem 取得 po1，自動產生 Panel


3. 當 CheckBox 勾選時，對應 Panel 內的 TextBox 會填入該 po1 的數據


4. 當 CheckBox 取消勾選時，清空 TextBox


5. PostBack 之後仍能保持狀態




---

✅ 結論

✅ po1 資料來自 dtsDetailItem (DataSet)
✅ 無需遍歷 GridView.Rows，直接從 ViewState 取得 DataTable
✅ CheckBox 控制 TextBox 內的 po1 值
✅ PostBack 之後仍能保持 CheckBox 和 TextBox 的狀態 🚀

