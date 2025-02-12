以下提供一個簡化的 VB WebForm 範例，示範如何將「已存在資料庫中的 EDI 850 欄位資料」（例如：ISA13, PONo, itemNo, NTCPartNo 等），轉換並插入到新的「EDI 855 欄位」(ISA01, ISA02, …, BAK01, BAK02…等) 的資料表中。請依實際需求調整欄位與邏輯。


---

1. 建立 EDI855Segments 資料表 (SQL 範例)

以下為可能的結構，實際可依你們的 Implementation Guide 新增或刪除欄位。

CREATE TABLE [dbo].[EDI855Segments] (
    [Id]       INT IDENTITY(1,1) NOT NULL PRIMARY KEY,
    -- ISA 區段（範例）
    [ISA01]    VARCHAR(2),
    [ISA02]    VARCHAR(10),
    [ISA03]    VARCHAR(2),
    [ISA04]    VARCHAR(10),
    [ISA05]    VARCHAR(2),
    [ISA06]    VARCHAR(15),
    [ISA07]    VARCHAR(2),
    [ISA08]    VARCHAR(15),
    [ISA09]    VARCHAR(6),
    [ISA10]    VARCHAR(6),
    [ISA11]    VARCHAR(1),
    [ISA12]    VARCHAR(5),
    [ISA13]    VARCHAR(9),  -- 與 EDI 850 對應但需新值
    [ISA14]    VARCHAR(1),
    [ISA15]    VARCHAR(1),
    [ISA16]    VARCHAR(1),

    -- GS 區段（範例）
    [GS01]     VARCHAR(2),
    [GS02]     VARCHAR(15),
    [GS03]     VARCHAR(15),
    [GS04]     VARCHAR(8),
    [GS05]     VARCHAR(8),
    [GS06]     VARCHAR(9),
    [GS07]     VARCHAR(2),
    [GS08]     VARCHAR(2),

    -- ST 區段（範例）
    [ST01]     VARCHAR(3),
    [ST02]     VARCHAR(9),

    -- BAK 區段（範例）
    [BAK01]    VARCHAR(2),
    [BAK02]    VARCHAR(30),
    [BAK03]    VARCHAR(8),
    [BAK04]    VARCHAR(20),

    -- 其他欄位（對應 EDI 850 的 itemNo, price, qty, date 等，以下僅示意）
    [ItemNo]        VARCHAR(50),
    [NTCPartNo]     VARCHAR(50),
    [AWSPartNo]     VARCHAR(50),
    [Price]         DECIMAL(18,2),
    [Qty]           INT,
    [DeliveryDate]  DATETIME,
    [RequestDate]   DATETIME
);

> 注意：上列只是常見欄位示例，你也可根據實際使用情況取捨；ISA 區段有 16 欄，GS 區段一般有 8 欄，ST/BAK 等欄位依照 EDI 855 結構擴增即可。




---

2. 從 EDI850 資料表讀取資料，並插入到 EDI855Segments

假設你的「原始 EDI 850 資料表」名為 EDI850Data，裡面已存在欄位(ISA13, PONo, ItemNo, …)。
以下程式碼在按下某個「產生 EDI855」按鈕後，

從 EDI850Data 讀取資料

填入 855 需要的 ISA01, ISA02, …, BAK04 等欄位

以 INSERT 寫入 EDI855Segments


EDI855Form.aspx

<%@ Page Language="VB" AutoEventWireup="true" CodeFile="EDI855Form.aspx.vb" Inherits="EDI855Form" %>
<!DOCTYPE html>
<html>
<head runat="server">
    <title>產生 EDI 855</title>
</head>
<body>
    <form id="form1" runat="server">
        <div>
            <!-- 這裡可放一些條件輸入，如 PO Number 或查詢範圍，下面只示範按鈕 -->
            <asp:Button ID="btnGenerate855" runat="server" Text="產生 EDI 855" />
        </div>
    </form>
</body>
</html>

EDI855Form.aspx.vb (CodeBehind)

Imports System.Data
Imports System.Data.SqlClient

