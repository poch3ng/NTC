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