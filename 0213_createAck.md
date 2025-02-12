以下範例示範在已知 EDI 850 資料的情況下，如何將其轉換並插入到「含 PO1、ACK 段」的 EDI 855 資料表中。程式以 ASP.NET VB WebForm 為例，重點是說明欄位對應與多筆細項（PO1, ACK）的處理流程。實際實作時，請依照交易夥伴規範(Implementation Guide)與資料庫設計來做調整。


---

1. 資料表設計範例

假設你想將整張 EDI 855 放在一張資料表 EDI855Header，再用一張 EDI855Detail 來存放多筆 PO1/ACK 資訊（因為一張 PO 可能有多個 Line Item），結構範例如下：

(1) EDI855Header

CREATE TABLE [dbo].[EDI855Header] (
    [Id]              INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    -- ISA / GS / ST / BAK 相關欄位(略)
    [ISA13]           VARCHAR(9),      -- Control Number
    [GS06]            VARCHAR(9),      -- Group Control Number
    [ST02]            VARCHAR(9),      -- ST Control Number
    [BAK01]           VARCHAR(2),
    [BAK02]           VARCHAR(30),
    [BAK03]           VARCHAR(8),
    [BAK04]           VARCHAR(20),     -- 這裡通常對應 EDI850 的 PO Number
    
    -- 其他 Header 級欄位(交易夥伴資訊等) 自行補充
    [CreateTime]      DATETIME NOT NULL DEFAULT(GETDATE())
);

(2) EDI855Detail

CREATE TABLE [dbo].[EDI855Detail] (
    [Id]          INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    [HeaderId]    INT NOT NULL,              -- 對應 EDI855Header.Id
    
    -- PO1 段常見欄位：
    [PO1_01]      INT,                       -- Line Item 序號
    [PO1_02]      INT,                       -- 數量
    [PO1_03]      VARCHAR(3),                -- 單位(如 EA)
    [PO1_04]      DECIMAL(18,2),             -- 單價
    [PO1_07]      VARCHAR(50),               -- 料號(可放 NTCPartNo/AWSPartNo)
    
    -- ACK 段常見欄位：用來確認每個 Line Item 的狀態或異動
    [ACK01]       VARCHAR(2),                -- Line Item Status Code (IA/IE/IR等)
    [ACK02]       INT,                       -- ACK 數量
    [ACK03]       VARCHAR(3),                -- ACK 數量的單位
    [ACK04]       DATETIME,                  -- 預計交期或其他日期
    
    -- 其他欄位(如需求日期、備註等) 視實際需求擴充
    [CreateTime]  DATETIME NOT NULL DEFAULT(GETDATE())
);

-- Foreign Key 連結
ALTER TABLE [dbo].[EDI855Detail]
ADD CONSTRAINT FK_EDI855Detail_Header 
FOREIGN KEY (HeaderId) REFERENCES EDI855Header(Id);

> 為了對應 EDI 850 裡多筆 Line Item，通常使用「主表 (Header) + 明細表(Detail)」的設計會較好維護。




---

2. 從 EDI 850 資料表產生 EDI 855 (範例程式)

以下程式假設你有一張 EDI850Header 和一張 EDI850Detail，裡面包含需要的資訊(例如 PO Number, itemNo, qty, price, 交期等)。按下「產生 855」按鈕後：

1. 先取得 EDI850 的 Header 資訊。


2. 建立對應的 EDI855Header 記錄 (ISA13, GS06, ST02, BAK04等)。


3. 再取得對應的 EDI850Detail 裡的多筆細項，逐筆產生 EDI855Detail(包含 PO1、ACK 欄位)。



> 備註：程式中的 ISA13/GS06/ST02 等控制號或狀態碼(ACK01) 皆為示範，實際請依照你的需求與 EDI 規範賦值。




---

2.1  WebForm 介面 (EDI855Form.aspx)

<%@ Page Language="VB" AutoEventWireup="true" CodeFile="EDI855Form.aspx.vb" Inherits="EDI855Form" %>
<!DOCTYPE html>
<html>
<head runat="server">
    <title>產生 EDI855</title>
</head>
<body>
    <form id="form1" runat="server">
        <div>
            <!-- 可以放條件(比如輸入 PO Number) -->
            <label for="txtPONumber">PO Number：</label>
            <asp:TextBox ID="txtPONumber" runat="server"></asp:TextBox>
            <br /><br />
            <asp:Button ID="btnGenerate855" runat="server" Text="產生 EDI 855" />
        </div>
    </form>
