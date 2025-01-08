-- 建立EDI主表
CREATE TABLE EdiMain (
    MainId INT PRIMARY KEY IDENTITY(1,1),
    
    -- ISA段落
    Isa01 VARCHAR(2),                -- 授權資訊限定符
    Isa02 VARCHAR(10),               -- 授權資訊
    Isa03 VARCHAR(2),                -- 安全資訊限定符
    Isa04 VARCHAR(10),               -- 安全資訊
    Isa05 VARCHAR(2),                -- 交換發送者ID限定符
    Isa06 VARCHAR(15),               -- 交換發送者ID
    Isa07 VARCHAR(2),                -- 交換接換者ID限定符
    Isa08 VARCHAR(15),               -- 交換接換者ID
    Isa09 VARCHAR(6),                -- 交換日期
    Isa10 VARCHAR(4),                -- 交換時間
    Isa11 VARCHAR(1),                -- 控制標準識別碼
    Isa12 VARCHAR(5),                -- 控制版本號
    Isa13 VARCHAR(9),                -- 控制編號
    Isa14 VARCHAR(1),                -- 確認要求標誌
    Isa15 VARCHAR(1),                -- 使用指示器
    Isa16 VARCHAR(1),                -- 元件分隔符號

    -- GS段落
    Gs01 VARCHAR(2),                 -- 功能識別碼
    Gs02 VARCHAR(15),                -- 應用發送者代碼
    Gs03 VARCHAR(15),                -- 應用接收者代碼
    Gs04 VARCHAR(8),                 -- 日期
    Gs05 VARCHAR(4),                 -- 時間
    Gs06 VARCHAR(9),                 -- 群組控制編號
    Gs07 VARCHAR(2),                 -- 負責機構代碼
    Gs08 VARCHAR(12),                -- 版本號

    -- ST段落
    St01 VARCHAR(3),                 -- 交易組識別碼
    St02 VARCHAR(9),                 -- 控制編號

    -- BEG段落
    Beg01 VARCHAR(2),                -- 交易組用途代碼
    Beg02 VARCHAR(2),                -- 採購單類型代碼
    Beg03 VARCHAR(22),               -- 採購單號碼
    Beg04 VARCHAR(8),                -- 採購單日期
    Beg05 VARCHAR(30),               -- 合約編號

    -- CUR段落
    Cur01 VARCHAR(2),                -- 實體類型限定符
    Cur02 VARCHAR(3),                -- 貨幣代碼

    -- FOB段落
    Fob01 VARCHAR(2),                -- 裝運條款代碼限定符
    Fob02 VARCHAR(30),               -- 裝運條款代碼
    Fob03 VARCHAR(2),                -- 運輸方式代碼

    -- ITD段落
    Itd01 VARCHAR(2),                -- 條款類型代碼
    Itd02 VARCHAR(2),                -- 條款基準代碼
    Itd03 VARCHAR(6),                -- 條款折扣百分比
    Itd04 VARCHAR(3),                -- 條款折扣天數
    Itd05 VARCHAR(3),                -- 條款淨天數

    -- MSG段落
    Msg01 VARCHAR(255),              -- 訊息文字

    -- SE段落
    Se01 VARCHAR(6),                 -- 包含的段落數
    Se02 VARCHAR(9),                 -- 控制編號

    -- GE段落
    Ge01 VARCHAR(6),                 -- 包含的組數
    Ge02 VARCHAR(9),                 -- 群組控制編號

    -- IEA段落
    Iea01 VARCHAR(6),                -- 包含的功能群組數
    Iea02 VARCHAR(9),                -- 控制編號

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME
);

