✅ 目標

1. 檢查 Panel 內的 itemNo 是否存在於 dtsDetailItem（DataSet）中


2. 如果 itemNo 不存在，顯示錯誤訊息


3. 確保 PostBack 之後仍能保持 Panel 和 TextBox 狀態


4. 整合 Quantity 檢核邏輯




---

🔹 解決方案

1. 從 ViewState("dtsDetailItem") 取得 DataTable


2. 遍歷 Panel 內的 itemNo


3. 比對 dtsDetailItem 是否包含該 itemNo


4. 如果 itemNo 不存在，顯示錯誤


5. 如果 itemNo 存在，再檢查 Quantity 總和




---

✅ 1️⃣ ValidateItemNumbersAndQuantities() - 驗證 itemNo 是否存在且數量正確

' ✅ 檢查 itemNo 是否存在於 dtsDetailItem，並檢查數量
Private Sub ValidateItemNumbersAndQuantities()
    Dim dtPo1 As DataTable = CType(ViewState("dtsDetailItem"), DataTable)
    Dim itemQuantitySum As New Dictionary(Of String, Integer) ' 存放 itemNo 的加總數量
    Dim existingItemNos As New HashSet(Of String) ' 存放 dtsDetailItem 內的 itemNo
    Dim errors As New List(Of String) ' 儲存錯誤訊息

    ' ✅ 取得所有 dtsDetailItem 內的 itemNo
    For Each row As DataRow In dtPo1.Rows
        existingItemNos.Add(row("ID").ToString())
    Next

    ' ✅ 遍歷所有 Panel，檢查 itemNo 是否存在於 dtsDetailItem 並累加數量
    For Each ctrl As Control In PlaceHolder1.Controls
        If TypeOf ctrl Is Panel Then
            Dim panel As Panel = CType(ctrl, Panel)
            Dim txtID As TextBox = CType(RecursiveFindControl(panel, "txt_ID_" & panel.ID), TextBox)
            Dim txtQuantity As TextBox = CType(RecursiveFindControl(panel, "txt_Quantity_" & panel.ID), TextBox)

            If txtID IsNot Nothing AndAlso txtQuantity IsNot Nothing Then
                Dim itemNo As String = txtID.Text.Trim()
                Dim quantity As Integer = 0
                Integer.TryParse(txtQuantity.Text.Trim(), quantity)

                ' ✅ 檢查 itemNo 是否存在
                If Not existingItemNos.Contains(itemNo) Then
                    errors.Add($"錯誤：ItemNo {itemNo} 不存在於原始資料內！")
                Else
                    ' ✅ 累加相同 itemNo 的數量
                    If itemQuantitySum.ContainsKey(itemNo) Then
                        itemQuantitySum(itemNo) += quantity
                    Else
                        itemQuantitySum(itemNo) = quantity
                    End If
                End If
            End If
        End If
    Next

    ' ✅ 檢查數量是否匹配
    For Each row As DataRow In dtPo1.Rows
        Dim itemNo As String = row("ID").ToString()
        Dim originalQuantity As Integer = Convert.ToInt32(row("Quantity"))

        ' ✅ 比對加總數量與原始數量
        If itemQuantitySum.ContainsKey(itemNo) AndAlso itemQuantitySum(itemNo) <> originalQuantity Then
            errors.Add($"錯誤：ItemNo {itemNo} 的數量總和 ({itemQuantitySum(itemNo)}) 與原始數量 ({originalQuantity}) 不符！")
        End If
    Next

    ' ✅ 顯示驗證結果
    If errors.Count > 0 Then
        lblValidationResult.Text = String.Join("<br/>", errors)
        lblValidationResult.ForeColor = System.Drawing.Color.Red
    Else
        lblValidationResult.Text = "所有數量驗證成功，且所有 itemNo 存在！"
        lblValidationResult.ForeColor = System.Drawing.Color.Green
    End If
End Sub


---

✅ 2️⃣ btnConfirm_Click() - 點擊確認按鈕時執行檢核

' ✅ 點擊「確認」按鈕時執行檢核
Protected Sub btnConfirm_Click(sender As Object, e As EventArgs)
    ValidateItemNumbersAndQuantities()
End Sub


---

