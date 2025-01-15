在 VB WebForms 使用 SQL Transaction 時，可以透過以下方式來捕捉每一段程式碼可能發生的錯誤，並記錄是哪一段出現問題：

實作步驟

1. 使用 Try-Catch 搭配 Transaction: 在每個需要執行的 SQL 語句周圍使用 Try-Catch，並記錄相關資訊。


2. 加入錯誤紀錄： 在 Catch 區塊中記錄錯誤訊息，包括哪一段程式碼發生錯誤。


3. 結合 Transaction Rollback： 在發生錯誤時，確保交易可以正確回滾。



範例程式碼

Imports System.Data.SqlClient

Partial Class WebForm1
    Inherits System.Web.UI.Page

    Protected Sub ExecuteTransaction()
        Dim connectionString As String = "Your_Connection_String"
        Using conn As New SqlConnection(connectionString)
            conn.Open()

            ' 開始 Transaction
            Dim transaction As SqlTransaction = conn.BeginTransaction()

            Try
                ' 第一段 SQL
                Try
                    Dim cmd1 As New SqlCommand("INSERT INTO Table1 (Column1) VALUES ('Value1')", conn, transaction)
                    cmd1.ExecuteNonQuery()
                Catch ex As Exception
                    Throw New Exception("第一段 SQL 錯誤: " & ex.Message)
                End Try

                ' 第二段 SQL
                Try
                    Dim cmd2 As New SqlCommand("INSERT INTO Table2 (Column1) VALUES ('Value2')", conn, transaction)
                    cmd2.ExecuteNonQuery()
                Catch ex As Exception
                    Throw New Exception("第二段 SQL 錯誤: " & ex.Message)
                End Try

                ' 提交 Transaction
                transaction.Commit()
            Catch ex As Exception
                ' 回滾 Transaction
                transaction.Rollback()
                ' 紀錄錯誤
                LogError(ex.Message)
            Finally
                conn.Close()
            End Try
        End Using
    End Sub

    ' 錯誤紀錄方法
    Private Sub LogError(message As String)
        ' 此處可改為寫入檔案或資料庫
        System.Diagnostics.Debug.WriteLine("錯誤訊息: " & message)
    End Sub
End Class

說明

1. 分段 Try-Catch：

每段 SQL 都有自己的 Try-Catch 區塊，以便在發生錯誤時可以精確捕捉是哪一段出現問題。

每個 Catch 區塊都會拋出帶有段落資訊的 Exception。



2. Transaction Rollback：

任何一段發生錯誤，整個交易會被回滾，確保資料一致性。



3. 錯誤記錄：

在 Catch 中使用 LogError 方法，可以紀錄錯誤訊息到檔案或其他儲存介質。




建議

1. 錯誤詳細性： 可以在錯誤訊息中加入額外資訊（例如 SQL 語句或參數值），便於偵錯。


2. 測試及驗證： 在測試環境模擬各種錯誤情境，確認 Transaction 和錯誤捕捉機制是否正確運作。


3. 結合 NLog 或其他紀錄框架： 若需詳細記錄錯誤，考慮使用專業的日誌工具來記錄錯誤訊息。



