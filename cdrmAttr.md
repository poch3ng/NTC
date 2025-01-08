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

