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