Partial Class EDI855Form
    Inherits System.Web.UI.Page

    Protected Sub Page_Load(sender As Object, e As EventArgs) Handles Me.Load
        If Not IsPostBack Then
            ' 第一次載入時，如需初始化可寫在這裡
        End If
    End Sub

    Protected Sub btnGenerate855_Click(sender As Object, e As EventArgs) Handles btnGenerate855.Click
        ' 1. 連接資料庫、讀取 EDI850Data
        Dim connStr As String = "Server=YOUR_SERVER;Database=YOUR_DB;User ID=YOUR_USER;Password=YOUR_PASS;"
        Dim dt As New DataTable()

        Using conn As New SqlConnection(connStr)
            conn.Open()

            ' 這裡示範只撈特定狀態的資料，也可依需求加條件
            Dim sqlSelect As String = "SELECT ISA13, PONo, ItemNo, NTCPartNo, AWSPartNo, Price, Qty, DeliveryDate, RequestDate " &
                                      "FROM EDI850Data WHERE SomeCondition = 1"

            Using da As New SqlDataAdapter(sqlSelect, conn)
                da.Fill(dt)  ' 將查出的資料放入 DataTable
            End Using

            ' 2. 對每筆 EDI850 資料進行轉換、插入 EDI855Segments
            For Each row As DataRow In dt.Rows
                ' (a) 準備對應 855 的欄位值
                '     以下示範部分欄位以固定值填入，如 ISA01, ISA02...；實務可從設定檔或參數帶入。

                Dim ISA01 As String = "00"
                Dim ISA02 As String = "          "  ' 10位空白
                Dim ISA03 As String = "00"
                Dim ISA04 As String = "          "  ' 10位空白
                Dim ISA05 As String = "01"
                Dim ISA06 As String = "SENDERID123"
                Dim ISA07 As String = "01"
                Dim ISA08 As String = "RECEIVERID"
                Dim ISA09 As String = DateTime.Now.ToString("yyMMdd")  ' EX: 230215
                Dim ISA10 As String = DateTime.Now.ToString("HHmm")    ' EX: 1530
                Dim ISA11 As String = "U"
                Dim ISA12 As String = "00401"
                Dim ISA13 As String = GenerateControlNumber()  ' 產生新的控制號 (自訂函式)
                Dim ISA14 As String = "0"
                Dim ISA15 As String = "P"
                Dim ISA16 As String = ">"

                Dim GS01 As String = "PR"         ' 855功能碼
                Dim GS02 As String = "SENDERGSID"
                Dim GS03 As String = "RECEIVERGS"
                Dim GS04 As String = DateTime.Now.ToString("yyyyMMdd")
                Dim GS05 As String = DateTime.Now.ToString("HHmm")
                Dim GS06 As String = GenerateControlNumber()   ' Group Control Number
                Dim GS07 As String = "X"
                Dim GS08 As String = "004010"

                Dim ST01 As String = "855"
                Dim ST02 As String = GenerateControlNumber()   ' ST控制號

                Dim BAK01 As String = "AC"  ' Acknowledge
                Dim BAK02 As String = "123456" ' 你們可自訂
                Dim BAK03 As String = DateTime.Now.ToString("yyyyMMdd") ' 回覆日期
                Dim BAK04 As String = row("PONo").ToString()   ' 對應原 850 PO Number

                ' (b) 取出原 850 資料中的值
                Dim itemNo As String = row("ItemNo").ToString()
                Dim ntcPartNo As String = row("NTCPartNo").ToString()
                Dim awsPartNo As String = row("AWSPartNo").ToString()
                Dim price As Decimal = If(IsDBNull(row("Price")), 0, Convert.ToDecimal(row("Price")))
                Dim qty As Integer = If(IsDBNull(row("Qty")), 0, Convert.ToInt32(row("Qty")))
                Dim deliveryDate As DateTime = If(IsDBNull(row("DeliveryDate")), DateTime.MinValue, Convert.ToDateTime(row("DeliveryDate")))
                Dim requestDate As DateTime = If(IsDBNull(row("RequestDate")), DateTime.MinValue, Convert.ToDateTime(row("RequestDate")))

                ' (c) 插入到 EDI855Segments
                Dim sqlInsert As String =
                    "INSERT INTO EDI855Segments (" &
                    "ISA01, ISA02, ISA03, ISA04, ISA05, ISA06, ISA07, ISA08, ISA09, ISA10, ISA11, ISA12, ISA13, ISA14, ISA15, ISA16," &
                    "GS01, GS02, GS03, GS04, GS05, GS06, GS07, GS08," &
                    "ST01, ST02," &
                    "BAK01, BAK02, BAK03, BAK04," &
                    "ItemNo, NTCPartNo, AWSPartNo, Price, Qty, DeliveryDate, RequestDate" &
                    ") VALUES (" &
                    "@ISA01,@ISA02,@ISA03,@ISA04,@ISA05,@ISA06,@ISA07,@ISA08,@ISA09,@ISA10,@ISA11,@ISA12,@ISA13,@ISA14,@ISA15,@ISA16," &
                    "@GS01,@GS02,@GS03,@GS04,@GS05,@GS06,@GS07,@GS08," &
                    "@ST01,@ST02," &
                    "@BAK01,@BAK02,@BAK03,@BAK04," &
                    "@ItemNo,@NTCPartNo,@AWSPartNo,@Price,@Qty,@DeliveryDate,@RequestDate)"

                Using cmd As New SqlCommand(sqlInsert, conn)
                    cmd.Parameters.AddWithValue("@ISA01", ISA01)
                    cmd.Parameters.AddWithValue("@ISA02", ISA02)
                    cmd.Parameters.AddWithValue("@ISA03", ISA03)
                    cmd.Parameters.AddWithValue("@ISA04", ISA04)
                    cmd.Parameters.AddWithValue("@ISA05", ISA05)
                    cmd.Parameters.AddWithValue("@ISA06", ISA06)
                    cmd.Parameters.AddWithValue("@ISA07", ISA07)
                    cmd.Parameters.AddWithValue("@ISA08", ISA08)
                    cmd.Parameters.AddWithValue("@ISA09", ISA09)
                    cmd.Parameters.AddWithValue("@ISA10", ISA10)
                    cmd.Parameters.AddWithValue("@ISA11", ISA11)
                    cmd.Parameters.AddWithValue("@ISA12", ISA12)
                    cmd.Parameters.AddWithValue("@ISA13", ISA13)
                    cmd.Parameters.AddWithValue("@ISA14", ISA14)
                    cmd.Parameters.AddWithValue("@ISA15", ISA15)
                    cmd.Parameters.AddWithValue("@ISA16", ISA16)

                    cmd.Parameters.AddWithValue("@GS01", GS01)
                    cmd.Parameters.AddWithValue("@GS02", GS02)
                    cmd.Parameters.AddWithValue("@GS03", GS03)
                    cmd.Parameters.AddWithValue("@GS04", GS04)
                    cmd.Parameters.AddWithValue("@GS05", GS05)
                    cmd.Parameters.AddWithValue("@GS06", GS06)
                    cmd.Parameters.AddWithValue("@GS07", GS07)
                    cmd.Parameters.AddWithValue("@GS08", GS08)

                    cmd.Parameters.AddWithValue("@ST01", ST01)
                    cmd.Parameters.AddWithValue("@ST02", ST02)

                    cmd.Parameters.AddWithValue("@BAK01", BAK01)
                    cmd.Parameters.AddWithValue("@BAK02", BAK02)
                    cmd.Parameters.AddWithValue("@BAK03", BAK03)
                    cmd.Parameters.AddWithValue("@BAK04", BAK04)

                    cmd.Parameters.AddWithValue("@ItemNo", itemNo)
                    cmd.Parameters.AddWithValue("@NTCPartNo", ntcPartNo)
                    cmd.Parameters.AddWithValue("@AWSPartNo", awsPartNo)
                    cmd.Parameters.AddWithValue("@Price", price)
                    cmd.Parameters.AddWithValue("@Qty", qty)
                    cmd.Parameters.AddWithValue("@DeliveryDate", If(deliveryDate = DateTime.MinValue, DBNull.Value, deliveryDate))
                    cmd.Parameters.AddWithValue("@RequestDate", If(requestDate = DateTime.MinValue, DBNull.Value, requestDate))

                    cmd.ExecuteNonQuery()
                End Using
            Next

            conn.Close()
        End Using

        ' 3. 給前端提示
        Response.Write("<script>alert('已完成 EDI 855 資料插入。');</script>")
    End Sub

    ' 產生新的控制號（依需求自訂邏輯）
    Private Function GenerateControlNumber() As String
        ' 這裡只是簡單用時間戳作為範例控制號
        Return DateTime.Now.ToString("HHmmssfff")
    End Function

End Class


---

3. 簡要說明

1. 建表

先在資料庫建立 EDI855Segments，將 EDI 855 常用欄位 (ISA, GS, ST, BAK等) 全部列出。



2. 讀取既有 EDI850 資料

從 EDI850Data 中撈出所需欄位(如: PONo, ItemNo, DeliveryDate...)。



3. 對應並填值

855 的 ISA01, ISA02… 通常按固定協議或動態參數；PO Number、ItemNo 等則從原 850 表內取得。

需新產生的控制號 (ISA13, GS06, ST02) 則呼叫一個簡易函式 GenerateControlNumber()。



4. 插入新的資料表

將每筆原始資料做一筆對應插入 EDI855Segments。



5. 後續

若還需要最終檔案 (X12) 字串，可再從 EDI855Segments 組合段落 (ISA|GS|ST|... )，輸出成 EDI 文件。




以上程式僅示範基本流程與欄位對應，實務中會依照交易夥伴協議 (Implementation Guide) 與內部邏輯做更完整的調整。

