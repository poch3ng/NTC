以下提供一個 VB.Net（WebForm 或 WinForm 均可）簡化範例，示範如何將資料庫中已儲存的資料（假設為 EDI 855 的 Header/Detail 資料）組裝成一個 X12 EDI 855 檔案。
使用「*」作為資料元素分隔符號，並以「~」作為段落結束符號。
實務上，你應根據實際資料表結構、欄位命名、交易夥伴之 Implementation Guide 做進一步調整。


---

1. 資料庫結構 (範例)

假設我們有以下兩張表：

EDI855Header (存放 855 Header 資訊: ISA、GS、ST、BAK、PO Number 等)

CREATE TABLE [dbo].[EDI855Header] (
    [Id] INT IDENTITY(1,1) PRIMARY KEY,
    [ISA13]  VARCHAR(9),     -- ISA控制號(示例)
    [GS06]   VARCHAR(9),     -- GS控制號(示例)
    [ST02]   VARCHAR(9),     -- ST控制號(示例)
    [BAK01]  VARCHAR(2),
    [BAK02]  VARCHAR(30),
    [BAK03]  VARCHAR(8),
    [BAK04]  VARCHAR(20),    -- 通常對應 850 的 PO Number
    [PODate] VARCHAR(8),     -- 訂單日期(格式 YYYYMMDD)
    -- ...可再加入其他欄位(如 交易夥伴資訊等)
    [CreateTime] DATETIME NOT NULL DEFAULT(GETDATE())
);

EDI855Detail (存放多筆行項的 PO1、ACK 資訊)

CREATE TABLE [dbo].[EDI855Detail] (
    [Id] INT IDENTITY(1,1) PRIMARY KEY,
    [HeaderId]   INT NOT NULL,      -- 對應 EDI855Header.Id
    [PO1_01]     INT,               -- 行項序號
    [PO1_02]     INT,               -- 數量
    [PO1_03]     VARCHAR(3),        -- 單位
    [PO1_04]     DECIMAL(18,2),     -- 單價
    [PO1_07]     VARCHAR(50),       -- 料號(或PartNo)
    [ACK01]      VARCHAR(2),        -- Item Accepted / Rejected / etc.
    [ACK02]      INT,               -- ACK 數量
    [ACK03]      VARCHAR(3),        -- ACK 單位
    [ACK04]      VARCHAR(8),        -- ACK 日期(YYYYMMDD)等
    -- ...可再加入其他欄位(如交期、備註)
    [CreateTime] DATETIME NOT NULL DEFAULT(GETDATE())
);

ALTER TABLE [dbo].[EDI855Detail]
ADD CONSTRAINT FK_EDI855Detail_Header
FOREIGN KEY (HeaderId) REFERENCES EDI855Header(Id);



---

2. 建立程式將資料組裝成 EDI 855 檔案

以下程式碼：

1. 從資料庫讀取特定 EDI855Header 與 EDI855Detail。


2. 根據常見的 X12 855 段落（ISA、GS、ST、BAK、PO1、ACK、CTT、SE、GE、IEA），以 * 分隔、以 ~ 結尾，拼裝成 純文字。


3. 將產生的 EDI 文字檔案內容儲存到指定路徑。



> 注意：此範例僅示範常見欄位與段落的產生方式，實務中還需依你的 Implementation Guide 完整填寫。



Imports System.Data
Imports System.Data.SqlClient
Imports System.IO

