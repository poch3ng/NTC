以下示範將每個 Segment 進一步拆成「多個欄位」的方式，並以常見的 EDI X12 格式（如 850）為例，逐一示範欄位如何命名、放置到哪個資料表。
注意：實際欄位多寡會依據你的 EDI 實際需求、版本(4010、5010等)、業務規範而定，以下僅是常見欄位拆解做示例。


---

一、範例欄位命名規則

ISA 段：ISA01, ISA02, ISA03, ISA04, …, ISA16

GS 段：GS01, GS02, GS03, GS04, …, GS08

ST 段：ST01, ST02, …

BEG 段：BEG01, BEG02, BEG03, BEG04, …

CUR 段：CUR01, CUR02, CUR03, …

FOB 段：FOB01, FOB02, …

ITD 段：ITD01, ITD02, …

DTM 段：DTM01, DTM02, …

MSG 段：MSG01 (一般只有一個欄位，存放訊息)

N9 段：N901, N902, N903, …

N1 段：N101, N102, N103, N104, …

N3 段：N301, N302, …

N4 段：N401, N402, N403, N404, …

PER 段：PER01, PER02, …

REF 段：REF01, REF02, …

SAC 段：SAC01, SAC02, …

PO1 段：PO101, PO102, …

PID 段：PID01, PID02, …

TXI 段：TXI01, TXI02, …

CTT 段：CTT01, CTT02, …

SE 段：SE01, SE02, …

GE 段：GE01, GE02, …

IEA 段：IEA01, IEA02, …



---

二、EdiMain：存放交易最外層 Header 及「唯一性控制」欄位

資料表結構範例

CREATE TABLE EdiMain (
    MainId INT PRIMARY KEY IDENTITY(1,1),

    -- === ISA 段(Interchange Control Header) ===
    ISA01 VARCHAR(10),
    ISA02 VARCHAR(15),
    ISA03 VARCHAR(10),
    ISA04 VARCHAR(15),
    ISA05 VARCHAR(2),
    ISA06 VARCHAR(15),
    ISA07 VARCHAR(2),
    ISA08 VARCHAR(15),
    ISA09 VARCHAR(6),
    ISA10 VARCHAR(6),
    ISA11 VARCHAR(1),
    ISA12 VARCHAR(5),
    ISA13 VARCHAR(9),    -- Interchange Control Number
    ISA14 VARCHAR(1),
    ISA15 VARCHAR(1),
    ISA16 VARCHAR(1),

    -- === GS 段(Functional Group Header) ===
    GS01 VARCHAR(2),
    GS02 VARCHAR(15),
    GS03 VARCHAR(15),
    GS04 VARCHAR(8),
    GS05 VARCHAR(8),
    GS06 VARCHAR(9),    -- Group Control Number
    GS07 VARCHAR(2),
    GS08 VARCHAR(6),    -- Version/Release/Industry Identifier Code

    -- === ST 段(Transaction Set Header) ===
    ST01 VARCHAR(3),
    ST02 VARCHAR(9),

    -- === BEG 段(Beginning Segment for Purchase Order) ===
    BEG01 VARCHAR(2),
    BEG02 VARCHAR(2),
    BEG03 VARCHAR(30),
    BEG04 VARCHAR(8),
    -- 若尚有 BEG05、BEG06 等，依實際需要增列

    -- === CUR 段(Currency) ===
    CUR01 VARCHAR(3),
    CUR02 VARCHAR(3),
    -- 其餘欄位依實際需要增列

    -- === SAC 段(Service, Promotion, Allowance, or Charge) [文件層級]
    SAC01 VARCHAR(2),
    SAC02 VARCHAR(2),
    SAC05 VARCHAR(10),   -- 常見: 金額
    -- ... 根據實際需要增列更多

    -- === FOB 段(Transportation Terms) ===
    FOB01 VARCHAR(2),
    FOB02 VARCHAR(30),
    -- ...

    -- === ITD 段(Terms of Sale/Deferred Terms of Sale) ===
    ITD01 VARCHAR(2),
    ITD02 VARCHAR(8),
    -- ...

    -- === DTM 段(Date/Time Reference) ===
    DTM01 VARCHAR(3),
    DTM02 VARCHAR(8),
    -- ...

    -- === MSG 段(Text) ===
    MSG01 TEXT,  -- 一般只有 1 個欄位, 可 TEXT 儲存

    -- === SE 段(Transaction Set Trailer) ===
    SE01 VARCHAR(10),
    SE02 VARCHAR(9),

    -- === GE 段(Functional Group Trailer) ===
    GE01 VARCHAR(6),
    GE02 VARCHAR(9),

    -- === IEA 段(Interchange Control Trailer) ===
    IEA01 VARCHAR(1),
    IEA02 VARCHAR(9),

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME
);

> 各段的額外欄位數量，實際要看你的 Implementation Guide (IG) 需求，這裡只展示常見欄位。

SAC 在文件層級若僅有一次，可放這裡；若會多筆出現，應改放到明細或另開表。





---

三、EdiDetail：存放 Header-Level 的重複 Loop (例如 N1、N9...等)

為何要拆 Segment？

1. N1、N3、N4、PER 通常出現多次，像是不同角色的地址(Ship To、Bill To...)。