-- 建立EDI段落表 (REF和DTM)
CREATE TABLE EdiSegment (
    SegmentId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT,                      -- 關聯到EdiMain
    DetailItemId INT NULL,           -- 關聯到EdiDetailItem (如果是項目層級)
    SegmentLevel VARCHAR(10),        -- HEADER/ITEM
    SegmentType VARCHAR(3),          -- REF/DTM
    SequenceNo INT,                  -- 序號

    -- REF段落欄位
    Ref01 VARCHAR(3),                -- 參考編號限定符
    Ref02 VARCHAR(50),               -- 參考編號
    Ref03 VARCHAR(80),               -- 描述

    -- DTM段落欄位
    Dtm01 VARCHAR(3),                -- 日期限定符
    Dtm02 VARCHAR(8),                -- 日期
    Dtm03 VARCHAR(8),                -- 時間
    Dtm04 VARCHAR(2),                -- 時區

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME,

    CONSTRAINT FK_EdiSegment_Main FOREIGN KEY (MainId) 
        REFERENCES EdiMain(MainId),
    CONSTRAINT FK_EdiSegment_DetailItem FOREIGN KEY (DetailItemId) 
        REFERENCES EdiDetailItem(DetailItemId)
);

-- 建立EDI明細表 (N1 Loop相關)
CREATE TABLE EdiDetail (
    DetailId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT,
    LoopType VARCHAR(10),            -- N1 或 N9
    LoopSequence INT,                -- Loop序號

    -- N1段落
    N101 VARCHAR(2),                 -- 實體識別碼限定符
    N102 VARCHAR(60),                -- 名稱
    N103 VARCHAR(2),                 -- 識別碼限定符
    N104 VARCHAR(80),                -- 識別碼

    -- N3段落
    N301 VARCHAR(55),                -- 地址資訊
    N302 VARCHAR(55),                -- 地址資訊

    -- N4段落
    N401 VARCHAR(30),                -- 城市名稱
    N402 VARCHAR(2),                 -- 州別代碼
    N403 VARCHAR(15),                -- 郵遞區號
    N404 VARCHAR(3),                 -- 國家代碼

    -- PER段落
    Per01 VARCHAR(2),                -- 聯絡功能代碼
    Per02 VARCHAR(60),               -- 聯絡人姓名
    Per03 VARCHAR(2),                -- 通訊號碼限定符
    Per04 VARCHAR(80),               -- 通訊號碼

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME,

    CONSTRAINT FK_EdiDetail_Main FOREIGN KEY (MainId) 
        REFERENCES EdiMain(MainId)
);

-- 建立EDI項目明細表
CREATE TABLE EdiDetailItem (
    DetailItemId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT,
    LoopSequence INT,                -- PO1 Loop序號

    -- PO1段落
    Po101 VARCHAR(6),                -- 行項目號碼
    Po102 VARCHAR(9),                -- 數量
    Po103 VARCHAR(2),                -- 單位代碼
    Po104 VARCHAR(17),               -- 單價
    Po105 VARCHAR(2),                -- 價格基準代碼
    Po106 VARCHAR(2),                -- 產品限定符
    Po107 VARCHAR(30),               -- 產品編號

    -- PID段落
    Pid01 VARCHAR(1),                -- 項目描述類型
    Pid02 VARCHAR(2),                -- 產品特性限定符
    Pid03 VARCHAR(2),                -- 代碼清單限定符
    Pid04 VARCHAR(12),               -- 代碼
    Pid05 VARCHAR(80),               -- 描述

    -- TXI段落
    Txi01 VARCHAR(2),                -- 稅務類型代碼
    Txi02 VARCHAR(18),               -- 稅務金額
    Txi03 VARCHAR(10),               -- 稅務百分比
    Txi04 VARCHAR(2),                -- 稅務管轄區限定符
    Txi05 VARCHAR(10),               -- 稅務管轄區代碼

    -- CTT段落
    Ctt01 VARCHAR(6),                -- 行項目總數
    Ctt02 VARCHAR(10),               -- 總重量

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME,

    CONSTRAINT FK_EdiDetailItem_Main FOREIGN KEY (MainId) 
        REFERENCES EdiMain(MainId)
);

