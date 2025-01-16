如果您目前的 LogError 方法已經適用於記錄 EDI 850 和 EDI 860 的錯誤資訊，但希望進一步區分並包含更詳細的 EDI 特定資訊（例如交易碼、控制編號、傳送方等），可以在現有方法上進行擴展。

以下是改進的建議：


---

改進 LogError 方法

加入 交易類型 和 交易相關資訊 的參數，讓 Log 更具可追溯性，同時保留現有功能。

改良版程式碼

Public Sub LogError(message As String,
                    transactionType As String, ' EDI 類型，例如 850 或 860
                    Optional controlNumber As String = "",
                    Optional senderID As String = "",
                    Optional receiverID As String = "",
                    <System.Runtime.CompilerServices.CallerMemberName> Optional callerName As String = "",
                    <System.Runtime.CompilerServices.CallerFilePath> Optional callerFile As String = "",
                    <System.Runtime.CompilerServices.CallerLineNumber> Optional callerLine As Integer = 0)
    ' 組合 Log 訊息
    Dim logMessage As String = $"[{DateTime.Now}] 交易類型: {transactionType}, 控制編號: {controlNumber}, 傳送方: {senderID}, 接收方: {receiverID}, 方法名稱: {callerName}, 檔案: {callerFile}, 行號: {callerLine}, 錯誤訊息: {message}"
    
    ' 紀錄到檔案、資料庫或 Console
    System.Diagnostics.Debug.WriteLine(logMessage)

    ' 如果需要，也可以寫入資料庫
    WriteLogToDatabase(logMessage)
End Sub

' 寫入資料庫（可選）
Private Sub WriteLogToDatabase(logMessage As String)
    Try
        Dim connectionString As String = "Your_Connection_String"
        Using conn As New SqlConnection(connectionString)
            conn.Open()
            Dim sql As String = "INSERT INTO SystemLogs (LogTime, LogLevel, Message) VALUES (@LogTime, @LogLevel, @Message)"
            Using cmd As New SqlCommand(sql, conn)
                cmd.Parameters.AddWithValue("@LogTime", DateTime.Now)
                cmd.Parameters.AddWithValue("@LogLevel", "ERROR")
                cmd.Parameters.AddWithValue("@Message", logMessage)
                cmd.ExecuteNonQuery()
            End Using
        End Using
    Catch ex As Exception
        ' 如果寫入資料庫失敗，記錄到 Console
        System.Diagnostics.Debug.WriteLine($"無法寫入資料庫: {ex.Message}")
    End Try
End Sub


---

如何呼叫 LogError

1. 記錄 EDI 850 錯誤

Try
    ' 模擬錯誤
    Throw New Exception("850 資料解析失敗")
Catch ex As Exception
    LogError(ex.Message, "850", "123456789", "SenderA", "ReceiverB")
End Try

2. 記錄 EDI 860 錯誤

Try
    ' 模擬錯誤
    Throw New Exception("860 無效 PO Number")
Catch ex As Exception
    LogError(ex.Message, "860", "987654321", "SenderA", "ReceiverB")
End Try


---

Log 輸出範例

850 錯誤：

[2025-01-15 14:00:00] 交易類型: 850, 控制編號: 123456789, 傳送方: SenderA, 接收方: ReceiverB, 方法名稱: Process850, 檔案: C:\Projects\EDIProcessor.vb, 行號: 35, 錯誤訊息: 850 資料解析失敗

860 錯誤：

[2025-01-15 14:05:00] 交易類型: 860, 控制編號: 987654321, 傳送方: SenderA, 接收方: ReceiverB, 方法名稱: Process860, 檔案: C:\Projects\EDIProcessor.vb, 行號: 50, 錯誤訊息: 860 無效 PO Number


---

改進點

1. 區分交易類型

將 transactionType 引入，輕鬆識別 Log 所屬的 EDI 類型（850 或 860）。



2. 加入核心交易資訊

控制編號、傳送方、接收方等資訊能幫助快速追蹤問題。



3. 記錄到資料庫

實現 WriteLogToDatabase 方法，讓 Log 持久化。



4. 保留 Caller Information

仍然記錄方法名稱、檔案路徑和行號，方便程式碼層級的問題追蹤。





---

資料表範例

CREATE TABLE SystemLogs (
    LogID INT IDENTITY(1,1) PRIMARY KEY,
    LogTime DATETIME NOT NULL,        -- 記錄時間
    LogLevel NVARCHAR(20) NOT NULL,  -- INFO, WARN, ERROR
    Message NVARCHAR(MAX) NOT NULL   -- 錯誤訊息
);


---

結論

透過改良後的 LogError，您可以：

1. 記錄詳細的 EDI 資訊（交易類型、控制編號、傳送方等）。


2. 快速區分 EDI 850 和 860 的錯誤。


3. 保存完整的錯誤上下文，無論是調試或分析都更高效。



根據需求，還可以進一步擴展，例如記錄檔案名稱或其他上下文資訊。

