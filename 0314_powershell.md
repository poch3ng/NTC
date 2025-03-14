$DeployFolder = "C:\Your\Target\Folder"

# 檢查資料夾是否存在，若不存在則建立
if (!(Test-Path -Path $DeployFolder)) {
    New-Item -ItemType Directory -Path $DeployFolder | Out-Null
}

# 複製檔案
Copy-Item -Path "C:\Source\*" -Destination $DeployFolder -Recurse