如果你的 Panel 內包含 純 HTML 的 <table>，以及 td 內的 Style，你可以 動態複製 LiteralControl，並保留所有 HTML Style。


---

✅ 解決方案

1. 從 Panel 內的 Table 取得 HTML


2. 將 Table 內容轉為 LiteralControl，動態複製


3. 確保 td 的 Style 也複製


4. PostBack 之後仍能保持 Table 及 td 的 Style




---

🔹 改進 CloneControl() 方法

' ✅ 複製控制項，包括 HTML Table
Private Function CloneControl(ctrl As Control, prefix As String) As Control
    If TypeOf ctrl Is LiteralControl Then
        ' ✅ 複製 LiteralControl（HTML 原始碼）
        Dim originalLiteral As LiteralControl = DirectCast(ctrl, LiteralControl)
        Dim newLiteral As New LiteralControl()
        newLiteral.Text = originalLiteral.Text
        Return newLiteral
    ElseIf TypeOf ctrl Is TextBox Then
        Dim originalTextBox As TextBox = DirectCast(ctrl, TextBox)
        Dim newTextBox As New TextBox()
        newTextBox.ID = prefix & "_" & originalTextBox.ID
        newTextBox.Text = originalTextBox.Text
        newTextBox.CssClass = originalTextBox.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalTextBox.Style.Keys
            newTextBox.Style(key) = originalTextBox.Style(key)
        Next

        Return newTextBox
    ElseIf TypeOf ctrl Is Label Then
        Dim originalLabel As Label = DirectCast(ctrl, Label)
        Dim newLabel As New Label()
        newLabel.ID = prefix & "_" & originalLabel.ID
        newLabel.Text = originalLabel.Text
        newLabel.CssClass = originalLabel.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalLabel.Style.Keys
            newLabel.Style(key) = originalLabel.Style(key)
        Next

        Return newLabel
    End If

    Return Nothing ' 其他類型的控制項可視需求添加
End Function


---

🔹 ASPX (HTML Table)

<asp:Panel ID="PanelTemplate" runat="server" CssClass="panel" BorderStyle="Solid" BorderWidth="1px" Width="100%" Visible="False">
    <div style="border: 1px solid black; padding: 10px; background-color: lightgray;">
        <table style="width: 100%; border-collapse: collapse;">
            <tr>
                <td style="border: 1px solid black; padding: 10px; background-color: yellow;">
                    <asp:Label ID="lblTitle" runat="server" Text="標題"></asp:Label>
                </td>
            </tr>
            <tr>
                <td style="border: 1px solid black; padding: 10px;">
                    <asp:TextBox ID="txtInput" runat="server" Style="border: 2px solid red; background-color: lightyellow; padding: 5px;"></asp:TextBox>
                </td>
            </tr>
        </table>
    </div>
</asp:Panel>


---

🔹 改進 ClonePanel() 方法

' ✅ 複製 Panel，包括 HTML 內容
Private Function ClonePanel(original As Panel, newID As String) As Panel
    Dim newPanel As New Panel()
    newPanel.ID = newID
    newPanel.CssClass = original.CssClass
    newPanel.BorderStyle = original.BorderStyle
    newPanel.BorderWidth = original.BorderWidth
    newPanel.Width = original.Width
    newPanel.Visible = True ' 讓複製的 Panel 顯示

    ' ✅ 複製 Style 屬性
    For Each key As String In original.Style.Keys
        newPanel.Style(key) = original.Style(key)
    Next

    ' ✅ 複製 Panel 內的所有 HTML 內容
    For Each ctrl As Control In original.Controls
        Dim newCtrl As Control = CloneControl(ctrl, newID)
        If newCtrl IsNot Nothing Then
            newPanel.Controls.Add(newCtrl)
        End If
    Next

    Return newPanel
End Function


---

🔹 測試方式

1. Panel 內的 HTML Table 保持原始結構


