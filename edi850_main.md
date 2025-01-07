以下是將 EDI 850 段落（如 ISA、GS、ST 等）寫入資料庫的完整 VB.NET 程式碼，包含儲存到資料表 EdiMain、EdiDetail 和 EdiDetailItem 的實作。


---

資料表設計

假設以下資料表結構：

1. EdiMain：儲存 ISA 和 GS 資訊。

CREATE TABLE EdiMain (
    ID INT IDENTITY PRIMARY KEY,
    ISA01 NVARCHAR(10),
    ISA02 NVARCHAR(10),
    ISA03 NVARCHAR(10),
    ISA04 NVARCHAR(10),
    ISA05 NVARCHAR(10),
    ISA06 NVARCHAR(15),
    ISA07 NVARCHAR(10),
    ISA08 NVARCHAR(15),
    ISA09 NVARCHAR(6),
    ISA10 NVARCHAR(4),
    ISA11 NVARCHAR(1),
    ISA12 NVARCHAR(5),
    ISA13 NVARCHAR(9),
    ISA14 NVARCHAR(1),
    ISA15 NVARCHAR(1),
    ISA16 NVARCHAR(1),
    GS01 NVARCHAR(5),
    GS02 NVARCHAR(15),
    GS03 NVARCHAR(15),
    GS04 NVARCHAR(8),
    GS05 NVARCHAR(4),
    GS06 NVARCHAR(5),
    GS07 NVARCHAR(1),
    GS08 NVARCHAR(6)
);


2. EdiDetail：儲存 ST 和 BEG 資訊。

CREATE TABLE EdiDetail (
    ID INT IDENTITY PRIMARY KEY,
    MainID INT,
    ST01 NVARCHAR(3),
    ST02 NVARCHAR(9),
    BEG01 NVARCHAR(2),
    BEG02 NVARCHAR(2),
    BEG03 NVARCHAR(20),
    BEG05 NVARCHAR(8),
    FOREIGN KEY (MainID) REFERENCES EdiMain(ID)
);


3. EdiDetailItem：儲存 PO1 資訊。

CREATE TABLE EdiDetailItem (
    ID INT IDENTITY PRIMARY KEY,
    DetailID INT,
    PO101 NVARCHAR(5),
    PO102 INT,
    PO103 NVARCHAR(2),
    PO104 DECIMAL(10, 2),
    PO106 NVARCHAR(5),
    PO107 NVARCHAR(20),
    PO109 NVARCHAR(15),
    FOREIGN KEY (DetailID) REFERENCES EdiDetail(ID)
);




---

程式碼實作

1. 資料庫連線字串

設定資料庫連線：

Dim connectionString As String = "Server=YourServerName;Database=YourDatabaseName;User Id=YourUsername;Password=YourPassword;"
Dim connection As New SqlClient.SqlConnection(connectionString)

2. 插入主檔資訊 (ISA 和 GS)

Function InsertEdiMain(isaFields As String(), gsFields As String()) As Integer
    Dim query As String = "
        INSERT INTO EdiMain (
            ISA01, ISA02, ISA03, ISA04, ISA05, ISA06, ISA07, ISA08, ISA09, ISA10,
            ISA11, ISA12, ISA13, ISA14, ISA15, ISA16,
            GS01, GS02, GS03, GS04, GS05, GS06, GS07, GS08
        ) VALUES (
            @ISA01, @ISA02, @ISA03, @ISA04, @ISA05, @ISA06, @ISA07, @ISA08, @ISA09, @ISA10,
            @ISA11, @ISA12, @ISA13, @ISA14, @ISA15, @ISA16,
            @GS01, @GS02, @GS03, @GS04, @GS05, @GS06, @GS07, @GS08
        );
        SELECT SCOPE_IDENTITY();
    "

    Dim command As New SqlClient.SqlCommand(query, connection)
    For i As Integer = 0 To 15
        command.Parameters.AddWithValue($"@ISA{i + 1:00}", isaFields(i))
    Next

    For i As Integer = 0 To 7
        command.Parameters.AddWithValue($"@GS{i + 1:00}", gsFields(i))
    Next

    connection.Open()
    Dim mainId As Integer = Convert.ToInt32(command.ExecuteScalar())
    connection.Close()

    Return mainId