-- 建立索引
CREATE INDEX IX_EdiSegment_MainId ON EdiSegment(MainId);
CREATE INDEX IX_EdiSegment_DetailItemId ON EdiSegment(DetailItemId);
CREATE INDEX IX_EdiSegment_Type ON EdiSegment(SegmentType, SegmentLevel);
CREATE INDEX IX_EdiDetail_MainId ON EdiDetail(MainId);
CREATE INDEX IX_EdiDetailItem_MainId ON EdiDetailItem(MainId);

-- 建立檢查約束
ALTER TABLE EdiSegment ADD CONSTRAINT CK_SegmentType 
    CHECK (SegmentType IN ('REF', 'DTM'));
ALTER TABLE EdiSegment ADD CONSTRAINT CK_SegmentLevel 
    CHECK (SegmentLevel IN ('HEADER', 'ITEM'));
ALTER TABLE EdiDetail ADD CONSTRAINT CK_LoopType 
    CHECK (LoopType IN ('N1', 'N9'));

-- 新增單一欄位
ALTER TABLE table_name
ADD COLUMN column_name data_type [constraints];

-- 範例:新增必填的 VARCHAR 欄位
ALTER TABLE users
ADD COLUMN email VARCHAR(100) NOT NULL;

-- 新增多個欄位
ALTER TABLE products
ADD COLUMN description TEXT,
ADD COLUMN created_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP,
ADD COLUMN updated_at TIMESTAMP DEFAULT CURRENT_TIMESTAMP ON UPDATE CURRENT_TIMESTAMP;

-- 新增欄位並設定預設值
ALTER TABLE orders
ADD COLUMN status VARCHAR(20) DEFAULT 'pending' NOT NULL;

-- 新增欄位並加入外鍵約束
ALTER TABLE orders
ADD COLUMN user_id INT,
ADD CONSTRAINT fk_user
FOREIGN KEY (user_id) REFERENCES users(id);

-- 新增欄位並設定檢查條件
ALTER TABLE employees
ADD COLUMN age INT CHECK (age >= 18);

-- 新增唯一索引欄位
ALTER TABLE customers
ADD COLUMN phone VARCHAR(20),
ADD UNIQUE INDEX idx_phone (phone);


