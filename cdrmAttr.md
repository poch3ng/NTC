在 VB.NET 中，使用 CDATA 執行動態 SQL 時可能遇到的問題通常與語法或引用 SQL 的特殊字符相關。如果出錯，可能是以下原因：

1. SQL 語法錯誤
SQL 腳本可能存在語法問題，例如 FOR XML PATH 的用法或單引號未正確處理。


2. 動態 SQL 的單引號問題
在 VB.NET 中，動態 SQL 需要處理內部的單引號。VB.NET 使用雙引號包裹字串，SQL 使用單引號包裹值，這可能導致錯誤。


3. 資料庫連線或執行錯誤
使用的 SQL Server 版本可能不支援某些功能，或者 sp_executesql 無法正確執行。



以下是修正並嵌入 VB.NET 的完整範例：

修正的 SQL 腳本

將單引號進行正確轉義，並測試語法。

DECLARE @columns NVARCHAR(MAX);
DECLARE @sql NVARCHAR(MAX);

-- Step 1: 動態生成所有的 CASE 欄位
SELECT @columns = STUFF((
    SELECT ', MAX(CASE WHEN attribute = ''' + attribute + ''' THEN attributeValue END) AS [' + attribute + ']'
    FROM (SELECT DISTINCT attribute FROM ProductSpecEdit) AS distinct_attributes
    FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 2, '');

-- Step 2: 拼接完整 SQL 查詢
SET @sql = 'SELECT partNo, ' + @columns + ' FROM ProductSpecEdit GROUP BY partNo ORDER BY partNo;';

-- Step 3: 執行動態 SQL
EXEC sp_executesql @sql;

VB.NET 執行方式

使用 ADO.NET 執行此 SQL 腳本。

Imports System.Data.SqlClient

Module Module1
    Sub Main()
        Dim connectionString As String = "YourConnectionStringHere"
        Dim sql As String = "
DECLARE @columns NVARCHAR(MAX);
DECLARE @sql NVARCHAR(MAX);

-- Step 1: 動態生成所有的 CASE 欄位
SELECT @columns = STUFF((
    SELECT ', MAX(CASE WHEN attribute = ''''' + attribute + ''''' THEN attributeValue END) AS ['''' + attribute + '''']'
    FROM (SELECT DISTINCT attribute FROM ProductSpecEdit) AS distinct_attributes
    FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 2, '');

-- Step 2: 拼接完整 SQL 查詢
SET @sql = 'SELECT partNo, ' + @columns + ' FROM ProductSpecEdit GROUP BY partNo ORDER BY partNo;';

-- Step 3: 執行動態 SQL
EXEC sp_executesql @sql;
"
        Using connection As New SqlConnection(connectionString)
            connection.Open()
            Using command As New SqlCommand(sql, connection)
                command.ExecuteNonQuery()
            End Using
        End Using
    End Sub
End Module

注意事項

1. 雙引號與單引號的處理
SQL 中的單引號需要在 VB.NET 中寫成 ''''（四個單引號）。


2. 權限問題
確保資料庫使用者具備執行 sp_executesql 的權限。


3. 測試 SQL 語法
在 SQL Server Management Studio 測試動態 SQL 的生成結果。



如果問題仍未解決，提供錯誤訊息以便進一步協助！



如果你的 SQL Server 版本不支援 STRING_AGG，可以使用 FOR XML PATH 來實現動態 SQL 拼接。以下是適用於舊版本 SQL Server 的完整範例：

實現動態 SQL 拼接

DECLARE @columns NVARCHAR(MAX);
DECLARE @sql NVARCHAR(MAX);

-- Step 1: 動態生成所有的 CASE 欄位
SELECT 
    @columns = STUFF((
        SELECT 
            ', MAX(CASE WHEN attribute = ''' + attribute + ''' THEN attributeValue END) AS [' + attribute + ']'
        FROM (SELECT DISTINCT attribute FROM ProductSpecEdit) AS distinct_attributes
        FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 2, '');

-- Step 2: 拼接完整 SQL 查詢
SET @sql = '
    SELECT 
        partNo, 
        ' + @columns + '
    FROM ProductSpecEdit
    GROUP BY partNo
    ORDER BY partNo;
';

-- Step 3: 執行動態 SQL
EXEC sp_executesql @sql;

說明

1. FOR XML PATH

將每個 attribute 動態轉換為 SQL 中的欄位語法。

使用 STUFF 函數刪除拼接結果開頭的多餘逗號。



2. 動態生成的欄位

查詢會生成每個 attribute 的動態欄位，例如：

MAX(CASE WHEN attribute = 'name' THEN attributeValue END) AS [name],
MAX(CASE WHEN attribute = 'color' THEN attributeValue END) AS [color],
MAX(CASE WHEN attribute = 'size' THEN attributeValue END) AS [size]



3. EXEC sp_executesql

用於執行生成的動態查詢。




假設數據

結果

執行後，生成的 SQL 查詢如下：

SELECT 
    partNo, 
    MAX(CASE WHEN attribute = 'name' THEN attributeValue END) AS [name],
    MAX(CASE WHEN attribute = 'color' THEN attributeValue END) AS [color],
    MAX(CASE WHEN attribute = 'size' THEN attributeValue END) AS [size]
FROM ProductSpecEdit
GROUP BY partNo
ORDER BY partNo;

查詢結果：

此方法適用於不支援 STRING_AGG 的 SQL Server 版本，並能動態生成查詢以處理不固定的欄位名稱需求。



在 SQL Server Management Studio (SSMS) 中，你可以使用動態 SQL 來生成並執行動態查詢。以下是詳細的步驟和範例：

動態 SQL 實現步驟

1. 使用 STRING_AGG 或 FOR XML PATH 來拼接動態欄位。


2. 將拼接好的欄位用於最終查詢。


3. 使用 EXEC 執行動態生成的 SQL。



實現範例

假設你的表是 ProductSpecEdit，結構如下：

以下是完整的動態 SQL 程式碼：

DECLARE @columns NVARCHAR(MAX);
DECLARE @sql NVARCHAR(MAX);

-- Step 1: 動態生成所有的 CASE 欄位
SELECT 
    @columns = STRING_AGG(
        'MAX(CASE WHEN attribute = ''' + attribute + ''' THEN attributeValue END) AS [' + attribute + ']',
        ', '
    )
FROM (SELECT DISTINCT attribute FROM ProductSpecEdit) AS distinct_attributes;

-- Step 2: 拼接完整 SQL 查詢
SET @sql = '
    SELECT 
        partNo, 
        ' + @columns + '
    FROM ProductSpecEdit
    GROUP BY partNo
    ORDER BY partNo;
';

-- Step 3: 執行動態 SQL
EXEC sp_executesql @sql;

關鍵點說明

1. STRING_AGG
用於動態拼接欄位名稱（SQL Server 2017+ 支援）。如果是更舊的版本，可以改用 FOR XML PATH 方法。

替代方式（FOR XML PATH）：

SELECT 
    @columns = STUFF((
        SELECT 
            ', MAX(CASE WHEN attribute = ''' + attribute + ''' THEN attributeValue END) AS [' + attribute + ']'
        FROM (SELECT DISTINCT attribute FROM ProductSpecEdit) AS distinct_attributes
        FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)'), 1, 2, '');


2. sp_executesql
安全地執行動態生成的 SQL。


3. 動態欄位名 依據表中的 attribute 動態生成。



執行結果

如果表中的數據如下：

執行後的查詢結果為：

這樣就能在 SQL Server 中實現動態生成欄位和查詢的需求！




要一次選取所有的 attribute，可以使用動態 SQL 或其他方式來達到目的。以下是調整後的 SQL 範例，假設你的表結構固定，且你希望將每個 attribute 轉為欄位：

SQL 查詢範例

SELECT 
    partNo,
    MAX(CASE WHEN attribute = 'name' THEN attributeValue END) AS name,
    MAX(CASE WHEN attribute = 'color' THEN attributeValue END) AS color,
    MAX(CASE WHEN attribute = 'size' THEN attributeValue END) AS size,
    MAX(CASE WHEN attribute = 'price' THEN attributeValue END) AS price
FROM ProductSpecEdit
GROUP BY partNo
ORDER BY partNo;

自動化處理（如果 attribute 不固定）

若 attribute 不固定，你可以考慮以下方式：

1. 動態 SQL（MySQL 範例）： 先查詢所有的 attribute 值，然後動態生成 SQL：

SET @sql = NULL;
SELECT
    GROUP_CONCAT(
        CONCAT(
            'MAX(CASE WHEN attribute = ''',
            attribute,
            ''' THEN attributeValue END) AS ',
            attribute
        )
    ) INTO @sql
FROM (SELECT DISTINCT attribute FROM ProductSpecEdit) AS attributes;

SET @sql = CONCAT('SELECT partNo, ', @sql, ' FROM ProductSpecEdit GROUP BY partNo ORDER BY partNo;');
PREPARE stmt FROM @sql;
EXECUTE stmt;
DEALLOCATE PREPARE stmt;


2. 使用程式碼生成 SQL
如果使用程式語言（如 Python、VB.NET），可以先動態取得 attribute 列表，再拼接查詢。



這樣可以實現自動適應不同的 attribute 結構。

