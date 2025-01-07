-- 主表：儲存整個 EDI 文件的基本信息
CREATE TABLE EdiMain (
    MainId INT PRIMARY KEY IDENTITY(1,1),
    -- ISA (Interchange Control Header)
    IsaControlNumber VARCHAR(50),
    IsaSenderId VARCHAR(50),
    IsaReceiverId VARCHAR(50),
    IsaDate DATETIME,
    -- GS (Functional Group Header)
    GsControlNumber VARCHAR(50),
    GsApplicationSenderCode VARCHAR(50),
    GsApplicationReceiverCode VARCHAR(50),
    -- ST (Transaction Set Header)
    StControlNumber VARCHAR(50),
    -- BEG (Beginning Segment)
    PurchaseOrderNumber VARCHAR(50),
    PurchaseOrderDate DATETIME,
    -- CUR (Currency)
    CurrencyCode VARCHAR(3),
    -- FOB (FOB Related Instructions)
    FobPayCode VARCHAR(50),
    FobLocationQualifier VARCHAR(50),
    FobLocationDescription VARCHAR(100),
    -- ITD (Terms of Sale)
    TermsTypeCode VARCHAR(50),
    TermsBasisDateCode VARCHAR(50),
    -- DTM (Date/Time Reference)
    DateTimeQualifier VARCHAR(50),
    OrderDate DATETIME,
    -- SE (Transaction Set Trailer)
    SeControlNumber VARCHAR(50),
    -- GE (Functional Group Trailer)
    GeControlNumber VARCHAR(50),
    -- IEA (Interchange Control Trailer)
    IeaControlNumber VARCHAR(50),
    -- CTT (Transaction Totals)
    TotalLineItems INT,
    HashTotal DECIMAL(18,2),
    CreateDate DATETIME DEFAULT GETDATE()
);

-- 訂單明細表：儲存地址和參考號等信息
CREATE TABLE EdiDetail (
    DetailId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT FOREIGN KEY REFERENCES EdiMain(MainId),
    -- Loop N1 (Name)
    EntityIdentifierCode VARCHAR(50),
    EntityName VARCHAR(100),
    -- N3 (Address Information)
    AddressInformation VARCHAR(200),
    -- N4 (Geographic Location)
    CityName VARCHAR(100),
    StateCode VARCHAR(2),
    PostalCode VARCHAR(10),
    CountryCode VARCHAR(3),
    -- PER (Administrative Communications Contact)
    ContactFunctionCode VARCHAR(50),
    ContactName VARCHAR(100),
    ContactNumber VARCHAR(50),
    -- Loop N9 (Reference Number)
    ReferenceIdentificationQualifier VARCHAR(50),
    ReferenceIdentification VARCHAR(100),
    -- MSG (Message Text)
    MessageText VARCHAR(1000),
    -- SAC (Service, Promotion, Allowance, or Charge Information)
    SacIndicator VARCHAR(50),
    SacCode VARCHAR(50),
    SacAmount DECIMAL(18,2),
    LoopIdentifier VARCHAR(10),  -- 用於標識是哪個Loop的資料
    CreateDate DATETIME DEFAULT GETDATE()
);

-- 訂單商品明細表：儲存每個商品的詳細信息
CREATE TABLE EdiDetailItem (
    ItemId INT PRIMARY KEY IDENTITY(1,1),
    DetailId INT FOREIGN KEY REFERENCES EdiDetail(DetailId),
    -- PO1 (Baseline Item Data)
    LineItemNumber VARCHAR(50),
    Quantity DECIMAL(18,2),
    UnitOfMeasure VARCHAR(50),
    UnitPrice DECIMAL(18,4),
    ProductId VARCHAR(50),
    ProductIdQualifier VARCHAR(50),
    -- PID (Product/Item Description)
    ItemDescription VARCHAR(500),
    -- REF (Reference Information)
    ReferenceQualifier VARCHAR(50),
    ReferenceNumber VARCHAR(100),
    -- N1 (Name) - Item Level
    ItemEntityIdentifierCode VARCHAR(50),
    ItemEntityName VARCHAR(100),
    -- N3 (Address Information) - Item Level
    ItemAddressInformation VARCHAR(200),
    -- N4 (Geographic Location) - Item Level
    ItemCityName VARCHAR(100),
    ItemStateCode VARCHAR(2),
    ItemPostalCode VARCHAR(10),
    ItemCountryCode VARCHAR(3),
    -- TXI (Tax Information)
    TaxTypeCode VARCHAR(50),
    TaxAmount DECIMAL(18,2),
    TaxPercentage DECIMAL(5,2),
    CreateDate DATETIME DEFAULT GETDATE()
);