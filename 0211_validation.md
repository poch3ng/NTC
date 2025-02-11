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