2. **每個 td 的 Style（邊框





如果你的 Panel 內 包含 Table 和 td，並且希望完整複製 Table 內的所有 td 的 Style，你需要 遞迴複製 Panel 內所有控制項（包含 Table、tr、td）。


---

✅ 解決方案

1. 確保 Panel 內的 Table 結構不變


2. 遞迴複製 Table 及其內部的 tr、td，保留 Style


3. 確保 TextBox、Label、DropDownList 也完整複製


4. 在 PostBack 之後仍能保持 Table 及 td 的 Style




---

🔹 改進 CloneControl() 方法

' ✅ 遞迴複製控制項，包含 Table、tr、td
Private Function CloneControl(ctrl As Control, prefix As String) As Control
    If TypeOf ctrl Is Table Then
        Dim originalTable As Table = DirectCast(ctrl, Table)
        Dim newTable As New Table()
        newTable.ID = prefix & "_" & originalTable.ID
        newTable.CssClass = originalTable.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalTable.Style.Keys
            newTable.Style(key) = originalTable.Style(key)
        Next

        ' ✅ 複製 Table 內的 tr 和 td
        For Each row As TableRow In originalTable.Rows
            Dim newRow As New TableRow()
            For Each cell As TableCell In row.Cells
                Dim newCell As New TableCell()

                ' ✅ 複製 td 的 Style
                For Each key As String In cell.Style.Keys
                    newCell.Style(key) = cell.Style(key)
                Next

                ' ✅ 複製 td 內的所有控制項
                For Each innerCtrl As Control In cell.Controls
                    Dim newInnerCtrl As Control = CloneControl(innerCtrl, prefix)
                    If newInnerCtrl IsNot Nothing Then
                        newCell.Controls.Add(newInnerCtrl)
                    End If
                Next

                newRow.Cells.Add(newCell)
            Next
            newTable.Rows.Add(newRow)
        Next

        Return newTable

    ElseIf TypeOf ctrl Is TableRow Then
        Dim originalRow As TableRow = DirectCast(ctrl, TableRow)
        Dim newRow As New TableRow()

        ' ✅ 複製 tr 內的 td
        For Each cell As TableCell In originalRow.Cells
            Dim newCell As New TableCell()
            newCell.Text = cell.Text

            ' ✅ 複製 td 的 Style
            For Each key As String In cell.Style.Keys
                newCell.Style(key) = cell.Style(key)
            Next

            newRow.Cells.Add(newCell)
        Next

        Return newRow

    ElseIf TypeOf ctrl Is TableCell Then
        Dim originalCell As TableCell = DirectCast(ctrl, TableCell)
        Dim newCell As New TableCell()
        newCell.Text = originalCell.Text

        ' ✅ 複製 td 的 Style
        For Each key As String In originalCell.Style.Keys
            newCell.Style(key) = originalCell.Style(key)
        Next

        Return newCell

    ElseIf TypeOf ctrl Is TextBox Then
        Dim originalTextBox As TextBox = DirectCast(ctrl, TextBox)
        Dim newTextBox As New TextBox()
        newTextBox.ID = prefix & "_" & originalTextBox.ID
        newTextBox.Text = originalTextBox.Text
        newTextBox.CssClass = originalTextBox.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalTextBox.Style.Keys
            newTextBox.Style(key) = originalTextBox.Style(key)
        Next

        Return newTextBox

    ElseIf TypeOf ctrl Is Label Then
        Dim originalLabel As Label = DirectCast(ctrl, Label)
        Dim newLabel As New Label()
        newLabel.ID = prefix & "_" & originalLabel.ID
        newLabel.Text = originalLabel.Text
        newLabel.CssClass = originalLabel.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalLabel.Style.Keys
            newLabel.Style(key) = originalLabel.Style(key)
        Next

        Return newLabel

    ElseIf TypeOf ctrl Is DropDownList Then
        Dim originalDropDown As DropDownList = DirectCast(ctrl, DropDownList)
        Dim newDropDown As New DropDownList()
        newDropDown.ID = prefix & "_" & originalDropDown.ID
        newDropDown.CssClass = originalDropDown.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalDropDown.Style.Keys
            newDropDown.Style(key) = originalDropDown.Style(key)
        Next

        ' 複製選項
        For Each item As ListItem In originalDropDown.Items
            newDropDown.Items.Add(New ListItem(item.Text, item.Value))
        Next

        Return newDropDown
    End If

    Return Nothing ' 其他類型的控制項可視需求添加