-- 1. 訂單主表 (Header Level)
CREATE TABLE EDI_850_Header (
    HeaderID INT IDENTITY(1,1) PRIMARY KEY,
    -- ISA Segment
    ISA_AuthorizationQual VARCHAR(2),      -- ISA01
    ISA_AuthorizationInfo VARCHAR(10),     -- ISA02
    ISA_SecurityQual VARCHAR(2),           -- ISA03
    ISA_SecurityInfo VARCHAR(10),          -- ISA04
    ISA_SenderQual VARCHAR(2),             -- ISA05
    ISA_SenderID VARCHAR(15),              -- ISA06
    ISA_ReceiverQual VARCHAR(2),           -- ISA07
    ISA_ReceiverID VARCHAR(15),            -- ISA08
    ISA_Date VARCHAR(6),                   -- ISA09
    ISA_Time VARCHAR(4),                   -- ISA10
    ISA_StandardsID VARCHAR(1),            -- ISA11
    ISA_Version VARCHAR(5),                -- ISA12
    ISA_ControlNumber VARCHAR(9),          -- ISA13
    ISA_AckRequested VARCHAR(1),           -- ISA14
    ISA_TestIndicator VARCHAR(1),          -- ISA15
    
    -- GS Segment
    GS_FunctionalIDCode VARCHAR(2),        -- GS01
    GS_SenderCode VARCHAR(15),             -- GS02
    GS_ReceiverCode VARCHAR(15),           -- GS03
    GS_Date VARCHAR(8),                    -- GS04
    GS_Time VARCHAR(4),                    -- GS05
    GS_ControlNumber VARCHAR(9),           -- GS06
    GS_ResponsibleAgency VARCHAR(2),       -- GS07
    GS_Version VARCHAR(12),                -- GS08
    
    -- ST Segment
    ST_TransactionSetID VARCHAR(3),        -- ST01 (850)
    ST_ControlNumber VARCHAR(9),           -- ST02
    
    -- BEG Segment (Beginning Segment for Purchase Order)
    BEG_TransactionSetPurpose VARCHAR(2),  -- BEG01
    BEG_POType VARCHAR(2),                 -- BEG02
    BEG_PONumber VARCHAR(22),              -- BEG03
    BEG_ReleaseNumber VARCHAR(30),         -- BEG04
    BEG_PODate DATE,                       -- BEG05
    BEG_ContractNumber VARCHAR(30),        -- BEG06
    BEG_AckType VARCHAR(2),                -- BEG07
    BEG_InvoiceType VARCHAR(2),            -- BEG08
    BEG_ContractTypeQual VARCHAR(2),       -- BEG09
    BEG_ContractType VARCHAR(2),           -- BEG10
    
    -- CUR Segment (Currency)
    CUR_EntityIdentifier VARCHAR(2),       -- CUR01
    CUR_CurrencyCode VARCHAR(3),           -- CUR02
    CUR_ExchangeRate DECIMAL(18,6),        -- CUR03
    
    -- FOB Segment (F.O.B. Related Instructions)
    FOB_ShipmentMethod VARCHAR(2),         -- FOB01
    FOB_LocationQualifier VARCHAR(2),      -- FOB02
    FOB_Description VARCHAR(30),           -- FOB03
    
    -- ITD Segment (Terms of Sale/Deferred Terms of Sale)
    ITD_TermsType VARCHAR(2),              -- ITD01
    ITD_TermsBasisQual VARCHAR(2),         -- ITD02
    ITD_TermsDiscountPercent DECIMAL(8,4), -- ITD03
    ITD_TermsDiscountDays INT,             -- ITD04
    ITD_NetDays INT,                       -- ITD05
    ITD_TermsDiscountAmt DECIMAL(18,2),    -- ITD06
    ITD_TermsDiscountDate DATE,            -- ITD07
    ITD_DeferredAmtDue DECIMAL(18,2),      -- ITD08
    ITD_DeferredDueDate DATE,              -- ITD09
    ITD_PercentOfInvoice DECIMAL(8,4),     -- ITD10
    
    -- DTM Segment (Date/Time Reference)
    DTM_DeliveryRequested DATE,            -- From DTM where DTM01 = '002'
    DTM_ShipNotBefore DATE,                -- From DTM where DTM01 = '037'
    DTM_ShipNoLater DATE,                  -- From DTM where DTM01 = '038'
    DTM_PromisedForDelivery DATE,          -- From DTM where DTM01 = '069'
    
    -- TD5 Segment (Carrier Details)
    TD5_RoutingSequence VARCHAR(2),        -- TD501
    TD5_RouteQual VARCHAR(2),              -- TD502
    TD5_RoutingID VARCHAR(35),             -- TD503
    TD5_TransMethodCode VARCHAR(2),         -- TD504
    TD5_TransMethodDesc VARCHAR(50),        -- TD505
    
    -- N9 Reference Numbers (Header)
    ReferenceQualifier VARCHAR(3),         -- N901
    ReferenceID VARCHAR(30),               -- N902
    ReferenceDescription VARCHAR(45),       -- N903
    
    -- CTT Segment (Transaction Totals)
    CTT_NumberOfLineItems INT,             -- CTT01
    CTT_HashTotal DECIMAL(18,2),           -- CTT02
    
    -- SE Segment
    SE_NumberOfSegments INT,               -- SE01
    SE_ControlNumber VARCHAR(9),           -- SE02
    
    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME,
    ProcessStatus VARCHAR(20),
    
    CONSTRAINT UX_PONumber UNIQUE (BEG_PONumber)
);

