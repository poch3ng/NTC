# 確保 PowerShell 使用 UTF-8 編碼
$PSDefaultParameterValues['*:Encoding'] = 'utf8'

# 定義 Azure DevOps 組織和專案資訊
$organization = "YourOrganizationName"  # 組織名稱（通常是英文）
$project = "你的中文專案名稱"          # 替換為實際的中文專案名稱
$pat = "YourPersonalAccessToken"       # 你的個人存取令牌

# 將 PAT 轉換為 Base64 格式
$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$pat"))

# 計算三十天前的日期
$thirtyDaysAgo = (Get-Date).AddDays(-30).ToString("yyyy-MM-dd")

# 定義 WIQL 查詢，查詢開始日期前三十天的工作項目
$wiqlQuery = @"
SELECT [System.Id], [System.Title], [System.WorkItemType], [Microsoft.VSTS.Scheduling.StartDate]
FROM WorkItems
WHERE [Microsoft.VSTS.Scheduling.StartDate] <= '$thirtyDaysAgo'
AND [System.TeamProject] = '$project'
ORDER BY [Microsoft.VSTS.Scheduling.StartDate] DESC
"@

# 定義 REST API 端點
$uri = "https://dev.azure.com/$organization/$project/_apis/wit/wiql?api-version=7.0"

# 準備查詢 Body，確保使用 UTF-8 編碼
$body = @{ query = $wiqlQuery } | ConvertTo-Json -Depth 10 -Compress

# 執行 WIQL 查詢以獲取工作項目 ID
$response = Invoke-RestMethod -Uri $uri `
    -Method Post `
    -Headers @{ Authorization = "Basic $base64AuthInfo" } `
    -ContentType "application/json; charset=utf-8" `
    -Body ([System.Text.Encoding]::UTF8.GetBytes($body))

# 檢查是否有工作項目
if ($response.workItems.Count -eq 0) {
    Write-Host "沒有找到符合條件的工作項目。"
    exit
}

# 獲取工作項目詳細資訊
$workItemIds = $response.workItems.id -join ","
$workItemUri = "https://dev.azure.com/$organization/_apis/wit/workitems?ids=$workItemIds&api-version=7.0"

$workItems = Invoke-RestMethod -Uri $workItemUri `
    -Method Get `
    -Headers @{ Authorization = "Basic $base64AuthInfo" } `
    -ContentType "application/json; charset=utf-8"

# 輸出結果
foreach ($item in $workItems.value) {
    Write-Host "工作項目 ID: $($item.id)"
    Write-Host "標題: $($item.fields.'System.Title')"
    Write-Host "開始日期: $($item.fields.'Microsoft.VSTS.Scheduling.StartDate')"
    Write-Host "類型: $($item.fields.'System.WorkItemType')"
    Write-Host "-----------------------------------"
}

要使用 PowerShell 在 Azure DevOps 中抓取工作項目開始日期前三十天的工作項目，你可以使用 Azure DevOps 的 REST API 搭配 WIQL (Work Item Query Language) 來查詢。以下是一個完整的 PowerShell 腳本，示範如何達成這個需求：

### 前提條件
1. **個人存取令牌 (PAT)**：你需要在 Azure DevOps 中建立一個具有讀取工作項目權限的 PAT。
2. **組織與專案資訊**：需要知道你的 Azure DevOps 組織名稱和專案名稱。
3. **PowerShell 環境**：確保你的環境可以執行 PowerShell 腳本。

### PowerShell 腳本範例
以下腳本會查詢開始日期 (`Microsoft.VSTS.Scheduling.StartDate`) 在當前日期前三十天的工作項目：

```powershell
# 定義 Azure DevOps 組織和專案資訊
$organization = "YourOrganizationName"
$project = "YourProjectName"
$pat = "YourPersonalAccessToken"

# 將 PAT 轉換為 Base64 格式
$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$pat"))

# 計算三十天前的日期
$thirtyDaysAgo = (Get-Date).AddDays(-30).ToString("yyyy-MM-dd")

# 定義 WIQL 查詢
$wiqlQuery = @"
SELECT [System.Id], [System.Title], [System.WorkItemType], [Microsoft.VSTS.Scheduling.StartDate]
FROM WorkItems
WHERE [Microsoft.VSTS.Scheduling.StartDate] <= '$thirtyDaysAgo'
AND [System.TeamProject] = '$project'
ORDER BY [Microsoft.VSTS.Scheduling.StartDate] DESC
"@

# 定義 REST API 端點
$uri = "https://dev.azure.com/$organization/$project/_apis/wit/wiql?api-version=7.0"

# 準備查詢 Body
$body = @{ query = $wiqlQuery } | ConvertTo-Json

