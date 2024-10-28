你可以在 SQL 中使用一個欄位儲存多個錯誤代碼（例如 `J1;K3;`），然後通過查詢將這些錯誤代碼轉換為相應的錯誤名稱。以下是一個解決方案：

### 1. 建立含錯誤代碼的主資料表
假設你有一個資料表 `ErrCodeTable`，裡面有 `ErrCode` 和 `ErrName` 欄位：

```sql
CREATE TABLE ErrCodeTable (
    ErrCode CHAR(2),
    ErrName VARCHAR(50)
);

-- 插入一些範例數據
INSERT INTO ErrCodeTable (ErrCode, ErrName)
VALUES ('J1', 'Error J1 Description'),
       ('K3', 'Error K3 Description'),
       ('L5', 'Error L5 Description');
```

### 2. 使用含多個錯誤代碼的欄位
假設你有另一個資料表 `MainTable`，其中 `ErrorCodes` 欄位用來儲存多個錯誤代碼（例如 `J1;K3;`）：

```sql
CREATE TABLE MainTable (
    ID INT,
    ErrorCodes VARCHAR(50) -- 儲存多個錯誤代碼，例如 'J1;K3;'
);

-- 插入範例數據
INSERT INTO MainTable (ID, ErrorCodes)
VALUES (1, 'J1;K3;'),
       (2, 'L5;J1;');
```

### 3. 查詢並顯示錯誤名稱
可以使用 `STRING_SPLIT` 函數將 `ErrorCodes` 欄位中的錯誤代碼分割為獨立的行，然後通過 `JOIN` 對應 `ErrName`。

```sql
SELECT m.ID,
       m.ErrorCodes,
       STRING_AGG(e.ErrName, '; ') AS ErrorNames -- 將所有錯誤名稱合併為一個欄位
FROM MainTable m
CROSS APPLY STRING_SPLIT(m.ErrorCodes, ';') AS s -- 將 ErrorCodes 分割成單獨的錯誤代碼
JOIN ErrCodeTable e ON s.value = e.ErrCode -- 將分割的錯誤代碼與 ErrCodeTable 關聯
GROUP BY m.ID, m.ErrorCodes;
```

### 查詢結果範例
執行這段 SQL 後的結果會如下：

| ID  | ErrorCodes | ErrorNames                        |
|-----|------------|-----------------------------------|
| 1   | J1;K3;     | Error J1 Description; Error K3 Description |
| 2   | L5;J1;     | Error L5 Description; Error J1 Description |

### 說明
1. `STRING_SPLIT` 用來將 `ErrorCodes` 以分號（`;`）分割成獨立的錯誤代碼。
2. `JOIN` 將分割後的錯誤代碼與 `ErrCodeTable` 關聯，獲取對應的 `ErrName`。
3. `STRING_AGG` 將多個錯誤名稱合併成單一欄位，使用分號分隔。

這樣就能夠從一個欄位中提取多個錯誤代碼並顯示相應的錯誤名稱。

SQL 本身不支援直接的迴圈語法，但我們可以使用 `WHILE` 迴圈來達成這個需求。以下是在 SQL Server 中使用 `WHILE` 迴圈來複製 `ErrCode = 'J'` 的資料，並生成 `J1` 到 `J9` 的方法：

### 方法：使用 `WHILE` 迴圈與 `INSERT INTO`

```sql
DECLARE @i INT = 1; -- 初始化計數器
DECLARE @ErrCodePrefix CHAR(1) = 'J'; -- ErrCode 前綴

WHILE @i <= 9
BEGIN
    -- 插入複製資料，並設置 ErrCode 為 'J1' 到 'J9'
    INSERT INTO ErrCodeTable (ErrCode, ErrName, 其他欄位)
    SELECT @ErrCodePrefix + CAST(@i AS CHAR(1)), ErrName, 其他欄位
    FROM ErrCodeTable
    WHERE ErrCode = 'J';

    SET @i = @i + 1; -- 計數器遞增
END;
```

### 說明
1. `@i` 是計數器，從 `1` 到 `9`。
2. `@ErrCodePrefix` 是 `ErrCode` 的前綴字母 `'J'`。
3. 迴圈內的 `INSERT INTO` 語句會將 `ErrCode = 'J'` 的資料複製並修改 `ErrCode` 為 `J1` 到 `J9`。
4. 每次迴圈執行時，`@i` 增加 1，直到 `@i` 等於 10 時停止。

這樣可以一次性生成 `J1` 到 `J9` 的多筆記錄，無需手動編寫多條 `INSERT` 語句。

如果 `ErrCode` 和 `ErrName` 的值完全相同，並且 SQL Server 無法刪除重複的資料，你可以透過以下方法來刪除這些重複的記錄。

### 方法 1：使用 `ROW_NUMBER()` 輔助刪除重複資料

這個方法會使用 `ROW_NUMBER()` 來給每組相同的 `ErrCode` 和 `ErrName` 編號，然後保留第一筆資料並刪除其餘的重複資料。

```sql
WITH DuplicateRecords AS (
    SELECT ErrCode, ErrName,
           ROW_NUMBER() OVER(PARTITION BY ErrCode, ErrName ORDER BY (SELECT NULL)) AS RowNum
    FROM ErrCodeTable -- 請將 ErrCodeTable 替換為實際的資料表名稱
)
DELETE FROM DuplicateRecords
WHERE RowNum > 1;
```

### 方法 2：利用 `CTE` 與 `MIN` 輔助刪除重複資料

如果資料表有 `ID` 或其他唯一識別欄位，你也可以使用以下方式來刪除重複項，只保留每組重複資料中的一條記錄：

```sql
WITH DuplicateRecords AS (
    SELECT MIN(ID) AS MinID -- 假設有唯一識別的 ID 欄位
    FROM ErrCodeTable
    GROUP BY ErrCode, ErrName
)
DELETE FROM ErrCodeTable
WHERE ID NOT IN (SELECT MinID FROM DuplicateRecords);
```

### 方法 3：利用 `JOIN` 刪除重複資料

如果無法使用 `ROW_NUMBER()`，可以利用 `JOIN` 的方式來刪除重複的記錄：

```sql
DELETE T1
FROM ErrCodeTable T1
JOIN ErrCodeTable T2
ON T1.ErrCode = T2.ErrCode
AND T1.ErrName = T2.ErrName
AND T1.ID > T2.ID; -- 假設 ID 是唯一識別欄位
```

以上方法會刪除重複的 `ErrCode` 和 `ErrName` 記錄，但保留每組中最早的一條。執行時請謹慎，建議在操作前進行資料備份。

在 SQL Server Management Studio (SSMS) 中，你可以使用以下 SQL 語句將 `ErrCode` 表中的欄位類型從 `char(1)` 更改為 `char(2)`：

```sql
ALTER TABLE ErrCode
ALTER COLUMN 欄位名稱 CHAR(2);
```

將 `欄位名稱` 替換為你要更改的實際欄位名稱。此語句會直接在 SSMS 查詢視窗中執行即可。