</body>
</html>


---

2.2 後端程式 (EDI855Form.aspx.vb)

Imports System.Data
Imports System.Data.SqlClient

Partial Class EDI855Form
    Inherits System.Web.UI.Page

    Protected Sub btnGenerate855_Click(sender As Object, e As EventArgs) Handles btnGenerate855.Click
        '=== 1. 取得用戶輸入的 PO Number，用來查 EDI850 資料 ===
        Dim inputPONo As String = txtPONumber.Text.Trim()
        If String.IsNullOrEmpty(inputPONo) Then
            Response.Write("<script>alert('請輸入 PO Number');</script>")
            Return
        End If

        '=== 2. 查詢 EDI850Header ===
        Dim connStr As String = "Server=YOUR_SERVER;Database=YOUR_DB;User ID=YOUR_USER;Password=YOUR_PASS;"
        Dim dtHeader As New DataTable()

        Using conn As New SqlConnection(connStr)
            conn.Open()

            Dim sqlHeader As String = "SELECT TOP 1 * FROM EDI850Header " &
                                      "WHERE PONumber = @PONo"
            Using da As New SqlDataAdapter(sqlHeader, conn)
                da.SelectCommand.Parameters.AddWithValue("@PONo", inputPONo)
                da.Fill(dtHeader)
            End Using

            ' 如果查不到對應 850 Header，就提示
            If dtHeader.Rows.Count = 0 Then
                Response.Write("<script>alert('查無對應的 EDI850 Header 資料');</script>")
                Return
            End If

            '=== 3. 插入 EDI855Header ===
            Dim rowH As DataRow = dtHeader.Rows(0)

            ' 以下欄位以示範為主，實務可依需求或協議
            Dim newISA13 As String = GenerateControlNumber() ' 產生新的控制號
            Dim newGS06 As String = GenerateControlNumber()
            Dim newST02 As String = GenerateControlNumber()

            Dim bak01 As String = "AC" ' Acknowledge
            Dim bak02 As String = "123456"
            Dim bak03 As String = DateTime.Now.ToString("yyyyMMdd") ' 回覆日期
            Dim bak04 As String = rowH("PONumber").ToString()       ' 對應 EDI850

            ' 建立 Insert 語法
            Dim sqlInsert855Header As String =
                "INSERT INTO EDI855Header (ISA13, GS06, ST02, BAK01, BAK02, BAK03, BAK04) " &
                "OUTPUT INSERTED.Id " &  ' 取得剛剛插入的主鍵
                "VALUES (@ISA13, @GS06, @ST02, @BAK01, @BAK02, @BAK03, @BAK04)"

            Dim new855HeaderId As Integer
            Using cmd As New SqlCommand(sqlInsert855Header, conn)
                cmd.Parameters.AddWithValue("@ISA13", newISA13)
                cmd.Parameters.AddWithValue("@GS06", newGS06)
                cmd.Parameters.AddWithValue("@ST02", newST02)
                cmd.Parameters.AddWithValue("@BAK01", bak01)
                cmd.Parameters.AddWithValue("@BAK02", bak02)
                cmd.Parameters.AddWithValue("@BAK03", bak03)
                cmd.Parameters.AddWithValue("@BAK04", bak04)

                new855HeaderId = Convert.ToInt32(cmd.ExecuteScalar())
            End Using

            '=== 4. 查詢 EDI850Detail (多筆 item) ===
            Dim dtDetail As New DataTable()
            Dim sqlDetail As String = "SELECT * FROM EDI850Detail WHERE HeaderId = @HeaderId"
            Using da2 As New SqlDataAdapter(sqlDetail, conn)
                da2.SelectCommand.Parameters.AddWithValue("@HeaderId", rowH("Id")) ' EDI850Header裡的主鍵
                da2.Fill(dtDetail)
            End Using

            '=== 5. 將多筆 EDI850Detail 轉成 EDI855Detail (PO1 + ACK) ===
            For Each rowD As DataRow In dtDetail.Rows
                ' PO1
                Dim po1_01 As Integer = Convert.ToInt32(rowD("LineNo")) ' 假設 EDI850Detail 有行號欄位
                Dim po1_02 As Integer = Convert.ToInt32(rowD("Qty"))
                Dim po1_03 As String = "EA"
                Dim po1_04 As Decimal = Convert.ToDecimal(rowD("Price"))
                Dim po1_07 As String = rowD("ItemNo").ToString() ' 或 NTCPartNo/AWSPartNo
                
                ' ACK (示範：全數接受)
                Dim ack01 As String = "IA"  ' Item Accepted
                Dim ack02 As Integer = po1_02
                Dim ack03 As String = "EA"
                Dim ack04 As DateTime = Convert.ToDateTime(rowD("DeliveryDate")) ' ACK 裡可放交期

                ' Insert 到 EDI855Detail
                Dim sqlInsert855Detail As String =
                    "INSERT INTO EDI855Detail " &
                    "(HeaderId, PO1_01, PO1_02, PO1_03, PO1_04, PO1_07, ACK01, ACK02, ACK03, ACK04) " &
                    "VALUES " &
                    "(@HeaderId, @PO1_01, @PO1_02, @PO1_03, @PO1_04, @PO1_07, @ACK01, @ACK02, @ACK03, @ACK04)"

                Using cmd2 As New SqlCommand(sqlInsert855Detail, conn)
                    cmd2.Parameters.AddWithValue("@HeaderId", new855HeaderId)
                    cmd2.Parameters.AddWithValue("@PO1_01", po1_01)
                    cmd2.Parameters.AddWithValue("@PO1_02", po1_02)
                    cmd2.Parameters.AddWithValue("@PO1_03", po1_03)
                    cmd2.Parameters.AddWithValue("@PO1_04", po1_04)
                    cmd2.Parameters.AddWithValue("@PO1_07", po1_07)
                    cmd2.Parameters.AddWithValue("@ACK01", ack01)
                    cmd2.Parameters.AddWithValue("@ACK02", ack02)
                    cmd2.Parameters.AddWithValue("@ACK03", ack03)
                    cmd2.Parameters.AddWithValue("@ACK04", ack04)
                    cmd2.ExecuteNonQuery()
                End Using
            Next

            conn.Close()
        End Using

        Response.Write("<script>alert('EDI 855 (含PO1/ACK) 已產生完畢');</script>")
    End Sub

    ' 簡易產生 ControlNumber 的函式 (示範用時間戳)
    Private Function GenerateControlNumber() As String
        Return DateTime.Now.ToString("HHmmssfff")
    End Function

