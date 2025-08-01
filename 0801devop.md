你的問題是 Azure DevOps API 在查詢工作項目時，請求了 230 個工作項目，但遇到了單次 API 呼叫最多只能返回 200 個工作項目的限制。這個限制是 Azure DevOps REST API 的內建行為，特別是在使用 `Work Items - List` 或 `Work Items - Get Work Items Batch` 端點時。以下是解決方案，針對你的中文專案名稱和 PowerShell 環境進行調整，確保能完整抓取超過 200 個的工作項目。

### 問題背景
根據 Azure DevOps 官方文檔，`Work Items - Get Work Items Batch` API 一次最多支援 200 個工作項目 ID。 如果你的查詢結果超過 200 個（如 230 個），需要分批處理。以下腳本會透過分批查詢來解決這個問題，並考慮到你的專案名稱是中文。[](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/work-items/get-work-items-batch?view=azure-devops-rest-7.1)

### 解決方案：分批查詢工作項目
以下是修改後的 PowerShell 腳本，基於你提供的原始需求（查詢開始日期前三十天的工作項目），並解決 200 個工作項目的限制：

```powershell
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

# 分批處理工作項目 ID（每批最多 200 個）
$batchSize = 200
$workItemIds = $response.workItems.id
$totalItems = $workItemIds.Count
$allWorkItems = @()

Write-Host "總共找到 $totalItems 個工作項目。"

for ($i = 0; $i -lt $totalItems; $i += $batchSize) {
    # 取得當前批次的 ID
    $batchIds = $workItemIds[$i..[math]::Min($i + $batchSize - 1, $totalItems - 1)] -join ","
    $workItemUri = "https://dev.azure.com/$organization/_apis/wit/workitems?ids=$batchIds&api-version=7.0"

    # 查詢當前批次的工作項目詳細資訊
    $batchResponse = Invoke-RestMethod -Uri $workItemUri `
        -Method Get `
        -Headers @{ Authorization = "Basic $base64AuthInfo" } `
        -ContentType "application/json; charset=utf-8"

    # 將結果添加到總集合
    $allWorkItems += $batchResponse.value

    Write-Host "已處理 $($i + [math]::Min($batchSize, $totalItems - $i)) / $totalItems 個工作項目。"
}

# 輸出結果
foreach ($item in $allWorkItems) {
    Write-Host "工作項目 ID: $($item.id)"
    Write-Host "標題: $($item.fields.'System.Title')"
    Write-Host "開始日期: $($item.fields.'Microsoft.VSTS.Scheduling.StartDate')"
    Write-Host "類型: $($item.fields.'System.WorkItemType')"
    Write-Host "-----------------------------------"
}

# 選擇性：將結果匯出到 CSV
$allWorkItems | Select-Object @{Name='ID';Expression={$_.id}}, 
                              @{Name='Title';Expression={$_.fields.'System.Title'}}, 
                              @{Name='StartDate';Expression={$_.fields.'Microsoft.VSTS.Scheduling.StartDate'}}, 
                              @{Name='Type';Expression={$_.fields.'System.WorkItemType'}} | 
    Export-Csv -Path "WorkItems.csv" -NoTypeInformation -Encoding UTF8