✅ 3️⃣ Page_Load() - 確保「確認」按鈕存在

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

    ' ✅ 確保「確認」按鈕存在
    If PlaceHolder1.FindControl("btnConfirm") Is Nothing Then
        Dim btnConfirm As New Button()
        btnConfirm.ID = "btnConfirm"
        btnConfirm.Text = "確認"
        AddHandler btnConfirm.Click, AddressOf btnConfirm_Click
        PlaceHolder1.Controls.Add(New LiteralControl("<br/>"))
        PlaceHolder1.Controls.Add(btnConfirm)

        ' ✅ 顯示驗證結果的 Label
        Dim lblValidation As New Label()
        lblValidation.ID = "lblValidationResult"
        PlaceHolder1.Controls.Add(New LiteralControl("<br/>"))
        PlaceHolder1.Controls.Add(lblValidation)
    End If
End Sub


---

✅ 測試方式

1. 點擊「複製」按鈕，允許拆分 itemNo


2. 手動輸入 Quantity


3. 嘗試輸入不存在於 dtsDetailItem 的 itemNo


4. 點擊「確認」按鈕

如果 itemNo 存在且數量正確，顯示綠色成功訊息

如果 itemNo 不存在或數量不符，顯示紅色錯誤訊息



5. PostBack 之後仍能保持 Panel 和 TextBox 狀態




---

✅ 結論

✅ 檢查 Panel 內的 itemNo 是否存在於 dtsDetailItem
✅ 檢查 Panel 內的 Quantity 是否等於 dtsDetailItem 的 Quantity
✅ 點擊「確認」按鈕時檢查所有數據
✅ PostBack 之後仍保持 Panel 和 TextBox 狀態 🚀



✅ 目標

1. 新增「確認」按鈕


2. 檢核相同 itemNo 的 Quantity 總和

比對 Panel 內的 Quantity 加總

確認是否等於 dtsDetailItem 內的 Quantity



3. 如不相符，顯示錯誤訊息


4. PostBack 之後仍能保持 Panel 和 TextBox 狀態




---

🔹 解決方案

1. 遍歷所有 Panel，收集 itemNo 的 Quantity


2. 從 dtsDetailItem 取得原始 Quantity


3. 檢查 Panel 內的 Quantity 是否匹配


4. 如不匹配，顯示錯誤訊息


5. 如匹配，顯示確認訊息




---

✅ 1️⃣ ValidateQuantities() - 驗證 Quantity 總和

' ✅ 驗證 Panel 內的 Quantity 是否與 dtsDetailItem 一致
Private Sub ValidateQuantities()
    Dim dtPo1 As DataTable = CType(ViewState("dtsDetailItem"), DataTable)
    Dim itemQuantitySum As New Dictionary(Of String, Integer) ' 存放 itemNo 的加總數量

    ' ✅ 遍歷所有 Panel，收集 Quantity
    For Each ctrl As Control In PlaceHolder1.Controls
        If TypeOf ctrl Is Panel Then
            Dim panel As Panel = CType(ctrl, Panel)
            Dim txtID As TextBox = CType(RecursiveFindControl(panel, "txt_ID_" & panel.ID), TextBox)
            Dim txtQuantity As TextBox = CType(RecursiveFindControl(panel, "txt_Quantity_" & panel.ID), TextBox)

            If txtID IsNot Nothing AndAlso txtQuantity IsNot Nothing Then
                Dim itemNo As String = txtID.Text.Trim()
                Dim quantity As Integer = 0
                Integer.TryParse(txtQuantity.Text.Trim(), quantity)

                ' ✅ 累加相同 itemNo 的數量
                If itemQuantitySum.ContainsKey(itemNo) Then
                    itemQuantitySum(itemNo) += quantity
                Else
                    itemQuantitySum(itemNo) = quantity
                End If
            End If
        End If
    Next

    ' ✅ 遍歷 dtsDetailItem，比對數量
    Dim errors As New List(Of String)
    For Each row As DataRow In dtPo1.Rows
        Dim itemNo As String = row("ID").ToString()
        Dim originalQuantity As Integer = Convert.ToInt32(row("Quantity"))

        ' ✅ 比對加總數量與原始數量
        If itemQuantitySum.ContainsKey(itemNo) AndAlso itemQuantitySum(itemNo) <> originalQuantity Then
            errors.Add($"ItemNo {itemNo} 的數量總和 ({itemQuantitySum(itemNo)}) 與原始數量 ({originalQuantity}) 不符")
        End If
    Next

    ' ✅ 顯示驗證結果
    If errors.Count > 0 Then
        lblValidationResult.Text = String.Join("<br/>", errors)
        lblValidationResult.ForeColor = System.Drawing.Color.Red
    Else
        lblValidationResult.Text = "所有數量驗證成功！"
        lblValidationResult.ForeColor = System.Drawing.Color.Green
    End If