-- 2. 相關方資訊表 (N1 Loop)
CREATE TABLE EDI_850_Party (
    PartyID INT IDENTITY(1,1) PRIMARY KEY,
    HeaderID INT NOT NULL,
    LineItemNumber VARCHAR(20) NULL,     -- NULL表示header level
    
    -- N1 Segment
    N1_EntityIdentifier VARCHAR(3),      -- N101 (e.g., BY, ST, SE)
    N1_Name VARCHAR(60),                 -- N102
    N1_IdQualifier VARCHAR(2),           -- N103
    N1_IdCode VARCHAR(80),               -- N104
    
    -- N2 Segment (Additional Name)
    N2_Name1 VARCHAR(60),                -- N201
    N2_Name2 VARCHAR(60),                -- N202
    
    -- N3 Segment (Address)
    N3_AddressLine1 VARCHAR(55),         -- N301
    N3_AddressLine2 VARCHAR(55),         -- N302
    
    -- N4 Segment (Geographic Location)
    N4_City VARCHAR(30),                 -- N401
    N4_State VARCHAR(2),                 -- N402
    N4_PostalCode VARCHAR(15),           -- N403
    N4_CountryCode VARCHAR(3),           -- N404
    N4_LocationQualifier VARCHAR(2),     -- N405
    N4_LocationID VARCHAR(30),           -- N406
    
    -- PER Segment (Administrative Communications Contact)
    PER_ContactFunction VARCHAR(2),      -- PER01
    PER_ContactName VARCHAR(60),         -- PER02
    PER_CommQual1 VARCHAR(2),           -- PER03
    PER_CommNumber1 VARCHAR(256),        -- PER04
    PER_CommQual2 VARCHAR(2),           -- PER05
    PER_CommNumber2 VARCHAR(256),        -- PER06
    PER_CommQual3 VARCHAR(2),           -- PER07
    PER_CommNumber3 VARCHAR(256),        -- PER08
    
    CONSTRAINT FK_Party_Header 
        FOREIGN KEY (HeaderID) 
        REFERENCES EDI_850_Header(HeaderID)
);

-- 3. 訂單明細表 (PO1 Loop)
CREATE TABLE EDI_850_Detail (
    DetailID INT IDENTITY(1,1) PRIMARY KEY,
    HeaderID INT NOT NULL,
    
    -- PO1 Segment
    PO1_LineNumber VARCHAR(20),           -- Assigned line number
    PO1_Quantity DECIMAL(18,4),           -- PO102
    PO1_UnitOfMeasure VARCHAR(2),         -- PO103
    PO1_UnitPrice DECIMAL(18,4),          -- PO104
    PO1_BasisOfUnitPrice VARCHAR(2),      -- PO105
    PO1_ProductQual1 VARCHAR(2),          -- PO106
    PO1_ProductID1 VARCHAR(48),           -- PO107
    PO1_ProductQual2 VARCHAR(2),          -- PO108
    PO1_ProductID2 VARCHAR(48),           -- PO109
    PO1_ProductQual3 VARCHAR(2),          -- PO110
    PO1_ProductID3 VARCHAR(48),           -- PO111
    
    -- CUR Segment (Line Level Currency)
    CUR_EntityIdentifier VARCHAR(2),      
    CUR_CurrencyCode VARCHAR(3),
    CUR_ExchangeRate DECIMAL(18,6),
    
    -- PID Segment (Product Description)
    PID_ItemDescriptionType VARCHAR(1),    -- PID01
    PID_ProductCharCode VARCHAR(3),        -- PID02
    PID_AgencyQualCode VARCHAR(2),         -- PID03
    PID_ProductDescription VARCHAR(1000),  -- PID04
    
    -- MEA Segment (Measurements)
    MEA_MeasurementRef VARCHAR(2),         -- MEA01
    MEA_MeasurementQual VARCHAR(3),        -- MEA02
    MEA_MeasurementValue DECIMAL(18,4),    -- MEA03
    MEA_CompositeMeasurement VARCHAR(20),  -- MEA04
    MEA_RangeMin DECIMAL(18,4),            -- MEA05
    MEA_RangeMax DECIMAL(18,4),            -- MEA06
    
    -- PKG Segment (Marking, Packaging, Loading)
    PKG_ItemDescriptionType VARCHAR(1),     -- PKG01
    PKG_PackagingCharCode VARCHAR(3),       -- PKG02
    PKG_AgencyQualCode VARCHAR(2),          -- PKG03
    PKG_PackagingDescription VARCHAR(1000), -- PKG04
    
    -- DTM Segment (Line Item Dates)
    DTM_DeliveryRequested DATE,            -- From DTM where DTM01 = '002'
    DTM_ShipNotBefore DATE,                -- From DTM where DTM01 = '037'
    DTM_ShipNoLater DATE,                  -- From DTM where DTM01 = '038'
    DTM_PromisedForDelivery DATE,          -- From DTM where DTM01 = '069'
    
    CONSTRAINT FK_Detail_Header 
        FOREIGN KEY (HeaderID) 
        REFERENCES EDI_850_Header(HeaderID)
);

