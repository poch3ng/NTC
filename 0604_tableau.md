從 Azure DevOps 的 DB（TFS）抓資料到 Tableau，可行但需釐清你使用的是哪種架構：

On-premises TFS / Azure DevOps Server ✅ 可以直接連接 SQL Server 資料庫

Azure DevOps Services（雲端版） ❌ 無法直接存取 SQL，需要用 Analytics View 或 REST API



---

✅ 如果你是使用 TFS / Azure DevOps Server（On-Prem）

🔧 1. 找到你的 SQL 資料庫名稱

TFS 安裝後會建立這些核心資料庫（每個 collection 一個）：

Tfs_Configuration

Tfs_DefaultCollection（或其他 collection 名）


常見重要資料表（以 Git 與 CI/CD 為主）：

功能	表名稱	說明

Commit	tbl_Commit / tbl_Changeset	儲存提交紀錄
Pull Request	tbl_PullRequest / tbl_PullRequestThread	PR 資訊與討論
Build	tbl_Build / tbl_BuildDefinition	CI 建置歷史
Release	tbl_Release / tbl_ReleaseDeployment	CD 部署紀錄
Project	tbl_Project	專案資訊（與其他表 Join 用）
User	tbl_Identity / tbl_IdentityMembership	使用者資訊



---

🛠 2. 使用 SQL 查詢資料（範例）

▶ 每日 Commit 數量統計

SELECT 
    CAST(c.CreationDate AS DATE) AS CommitDate,
    COUNT(*) AS CommitCount
FROM tbl_Commit c
GROUP BY CAST(c.CreationDate AS DATE)
ORDER BY CommitDate

▶ 每位開發者的 PR 數量

SELECT 
    pr.CreatedBy,
    COUNT(*) AS PRCount
FROM tbl_PullRequest pr
GROUP BY pr.CreatedBy
ORDER BY PRCount DESC

▶ 每日 Build 成功率

SELECT 
    CAST(b.StartTime AS DATE) AS BuildDate,
    SUM(CASE WHEN b.Status = 2 THEN 1 ELSE 0 END) AS SuccessCount,
    SUM(CASE WHEN b.Status != 2 THEN 1 ELSE 0 END) AS FailCount
FROM tbl_Build b
GROUP BY CAST(b.StartTime AS DATE)
ORDER BY BuildDate


---

📈 3. 在 Tableau 匯入 TFS 資料

步驟如下：

1. 開啟 Tableau Desktop


2. 點選「Microsoft SQL Server」作為資料來源


3. 輸入 SQL Server 主機位址、資料庫名稱、認證


4. 將 SQL 查詢結果當成資料表匯入


5. 建立圖表（如 Line Chart、Bar Chart、Heatmap 等）




---

❗注意事項

權限問題：你需有 Tfs_Collection 資料庫的 讀取權限（SELECT）

表格結構：TFS 資料庫不是公開 schema，有些欄位命名混亂，需要比對

SQL 效能：不要直接 SELECT *，會拖慢 Tableau 和 DB



---

如果你不是 TFS 是用雲端 Azure DevOps Service？

那就無法抓 DB，只能改用：

✅ Analytics View（使用 OData）

✅ REST API + 自動寫入資料庫



---

你可以告訴我以下幾件事，我能幫你生出 SQL 查詢或整合架構：

1. 你是用 TFS (On-prem) 還是 Azure DevOps Services (雲端)？


2. 想從 DB 抓哪些資料？（如 PR、Commit、Build、Release）


3. 你目前 Tableau 是 Desktop 還是有 Server？


4. 你有 SQL Server 權限嗎？要連的是哪個 collection？



提供後我可以直接幫你產生查詢語法或資料表 join 架構。