End Sub


---

✅ 2️⃣ btnConfirm_Click() - 點擊確認按鈕時檢核

' ✅ 點擊「確認」按鈕時執行檢核
Protected Sub btnConfirm_Click(sender As Object, e As EventArgs)
    ValidateQuantities()
End Sub


---

✅ 3️⃣ Page_Load() - 確保 確認 按鈕存在

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

    ' ✅ 確保「確認」按鈕存在
    If PlaceHolder1.FindControl("btnConfirm") Is Nothing Then
        Dim btnConfirm As New Button()
        btnConfirm.ID = "btnConfirm"
        btnConfirm.Text = "確認"
        AddHandler btnConfirm.Click, AddressOf btnConfirm_Click
        PlaceHolder1.Controls.Add(New LiteralControl("<br/>"))
        PlaceHolder1.Controls.Add(btnConfirm)

        ' ✅ 顯示驗證結果的 Label
        Dim lblValidation As New Label()
        lblValidation.ID = "lblValidationResult"
        PlaceHolder1.Controls.Add(New LiteralControl("<br/>"))
        PlaceHolder1.Controls.Add(lblValidation)
    End If
End Sub


---

✅ 測試方式

1. 點擊「複製」按鈕，允許拆分 itemNo


2. 手動輸入 Quantity


3. 點擊「確認」按鈕

如果數量正確，顯示綠色成功訊息

如果數量不符，顯示紅色錯誤訊息



4. PostBack 之後仍能保持 Panel 和 TextBox 狀態




---

✅ 結論

✅ 允許 itemNo 拆分成多筆
✅ 允許 Quantity 可自行輸入
✅ 點擊「確認」按鈕時檢查總數是否正確
✅ PostBack 之後仍保持狀態 🚀



在 VB WebForm (ASP.NET WebForms) 中，可以使用以下方法來限制 TextBox 只能輸入數字：

方法 1：使用 TextMode="Number"

如果 TextBox 只需要接受整數，則可以設定 TextMode="Number"：

<asp:TextBox ID="txtNumber" runat="server" TextMode="Number"></asp:TextBox>

⚠️ 注意：這個方法只在現代瀏覽器有效，且不會阻止手動輸入非數字字符。


---

方法 2：使用 JavaScript 限制輸入

<asp:TextBox ID="txtNumber" runat="server" onkeypress="return isNumber(event)"></asp:TextBox>

<script type="text/javascript">
    function isNumber(evt) {
        var charCode = (evt.which) ? evt.which : evt.keyCode;
        if (charCode < 48 || charCode > 57) {
            return false;
        }
        return true;
    }
</script>

這段 JavaScript 會限制輸入框只能輸入 0-9 的數字。


---

方法 3：使用 RegularExpressionValidator

如果需要進一步驗證輸入內容，可使用 RegularExpressionValidator：

<asp:TextBox ID="txtNumber" runat="server"></asp:TextBox>
<asp:RegularExpressionValidator 
    ID="revNumber" 
    runat="server" 
    ControlToValidate="txtNumber"
    ValidationExpression="^\d+$" 
    ErrorMessage="請輸入數字"
    ForeColor="Red">
</asp:RegularExpressionValidator>

這個方法可以確保用戶提交表單時，TextBox 內容符合數字格式。


---

方法 4：在 VB 後端程式碼驗證

如果想在後端 (VB.NET) 確保輸入為數字，可以在 Button 點擊事件中檢查：

Protected Sub btnSubmit_Click(ByVal sender As Object, ByVal e As EventArgs) Handles btnSubmit.Click
    If Not IsNumeric(txtNumber.Text) Then
        lblMessage.Text = "請輸入有效的數字"
        Exit Sub
    End If
    lblMessage.Text = "輸入正確"
End Sub

這種方法可防止用戶繞過前端驗證。


---

最佳做法

前端 JavaScript (方法 2) 限制輸入，即時反饋用戶

RegularExpressionValidator (方法 3) 驗證格式，避免非數字輸入

後端檢查 (方法 4)，確保安全性，避免惡意攻擊


這樣可以確保 TextBox 只允許輸入數字並防止非數字提交！

