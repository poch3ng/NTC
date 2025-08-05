我會幫你撰寫 SQL Server Management Studio (SSMS) 的 SQL 指令來建立這兩個表，並確保子表透過外鍵關聯到主表。以下是具體的 SQL 程式碼：

```sql
-- 建立主表 Result
CREATE TABLE Result (
    ResultID INT PRIMARY KEY IDENTITY(1,1), -- 主鍵，自動遞增
    ResultName NVARCHAR(100) NOT NULL,     -- 結果名稱
    CreateDate DATETIME DEFAULT GETDATE()   -- 建立日期
);

-- 建立子表 ResultSubItem
CREATE TABLE ResultSubItem (
    SubItemID INT PRIMARY KEY IDENTITY(1,1), -- 子表主鍵，自動遞增
    ResultID INT NOT NULL,                  -- 外鍵，關聯到主表
    SubItemName NVARCHAR(100) NOT NULL,     -- 子項目名稱
    SubItemValue NVARCHAR(200),             -- 子項目值
    CONSTRAINT FK_ResultSubItem_Result FOREIGN KEY (ResultID) 
        REFERENCES Result(ResultID) ON DELETE CASCADE
);
```

### 說明：
1. **主表 Result**：
   - `ResultID`：主鍵，自動遞增，作為表的唯一識別。
   - `ResultName`：儲存結果的名稱，假設為必要欄位。
   - `CreateDate`：記錄建立時間，預設為當前時間。

2. **子表 ResultSubItem**：
   - `SubItemID`：子表的主鍵，自動遞增。
   - `ResultID`：外鍵，關聯到主表的 `ResultID`，確保子表記錄對應的主表項目。
   - `SubItemName`：子項目名稱，假設為必要欄位。
   - `SubItemValue`：子項目值，可選欄位，允許空值。
   - `FOREIGN KEY`：建立外鍵約束，確保 `ResultID` 必須對應到主表中的有效 `ResultID`。
   - `ON DELETE CASCADE`：當主表記錄被刪除時，相關的子表記錄也會自動刪除。

### 使用範例：
插入資料以測試表結構：
```sql
-- 插入主表資料
INSERT INTO Result (ResultName) VALUES ('Test Result 1');

-- 插入子表資料，關聯到主表
INSERT INTO ResultSubItem (ResultID, SubItemName, SubItemValue) 
VALUES (1, 'SubItem 1', 'Value 1'), (1, 'SubItem 2', 'Value 2');

-- 查詢主表和子表
SELECT * FROM Result;
SELECT * FROM ResultSubItem;
```

### 注意事項：
- 你可以根據需求調整欄位型別（例如 `NVARCHAR` 的長度）或新增其他欄位。
- 如果不需要 `ON DELETE CASCADE`，可以移除或改為 `ON DELETE SET NULL`（需確保欄位允許空值）。
- 執行前請確認資料庫環境重建表許可權。

如果有其他需求（例如新增欄位或調整結構），請告訴我！