2. N9 Loop 裡面可能有 N9 與 MSG，或加一些 REF、其他段等等。



CREATE TABLE EdiDetail (
    DetailId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT NOT NULL FOREIGN KEY REFERENCES EdiMain(MainId),

    LoopType VARCHAR(10),     -- 'N1' or 'N9' or 其他
    LoopSequence INT,         -- 第幾個同類型 Loop

    -- === N1 段 ===
    N101 VARCHAR(2),
    N102 VARCHAR(60),
    N103 VARCHAR(2),
    N104 VARCHAR(15),

    -- === N3 段 ===
    N301 VARCHAR(55),
    N302 VARCHAR(55),

    -- === N4 段 ===
    N401 VARCHAR(30),
    N402 VARCHAR(2),
    N403 VARCHAR(15),
    N404 VARCHAR(3),

    -- === PER 段(若此 Loop 會包含聯絡資訊) ===
    PER01 VARCHAR(2),
    PER02 VARCHAR(60),
    PER03 VARCHAR(2),
    PER04 VARCHAR(15),
    -- ...

    -- === N9 段 ===
    N901 VARCHAR(3),
    N902 VARCHAR(30),
    N903 VARCHAR(30),
    -- ...

    -- === MSG 段(若在 N9 Loop 中出現) ===
    MSG01 TEXT,

    -- === REF 段(若在該 Loop 需要用到) ===
    REF01 VARCHAR(3),
    REF02 VARCHAR(30),
    REF03 VARCHAR(30),

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME
);

> LoopType + LoopSequence，可幫你識別「哪一種 Loop」「第幾次出現」。

在同一張表同時存 N1 Loop、N9 Loop，只要把不使用的欄位留空(Null)即可。





---

四、EdiDetailItem：存放 Item-Level (PO1) 的重複明細

常見段：PO1、PID、REF、N1/N3/N4(若項目層次有地址需求)、TXI、CTT

CREATE TABLE EdiDetailItem (
    DetailItemId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT NOT NULL FOREIGN KEY REFERENCES EdiMain(MainId),

    -- 用來標示此 PO1 順序 (Line Number)
    LineSequence INT,     

    -- === PO1 段(最常見的訂單明細) ===
    PO101 VARCHAR(20),
    PO102 VARCHAR(10),
    PO103 VARCHAR(2),
    PO104 VARCHAR(10),
    PO105 VARCHAR(2),
    PO106 VARCHAR(2),
    PO107 VARCHAR(2),
    -- ...

    -- === PID 段(產品描述) ===
    PID01 VARCHAR(2),
    PID02 VARCHAR(2),
    PID03 VARCHAR(3),
    PID04 VARCHAR(12),
    PID05 VARCHAR(80),
    -- ...

    -- === REF 段(項目層級) ===
    REF01 VARCHAR(3),
    REF02 VARCHAR(30),
    REF03 VARCHAR(30),

    -- === 若項目層級有自己的 N1/N3/N4 段 ===
    N101 VARCHAR(2),
    N102 VARCHAR(60),
    N103 VARCHAR(2),
    N104 VARCHAR(15),
    N301 VARCHAR(55),
    N302 VARCHAR(55),
    N401 VARCHAR(30),
    N402 VARCHAR(2),
    N403 VARCHAR(15),
    N404 VARCHAR(3),

    -- === TXI 段(稅務資訊) ===
    TXI01 VARCHAR(3),
    TXI02 VARCHAR(12),
    TXI03 VARCHAR(3),
    -- ...

    -- === CTT 段(通常放在交易最後，但若要與每個 PO1 對應，也可放這裡) ===
    CTT01 VARCHAR(10),
    CTT02 VARCHAR(10),
    -- ...

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME
);

> 在 850 的常見作法是：PO1 + PID + REF + SAC + SLN + TXI 等等都會跟隨著一個「Line Item」Loop 出現，因此在這裡可視需要再延伸。

若同一個 PO1 可能包含多個 PID、REF、SAC，則要考慮做多張表來解：EdiDetailItem(主檔) + EdiDetailItemPID(PID 多筆) + EdiDetailItemSAC(SAC 多筆)…





---

五、總結建議

1. 若同一段重複出現多次

需要拆成「1:N」結構。例如同一張 PO 可能出現多個 DTM、多個 N9；你可以把 DTM 或 N9 拆開至 EdiDetail，並用 LoopType='N9' 做區分。



2. 若同一 PO1 需要多個 PID、REF、SAC

可能再細分表結構，如 EdiDetailItemPID, EdiDetailItemREF, EdiDetailItemSAC 等，讓每個段更靈活 1:N。

這就會變得資料庫欄位更細、表也更多，但彈性最高。



3. 欄位數量非常龐大

通常會結合程式邏輯來動態解析 EDI 段，再逐段寫入對應欄位，也會需要對資料完整性做更多的控管。

若要做高度彈性的解析(不想一直改資料庫 schema)，也可以考慮將解析後的 key/value 存成 JSON 格式，再寫入 NoSQL 或 JSON 欄位。




依照以上結構，你就能把每個 Segment 的子元素(ISA01, ISA02, ...)都對應到獨立欄位，並可清楚區分哪個 Loop/Line 對應哪一筆資料。後續若要擴充更多段或欄位，也可以照此模式持續拆解、新增。