# 執行 WIQL 查詢以獲取工作項目 ID
$response = Invoke-RestMethod -Uri $uri `
    -Method Post `
    -Headers @{ Authorization = "Basic $base64AuthInfo" } `
    -ContentType "application/json" `
    -Body $body

# 檢查是否有工作項目
if ($response.workItems.Count -eq 0) {
    Write-Host "沒有找到符合條件的工作項目。"
    exit
}

# 獲取工作項目詳細資訊
$workItemIds = $response.workItems.id -join ","
$workItemUri = "https://dev.azure.com/$organization/_apis/wit/workitems?ids=$workItemIds&api-version=7.0"

$workItems = Invoke-RestMethod -Uri $workItemUri `
    -Method Get `
    -Headers @{ Authorization = "Basic $base64AuthInfo" } `
    -ContentType "application/json"

# 輸出結果
foreach ($item in $workItems.value) {
    Write-Host "工作項目 ID: $($item.id)"
    Write-Host "標題: $($item.fields.'System.Title')"
    Write-Host "開始日期: $($item.fields.'Microsoft.VSTS.Scheduling.StartDate')"
    Write-Host "類型: $($item.fields.'System.WorkItemType')"
    Write-Host "-----------------------------------"
}
```

### 腳本說明
1. **設定基本資訊**：
   - `$organization`：你的 Azure DevOps 組織名稱。
   - `$project`：你的專案名稱。
   - `$pat`：你的個人存取令牌。

2. **計算日期**：
   - 使用 `(Get-Date).AddDays(-30)` 計算三十天前的日期，並格式化為 `yyyy-MM-dd` 以符合 WIQL 查詢需求。

3. **WIQL 查詢**：
   - 查詢語句選取 `System.Id`、`System.Title`、`System.WorkItemType` 和 `Microsoft.VSTS.Scheduling.StartDate`。
   - 使用 `Microsoft.VSTS.Scheduling.StartDate <= '$thirtyDaysAgo'` 篩選開始日期在三十天前的工作項目。
   - `System.TeamProject = '$project'` 確保只查詢指定專案的工作項目。

4. **REST API 呼叫**：
   - 第一次呼叫 `/wit/wiql` 端點以執行 WIQL 查詢，獲取工作項目 ID。
   - 第二次呼叫 `/wit/workitems` 端點以獲取詳細的工作項目資訊。

5. **輸出結果**：
   - 迴圈遍歷每個工作項目，輸出其 ID、標題、開始日期和類型。

### 注意事項
- **開始日期欄位**：`Microsoft.VSTS.Scheduling.StartDate` 可能不是所有工作項目類型都預設啟用。如果你的工作項目沒有這個欄位，可能需要檢查專案的流程設定，確保該欄位已添加到工作項目類型中。
- **PAT 權限**：確保 PAT 具有「Work Items - Read」權限。
- **API 版本**：腳本使用 API 版本 7.0，確認你的 Azure DevOps 版本是否支援。
- **錯誤處理**：為生產環境使用，建議添加錯誤處理邏輯，例如檢查 API 回應狀態碼或處理網路問題。
- **日期格式**：WIQL 查詢需要日期格式為 `yyyy-MM-dd`，確保一致性。

### 測試與驗證
1. 將腳本中的 `$organization`、`$project` 和 `$pat` 替換為實際值。
2. 執行腳本，檢查是否正確輸出工作項目。
3. 如果沒有結果，確認：
   - 工作項目是否包含 `StartDate` 欄位。
   - WIQL 查詢語法是否正確。
   - 專案中是否存在符合條件的數據。

### 替代方案
如果不希望使用 REST API，也可以在 Azure DevOps 的查詢編輯器中手動建立查詢，然後使用 `az boards query` CLI 命令執行查詢並匯出結果。例如：

```bash
az boards query --wiql "SELECT [System.Id], [System.Title], [Microsoft.VSTS.Scheduling.StartDate] FROM WorkItems WHERE [Microsoft.VSTS.Scheduling.StartDate] <= @Today - 30 AND [System.TeamProject] = 'YourProjectName'" --output json
```

然後用 PowerShell 解析 JSON 輸出。

### 參考資料
- [Azure DevOps REST API - Work Items](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/?view=azure-devops-rest-7.0)[](https://learn.microsoft.com/en-us/azure/devops/integrate/quickstarts/work-item-quickstart?view=azure-devops)
- [WIQL 語法參考](https://learn.microsoft.com/en-us/azure/devops/boards/queries/wiql-syntax?view=azure-devops)[](https://learn.microsoft.com/en-us/azure/devops/boards/queries/query-by-date-or-current-iteration?view=azure-devops)

如果有其他問題或需要進一步客製化，請告訴我！