End Function


---

3. 插入交易明細 (ST 和 BEG)

Function InsertEdiDetail(mainId As Integer, stFields As String(), begFields As String()) As Integer
    Dim query As String = "
        INSERT INTO EdiDetail (
            MainID, ST01, ST02, BEG01, BEG02, BEG03, BEG05
        ) VALUES (
            @MainID, @ST01, @ST02, @BEG01, @BEG02, @BEG03, @BEG05
        );
        SELECT SCOPE_IDENTITY();
    "

    Dim command As New SqlClient.SqlCommand(query, connection)
    command.Parameters.AddWithValue("@MainID", mainId)
    command.Parameters.AddWithValue("@ST01", stFields(0))
    command.Parameters.AddWithValue("@ST02", stFields(1))
    command.Parameters.AddWithValue("@BEG01", begFields(0))
    command.Parameters.AddWithValue("@BEG02", begFields(1))
    command.Parameters.AddWithValue("@BEG03", begFields(2))
    command.Parameters.AddWithValue("@BEG05", begFields(4))

    connection.Open()
    Dim detailId As Integer = Convert.ToInt32(command.ExecuteScalar())
    connection.Close()

    Return detailId
End Function


---

4. 插入商品明細 (PO1)

Sub InsertEdiDetailItem(detailId As Integer, po1Fields As String())
    Dim query As String = "
        INSERT INTO EdiDetailItem (
            DetailID, PO101, PO102, PO103, PO104, PO106, PO107, PO109
        ) VALUES (
            @DetailID, @PO101, @PO102, @PO103, @PO104, @PO106, @PO107, @PO109
        )
    "

    Dim command As New SqlClient.SqlCommand(query, connection)
    command.Parameters.AddWithValue("@DetailID", detailId)
    command.Parameters.AddWithValue("@PO101", po1Fields(0))
    command.Parameters.AddWithValue("@PO102", Convert.ToInt32(po1Fields(1)))
    command.Parameters.AddWithValue("@PO103", po1Fields(2))
    command.Parameters.AddWithValue("@PO104", Convert.ToDecimal(po1Fields(3)))
    command.Parameters.AddWithValue("@PO106", po1Fields(5))
    command.Parameters.AddWithValue("@PO107", po1Fields(6))
    command.Parameters.AddWithValue("@PO109", po1Fields(8))

    connection.Open()
    command.ExecuteNonQuery()
    connection.Close()
End Sub


---

5. 整合主流程

Sub ProcessEDI850(filePath As String)
    Dim segments = ParseEDI850(filePath)

    Dim isaFields(15) As String
    Dim gsFields(7) As String
    Dim stFields(1) As String
    Dim begFields(4) As String
    Dim mainId As Integer
    Dim detailId As Integer

    For Each segment As String In segments
        Dim elements = segment.Split("*"c)

        Select Case elements(0)
            Case "ISA"
                Array.Copy(elements, 1, isaFields, 0, 16)
            Case "GS"
                Array.Copy(elements, 1, gsFields, 0, 8)
                mainId = InsertEdiMain(isaFields, gsFields)
            Case "ST"
                Array.Copy(elements, 1, stFields, 0, 2)
            Case "BEG"
                Array.Copy(elements, 1, begFields, 0, 5)
                detailId = InsertEdiDetail(mainId, stFields, begFields)
            Case "PO1"
                InsertEdiDetailItem(detailId, elements.Skip(1).ToArray())
        End Select
    Next
End Sub


---

完整執行範例

呼叫主流程：

ProcessEDI850("path_to_edi_file.edi")

該程式會依照段落逐一解析並將資料存入 EdiMain、EdiDetail 和 EdiDetailItem 資料表。


---

測試

測試用的 EDI 850 範例文件可以參考我先前提供的範例。