End Function


---

🔹 ASPX 範例

<asp:Panel ID="PanelTemplate" runat="server" CssClass="panel" BorderStyle="Solid" BorderWidth="1px" Width="100%" Visible="False">
    <asp:Table ID="tblMain" runat="server" CssClass="table-style" Style="border-collapse: collapse; width: 100%; border: 1px solid black;">
        <asp:TableRow>
            <asp:TableCell Style="border: 1px solid black; padding: 10px; background-color: lightgray;">
                <asp:Label ID="lblTitle" runat="server" Text="標題"></asp:Label>
            </asp:TableCell>
        </asp:TableRow>
        <asp:TableRow>
            <asp:TableCell Style="border: 1px solid black; padding: 10px;">
                <asp:TextBox ID="txtInput" runat="server" Style="border: 2px solid red; background-color: lightyellow; padding: 5px;"></asp:TextBox>
            </asp:TableCell>
        </asp:TableRow>
        <asp:TableRow>
            <asp:TableCell Style="border: 1px solid black; padding: 10px;">
                <asp:DropDownList ID="ddlOptions" runat="server" Style="width: 200px; font-weight: bold;">
                    <asp:ListItem Text="選項 1" Value="1"></asp:ListItem>
                    <asp:ListItem Text="選項 2" Value="2"></asp:ListItem>
                    <asp:ListItem Text="選項 3" Value="3"></asp:ListItem>
                </asp:DropDownList>
            </asp:TableCell>
        </asp:TableRow>
    </asp:Table>
</asp:Panel>


---

🔹 測試結果

1. Panel 內的 Table 保持原始結構


2. 每個 td 的 Style（邊框、背景色、padding）完整複製


3. **TextBox





如果你希望 複製 Panel 內所有控制項的 Style，可以在 CloneControl() 方法中 將原本 TextBox、Label、DropDownList 等控制項的 Style 屬性完整複製到新控制項。


---

🔹 改進 CloneControl() 方法

' ✅ 複製控制項，包含 CSSClass 和 Style
Private Function CloneControl(ctrl As Control, prefix As String) As Control
    If TypeOf ctrl Is TextBox Then
        Dim originalTextBox As TextBox = DirectCast(ctrl, TextBox)
        Dim newTextBox As New TextBox()
        newTextBox.ID = prefix & "_" & originalTextBox.ID
        newTextBox.Text = originalTextBox.Text
        newTextBox.CssClass = originalTextBox.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalTextBox.Style.Keys
            newTextBox.Style(key) = originalTextBox.Style(key)
        Next

        Return newTextBox

    ElseIf TypeOf ctrl Is Label Then
        Dim originalLabel As Label = DirectCast(ctrl, Label)
        Dim newLabel As New Label()
        newLabel.ID = prefix & "_" & originalLabel.ID
        newLabel.Text = originalLabel.Text
        newLabel.CssClass = originalLabel.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalLabel.Style.Keys
            newLabel.Style(key) = originalLabel.Style(key)
        Next

        Return newLabel

    ElseIf TypeOf ctrl Is DropDownList Then
        Dim originalDropDown As DropDownList = DirectCast(ctrl, DropDownList)
        Dim newDropDown As New DropDownList()
        newDropDown.ID = prefix & "_" & originalDropDown.ID
        newDropDown.CssClass = originalDropDown.CssClass

        ' ✅ 複製 Style 屬性
        For Each key As String In originalDropDown.Style.Keys
            newDropDown.Style(key) = originalDropDown.Style(key)
        Next

        ' 複製選項
        For Each item As ListItem In originalDropDown.Items
            newDropDown.Items.Add(New ListItem(item.Text, item.Value))
        Next

        Return newDropDown
    End If

    Return Nothing ' 其他類型的控制項可視需求添加