Write-Host "結果已匯出至 WorkItems.csv"
```

### 腳本說明
1. **分批處理**：
   - 使用 `$batchSize = 200` 定義每批次最多 200 個工作項目。
   - 透過 `for` 迴圈將工作項目 ID 分成多批，每批最多 200 個，使用陣列切片 (`$workItemIds[$i..[math]::Min($i + $batchSize - 1, $totalItems - 1)]`)。
   - 對每批 ID 呼叫 `Work Items - List` API，獲取詳細資訊。

2. **中文專案名稱與編碼**：
   - 腳本保持 UTF-8 編碼設定，確保中文專案名稱正確處理。
   - `$project` 直接使用中文名稱，API 會自動處理 URL 編碼。

3. **進度提示**：
   - 增加進度提示，顯示已處理的工作項目數量（例如「已處理 200 / 230 個工作項目」）。

4. **匯出 CSV**：
   - 添加將結果匯出到 CSV 的功能，檔案編碼為 UTF-8，方便後續分析。

### 如何使用
1. **替換參數**：
   - `$organization`：你的 Azure DevOps 組織名稱。
   - `$project`：你的中文專案名稱，例如 `"我的專案"`。
   - `$pat`：你的個人存取令牌。

2. **保存腳本**：
   - 將腳本保存為 `.ps1` 檔案（例如 `GetWorkItemsBatch.ps1`），確保檔案編碼為 UTF-8。

3. **執行腳本**：
   ```powershell
   .\GetWorkItemsBatch.ps1
   ```

4. **檢查輸出**：
   - 腳本會在控制台輸出每個工作項目的詳細資訊。
   - 結果會匯出到 `WorkItems.csv` 檔案，位於腳本執行目錄。

### 解決 200 限制的關鍵
- **WIQL 查詢**：WIQL 查詢本身可能返回超過 20,000 個工作項目 ID，但本腳本已處理後續的詳細查詢限制（200 個/批次）。[](https://www.azuredevopsguide.com/vs402337-the-number-of-work-items-returned-exceeds-the-size-limit-of-20000-azure-devops/)
- **分批 API 呼叫**：透過將 ID 分批，每批不超過 200 個，繞過 `Work Items - Get Work Items Batch` 的限制。[](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/work-items/get-work-items-batch?view=azure-devops-rest-7.1)
- **進度追蹤**：分批處理確保即使有更多工作項目（例如 1,000 個），也能穩定執行。

### 其他注意事項
- **速率限制**：Azure DevOps 有 API 速率限制（預設每分鐘 60 個請求，或 200 TSTU 每 5 分鐘）。如果查詢頻繁，建議在迴圈中添加延遲（例如 `Start-Sleep -Milliseconds 500`）。[](https://www.restack.io/p/azure-devops-knowledge-answer-work-items-limit)[](https://learn.microsoft.com/en-us/azure/devops/integrate/concepts/rate-limits?view=azure-devops)
- **欄位確認**：確保工作項目包含 `Microsoft.VSTS.Scheduling.StartDate` 欄位。如果部分工作項目缺少此欄位，可能導致輸出中缺少該欄位值。
- **查詢優化**：如果 230 個工作項目只是部分結果，建議在 WIQL 中添加更多篩選條件（例如工作項目類型或狀態），以減少結果數量，提高性能。[](https://www.restack.io/p/azure-devops-knowledge-answer-work-items-limit)
- **錯誤處理**：為生產環境使用，建議添加 try-catch 塊來處理 API 錯誤，例如：
  ```powershell
  try {
      $batchResponse = Invoke-RestMethod -Uri $workItemUri -Method Get -Headers @{ Authorization = "Basic $base64AuthInfo" } -ContentType "application/json; charset=utf-8"
  } catch {
      Write-Host "API 呼叫失敗: $($_.Exception.Message)"
      exit
  }
  ```

### 替代方案
如果不希望自行分批處理，可以考慮以下方法：
1. **Azure DevOps 查詢 UI**：
   - 在 Azure DevOps 的 Boards > Queries 中手動建立相同的 WIQL 查詢，然後匯出結果為 CSV。
   - 缺點：不適合自動化，且匯出可能受限於 2,000 個工作項目。[](https://www.restack.io/p/azure-devops-knowledge-answer-work-items-limit)

2. **Azure CLI**：
   - 使用 `az boards query` 執行 WIQL 查詢，並處理結果：
     ```bash
     az boards query --wiql "SELECT [System.Id], [System.Title], [Microsoft.VSTS.Scheduling.StartDate] FROM WorkItems WHERE [Microsoft.VSTS.Scheduling.StartDate] <= @Today - 30 AND [System.TeamProject] = '你的中文專案名稱'" --output json > output.json
     ```
   - 然後用 PowerShell 分批查詢詳細資訊，類似上述腳本。

3. **Power BI 整合**：
   - 使用 Power BI 連接到 Azure DevOps，執行 WIQL 查詢並處理分批邏輯。Power BI 內建分頁功能，可自動處理 200 個工作項目的限制。[](https://endjin.com/blog/2016/07/querying-the-vsts-work-items-api-directly-from-power-bi)

### 參考資料
- Azure DevOps REST API - Work Items Batch: https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/work-items/get-work-items-batch[](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/work-items/get-work-items-batch?view=azure-devops-rest-7.1)
- Azure DevOps 查詢限制: https://learn.microsoft.com/en-us/azure/devops/boards/queries/query-limits[](https://www.restack.io/p/azure-devops-knowledge-answer-work-items-limit)
- WIQL 語法: https://learn.microsoft.com/en-us/azure/devops/boards/queries/wiql-syntax[](https://www.azuredevopsguide.com/vs402337-the-number-of-work-items-returned-exceeds-the-size-limit-of-20000-azure-devops/)

如果還有其他問題（例如需要特定欄位、匯出格式調整或處理更多工作項目），請告訴我，我可以進一步優化腳本！

如果你的 Azure DevOps 專案名稱是中文，PowerShell 腳本仍然適用，但需要確保正確處理中文字符編碼。Azure DevOps 的 REST API 支援中文專案名稱，只需在腳本中正確設定 `$project` 變數，並確保 PowerShell 環境使用 UTF-8 編碼來處理中文。以下是針對中文專案名稱的調整與完整腳本：

### 注意事項
1. **編碼問題**：PowerShell 預設可能使用非 UTF-8 編碼，需確保腳本檔案和 PowerShell 環境支援 UTF-8。
2. **專案名稱**：中文專案名稱直接填入 `$project` 變數，API 會正確處理。
3. **日期格式**：與英文專案名稱相同，WIQL 查詢中的日期格式仍為 `yyyy-MM-dd`。

### 調整後的 PowerShell 腳本
以下是針對中文專案名稱的 PowerShell 腳本，查詢開始日期前三十天的工作項目：

```powershell
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
```

### 關鍵調整
1. **UTF-8 編碼**：
   - 腳本開頭添加 `$PSDefaultParameterValues['*:Encoding'] = 'utf8'`，確保 PowerShell 輸出和處理使用 UTF-8。
   - 在 `Invoke-RestMethod` 的 `ContentType` 中明確指定 `charset=utf-8`。
   - 使用 `[System.Text.Encoding]::UTF8.GetBytes($body)` 確保查詢 Body 正確編碼。

2. **中文專案名稱**：
   - 將 `$project` 設為你的中文專案名稱，例如 `"我的專案"`。
   - Azure DevOps API 會自動處理 URL 編碼，無需手動轉換中文。

3. **腳本檔案編碼**：
   - 確保腳本檔案本身保存為 UTF-8 格式（在編輯器如 VS Code 中，選擇「UTF-8 with BOM」或「UTF-8」）。
   - 如果使用 PowerShell ISE，可能需要額外確認編碼設置。

### 執行步驟
1. 將 `$organization`、`$project` 和 `$pat` 替換為實際值，例如：
   - `$organization = "myorg"`
   - `$project = "我的專案名稱"`
   - `$pat = "your_personal_access_token"`
2. 將腳本保存為 `.ps1` 檔案（例如 `GetWorkItems.ps1`），確保檔案編碼為 UTF-8。
3. 在 PowerShell 中執行：
   ```powershell
   .\GetWorkItems.ps1
   ```

### 常見問題與解決方法
- **問題 1**：腳本報錯「無法識別專案名稱」或亂碼。
  - **解決**：檢查腳本檔案是否為 UTF-8 編碼。在 VS Code 中，點擊右下角的編碼選項，選擇「Save with Encoding」並選 UTF-8。
  - 確認 PowerShell 環境使用 UTF-8，可運行以下命令檢查：
    ```powershell
    $PSDefaultParameterValues['*:Encoding']
    ```
    如果不是 `utf8`，執行 `$PSDefaultParameterValues['*:Encoding'] = 'utf8'`。

- **問題 2**：查詢無結果。
  - **解決**：
    - 確認專案中工作項目是否有 `Microsoft.VSTS.Scheduling.StartDate` 欄位。
    - 在 Azure DevOps 的查詢編輯器中測試相同的 WIQL，確保語法正確。
    - 檢查 PAT 是否有「Work Items - Read」權限。

- **問題 3**：API 回應 400 或 404 錯誤。
  - **解決**：
    - 檢查 `$organization` 和 `$project` 是否正確。
    - 確保 API 版本 (`api-version=7.0`) 與你的 Azure DevOps 版本相容。

### 替代方法：使用 Azure CLI
如果不想直接使用 REST API，可以使用 Azure CLI 的 `az boards query` 命令：
```bash
az boards query --wiql "SELECT [System.Id], [System.Title], [Microsoft.VSTS.Scheduling.StartDate] FROM WorkItems WHERE [Microsoft.VSTS.Scheduling.StartDate] <= @Today - 30 AND [System.TeamProject] = '你的中文專案名稱'" --output json > output.json
```
然後用 PowerShell 解析 `output.json`：
```powershell
$results = Get-Content -Path "output.json" -Encoding UTF8 | ConvertFrom-Json
foreach ($item in $results) {
    Write-Host "ID: $($item.id), 標題: $($item.fields.'System.Title'), 開始日期: $($item.fields.'Microsoft.VSTS.Scheduling.StartDate')"
}
```

### 參考資料
- [Azure DevOps REST API - WIQL](https://learn.microsoft.com/en-us/rest/api/azure/devops/wit/wiql?view=azure-devops-rest-7.0)
- [PowerShell UTF-8 編碼處理](https://learn.microsoft.com/en-us/powershell/module/microsoft.powershell.core/about/about_Character_Encoding?view=powershell-7.3)

如果還有其他問題或需要進一步協助（例如輸出到 CSV 或篩選特定工作項目類型），請告訴我！