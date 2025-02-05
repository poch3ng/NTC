在 SSMS (SQL Server Management Studio) 中，可以使用 UPDATE 語句來更新 SerialNo，條件是 FpgID = 'a'。請使用以下 SQL 指令：

UPDATE YourTableName
SET SerialNo = '20250120-014'
WHERE FpgID = 'a';

注意：

1. 確認資料表名稱 (YourTableName)

請替換成你的實際資料表名稱。



2. 確認 FpgID 是否為 VARCHAR 或 CHAR

若 FpgID 是字串類型 (VARCHAR、CHAR)，則 'a' 需要加單引號。

若 FpgID 是 INT 或數字類型，則應該去掉單引號，例如 WHERE FpgID = 1;。



3. 先測試後執行

可先執行 SELECT 確認受影響的資料：

SELECT * FROM YourTableName WHERE FpgID = 'a';

再執行 UPDATE。



4. 若要更新多個條件

可加 AND 來限制範圍，例如：

UPDATE YourTableName
SET SerialNo = '20250120-014'
WHERE FpgID = 'a' AND Status = 'Active';




執行後，可以再用 SELECT 確認變更是否正確：

SELECT * FROM YourTableName WHERE FpgID = 'a';

如果有其他需求，可以補充詳細資訊。



在 SQL Server Management Studio (SSMS) 中，如果你的 SQL Server 版本是 2017+，可以使用 STRING_AGG；如果是 2016 或更早版本，則需要使用 FOR XML PATH 來合併字串。


---

1. SQL Server 2017+（推薦方式）

SELECT STRING_AGG(column_name, ', ') AS merged_result
FROM table_name;


---

2. SQL Server 2016 或更早版本（使用 FOR XML PATH）

SELECT STUFF(
    (SELECT ', ' + column_name
     FROM table_name
     FOR XML PATH(''), TYPE).value('.', 'NVARCHAR(MAX)')
, 1, 2, '') AS merged_result;

> 解釋：

FOR XML PATH('') 會將多行合併成 XML 字串格式。

STUFF(..., 1, 2, '') 用來去除最前面的 , 。





---

範例

假設 table_name 有以下數據：

column_name
---------
Apple
Banana
Cherry
Date

執行後的結果：

merged_result
-------------
Apple, Banana, Cherry, Date

如果你的 SQL Server 是 2017 以前的版本，請使用 方法 2（FOR XML PATH）。如果是 2017 以上，建議用 STRING_AGG，更簡潔高效。

