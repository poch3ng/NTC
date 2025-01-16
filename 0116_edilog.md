是的，使用函數名稱來記載 Log 是非常實用的方式，因為它能幫助快速定位問題所在的程式邏輯。以下是如何在 VB.NET 中實現這個功能的方法。


---

1. 使用 MethodBase 自動取得函數名稱

可以透過 .NET 提供的 System.Reflection.MethodBase.GetCurrentMethod() 來取得目前執行的函數名稱。

範例程式碼

Imports System.Reflection

Public Sub LogError(message As String)
    Dim methodName As String = MethodBase.GetCurrentMethod().Name
    Dim logMessage As String = $"[{DateTime.Now}] 方法名稱: {methodName}, 錯誤訊息: {message}"
    ' 這裡可以改為記錄到檔案或資料庫
    System.Diagnostics.Debug.WriteLine(logMessage)
End Sub

執行結果

若 LogError 在 ExecuteTransaction 方法中被呼叫，則輸出會是：

[2025-01-15 10:45:30] 方法名稱: ExecuteTransaction, 錯誤訊息: INSERT INTO Table1 錯誤。


---

2. 使用 Caller Information 取得呼叫方資訊

除了取得目前方法的名稱，您也可以利用 VB.NET 的 Caller Information 特性，記錄呼叫 LogError 的方法名稱。

修改 LogError 方法

Public Sub LogError(message As String,
                    <System.Runtime.CompilerServices.CallerMemberName> Optional callerName As String = "")
    Dim logMessage As String = $"[{DateTime.Now}] 方法名稱: {callerName}, 錯誤訊息: {message}"
    ' 紀錄 Log
    System.Diagnostics.Debug.WriteLine(logMessage)
End Sub

呼叫範例

在 ExecuteTransaction 方法中：

Public Sub ExecuteTransaction()
    Try
        ' 模擬 SQL 錯誤
        Throw New Exception("INSERT INTO Table1 錯誤。")
    Catch ex As Exception
        LogError(ex.Message)
    End Try
End Sub

執行結果

[2025-01-15 10:50:00] 方法名稱: ExecuteTransaction, 錯誤訊息: INSERT INTO Table1 錯誤。


---

3. 建立通用 Log 方法

如果需要記錄更詳細的資訊（如執行的類別名稱），可以進一步擴展 Log 方法。

範例程式碼

Public Sub LogError(message As String,
                    <System.Runtime.CompilerServices.CallerMemberName> Optional callerName As String = "",
                    <System.Runtime.CompilerServices.CallerFilePath> Optional callerFile As String = "",
                    <System.Runtime.CompilerServices.CallerLineNumber> Optional callerLine As Integer = 0)
    Dim logMessage As String = $"[{DateTime.Now}] 方法名稱: {callerName}, 檔案: {callerFile}, 行號: {callerLine}, 錯誤訊息: {message}"
    ' 紀錄 Log
    System.Diagnostics.Debug.WriteLine(logMessage)
End Sub

呼叫範例

Try
    ' 模擬錯誤
    Throw New Exception("資料庫連線失敗。")
Catch ex As Exception
    LogError(ex.Message)
End Try

執行結果

[2025-01-15 10:55:00] 方法名稱: ExecuteTransaction, 檔案: C:\Projects\MyApp\WebForm1.aspx.vb, 行號: 35, 錯誤訊息: 資料庫連線失敗。


---

4. 日誌結構化建議

將函數名稱記錄到 Log 中可以幫助問題定位，以下是 Log 的結構化內容建議：

時間戳 (Timestamp)

函數名稱 (MethodName)

類別名稱 (ClassName)

檔案名稱與行號 (FilePath 和 LineNumber)

錯誤訊息 (Message)

其他資訊（如執行的 SQL 語句或參數值）



---

5. 結合 Log 框架

若需更進階的功能（例如支援 JSON 格式或多輸出目標），可以結合第三方框架如：

NLog

Serilog


例如使用 NLog 可自動記錄方法名稱，並支援結構化日誌輸出到檔案或資料庫。


---

透過以上方法，您能夠有效利用函數名稱記錄錯誤資訊，並輔助快速排除問題。