Public Class EDI855Generator
    ' 你可在 WebForm/WinForm 中呼叫本函式，或直接改成 Page_Load / 按鈕事件
    Public Sub GenerateEDI855File(headerId As Integer, outputFilePath As String)
        Dim connStr As String = "Server=YOUR_SERVER;Database=YOUR_DB;User ID=YOUR_USER;Password=YOUR_PASS;"
        
        ' 1) 讀取 EDI855Header
        Dim headerRow As DataRow = Nothing
        Dim dtHeader As New DataTable()
        Using conn As New SqlConnection(connStr)
            conn.Open()

            Dim sqlHeader As String = "SELECT * FROM EDI855Header WHERE Id = @Id"
            Using da As New SqlDataAdapter(sqlHeader, conn)
                da.SelectCommand.Parameters.AddWithValue("@Id", headerId)
                da.Fill(dtHeader)
            End Using

            If dtHeader.Rows.Count = 0 Then
                Throw New Exception("找不到指定的 EDI855 Header 資料 (Id=" & headerId & ")。")
            End If
            headerRow = dtHeader.Rows(0)

            ' 2) 讀取 EDI855Detail (多筆)
            Dim dtDetail As New DataTable()
            Dim sqlDetail As String = "SELECT * FROM EDI855Detail WHERE HeaderId = @HeaderId ORDER BY PO1_01"
            Using da2 As New SqlDataAdapter(sqlDetail, conn)
                da2.SelectCommand.Parameters.AddWithValue("@HeaderId", headerId)
                da2.Fill(dtDetail)
            End Using

            ' -- 3) 開始組裝 EDI 855 字串 --
            Dim ediLines As New List(Of String)()

            ' ============= ISA segment =============
            ' 常見欄位：ISA01, ISA02, ISA03, ISA04, ISA05, ISA06, ISA07, ISA08, ISA09, ISA10, ISA11, ISA12, ISA13, ISA14, ISA15, ISA16
            ' 以下僅以簡單範例給固定值，實務中請依協議或資料庫欄位填入
            Dim ISA13 As String = headerRow("ISA13").ToString().PadLeft(9, "0"c)  ' 控制號(左側補0到9位)
            Dim isaSegment As String = String.Join("*", {
                "ISA",
                "00",                     ' ISA01
                "".PadRight(10),         ' ISA02 (10空白)
                "00",                     ' ISA03
                "".PadRight(10),         ' ISA04 (10空白)
                "01",                     ' ISA05 (Qualifier)
                "SENDERID      ".Substring(0, 15), ' ISA06 (15位)
                "01",                     ' ISA07 (Qualifier)
                "RECEIVERID    ".Substring(0, 15), ' ISA08 (15位)
                Now.ToString("yyMMdd"),  ' ISA09 (日期)
                Now.ToString("HHmm"),    ' ISA10 (時間)
                "U",                     ' ISA11
                "00401",                 ' ISA12 (Version)
                ISA13,                   ' ISA13 (Control Number)
                "0",                     ' ISA14 (Ack Requested)
                "P",                     ' ISA15 (Usage Indicator: P=Production/T=Test)
                ">"                      ' ISA16 (Component Element Separator)
            }) & "~"
            ediLines.Add(isaSegment)

            ' ============= GS segment =============
            ' GS01, GS02, GS03, GS04, GS05, GS06, GS07, GS08
            Dim GS06 As String = headerRow("GS06").ToString()
            Dim gsSegment As String = String.Join("*", {
                "GS",
                "PR",                        ' GS01 (Functional ID for 855)
                "SENDERGSID",                ' GS02
                "RECEIVERGSID",              ' GS03
                Now.ToString("yyyyMMdd"),    ' GS04 (Date)
                Now.ToString("HHmm"),        ' GS05 (Time)
                GS06,                        ' GS06 (Group Control Number)
                "X",                         ' GS07
                "004010"                     ' GS08 (Version/Release)
            }) & "~"
            ediLines.Add(gsSegment)

            ' ============= ST segment =============
            Dim ST02 As String = headerRow("ST02").ToString()
            Dim stSegment As String = String.Join("*", {
                "ST",
                "855",       ' ST01 - 855 transaction
                ST02         ' ST02 - Transaction Set Control Number
            }) & "~"
            ediLines.Add(stSegment)

            ' ============= BAK segment =============
            ' 常見欄位：BAK01, BAK02, BAK03(PO日期), BAK04(PO編號)
            Dim bak01 As String = headerRow("BAK01").ToString()
            Dim bak02 As String = headerRow("BAK02").ToString()
            Dim bak03 As String = headerRow("BAK03").ToString()  ' YYYYMMDD
            Dim bak04 As String = headerRow("BAK04").ToString()  ' PO Number
            Dim bakSegment As String = String.Join("*", {
                "BAK",
                bak01,
                bak02,
                bak03,
                bak04
            }) & "~"
            ediLines.Add(bakSegment)

            ' (若有 REF 或 N1 段，也可在這裡組裝)

            ' ============= PO1 & ACK segments =============
            ' 迭代每一筆行項，分別加入 PO1、ACK 段
            Dim lineCount As Integer = 0
            For Each dr As DataRow In dtDetail.Rows
                lineCount += 1

                ' --- PO1 ---
                ' 常見欄位：PO1_01=行項序號, PO1_02=數量, PO1_03=單位, PO1_04=單價, PO1_07=料號等
                Dim po1_01 As String = dr("PO1_01").ToString()
                Dim po1_02 As String = dr("PO1_02").ToString()
                Dim po1_03 As String = dr("PO1_03").ToString()
                Dim po1_04 As String = dr("PO1_04").ToString()
                Dim po1_07 As String = dr("PO1_07").ToString()
                Dim po1Segment As String = String.Join("*", {
                    "PO1",
                    po1_01,
                    po1_02,
                    po1_03,
                    po1_04,
                    "", "",  ' PO1_05, PO1_06 (若不用可留空)
                    po1_07
                }) & "~"
                ediLines.Add(po1Segment)

                ' --- ACK ---
                ' 常見欄位：ACK01=狀態, ACK02=數量, ACK03=單位, ACK04=日期(YYYYMMDD) ...
                Dim ack01 As String = dr("ACK01").ToString()
                Dim ack02 As String = dr("ACK02").ToString()
                Dim ack03 As String = dr("ACK03").ToString()
                Dim ack04 As String = dr("ACK04").ToString()  ' YYYYMMDD
                Dim ackSegment As String = String.Join("*", {
                    "ACK",
                    ack01,
                    ack02,
                    ack03,
                    ack04
                }) & "~"
                ediLines.Add(ackSegment)
            Next

            ' ============= CTT segment (行數總計) =============
            Dim cttSegment As String = String.Join("*", {
                "CTT",
                lineCount.ToString()
            }) & "~"
            ediLines.Add(cttSegment)

            ' ============= SE segment (Transaction Set Trailer) =============
            ' SE01 = Segment Count (ST,BAK,PO1,ACK,CTT,SE ...等所有段的數量)
            ' SE02 = ST02 (與 ST 段對應)
            Dim segmentCount As Integer = ediLines.Count + 1 ' +1 是因為 SE 自己也算一段
            Dim seSegment As String = String.Join("*", {
                "SE",
                segmentCount.ToString(),
                ST02
            }) & "~"
            ediLines.Add(seSegment)

            ' ============= GE segment (Functional Group Trailer) =============
            ' GE01 = # of Transaction Sets Included
            ' GE02 = GS06
            Dim geSegment As String = String.Join("*", {
                "GE",
                "1",    ' 本範例只有 1 個 ST
                GS06
            }) & "~"
            ediLines.Add(geSegment)

            ' ============= IEA segment (Interchange Control Trailer) =============
            ' IEA01 = # of Functional Groups Included
            ' IEA02 = ISA13
            Dim ieaSegment As String = String.Join("*", {
                "IEA",
                "1",     ' 1 個 GS
                ISA13
            }) & "~"
            ediLines.Add(ieaSegment)

            ' 4) 將組好的 EDI 文字內容存檔
            Dim ediContent As String = String.Join("", ediLines)
            File.WriteAllText(outputFilePath, ediContent)

        End Using
    End Sub

    ' (如果需要在 UI 上直接顯示或下載，也可改用 Response.Write() / Response.ContentType 等做法)
