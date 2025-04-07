以下是產生你下拉選單的 SQL 查詢語法，可用於 MS SQL Server / MySQL 等常見資料庫：


---

一、資料表建置語法（Create Table）

CREATE TABLE ApprovalLevelOptions (
    OptionID INT PRIMARY KEY AUTO_INCREMENT,
    OptionCode VARCHAR(50) NOT NULL,
    DisplayName VARCHAR(100) NOT NULL,
    MaxLevel INT NULL,
    SkipTitles VARCHAR(100) NULL,
    IsActive BIT DEFAULT 1,
    SortOrder INT DEFAULT 0
);


---

二、插入範例資料（Insert Data）

INSERT INTO ApprovalLevelOptions (OptionCode, DisplayName, MaxLevel, SkipTitles, SortOrder)
VALUES
('skip', '略過主管', NULL, NULL, 1),
('level80', '上級主管（課經理）', 80, '副處長', 2),
('level70', '二級主管（部經理）', 70, '副處長', 3),
('level60', '一級主管（處長）', 60, '副處長', 4);


---

三、查詢下拉選單資料（Select）

SELECT OptionCode, DisplayName
FROM ApprovalLevelOptions
WHERE IsActive = 1
ORDER BY SortOrder;


---

這樣可以確保：

易於擴充（新增層級或特殊條件）

控制略過職稱（如副處長）

顯示順序固定且清楚


需要的話我也可以幫你補一段對應的 VB.NET 讀取下拉選單的程式碼。

