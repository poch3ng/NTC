✅ 目標

1. 將所有 Panel 內的 itemNo、Price、Quantity、Date 存入資料庫


2. 確保 PostBack 之後仍能保持 Panel 和 TextBox 狀態


3. 顯示資料庫存入結果（成功或失敗）




---

🔹 解決方案

1. 遍歷 Panel，收集所有 TextBox 資料


2. 使用 SQL 語法將資料寫入資料庫


3. 顯示存入結果


4. 確保 PostBack 之後仍保持 Panel 狀態




---

✅ 1️⃣ SavePanelsToDatabase() - 將 Panel 內資料存入資料庫

Imports System.Data.SqlClient

' ✅ 儲存 Panel 內的資料至資料庫
Private Sub SavePanelsToDatabase()
    Dim connectionString As String = "your_connection_string_here" ' ✅ 修改為你的資料庫連線字串
    Dim query As String = "INSERT INTO YourTable (ItemNo, Price, Quantity, Date) VALUES (@ItemNo, @Price, @Quantity, @Date)"
    
    Using conn As New SqlConnection(connectionString)
        conn.Open()

        For Each ctrl As Control In PlaceHolder1.Controls
            If TypeOf ctrl Is Panel Then
                Dim panel As Panel = CType(ctrl, Panel)
                Dim txtID As TextBox = CType(RecursiveFindControl(panel, "txt_ID_" & panel.ID), TextBox)
                Dim txtPrice As TextBox = CType(RecursiveFindControl(panel, "txt_Price_" & panel.ID), TextBox)
                Dim txtQuantity As TextBox = CType(RecursiveFindControl(panel, "txt_Quantity_" & panel.ID), TextBox)
                Dim txtDate As TextBox = CType(RecursiveFindControl(panel, "txt_Date_" & panel.ID), TextBox)

                If txtID IsNot Nothing AndAlso txtPrice IsNot Nothing AndAlso txtQuantity IsNot Nothing AndAlso txtDate IsNot Nothing Then
                    Using cmd As New SqlCommand(query, conn)
                        cmd.Parameters.AddWithValue("@ItemNo", txtID.Text.Trim())
                        cmd.Parameters.AddWithValue("@Price", Convert.ToDecimal(txtPrice.Text.Trim()))
                        cmd.Parameters.AddWithValue("@Quantity", Convert.ToInt32(txtQuantity.Text.Trim()))
                        cmd.Parameters.AddWithValue("@Date", Convert.ToDateTime(txtDate.Text.Trim()))

                        Try
                            cmd.ExecuteNonQuery() ' ✅ 寫入資料庫
                        Catch ex As Exception
                            lblSaveResult.Text = "錯誤：" & ex.Message
                            lblSaveResult.ForeColor = System.Drawing.Color.Red
                            Exit Sub
                        End Try
                    End Using
                End If
            End If
        Next

        lblSaveResult.Text = "所有資料成功存入資料庫！"
        lblSaveResult.ForeColor = System.Drawing.Color.Green
    End Using
End Sub


---

✅ 2️⃣ btnSave_Click() - 點擊存入按鈕時執行資料庫存入

' ✅ 點擊「存入資料庫」按鈕時執行儲存
Protected Sub btnSave_Click(sender As Object, e As EventArgs)
    SavePanelsToDatabase()
End Sub


---

✅ 3️⃣ Page_Load() - 確保「存入資料庫」按鈕存在

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

    ' ✅ 確保「存入資料庫」按鈕存在
    If PlaceHolder1.FindControl("btnSave") Is Nothing Then
        Dim btnSave As New Button()
        btnSave.ID = "btnSave"
        btnSave.Text = "存入資料庫"
        AddHandler btnSave.Click, AddressOf btnSave_Click
        PlaceHolder1.Controls.Add(New LiteralControl("<br/>"))
        PlaceHolder1.Controls.Add(btnSave)

        ' ✅ 顯示存入結果的 Label
        Dim lblSave As New Label()
        lblSave.ID = "lblSaveResult"
        PlaceHolder1.Controls.Add(New LiteralControl("<br/>"))
        PlaceHolder1.Controls.Add(lblSave)
    End If
End Sub


---

✅ 測試方式

1. 點擊「複製」按鈕，允許新增 Panel


2. 手動輸入 Quantity、Date


3. 點擊「存入資料庫」按鈕

如果存入成功，顯示綠色成功訊息

如果存入失敗，顯示紅色錯誤訊息



4. PostBack 之後仍能保持 Panel 和 TextBox 狀態




---

✅ 結論

✅ 允許 itemNo 拆分成多筆
✅ 允許 Quantity、Date 可自行輸入
✅ 將所有 Panel 內的資料存入資料庫
✅ PostBack 之後仍保持狀態 🚀

