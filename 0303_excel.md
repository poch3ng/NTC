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