End Class


---

3. 說明重點

1. Header / Detail 分開

EDI 855 最常見的結構就是：Header 存放 ISA/GS/ST/BAK 等整筆交易的資訊，Detail 則依據每個 Line Item 建立多筆 PO1、ACK 等資訊。

若你的實際需求是放在同一張表，也可以把 PO1_01, ACK01 之類的欄位加在一起，但不利於管理多筆細項。



2. 對應關係

EDI850Header -> EDI850Detail = 1對多

EDI855Header -> EDI855Detail = 1對多

用外鍵 (FK) 來關聯 EDI855Header.Id 與 EDI855Detail.HeaderId。



3. PO1 與 ACK

PO1 段：通常放行項序號(PO1_01)、數量(PO1_02)、單位(PO1_03)、單價(PO1_04)、料號(PO1_07)等。

ACK 段：對每個行項進行「接受/拒絕/部分接受」等確認(ACK01)，以及確認數量(ACK02)、單位(ACK03)、交期(ACK04)等。

若實際使用 ACK01 = “IA” (Item Accepted)，“IR” (Item Rejected)，或 “IP” (Item Accepted – Price Change) 等狀態碼，請依交易夥伴要求。



4. 產生控制號

ISA13, GS06, ST02 通常都要有新的唯一控制號，以利追蹤這筆 855 交易。

可以透過序號管理表或用時間戳方式暫時處理（範例中以 GenerateControlNumber() 取代）。



5. 後續產生 EDI 檔

如果要真正輸出「X12 檔案格式」，可再由 EDI855Header & EDI855Detail 拼裝段落(ISA|GS|ST|...|SE|GE|IEA)。

這裡只示範將資料「存入資料庫」。





---

簡結

核心做法是：先將 EDI 850 的欄位(PO Number, ItemNo, Qty, 交期…) 與 855 需要的段(PO1, ACK)對應起來，並分成「Header 表 + Detail 表」存放。

以上範例示範最基本的 CRUD 流程，請依實際欄位命名、資料型別、業務邏輯加以修改。


這樣就能讓你在程式中完成「將原始 EDI 850 多筆行項資料，轉為含 PO1、ACK 段的 EDI 855 資料表」的流程。祝一切順利完成!

