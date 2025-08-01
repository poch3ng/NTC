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