以下整理兩種情境對應的操作方式，供你判斷適用哪一種：


---

情境一：整個 DevOps 搬到新 SQL Server（建議）

用途：若你所有 Collection DB + Configuration DB 都要搬到新 SQL Server（NTCOADB02,14332）

步驟：

1. 停用 DevOps 服務

services.msc → 停用：
- Visual Studio Team Foundation Background Job Agent
- World Wide Web Publishing Service（或 IIS）

2. Remap 所有資料庫（包含 Configuration 和所有 Collection）

TFSConfig RemapDBs /DatabaseName:AzureDevops_Configuration /SQLInstances:"ntcoat068\devops;NTCOADB02,14332" /UseTrustedConnection

3. 重新註冊 Application Tier

TFSConfig RegisterDB /SQLInstance:"NTCOADB02,14332" /DatabaseName:AzureDevops_Configuration /UseTrustedConnection

4. 啟用服務 & 驗證 DevOps 正常


---

情境二：只搬移部分 Collection，其他維持原狀

用途：只把某些 Collection 移動到新 SQL Server，其餘仍留在舊的 SQL Server。

步驟：

A. 移除舊 Collection（可用 UI 或指令）

TFSServiceControl quiesce
TFSConfig DetachDB /DatabaseName:AzureDevops_Collection_XX /SQLInstance:ntcoat068\devops

B. 將資料庫搬移到新 SQL Server（透過 SQL 備份還原）

C. 在新 SQL Server Attach Collection

TFSConfig AttachDB /DatabaseName:AzureDevops_Collection_XX /SQLInstance:NTCOADB02,14332 /UseTrustedConnection

D. 恢復服務

TFSServiceControl unquiesce


---

如何選擇？

情況	建議做法

所有資料庫都要搬走	情境一（RemapDBs）
有些還要留在舊伺服器	情境二（Detach + Attach）



---

若你確定只是一個 Collection 需要搬家，不要使用 RemapDBs，會連 Configuration 都改掉。


---

如需進一步協助你列出目前有哪些 collection，以及給出具體 SQL 還原與連線設定方式，也可以提供清單，我幫你補上。是否要這樣做？

