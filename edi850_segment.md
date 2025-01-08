如果 ITD 和 MSG 也需要與 REF 和 DTM 相同方式儲存，那可以擴展現有的 EdiSegment 表結構，將 ITD 和 MSG 的欄位統一加入 EdiSegment，並通過 SegmentType 區分資料類型。這樣能夠保持設計的簡潔性和可擴展性。


---

改進後的 EdiSegment 表結構

CREATE TABLE EdiSegment (
    SegmentId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT,                      -- 關聯到 EdiMain
    DetailItemId INT NULL,           -- 關聯到 EdiDetailItem (如果是項目層級)
    SegmentLevel VARCHAR(10),        -- HEADER/ITEM
    SegmentType VARCHAR(3),          -- REF/DTM/ITD/MSG
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

    -- ITD段落欄位
    Itd01 VARCHAR(2),                -- 條款類型代碼
    Itd02 VARCHAR(2),                -- 條款基準代碼
    Itd03 DECIMAL(6,2),              -- 條款折扣百分比
    Itd04 INT,                       -- 條款折扣天數
    Itd05 INT,                       -- 條款淨天數

    -- MSG段落欄位
    Msg01 VARCHAR(255),              -- 訊息文字

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME DEFAULT GETDATE(),

    CONSTRAINT FK_EdiSegment_Main FOREIGN KEY (MainId) 
        REFERENCES EdiMain(MainId),
    CONSTRAINT FK_EdiSegment_DetailItem FOREIGN KEY (DetailItemId) 
        REFERENCES EdiDetailItem(DetailItemId)
);


---

資料儲存規則

1. SegmentType 區分類型：

REF：使用 Ref01, Ref02, Ref03。

DTM：使用 Dtm01, Dtm02, Dtm03, Dtm04。

ITD：使用 Itd01, Itd02, Itd03, Itd04, Itd05。

MSG：使用 Msg01。



2. SequenceNo 區分多筆資料：

同一類型多筆資料使用 SequenceNo 區分。





---

新增資料範例

儲存 ITD 資料

INSERT INTO EdiSegment (MainId, SegmentLevel, SegmentType, SequenceNo, Itd01, Itd02, Itd03, Itd04, Itd05)
VALUES 
    (1, 'HEADER', 'ITD', 1, '01', '2', 2.5, 10, 30), -- 條款類型代碼、基準代碼、折扣百分比、折扣天數、淨天數
    (1, 'HEADER', 'ITD', 2, '02', '3', 5.0, 5, 60);

儲存 MSG 資料

INSERT INTO EdiSegment (MainId, SegmentLevel, SegmentType, SequenceNo, Msg01)
VALUES 
    (1, 'HEADER', 'MSG', 1, 'This is a test message.'),
    (1, 'HEADER', 'MSG', 2, 'Another important message.');

儲存 REF 資料

INSERT INTO EdiSegment (MainId, SegmentLevel, SegmentType, SequenceNo, Ref01, Ref02, Ref03)
VALUES 
    (1, 'HEADER', 'REF', 1, 'ZZ', 'RefCode1', 'Description1'),
    (1, 'HEADER', 'REF', 2, 'AB', 'RefCode2', 'Description2');

儲存 DTM 資料

INSERT INTO EdiSegment (MainId, SegmentLevel, SegmentType, SequenceNo, Dtm01, Dtm02, Dtm03, Dtm04)
VALUES 
    (1, 'HEADER', 'DTM', 1, '007', '20250101', '1200', 'ET'),
    (1, 'HEADER', 'DTM', 2, '011', '20250115', '1400', 'CT');


---

查詢範例

查詢所有 ITD 資料

SELECT 
    SegmentType,
    SequenceNo,
    Itd01 AS TermsTypeCode,
    Itd02 AS TermsBasisDateCode,
    Itd03 AS DiscountPercent,
    Itd04 AS DiscountDays,
    Itd05 AS NetDays
FROM EdiSegment
WHERE MainId = 1 AND SegmentType = 'ITD';

查詢所有 MSG 資料

SELECT 
    SegmentType,
    SequenceNo,
    Msg01 AS MessageText
FROM EdiSegment
WHERE MainId = 1 AND SegmentType = 'MSG';

查詢所有 REF 資料

SELECT 
    SegmentType,
    SequenceNo,
    Ref01 AS ReferenceQualifier,
    Ref02 AS ReferenceNumber,
    Ref03 AS Description
FROM EdiSegment
WHERE MainId = 1 AND SegmentType = 'REF';

查詢所有 DTM 資料

SELECT 
    SegmentType,
    SequenceNo,
    Dtm01 AS DateQualifier,
    Dtm02 AS Date,
    Dtm03 AS Time,
    Dtm04 AS TimeZone
FROM EdiSegment
WHERE MainId = 1 AND SegmentType = 'DTM';


---

優化與優勢

1. 統一表結構：

所有段落資料（REF、DTM、ITD、MSG）集中存儲，設計簡潔。

增加段落時，只需在 EdiSegment 表中擴展新欄位即可。



2. 靈活性高：

支援多筆段落資料存儲，通過 SegmentType 和 SequenceNo 區分。

支援多層級（Header/Item），可通過 SegmentLevel 區分資料層級。



3. 查詢方便：

使用索引（SegmentType, SegmentLevel）優化查詢性能。

可輕鬆按段落類型或層級查詢所需資料。





---

未來擴展

如果需要新增其他段落（如 SAC 或 PID），可以採用類似方式，擴展欄位並使用 SegmentType 區分類型，無需更改整體結構。

這樣設計既滿足了您對多筆資料統一存儲的需求，又保留了靈活性和可擴展性。如需進一步調整，請提供需求細節！