End Class


---

3. 使用方式示例

在你的 VB WebForm (或 WinForm) 中，新建或呼叫 EDI855Generator 類別。

傳入要生成的 HeaderId 及輸出檔案路徑，例如：


Protected Sub btnExport_Click(sender As Object, e As EventArgs) Handles btnExport.Click
    Dim headerId As Integer = 123  ' 假設要匯出 Id=123 的那筆 EDI855 資料
    Dim outputPath As String = Server.MapPath("~/EDI_Output/EDI855_123.txt") ' WebForm 範例

    Dim generator As New EDI855Generator()
    generator.GenerateEDI855File(headerId, outputPath)

    Response.Write("<script>alert('EDI855 檔案已產生，路徑：" & outputPath & "');</script>")
End Sub


---

4. 重點整理

1. Segment 與分隔符號

每段資料以「~」結束。

段落內以「*」分隔資料元素。

程式上就是不斷用 String.Join("*", {...}) & "~" 來拼裝每個段落。



2. 資料來源

本範例直接從 EDI855Header、EDI855Detail 取得欄位來組裝。

若你的系統裡只有 EDI 850 的資料，需要先將其轉換/插入到 855 的表或暫存表，再組裝成 855。



3. 欄位對應

例如：ISA13、GS06、ST02 通常是你的系統對 855 交易產生的控制號，用以區分每筆交易。

BAK04 通常填入對應 850 的 PO Number；ACK 段放每個 Line Item 的確認結果。

可根據 Implementation Guide 補充或修改段落(REF、N1、DTM...)。



4. 段落計數

SE01 需包含整個 Transaction Set 的段數（含 ST 與 SE 本身），在程式組裝時可先把所有段裝入清單，再計算總數。



5. 實際測試

產生後的文字檔可用 EDI 檢驗工具（如 bots、GEIS、對方指定工具）測試是否符合 ANSI X12 855 標準。





---

結論
以上程式說明了如何從資料庫取出已存的 855 資訊，拼裝成標準的 X12 格式檔案 (* 分隔, ~ 結尾)。你可依業務需求、交易夥伴規範做更進一步的欄位與段落設定，即可完成 EDI 855 的自動產生。祝專案順利!

