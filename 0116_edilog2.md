在接收 EDI 850 時，您記錄的重點應該側重於以下幾個方面：

1. 檔案層級資訊

檔案是何時接收到的、由誰發送、檔案格式是否有效。



2. 處理狀態

是否成功解析、驗證數據，或發生錯誤。



3. 交易追蹤資訊

控制編號、傳送方和接收方 ID 等，便於追溯。





---

需要紀錄的欄位

1. 通用欄位

這些欄位適用於記錄任何 Log：

LogTime: 接收到檔案的時間。

LogLevel: 日誌級別，例如 INFO、WARN、ERROR。

Message: 描述接收過程中的狀態或錯誤。



---

2. EDI 850 特定欄位

在接收時，重點記錄以下幾個欄位：

1. FileName

接收到的 EDI 檔案名稱。

資料型別：NVARCHAR(255)



2. Sender ID (ISA06)

傳送方的 EDI ID，用於識別發送方。

資料型別：NVARCHAR(50)



3. Receiver ID (ISA08)

接收方的 EDI ID，用於識別本系統。

資料型別：NVARCHAR(50)



4. Control Number (ST02)

交易控制編號，用於追蹤 EDI 文件。

資料型別：NVARCHAR(20)



5. TransactionSetID (ST01)

EDI 交易碼，例如 850。

資料型別：NVARCHAR(10)



6. Validation Status

接收到的檔案是否通過初步驗證，例如檔案格式正確與否。

資料型別：NVARCHAR(20)，如 VALID、INVALID。



7. Error Details

如果有錯誤，記錄詳細的錯誤資訊。

資料型別：NVARCHAR(MAX)



8. Raw Data

可選：原始 EDI 檔案的部分數據（例如 Header），便於排錯。

資料型別：NVARCHAR(MAX)





---

資料表設計

CREATE TABLE EDI850ReceiveLogs (
    LogID INT IDENTITY(1,1) PRIMARY KEY,
    LogTime DATETIME NOT NULL,        -- 接收時間
    LogLevel NVARCHAR(20) NOT NULL,  -- INFO, WARN, ERROR
    Message NVARCHAR(MAX) NOT NULL,  -- 狀態描述
    FileName NVARCHAR(255),          -- 接收到的檔案名稱
    SenderID NVARCHAR(50),           -- 傳送方 ID (ISA06)
    ReceiverID NVARCHAR(50),         -- 接收方 ID (ISA08)
    ControlNumber NVARCHAR(20),      -- 交易控制號碼 (ST02)
    TransactionSetID NVARCHAR(10),   -- 交易類型 (ST01)
    ValidationStatus NVARCHAR(20),   -- 驗證狀態
    ErrorDetails NVARCHAR(MAX),      -- 錯誤資訊
    RawData NVARCHAR(MAX)            -- 原始數據 (可選)
);


---

記錄範例

成功案例：

錯誤案例：


---

處理流程建議

1. 檔案接收與初步驗證

檢查 EDI 檔案的格式是否正確（例如段落和元素完整性）。

驗證是否包含必要的欄位（如 BEG03 採購單編號）。


2. 記錄成功與錯誤

成功： 只需記錄檔案名稱、傳送方、接收方等基本資訊。

錯誤： 記錄詳細的錯誤資訊，包括哪些欄位缺失或格式錯誤。


3. 可選擇存儲原始數據

若需進一步排錯，可存儲部分原始數據（如 Header 或整個檔案）。



---

查詢範例

1. 查詢最近的錯誤：

SELECT * 
FROM EDI850ReceiveLogs 
WHERE LogLevel = 'ERROR'
ORDER BY LogTime DESC;


2. 查詢特定傳送方的接收記錄：

SELECT * 
FROM EDI850ReceiveLogs 
WHERE SenderID = 'ABCD';




---

結論

在接收 EDI 850 時，應重點記錄檔案的接收狀態、基本交易資訊（如控制編號、傳送方 ID）、驗證結果以及錯誤詳情。如果需要進一步排錯，可以儲存部分原始數據或完整 EDI 檔案內容。