-- 4. 允許和扣款表 (SAC Segments)
CREATE TABLE EDI_850_Allowance_Charge (
    SACID INT IDENTITY(1,1) PRIMARY KEY,
    HeaderID INT NOT NULL,
    LineItemNumber VARCHAR(20) NULL,    -- NULL表示header level
    
    SAC_AllowanceIndicator VARCHAR(1),  -- SAC01
    SAC_ServiceCode VARCHAR(4),         -- SAC02
    SAC_AgencyQualCode VARCHAR(2),      -- SAC03
    SAC_AgencyCode VARCHAR(10),         -- SAC04
    SAC_Amount DECIMAL(18,2),           -- SAC05
    SAC_PercentBasis VARCHAR(2),        -- SAC06
    SAC_Rate DECIMAL(8,4),              -- SAC07
    SAC_UnitOfMeasure VARCHAR(2),       -- SAC08
    SAC_Quantity DECIMAL(18,4),         -- SAC09
    
    CONSTRAINT FK_SAC_Header 
        FOREIGN KEY (HeaderID) 
        REFERENCES EDI_850_Header(HeaderID)
);

-- 5. 參考編號表 (REF Segments)
CREATE TABLE EDI_850_References (
    ReferenceID INT IDENTITY(1,1) PRIMARY KEY,
    HeaderID INT NOT NULL,
    LineItemNumber VARCHAR(20) NULL,    -- NULL表示header level
    
    REF_ReferenceQualifier VARCHAR(3),  -- REF01
    REF_ReferenceID VARCHAR(50),        -- REF02
    REF_Description VARCHAR(80),        -- REF03
    
    CONSTRAINT FK_Reference_Header 
        FOREIGN KEY (HeaderID) 
        REFERENCES EDI_850_Header(HeaderID)
);

-- 6. 訊息表 (MSG Segments)
CREATE TABLE EDI_850_Messages (
    MessageID INT IDENTITY(1,1) PRIMARY KEY,
    HeaderID INT NOT NULL,
    LineItemNumber VARCHAR(20) NULL,    -- NULL表示header level
    ParentSegment VARCHAR(3),           -- 例如: N9, PO1等
    
    MSG_FreeFormMessage VARCHAR(1000),  -- MSG01
    
    CONSTRAINT FK_Message_Header 
        FOREIGN KEY (HeaderID) 
        REFERENCES EDI_850_Header(HeaderID)
);

-- 建立索引
CREATE INDEX IX_EDI850_Header_PONumber ON EDI_850_Header(BEG_PONumber);
CREATE INDEX IX_EDI850_Header_PODate ON EDI_850_Header(BEG_PODate);
CREATE INDEX IX_EDI850_Party_Header ON EDI_850_Party(HeaderID);
CREATE INDEX IX_EDI850_Detail_Header ON EDI_850_Detail(HeaderID);
CREATE INDEX IX_EDI850_SAC_Header ON EDI_850_Allowance_Charge(HeaderID);
CREATE INDEX IX_EDI850_References_Header ON EDI_850_References(HeaderID);
CREATE INDEX IX_EDI850_Messages_Header ON EDI_850_Messages(HeaderID);