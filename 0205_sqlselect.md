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

