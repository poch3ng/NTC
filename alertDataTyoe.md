在 SQL Server Management Studio (SSMS) 中，你可以使用以下 SQL 語句將 `ErrCode` 表中的欄位類型從 `char(1)` 更改為 `char(2)`：

```sql
ALTER TABLE ErrCode
ALTER COLUMN 欄位名稱 CHAR(2);
```

將 `欄位名稱` 替換為你要更改的實際欄位名稱。此語句會直接在 SSMS 查詢視窗中執行即可。