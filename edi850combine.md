以下示範「在同一張 EdiDetail 表格中，針對 N9 也做欄位拆解 (N901, N902, N903...)」，並同時保留其他段 (N1, SAC, REF, FOB, ITD, DTM, MSG…) 的欄位。核心做法依舊是：

1. LoopType：用來標示這筆記錄是哪個 Segment (N9, N1, SAC, …)。


2. LoopSequence：用來標示同一種 Segment 出現的第幾筆。


3. ParentDetailId：用來建立巢狀關係（例如「某個N9底下再掛N1」的狀況）。


4. 欄位展開：像 N9 就拆成 N901、N902、N903…；N1 拆成 N101、N102、N103、N104… 等等。



下方提供「整合後」的表結構範例，供參考。


---

一、EdiDetail 表 (範例)

CREATE TABLE EdiDetail (
    DetailId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT NOT NULL 
        FOREIGN KEY REFERENCES EdiMain(MainId),

    -- 若要支援 N9 裡面再掛 N1 或其他段，需要巢狀關係
    ParentDetailId INT NULL 
        REFERENCES EdiDetail(DetailId),

    -- 用來分辨是哪個 Segment: 'N9', 'N1', 'SAC', 'REF', 'FOB', 'ITD', 'DTM', 'MSG'...
    LoopType VARCHAR(10),

    -- 同一種類段多次出現時，用 LoopSequence 區分第幾筆
    LoopSequence INT,

    -- -------------------------------------------------------------------
    -- 以下是各種段可能會用到的欄位。只要此筆資料的 LoopType 與段對應，
    -- 才會填寫相應欄位，其它欄位就留 NULL。
    -- -------------------------------------------------------------------

    /* ======================
       1. N9 段 (Reference Identification)
       ====================== */
    N901 VARCHAR(3),   -- Reference Identification Qualifier
    N902 VARCHAR(30),  -- Reference Identification
    N903 VARCHAR(45),  -- Free-form Description (可依需求增減)
    N904 VARCHAR(8),   -- Date (YYMMDD 或 CCYYMMDD, 視規範而定)
    N905 VARCHAR(6),   -- Time (HHMMSS, 視規範而定)
    -- 如果還有更多欄位需求(如子元素 N906…)，依業務需求增列

    /* ======================
       2. N1 段 (Party Identification)
       ====================== */
    N101 VARCHAR(2),   -- Entity Identifier Code (e.g. ST, BT, RE, SF…)
    N102 VARCHAR(60),  -- Name
    N103 VARCHAR(2),   -- Identification Code Qualifier
    N104 VARCHAR(15),  -- Identification Code

    /* 同一個 N1 Loop 常會有 N3、N4、PER 等段，可看需求要不要一起放這裡 */
    N301 VARCHAR(55),
    N302 VARCHAR(55),
    N401 VARCHAR(30),  -- City Name
    N402 VARCHAR(2),   -- State or Province Code
    N403 VARCHAR(15),  -- Postal Code
    N404 VARCHAR(3),   -- Country Code
    PER01 VARCHAR(2),
    PER02 VARCHAR(60),
    PER03 VARCHAR(2),
    PER04 VARCHAR(25),
    -- … 依實際需要增減

    /* ======================
       3. SAC 段 (Allowance or Charge)
       ====================== */
    SAC01 VARCHAR(1),  -- Allowance or Charge Indicator (C or A)
    SAC02 VARCHAR(4),  -- SAC Code
    SAC05 VARCHAR(20), -- Amount or Rate
    -- … 依實際需要增減

    /* ======================
       4. REF 段 (Reference Identification)
       ====================== */
    REF01 VARCHAR(3),
    REF02 VARCHAR(30),
    REF03 VARCHAR(30),
    -- … 依實際需要增減

    /* ======================
       5. FOB 段 (F.O.B. Related Instructions)
       ====================== */
    FOB01 VARCHAR(2),
    FOB02 VARCHAR(30),
    -- … 依實際需要增減

    /* ======================
       6. ITD 段 (Terms of Sale/Deferred Terms of Sale)
       ====================== */
    ITD01 VARCHAR(2),
    ITD02 VARCHAR(2),
    ITD03 VARCHAR(10),
    -- … 依實際需要增減

    /* ======================
       7. DTM 段 (Date/Time Reference)
       ====================== */
    DTM01 VARCHAR(3),
    DTM02 VARCHAR(8),
    -- … 依實際需要增減

    /* ======================
       8. MSG 段 (Message Text)
       ====================== */
    MSG01 TEXT,

    -- ----------------------------------------------------
    -- 系統管理欄位
    -- ----------------------------------------------------
    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME
);

設計重點

1. LoopType + LoopSequence

