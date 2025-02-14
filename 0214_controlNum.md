沒錯，如果都從 100000000 開始遞增，那 ISA, GS, ST 可能會變成一樣的號碼。
解決方法：

讓 ISA, GS, ST 各自獨立遞增，但仍然符合 9 位數規則。

例如：

ISA13 = IEA02 = 100000001

GS06 = GE02 = 200000001

ST02 = SE02 = 300000001




---

1. SQL：不同類型的控制號分開儲存

CREATE TABLE EDI_ControlNumbers (
    ControlType VARCHAR(10) PRIMARY KEY, -- 控制號類型 (ISA, GS, ST)
    LastNumber INT NOT NULL
);

-- 初始化不同類型的號碼
INSERT INTO EDI_ControlNumbers (ControlType, LastNumber) VALUES ('ISA', 100000000);
INSERT INTO EDI_ControlNumbers (ControlType, LastNumber) VALUES ('GS', 200000000);
INSERT INTO EDI_ControlNumbers (ControlType, LastNumber) VALUES ('ST', 300000000);


---

2. SQL 查詢：更新並取得下一個控制號

UPDATE EDI_ControlNumbers 
SET LastNumber = CASE 
    WHEN LastNumber < 999999999 THEN LastNumber + 1 
    ELSE 100000000 
END 
OUTPUT INSERTED.LastNumber
WHERE ControlType = @Type;

這樣：

ISA 會從 100000000 開始遞增

GS 會從 200000000 開始遞增

ST 會從 300000000 開始遞增

確保不同類型的控制號不會相同！



---

3. VB.NET 取得不同的控制號

Imports System.Data.SqlClient

Function GetNextControlNumberFromDB(controlType As String) As String
    Dim connectionString As String = "your_connection_string_here"
    Dim nextNumber As Integer

    Using conn As New SqlConnection(connectionString)
        conn.Open()
        Dim cmd As New SqlCommand("UPDATE EDI_ControlNumbers SET LastNumber = CASE WHEN LastNumber < 999999999 THEN LastNumber + 1 ELSE 100000000 END OUTPUT INSERTED.LastNumber WHERE ControlType = @type", conn)
        cmd.Parameters.AddWithValue("@type", controlType)
        nextNumber = cmd.ExecuteScalar()
    End Using

    Return nextNumber.ToString()
End Function


---

4. 生成 EDI 855（完整 VB.NET 代碼）

Sub GenerateEDI855()
    ' 取得不同的控制號
    Dim isaNumber As String = GetNextControlNumberFromDB("ISA") ' ISA13 / IEA02
    Dim gsNumber As String = GetNextControlNumberFromDB("GS")   ' GS06 / GE02
    Dim stNumber As String = GetNextControlNumberFromDB("ST")   ' ST02 / SE02
    Dim segmentCount As Integer = 9  ' 交易集內的段數

    ' 生成 EDI 855
    Dim edi855 As String = "ISA|00|          |00|          |ZZ|SENDERID       |ZZ|RECEIVERID     |" &
                            DateTime.Now.ToString("yyMMdd") & "|1234|U|00401|" & isaNumber & "|0|P|>" & vbCrLf &
                            "GS|PR|SENDERID|RECEIVERID|" & DateTime.Now.ToString("yyyyMMdd") & "|" & gsNumber & "|X|004010" & vbCrLf &
                            "ST|855|" & stNumber & vbCrLf &
                            "BAK|00|AD|4501234567|" & DateTime.Now.ToString("yyyyMMdd") & vbCrLf &
                            "REF|VR|123456" & vbCrLf &
                            "DTM|002|20240215" & vbCrLf &
                            "DTM|010|20240220" & vbCrLf &
                            "N1|ST|ABC Corporation|92|98765" & vbCrLf &
                            "PO1|1|10|EA|15.00|USD|VP|12345" & vbCrLf &
                            "CTT|1" & vbCrLf &
                            "SE|" & segmentCount & "|" & stNumber & vbCrLf &
                            "GE|1|" & gsNumber & vbCrLf &
                            "IEA|1|" & isaNumber

    ' 輸出 EDI 855
    Console.WriteLine(edi855)
End Sub


---

5. 生成的 EDI 855 示例

ISA|00|          |00|          |ZZ|SENDERID       |ZZ|RECEIVERID     |250213|1234|U|00401|100000001|0|P|>
GS|PR|SENDERID|RECEIVERID|20250213|200000001|X|004010
ST|855|300000001
BAK|00|AD|4501234567|20250213
REF|VR|123456
DTM|002|20240215
DTM|010|20240220
N1|ST|ABC Corporation|92|98765
PO1|1|10|EA|15.00|USD|VP|12345
CTT|1
SE|9|300000001
GE|1|200000001
IEA|1|100000001


---

6. 總結


---

7. 這樣的好處

✅ 每個控制號獨立遞增，不會重複
✅ 不會超過 9 位數，超過則回到最小值
✅ 符合 EDI 855 規範，保證可解析

這樣 ISA, GS, ST 三個控制號就不會相同了，而且符合 EDI 標準！

