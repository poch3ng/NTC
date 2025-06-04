以下是針對 TFS (On-Premises) 資料庫結構中，你可用來製作 Tableau 圖表的 SQL 資料表與查詢範例，主要針對 Git Commit、Build、Release、PR 等資料。我將提供更具體的表名、說明與可視覺化方向，方便給處長參考。


---

🔍 一、Git 相關（Commit / Pull Request）

✅ 表格與說明：

表格名稱	說明

tbl_Commit	Git commit 資訊，如時間、提交者、Repo 等
tbl_Changeset	TFS 的版本控制（非 Git）用
tbl_PullRequest	PR 記錄：標題、建立人、狀態、建立/完成時間
tbl_Repository	對應的 Git Repo 資訊


📈 圖表建議 & SQL 範例：

1️⃣ 每日 Commit 數量（折線圖）

SELECT 
    CAST(c.CreationDate AS DATE) AS CommitDate,
    COUNT(*) AS CommitCount
FROM tbl_Commit c
GROUP BY CAST(c.CreationDate AS DATE)
ORDER BY CommitDate

2️⃣ 開發者 Commit 排行（橫條圖）

SELECT 
    c.Committer AS Developer,
    COUNT(*) AS CommitCount
FROM tbl_Commit c
GROUP BY c.Committer
ORDER BY CommitCount DESC

3️⃣ PR 合併平均時長（箱型圖）

SELECT 
    pr.PullRequestId,
    pr.CreatedDate,
    pr.ClosedDate,
    DATEDIFF(HOUR, pr.CreatedDate, pr.ClosedDate) AS MergeHours
FROM tbl_PullRequest pr
WHERE pr.Status = 3  -- 3 表示已合併


---

🔧 二、Build（CI）

✅ 表格與說明：

表格名稱	說明

tbl_Build	每次建置紀錄：起始時間、結果、定義、觸發人
tbl_BuildDefinition	建置流程定義資訊
tbl_BuildRequest	建置請求來源（可追蹤觸發者）


📈 圖表建議 & SQL 範例：

4️⃣ 每日建置成功 vs 失敗（堆疊圖）

SELECT 
    CAST(b.StartTime AS DATE) AS BuildDate,
    COUNT(CASE WHEN b.Status = 2 THEN 1 END) AS SuccessCount,
    COUNT(CASE WHEN b.Status != 2 THEN 1 END) AS FailCount
FROM tbl_Build b
GROUP BY CAST(b.StartTime AS DATE)
ORDER BY BuildDate

5️⃣ 每位開發者觸發建置次數（橫條圖）

SELECT 
    br.RequestedBy AS Developer,
    COUNT(*) AS TriggerCount
FROM tbl_BuildRequest br
GROUP BY br.RequestedBy
ORDER BY TriggerCount DESC


---

🚀 三、Release（CD）

✅ 表格與說明：

表格名稱	說明

tbl_ReleaseDeployment	部署執行記錄
tbl_ReleaseEnvironment	部署的環境
tbl_Release	Release 主表，含版本、建立人、時間等資訊


📈 圖表建議 & SQL 範例：

6️⃣ 每週部署次數（折線圖）

SELECT 
    DATEPART(YEAR, r.CreatedOn) AS Year,
    DATEPART(WEEK, r.CreatedOn) AS Week,
    COUNT(*) AS DeploymentCount
FROM tbl_Release r
GROUP BY DATEPART(YEAR, r.CreatedOn), DATEPART(WEEK, r.CreatedOn)
ORDER BY Year, Week

7️⃣ 每個環境的部署成功率（堆疊長條圖）

SELECT 
    re.EnvironmentName,
    COUNT(CASE WHEN rd.Status = 'succeeded' THEN 1 END) AS SuccessCount,
    COUNT(CASE WHEN rd.Status != 'succeeded' THEN 1 END) AS FailCount
FROM tbl_ReleaseDeployment rd
JOIN tbl_ReleaseEnvironment re ON rd.EnvironmentId = re.EnvironmentId
GROUP BY re.EnvironmentName


---

📌 四、彙整建議的圖表供處長參考

圖表	重點洞察

🔹 每日 Commit 折線圖	團隊活躍程度
🔹 PR 合併所需時間箱型圖	協作效率
🔹 Build 成功/失敗堆疊圖	CI 穩定性
🔹 開發者觸發建置數排行	流程使用分布
🔹 每週部署次數趨勢圖	CD 頻率與變化
🔹 各環境部署成功率	品質控制狀況



---

❓你可以回覆我：

目前你的資料庫叫什麼名字（如 Tfs_DefaultCollection）？

你要用 SQL Server Management Studio (SSMS) 還是直接連 Tableau？

想先實作哪幾個圖表？我可以幫你一次產出 SQL。


我可以幫你整理出「整合用 View」或建立 ETL 流程，也可給你完整的 Tableau 資料來源設定流程。需要我這樣做嗎？



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