用來區分資料筆屬於哪個段，以及相同段第幾筆。

例如 LoopType='N9'、LoopSequence=1 表示第一筆 N9 段。

若同一張文件有多筆 N9，就插多筆資料，LoopType='N9'，LoopSequence=2,3,4… 以此類推。



2. ParentDetailId

若需要階層結構 (例如某個 N9 底下還掛 N1、REF…)，可以用 ParentDetailId 指回對應的 EdiDetail(DetailId)。

若該段直接屬於 EdiMain (最外層)，則 ParentDetailId=NULL。



3. 大量欄位

同一張表裡，你只會用到「對應 LoopType」的欄位，其餘留 NULL。

若你希望區分表結構，例如「N9 相關欄位 + N1 相關欄位太多想拆不同表」，也可以依業務需求再拆分。





---

二、插入範例：N9 段展開

假設要在文件 MainId=100 底下插入 2 筆 N9：

-- 第1筆 N9
INSERT INTO EdiDetail (
  MainId, ParentDetailId, LoopType, LoopSequence,
  N901, N902, N903, N904, N905
)
VALUES (
  100,      -- 此 N9 直接隸屬最外層
  NULL,     
  'N9',     -- N9 段
  1,        -- 第1筆 N9
  'PO',     -- N901
  '123456', -- N902
  'Special reference info', -- N903
  '20230101', -- N904
  '1230'      -- N905 (假設格式 HHMM)
);

-- 第2筆 N9
INSERT INTO EdiDetail (
  MainId, ParentDetailId, LoopType, LoopSequence,
  N901, N902, N903
)
VALUES (
  100,
  NULL,
  'N9',
  2,
  'TN',
  'ABC999',
  'Tracking Number ABC999'
);

這樣便把 N9 段 的子欄位 (N901, N902, N903, N904, N905…) 分開存進資料表中，而不是一整段放在 N9Segment TEXT。


---

三、若 N9 底下還有 N1、REF、MSG… (巢狀)

假設在剛剛的第一筆 N9 (DetailId=501) 底下插入一筆 N1、兩筆 REF、再插一筆 MSG，流程可參考：

-- 1) 假設上面 INSERT 的第一筆 N9 => DetailId = 501

-- 2) 在該 N9 下，插入一筆 N1
INSERT INTO EdiDetail (
  MainId, ParentDetailId, LoopType, LoopSequence,
  N101, N102, N103, N104
)
VALUES (
  100,             -- 同一份文件
  501,             -- 指向父層 N9
  'N1',
  1,
  'ST',
  'Some Ship-To Name',
  '92',
  'ShipTo001'
);

-- 3) 再插入兩筆 REF，都掛在 N9 DetailId=501 下面
INSERT INTO EdiDetail (
  MainId, ParentDetailId, LoopType, LoopSequence,
  REF01, REF02
)
VALUES
(
  100,
  501,
  'REF',
  1,
  'VR',
  '123'  -- 1st REF
),
(
  100,
  501,
  'REF',
  2,
  'CO',
  'ABC'  -- 2nd REF
);

-- 4) 再插入一筆 MSG，掛在 N9 DetailId=501 下面
INSERT INTO EdiDetail (
  MainId, ParentDetailId, LoopType, LoopSequence,
  MSG01
)
VALUES (
  100,
  501,
  'MSG',
  1,
  'This is a message under the first N9 loop'
);

如此，你就能在 同一張 EdiDetail 裡，不但拆解了所有段欄位 (包含 N9、N1、REF、MSG…)，還能用 ParentDetailId 表示巢狀關係。


---

四、結語

1. 欄位爆多

這是此種「單表 + 多Segment 欄位」的常見狀況。

優點：結構直觀，查詢相對簡單（不用跨多表）。

缺點：欄位多且多數會是 NULL，需要一套插入/解析規則。



2. N9 或其他段的欄位展開

與 N1、REF、ITD、DTM… 類似，直接把 N901、N902、N903… 這些子欄位拆開。

若後續還需要更多欄位 (N906…)，可視實際需求再度延伸。



3. ParentDetailId

若根本不需要巢狀結構 (N9 下不會再掛 N1 等)，可不使用或置 NULL；需要時再指向父層 ID。



4. 依實際業務分拆

若覺得「單表過於龐大」，也可以考慮「N9 與 REF 特化一張表」、「N1 與 PER、N3、N4 特化一張表」等做法，維持比較精簡、容易維護。




總之，以上範例就是把 N9（以及其他 N1, SAC, REF, FOB, ITD, DTM, MSG…）的欄位全部展開，並且透過 LoopType 與 LoopSequence 來處理多筆重複出現的段，最後透過 ParentDetailId 來建立可能的巢狀關係。

