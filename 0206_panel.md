是的，你可以透過 動態產生的 Panel 區分每組資料，並在填寫完成後寫入資料庫。每個 Panel 可以對應到一筆資料，Panel 內的 TextBox 則對應到不同欄位的值。


---

實作步驟

1. 從 SQL 取得資料列數量（筆數）與欄位數量


2. 動態產生 Panel，每個 Panel 代表一筆資料


3. 在 Panel 內建立 Label + TextBox


4. 按下「儲存」按鈕時，將所有 Panel 內的資料寫入資料庫




---

範例程式

Imports System.Data.SqlClient

Public Class Form1
    Private Panels As New List(Of Panel) ' 存儲動態產生的 Panel

    Private Sub LoadData()
        ' SQL 連線字串 (請依實際設定修改)
        Dim connectionString As String = "Server=your_server;Database=your_db;User Id=your_user;Password=your_password;"
        Dim query As String = "SELECT * FROM your_table"

        Using conn As New SqlConnection(connectionString)
            Using cmd As New SqlCommand(query, conn)
                conn.Open()
                Using reader As SqlDataReader = cmd.ExecuteReader()
                    Dim panelYOffset As Integer = 10 ' Panel 間距
                    Dim panelIndex As Integer = 0 ' Panel 計數
                    
                    While reader.Read()
                        Dim fieldCount As Integer = reader.FieldCount
                        Dim newPanel As New Panel()
                        newPanel.Name = "Panel_" & panelIndex
                        newPanel.BorderStyle = BorderStyle.FixedSingle
                        newPanel.Size = New Size(300, (fieldCount * 30) + 20)
                        newPanel.Location = New Point(10, panelYOffset)
                        Panels.Add(newPanel) ' 加入 List，方便後續讀取資料
                        PanelContainer.Controls.Add(newPanel)

                        Dim yOffset As Integer = 10
                        For i As Integer = 0 To fieldCount - 1
                            ' Label
                            Dim lbl As New Label()
                            lbl.Text = reader.GetName(i) ' 取得欄位名稱
                            lbl.Location = New Point(10, yOffset)
                            lbl.AutoSize = True
                            newPanel.Controls.Add(lbl)

                            ' TextBox
                            Dim txt As New TextBox()
                            txt.Name = "TextBox_" & panelIndex & "_" & i ' 唯一名稱
                            txt.Location = New Point(100, yOffset)
                            txt.Width = 150
                            txt.Text = reader(i).ToString() ' 設定初始值
                            newPanel.Controls.Add(txt)

                            yOffset += 30
                        Next

                        panelYOffset += newPanel.Height + 10 ' 調整下一個 Panel 位置
                        panelIndex += 1
                    End While
                End Using
            End Using
        End Using
    End Sub

    Private Sub SaveData()
        Dim connectionString As String = "Server=your_server;Database=your_db;User Id=your_user;Password=your_password;"
        Dim query As String = "UPDATE your_table SET {0} WHERE id = @id" ' 這裡的 SQL 需根據你的資料表調整

        Using conn As New SqlConnection(connectionString)
            conn.Open()
            For Each panel In Panels
                Dim updateQuery As String = "UPDATE your_table SET "
                Dim parameters As New List(Of SqlParameter)
                Dim idValue As String = "" ' 假設第一個欄位是 ID

                ' 遍歷每個 TextBox 並組合 SQL 語法
                For Each ctrl In panel.Controls
                    If TypeOf ctrl Is TextBox Then
                        Dim txt As TextBox = DirectCast(ctrl, TextBox)
                        Dim fieldName As String = txt.Name.Split("_")(2) ' 取得欄位名稱
                        If fieldName = "id" Then
                            idValue = txt.Text ' ID 值
                        Else
                            updateQuery &= fieldName & " = @" & fieldName & ", "
                            parameters.Add(New SqlParameter("@" & fieldName, txt.Text))
                        End If
                    End If
                Next

                ' 去除最後的逗號
                updateQuery = updateQuery.TrimEnd(", ".ToCharArray()) & " WHERE id = @id"
                parameters.Add(New SqlParameter("@id", idValue))

                ' 執行更新
                Using cmd As New SqlCommand(updateQuery, conn)
                    cmd.Parameters.AddRange(parameters.ToArray())
                    cmd.ExecuteNonQuery()
                End Using
            Next
        End Using

        MessageBox.Show("資料更新成功！")
    End Sub

    Private Sub ButtonLoad_Click(sender As Object, e As EventArgs) Handles ButtonLoad.Click
        LoadData()
    End Sub

    Private Sub ButtonSave_Click(sender As Object, e As EventArgs) Handles ButtonSave.Click
        SaveData()
    End Sub
End Class


---

說明

1. LoadData()

讀取 SQL 資料

依據筆數動態建立 Panel，每個 Panel 內部包含 Label（顯示欄位名稱）和 TextBox（輸入值）

Panel 依序排列，不會互相重疊



2. SaveData()

遍歷 Panels 內的 TextBox

組合 SQL UPDATE 語法，將修改後的值寫入資料庫



3. ButtonLoad_Click()

按下按鈕時執行 LoadData()，動態載入 Panel



4. ButtonSave_Click()

按下按鈕時執行 SaveData()，更新資料庫





---

結果

1. 點擊「載入資料」按鈕，Panel 會根據 SQL 取得的資料筆數產生。


2. 每個 Panel 內的 TextBox 會對應欄位值。


3. 修改 TextBox 內的數據後，點擊「儲存資料」按鈕，會將資料寫回 SQL。



這樣你就可以區分每個 Panel，並且正確存取與更新資料庫了！