End Function


---

🔹 說明

1. 完整複製 Style 屬性

For Each key As String In originalTextBox.Style.Keys
    newTextBox.Style(key) = originalTextBox.Style(key)
Next

這段程式碼會遍歷 原本的控制項（如 TextBox、Label、DropDownList）的 Style，並將它們逐一 複製到新控制項。

確保 font-size、color、background-color、margin、padding 等樣式不變。



2. 完整支援

TextBox

Label

DropDownList

你可以擴展到其他 Web 控制項，如 Button、CheckBox 等。





---

🔹 測試方式

在 PanelTemplate 設定一些 Style

<asp:Panel ID="PanelTemplate" runat="server" CssClass="panel" BorderStyle="Solid" BorderWidth="1px" Width="100%" Visible="False">
    <asp:Label ID="lblTitle" runat="server



如果你的 aspx 已經有一個設計好的 Panel，而你希望 根據 po1 數量動態複製這個 Panel，你可以使用 動態複製控制項的方法 來達成。


---

✅ 解決方案

1. 使用 FindControl 取得已存在的 Panel


2. 複製 Panel 內的所有控制項


3. 確保 PostBack 時不會消失


4. 保持 TextBox 內的輸入值




---

🔹 程式碼

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        ' 從 QueryString 或 Request 取得 po1 數量
        Dim po1Count As Integer = If(Request.QueryString("po1") IsNot Nothing, Convert.ToInt32(Request.QueryString("po1")), 0)

        ' 紀錄 Panel 數量
        ViewState("PanelCount") = po1Count

        ' 生成對應數量的 Panel
        DuplicatePanels(po1Count)
    Else
        ' PostBack 時，重新載入 Panel
        ReloadPanels()
    End If
End Sub

' ✅ 根據 po1 數量複製現有 Panel
Private Sub DuplicatePanels(po1Count As Integer)
    Dim originalPanel As Panel = CType(FindControl("PanelTemplate"), Panel)

    If originalPanel IsNot Nothing Then
        For i As Integer = 1 To po1Count
            Dim newPanel As Panel = ClonePanel(originalPanel, "Panel_" & i)
            PlaceHolder1.Controls.Add(newPanel)
        Next
    End If
End Sub

' ✅ 複製 Panel
Private Function ClonePanel(original As Panel, newID As String) As Panel
    Dim newPanel As New Panel()
    newPanel.ID = newID
    newPanel.CssClass = original.CssClass
    newPanel.BorderStyle = original.BorderStyle
    newPanel.BorderWidth = original.BorderWidth
    newPanel.Width = original.Width
    newPanel.Style("margin-bottom") = "10px"

    ' 複製 Panel 內的所有控制項
    For Each ctrl As Control In original.Controls
        Dim newCtrl As Control = CloneControl(ctrl, newID)
        If newCtrl IsNot Nothing Then
            newPanel.Controls.Add(newCtrl)
        End If
    Next

    Return newPanel
End Function

