以下提供一種常見且簡單的做法：
在 EdiDetail (或對應的明細表) 新增一個 ParentDetailId 欄位，讓「子 Loop」可以指向「父 Loop」的記錄。如此一來，就能表示「第 1 個 N9 底下的第 3 個 N1」之類的階層結構。


---

1. 在 EdiDetail 新增 ParentDetailId

假設我們原本的 EdiDetail 定義如下（僅節錄重點）：

CREATE TABLE EdiDetail (
    DetailId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT NOT NULL,
    LoopType VARCHAR(10),      -- 'N1', 'N9' 等
    LoopSequence INT,
    -- ...
    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME
);

現在加一欄 ParentDetailId 來建立父子關係：

ALTER TABLE EdiDetail
ADD ParentDetailId INT NULL;  -- 若是頂層 Loop(例如直接掛在 MainID 之下)，則為 NULL

或者在建表時就直接加：

CREATE TABLE EdiDetail (
    DetailId INT PRIMARY KEY IDENTITY(1,1),
    MainId INT NOT NULL FOREIGN KEY REFERENCES EdiMain(MainId),
    
    ParentDetailId INT NULL,   -- 指向同張表 EdiDetail.DetailId

    LoopType VARCHAR(10),      -- 'N9', 'N1' 等
    LoopSequence INT,          -- 同類型 loop 的出現順序

    -- N9、N1 的欄位拆解 (略) ...
    N101 VARCHAR(2),
    N102 VARCHAR(60),
    N901 VARCHAR(3),
    N902 VARCHAR(30),
    -- ...

    CreateDate DATETIME DEFAULT GETDATE(),
    LastUpdateDate DATETIME
);

> ParentDetailId 用來指向同一張表的 DetailId，表示「我屬於哪一個父層 Loop」。

若此筆是最外層 (例如最上層的 N9 loop，並沒有掛在其他 loop 下)，ParentDetailId 就可以是 NULL。





---

2. 實際範例：第一個 N9、底下第三個 N1

1. 插入第一個 N9

LoopType='N9'

LoopSequence=1 (代表此文件的第 1 個 N9)

ParentDetailId=NULL (因為它直接掛在整個文件底層)


INSERT INTO EdiDetail (
    MainId, ParentDetailId, LoopType, LoopSequence, N901, N902
)
VALUES (
    123,        -- 假設屬於 EdiMain.MainId = 123
    NULL,       -- 沒有父層
    'N9',       -- 這是一個 N9 Loop
    1,          -- 第一個 N9
    'PR',       -- N901
    '1234567'   -- N902
);

假設這筆插入後的 DetailId = 10。


2. 插入底下的第三個 N1

LoopType='N1'

LoopSequence=3 (第三個 N1)

ParentDetailId=10 (表示它屬於剛剛那筆 N9 之下)


INSERT INTO EdiDetail (
    MainId, ParentDetailId, LoopType, LoopSequence, N101, N102
)
VALUES (
    123,   -- 同一份文件
    10,    -- 父層是剛剛那筆 N9(DetailId=10)
    'N1',  -- 這是一個 N1 Loop
    3,     -- 第 3 個 N1
    'ST',  -- N101 (例如 ShipTo)
    'Some Company'  -- N102 (名稱)
);

這樣就表示：「在 DetailId=10 的 N9 Loop 之下，掛了一個 LoopSequence=3 的 N1」。




---

3. 查詢與邏輯關聯

若要查詢「第一個 N9 底下的所有 N1」，就可以根據 ParentDetailId = <N9的DetailId> 來篩選。

N9 和 N1 同在 EdiDetail：

N9 自己的 ParentDetailId 通常為 NULL（或指向更上層，如此類推）。

N1 只要標上 ParentDetailId = N9.DetailId 即可表示「從屬哪個 N9」。




---

4. 簡要結論

1. 多層 loop 關係：

在 同一張表 透過 ParentDetailId 建立父子層級。

LoopType + LoopSequence + ParentDetailId 就能幫你表示「第幾個 N9 的第幾個 N1」。



2. 巢狀結構：

需要更深層 (N9 裡面又有其他子 Loop) 也適用同樣方式一路往下帶。




如此，便能清楚地表示「第一個 N9 裡面含有第三個 N1」等關係。

