如果你的 E 欄 內容有 雙引號 (")，你需要在 Excel 公式中 正確處理雙引號，以確保生成的 SQL 語法正確。


---

最終解決方案

請使用這個公式，確保 E 欄的內容能安全地放入 SQL 查詢：

="UPDATE MainTable SET Remark='" & SUBSTITUTE(E2, "'", "''") & "' WHERE FPGID=" & C2 & ";"


---

詳細解釋

1. SUBSTITUTE(E2, "'", "''")

SQL 中，單引號 ' 需要 雙寫 ' → ''，否則會發生錯誤。

假如 E2 = 測試備註'sql，會變成：

UPDATE MainTable SET Remark='測試備註''sql' WHERE FPGID=1001;

這樣 SQL Server 就能正確解析單引號。



2. 完整範例

Excel 原始數據： | C (FPGID) | E (Remark) | |-----------|------------| | 1001      | 測試備註    | | 1002      | 這是 O'Neil | | 1003      | "SQL 測試"  |

公式結果：

UPDATE MainTable SET Remark='測試備註' WHERE FPGID=1001;
UPDATE MainTable SET Remark='這是 O''Neil' WHERE FPGID=1002;
UPDATE MainTable SET Remark='"SQL 測試"' WHERE FPGID=1003;





---

批量合併成一個 SQL 語句

如果你想 一次生成多條 SQL 指令，可以用 TEXTJOIN()：

=TEXTJOIN(" ", TRUE, "UPDATE MainTable SET Remark='" & SUBSTITUTE(E2:E100, "'", "''") & "' WHERE FPGID=" & C2:C100 & ";")

這樣會輸出：

UPDATE MainTable SET Remark='測試備註' WHERE FPGID=1001; UPDATE MainTable SET Remark='這是 O''Neil' WHERE FPGID=1002; UPDATE MainTable SET Remark='"SQL 測試"' WHERE FPGID=1003;

你可以直接 複製貼上 SQL Server 執行。


---

結論

✅ SUBSTITUTE(E2, "'", "''") 確保單引號安全
✅ TEXTJOIN() 可一次產生多條 SQL
✅ 公式簡單，不需 VBA 或巨集

這樣你的 Excel 就可以 自動產生 SQL 語法，貼到 SSMS 執行即可！



如果你要直接在 Excel 裡用 公式 來串接 SQL 查詢，你可以使用 CONCATENATE() 或 TEXTJOIN() 來產生 SQL 語法。

假設你的 Excel 欄位如下： | C (FPGID) | E (Remark) | |-----------|-----------| | 1001      | 這是一條備註 | | 1002      | 測試備註 | | 1003      | 另一條備註 |


---

方法 1：使用 CONCATENATE() (舊版 Excel)

在 Excel 中，在 G 欄 輸入：

="UPDATE MainTable SET Remark='" & E2 & "' WHERE FPGID=" & C2 & ";"

然後 向下拖曳複製，就能產生：

UPDATE MainTable SET Remark='這是一條備註' WHERE FPGID=1001;
UPDATE MainTable SET Remark='測試備註' WHERE FPGID=1002;
UPDATE MainTable SET Remark='另一條備註' WHERE FPGID=1003;


---

方法 2：使用 TEXTJOIN() (新版 Excel)

如果你想 一次性合併多筆 SQL，可以使用：

=TEXTJOIN(" ", TRUE, "UPDATE MainTable SET Remark='" & E2:E100 & "' WHERE FPGID=" & C2:C100 & ";")

這樣會輸出 一整串 SQL 指令，可以直接貼到 SSMS 執行。


---

注意事項

1. 如果 Remark 欄位有單引號 '，需替換成 ''
可用 SUBSTITUTE(E2, "'", "''") 來處理：

="UPDATE MainTable SET Remark='" & SUBSTITUTE(E2, "'", "''") & "' WHERE FPGID=" & C2 & ";"


2. 確保 FPGID 是數字型態，否則需要加單引號：

="UPDATE MainTable SET Remark='" & E2 & "' WHERE FPGID='" & C2 & "';"

如果 FPGID 是 VARCHAR，要加上 ' '。



這樣你可以直接在 Excel 產生 SQL 指令，然後 複製貼到 SSMS 執行！