' ✅ 複製 Label、TextBox、DropDownList
Private Function CloneControl(ctrl As Control, prefix As String) As Control
    If TypeOf ctrl Is TextBox Then
        Dim originalTextBox As TextBox = DirectCast(ctrl, TextBox)
        Dim newTextBox As New TextBox()
        newTextBox.ID = prefix & "_" & originalTextBox.ID
        newTextBox.Text = originalTextBox.Text
        newTextBox.CssClass = originalTextBox.CssClass
        Return newTextBox
    ElseIf TypeOf ctrl Is Label Then
        Dim originalLabel As Label = DirectCast(ctrl, Label)
        Dim newLabel As New Label()
        newLabel.ID = prefix & "_" & originalLabel.ID
        newLabel.Text = originalLabel.Text
        newLabel.CssClass = originalLabel.CssClass
        Return newLabel
    ElseIf TypeOf ctrl Is DropDownList Then
        Dim originalDropDown As DropDownList = DirectCast(ctrl, DropDownList)
        Dim newDropDown As New DropDownList()
        newDropDown.ID = prefix & "_" & originalDropDown.ID
        newDropDown.CssClass = originalDropDown.CssClass

        ' 複製選項
        For Each item As ListItem In originalDropDown.Items
            newDropDown.Items.Add(New ListItem(item.Text, item.Value))
        Next

        Return newDropDown
    End If

    Return Nothing ' 其他類型的控制項可視需求添加
End Function

' ✅ PostBack 時還原 Panel
Private Sub ReloadPanels()
    Dim po1Count As Integer = If(ViewState("PanelCount") IsNot Nothing, CInt(ViewState("PanelCount")), 0)
    DuplicatePanels(po1Count)
End Sub


---

🔹 ASPX 範例

<asp:Panel ID="PanelTemplate" runat="server" CssClass="panel" BorderStyle="Solid" BorderWidth="1px" Width="100%">
    <asp:Label ID="lblTitle" runat="server" Text="標題"></asp:Label>
    <br />
    <asp:TextBox ID="txtInput" runat="server"></asp:TextBox>
    <br />
    <asp:DropDownList ID="ddlOptions" runat="server">
        <asp:ListItem Text="選項 1" Value="1"></asp:ListItem>
        <asp:ListItem Text="選項 2" Value="2"></asp:ListItem>
        <asp:ListItem Text="選項 3" Value="3"></asp:ListItem>
    </asp:DropDownList>
</asp:Panel>

<asp:PlaceHolder ID="PlaceHolder1" runat="server"></asp:PlaceHolder>

<asp:Button ID="btnSave" runat="server" Text="儲存" OnClick="SavePanelData"/>


---

🔹 進階：儲存 Panel 內的輸入資料

如果你希望 暫存使用者填入的數據，可以使用 Session。

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
                ElseIf TypeOf ctrl Is DropDownList Then
                    Dim ddl As DropDownList = DirectCast(ctrl, DropDownList)
                    panelData(panelId & "_" & ddl.ID) = ddl.SelectedValue
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
                    ElseIf TypeOf ctrl Is DropDownList Then
                        Dim ddl As DropDownList = DirectCast(ctrl, DropDownList)
                        Dim key As String = panelId & "_" & ddl.ID
                        If panelData.ContainsKey(key) Then
                            ddl.SelectedValue = panelData(key) ' 還原選擇的值
                        End If
                    End If
                Next
            End If
        Next
    End If
End Sub

📌 更新 Page_Load()

Protected Sub Page_Load(ByVal sender As Object, ByVal e As EventArgs) Handles Me.Load
    If Not IsPostBack Then
        Dim po1Count As Integer = If(Request.QueryString("po1") IsNot Nothing, Convert.ToInt32(Request.QueryString("po1")), 0)
        ViewState("PanelCount") = po1Count
        DuplicatePanels(po1Count)
    Else
        ReloadPanels()
        RestorePanelData() ' 還原輸入值
    End If
End Sub


---

🔹 結果

✅ 根據 po1 數量動態複製 Panel
✅ 複製 Label、TextBox、DropDownList
✅ PostBack 不會消失
✅ 輸入資料可暫存並還原

這樣你的 ASP.NET WebForm 就能 根據 po1 數量，自動複製現有的 Panel，並保持 PostBack 後的數據！



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

