如果 SCH 代表需求日期，而 Email 內容是 英文，那麼我們需要：

1. 修改查詢 讓 SCH05（需求日期） 正確對應 PO1


2. 修改 Email 內容 顯示 需求日期 而不是排程數量


3. 保持 COMMIT 之後才發送 Email




---

1. SQL 查詢 - 獲取 PO1 與需求日期 (SCH05)

這個查詢會：

查詢 PO1 (LoopType = 'PO1')

查詢需求日期 (SCH05)

確保每個 PO1 產品能獲得對應的需求日期


Function GenerateEdiEmailContent(mainId As Integer, connection As SqlConnection) As String
    Dim query As String = "
        SELECT 
            e.St01 AS OrderType, e.Beg03 AS PurchaseOrderNumber, 
            e.Beg05 AS ContractNumber, i.Po107 AS ProductCode, 
            i.Po102 AS Quantity, i.Po104 AS UnitPrice, i.Po109 AS Amount,
            sch.Po102 AS RequirementDate
        FROM EdiMain e
        LEFT JOIN EdiDetailItem i ON e.MainId = i.MainId AND i.LoopType = 'PO1'
        LEFT JOIN EdiDetailItem sch ON i.MainId = sch.MainId 
             AND i.LoopSequence = sch.LoopSequence
             AND sch.LoopType = 'SCH'
        WHERE e.MainId = @MainId
        ORDER BY i.LoopSequence, sch.LoopSequence"
    
    Dim body As New StringBuilder()
    Dim orderType As String = ""
    Dim purchaseOrderNumber As String = ""
    Dim contractNumber As String = ""

    Using cmd As New SqlCommand(query, connection)
        cmd.Parameters.AddWithValue("@MainId", mainId)
        Using reader As SqlDataReader = cmd.ExecuteReader()
            While reader.Read()
                ' 訂單主資訊 (只設置一次)
                If orderType = "" Then
                    orderType = reader("OrderType").ToString()
                    purchaseOrderNumber = reader("PurchaseOrderNumber").ToString()
                    contractNumber = reader("ContractNumber").ToString()

                    ' 設定 Email 標題
                    If orderType = "850" Then
                        body.AppendLine($"[New Order Confirmation] PO {purchaseOrderNumber} Received")
                        body.AppendLine()
                        body.AppendLine($"Purchase Order Number: {purchaseOrderNumber}")
                        body.AppendLine($"Order Type: 850 (New Order)")
                    ElseIf orderType = "860" Then
                        body.AppendLine($"[Order Change Notification] PO {purchaseOrderNumber} Updated")
                        body.AppendLine()
                        body.AppendLine($"Purchase Order Number: {purchaseOrderNumber}")
                        body.AppendLine($"Order Type: 860 (Order Change)")
                    End If

                    body.AppendLine($"Contract Number: {contractNumber}")
                    body.AppendLine()
                    body.AppendLine("【Order Details】")
                End If

                ' 讀取 PO1 明細
                Dim productCode As String = reader("ProductCode").ToString()
                Dim quantity As String = reader("Quantity").ToString()
                Dim unitPrice As String = reader("UnitPrice").ToString()
                Dim amount As String = reader("Amount").ToString()
                Dim requirementDate As String = If(IsDBNull(reader("RequirementDate")), "N/A", reader("RequirementDate").ToString())

                ' 加入 Email 內容
                body.AppendLine($"- Product Code: {productCode}")
                body.AppendLine($"  Quantity: {quantity}")
                body.AppendLine($"  Unit Price: {unitPrice}")
                body.AppendLine($"  Total Amount: {amount}")
                body.AppendLine($"  Requirement Date (SCH05): {requirementDate}")
                body.AppendLine()
            End While
        End Using
    End Using
    
    Return body.ToString()
End Function


---

2. 交易 (COMMIT 後)

這段程式碼確保：

1. 解析 EDI 並存入資料庫


2. 成功後 COMMIT


3. 直接查詢 Email 內容


4. 發送 Email



Try
    Using connection As New SqlConnection(connectionString)
        connection.Open()
        Using transaction As SqlTransaction = connection.BeginTransaction()

            ' 解析 EDI 並存入資料庫
            Dim mainId As Integer = SaveEdiData(transaction, ediData)

            ' 提交交易
            transaction.Commit()

            ' **查詢 EDI 訂單內容，直接生成 Email**
            Dim emailContent As String = GenerateEdiEmailContent(mainId, connection)

            ' **發送通知信**
            SendEmail("sales@example.com", "EDI Order Notification", emailContent)
        End Using
    End Using
Catch ex As Exception
    ' 交易失敗處理
    LogError(ex.Message)
End Try


---

3. 直接發送 Email

Sub SendEmail(toEmail As String, subject As String, body As String)
    Dim smtp As New SmtpClient("smtp.example.com")
    smtp.Credentials = New NetworkCredential("your_email@example.com", "your_password")
    smtp.EnableSsl = True
    
    Dim mail As New MailMessage()
    mail.From = New MailAddress("noreply@example.com")
    mail.To.Add(toEmail)
    mail.Subject = subject
    mail.Body = body
    
    smtp.Send(mail)
End Sub


---

4. 最終 Email 格式

850 (New Order)

[New Order Confirmation] PO PO12345 Received

Purchase Order Number: PO12345
Order Type: 850 (New Order)
Contract Number: ABC123

【Order Details】
- Product Code: ITEM001
  Quantity: 100
  Unit Price: 50
  Total Amount: 5000
  Requirement Date (SCH05): 2025-02-10

- Product Code: ITEM002
  Quantity: 200
  Unit Price: 30
  Total Amount: 6000
  Requirement Date (SCH05): 2025-02-12

860 (Order Change)

[Order Change Notification] PO PO12345 Updated

Purchase Order Number: PO12345
Order Type: 860 (Order Change)
Contract Number: ABC123

【Order Details】
- Product Code: ITEM001
  Quantity: 150
  Unit Price: 45
  Total Amount: 6750
  Requirement Date (SCH05): 2025-02-15

- Product Code: ITEM003
  Quantity: 50



如果你不想要額外寫 Class，直接獲取 Email 內容，那可以直接 在 SQL 查詢後組合 Email 內容，避免使用 EdiOrderInfo 和 EdiOrderItem。


---

1. 直接產生 Email 內容

這個方法：

COMMIT 後查詢資料

直接在迴圈組合 Email 內容

不使用 EdiOrderInfo、EdiOrderItem


Function GenerateEdiEmailContent(mainId As Integer, connection As SqlConnection) As String
    Dim query As String = "
        SELECT 
            e.St01 AS OrderType, e.Beg03 AS PurchaseOrderNumber, 
            e.Beg05 AS ContractNumber, i.Po107 AS ProductCode, 
            i.Po102 AS Quantity, i.Po104 AS UnitPrice, i.Po109 AS Amount,
            sch.Po102 AS ScheduleQuantity
        FROM EdiMain e
        LEFT JOIN EdiDetailItem i ON e.MainId = i.MainId AND i.LoopType = 'PO1'
        LEFT JOIN EdiDetailItem sch ON i.MainId = sch.MainId 
             AND i.LoopSequence = sch.LoopSequence
             AND sch.LoopType = 'SCH'
        WHERE e.MainId = @MainId
        ORDER BY i.LoopSequence, sch.LoopSequence"
    
    Dim body As New StringBuilder()
    Dim orderType As String = ""
    Dim purchaseOrderNumber As String = ""
    Dim contractNumber As String = ""

    Using cmd As New SqlCommand(query, connection)
        cmd.Parameters.AddWithValue("@MainId", mainId)
        Using reader As SqlDataReader = cmd.ExecuteReader()
            While reader.Read()
                ' 訂單主資訊 (只設置一次)
                If orderType = "" Then
                    orderType = reader("OrderType").ToString()
                    purchaseOrderNumber = reader("PurchaseOrderNumber").ToString()
                    contractNumber = reader("ContractNumber").ToString()

                    ' 設定 Email 標題
                    If orderType = "850" Then
                        body.AppendLine($"[新訂單通知] 訂單 {purchaseOrderNumber} 已收到")
                        body.AppendLine()
                        body.AppendLine($"訂單號碼: {purchaseOrderNumber}")
                        body.AppendLine($"訂單類型: 850 (新訂單)")
                    ElseIf orderType = "860" Then
                        body.AppendLine($"[訂單變更通知] 訂單 {purchaseOrderNumber} 已變更")
                        body.AppendLine()
                        body.AppendLine($"訂單號碼: {purchaseOrderNumber}")
                        body.AppendLine($"訂單類型: 860 (變更訂單)")
                    End If

                    body.AppendLine($"合約編號: {contractNumber}")
                    body.AppendLine()
                    body.AppendLine("【產品清單】")
                End If

                ' 讀取 PO1 明細
                Dim productCode As String = reader("ProductCode").ToString()
                Dim quantity As String = reader("Quantity").ToString()
                Dim unitPrice As String = reader("UnitPrice").ToString()
                Dim amount As String = reader("Amount").ToString()
                Dim scheduleQuantity As String = If(IsDBNull(reader("ScheduleQuantity")), "無排程", reader("ScheduleQuantity").ToString())

                ' 加入 Email 內容
                body.AppendLine($"- 產品編號: {productCode}")
                body.AppendLine($"  數量: {quantity}")
                body.AppendLine($"  單價: {unitPrice}")
                body.AppendLine($"  總金額: {amount}")
                body.AppendLine($"  排程數量 (SCH05): {scheduleQuantity}")
                body.AppendLine()
            End While
        End Using
    End Using
    
    Return body.ToString()
End Function


---

2. 交易 (COMMIT 後)

這段程式碼會：

1. 解析 EDI 並存入資料庫


2. 成功後 COMMIT


3. 直接查詢 Email 內容


4. 發送 Email



Try
    Using connection As New SqlConnection(connectionString)
        connection.Open()
        Using transaction As SqlTransaction = connection.BeginTransaction()

            ' 解析 EDI 並存入資料庫
            Dim mainId As Integer = SaveEdiData(transaction, ediData)

            ' 提交交易
            transaction.Commit()

            ' **查詢 EDI 訂單內容，直接生成 Email**
            Dim emailContent As String = GenerateEdiEmailContent(mainId, connection)

            ' **發送通知信**
            SendEmail("sales@example.com", "EDI 訂單通知", emailContent)
        End Using
    End Using
Catch ex As Exception
    ' 交易失敗處理
    LogError(ex.Message)
End Try


---

3. 直接發送 Email

Sub SendEmail(toEmail As String, subject As String, body As String)
    Dim smtp As New SmtpClient("smtp.example.com")
    smtp.Credentials = New NetworkCredential("your_email@example.com", "your_password")
    smtp.EnableSsl = True
    
    Dim mail As New MailMessage()
    mail.From = New MailAddress("noreply@example.com")
    mail.To.Add(toEmail)
    mail.Subject = subject
    mail.Body = body
    
    smtp.Send(mail)
End Sub


---

4. 最終 Email 格式

850 (新訂單)

[新訂單通知] 訂單 PO12345 已收到

訂單號碼: PO12345
訂單類型: 850 (新訂單)
合約編號: ABC123

【產品清單】
- 產品編號: ITEM001
  數量: 100
  單價: 50
  總金額: 5000
  排程數量 (SCH05): 30, 50

- 產品編號: ITEM002
  數量: 200
  單價: 30
  總金額: 6000
  排程數量 (SCH05): 100

860 (訂單變更)

[訂單變更通知] 訂單 PO12345 已變更

訂單號碼: PO12345
訂單類型: 860 (變更訂單)
合約編號: ABC123

【變更後產品清單】
- 產品編號: ITEM001
  數量: 150
  單價: 45
  總金額: 6750
  排程數量 (SCH05): 50, 50, 20

- 產品編號: ITEM003
  數量: 50
  單價: 25
  總金額: 1250
  排程數量 (SCH05): 無排程


---

結論

✅ COMMIT 後直接查詢資料並組合 Email
✅ 避免寫 EdiOrderInfo、EdiOrderItem 類別，簡化程式碼
✅ 直接在 SQL 查詢後組合 Email 內容
✅ 確保 PO1 與 SCH05（排程數量）正確對應
✅ 高效能，不影響交易效能

這樣 程式碼更簡潔、直接生成 Email 內容、確保數據準確！



你想在 相同 PO1 之下 抓取 LoopType = 'SCH' 的 SCH05（排程數量），那我們需要：

1. 查詢 PO1 產品資訊


2. 查詢 LoopType = 'SCH' 的 SCH05（排程數量）


3. 將 SCH05 加入對應的 PO1 項目


4. 發送 Email，確保 PO1 與 SCH05 內容完整




---

1. 更新 SQL 查詢

這次我們要：

查詢 PO1 (LoopType = 'PO1')

查詢 SCH (LoopType = 'SCH')

確保 SCH05 與對應的 PO1 綁定

如果沒有 SCH05，則顯示「無排程」


Function GetEdiOrderDetailsWithItems(mainId As Integer, connection As SqlConnection) As EdiOrderInfo
    Dim query As String = "
        SELECT 
            e.MainId, e.St01 AS OrderType, e.Beg03 AS PurchaseOrderNumber, 
            e.Beg05 AS ContractNumber, i.Po102 AS Quantity, 
            i.Po104 AS UnitPrice, i.Po107 AS ProductCode, 
            i.Po109 AS Amount, sch.Po102 AS ScheduleQuantity
        FROM EdiMain e
        LEFT JOIN EdiDetailItem i ON e.MainId = i.MainId AND i.LoopType = 'PO1'
        LEFT JOIN EdiDetailItem sch ON i.MainId = sch.MainId 
             AND i.LoopSequence = sch.LoopSequence
             AND sch.LoopType = 'SCH'
        WHERE e.MainId = @MainId
        ORDER BY i.LoopSequence, sch.LoopSequence"
    
    Dim order As New EdiOrderInfo()
    order.ItemList = New List(Of EdiOrderItem)()

    Using cmd As New SqlCommand(query, connection)
        cmd.Parameters.AddWithValue("@MainId", mainId)
        Using reader As SqlDataReader = cmd.ExecuteReader()
            While reader.Read()
                ' 訂單主資訊 (只設定一次)
                If order.MainId = 0 Then
                    order.MainId = reader("MainId")
                    order.OrderType = reader("OrderType").ToString()
                    order.PurchaseOrderNumber = reader("PurchaseOrderNumber").ToString()
                    order.ContractNumber = reader("ContractNumber").ToString()
                End If

                ' 讀取 PO1 明細
                Dim item As New EdiOrderItem()
                item.ProductCode = reader("ProductCode").ToString()
                item.Quantity = reader("Quantity").ToString()
                item.UnitPrice = reader("UnitPrice").ToString()
                item.Amount = reader("Amount").ToString()
                item.ScheduleQuantity = If(IsDBNull(reader("ScheduleQuantity")), "無排程", reader("ScheduleQuantity").ToString())

                ' 檢查是否已存在相同 PO1 項目
                Dim existingItem = order.ItemList.FirstOrDefault(Function(x) x.ProductCode = item.ProductCode)
                If existingItem IsNot Nothing Then
                    ' PO1 已存在，只需要添加 SCH05
                    existingItem.ScheduleQuantity &= $", {item.ScheduleQuantity}"
                Else
                    ' 新 PO1 項目，加入清單
                    order.ItemList.Add(item)
                End If
            End While
        End Using
    End Using
    
    Return order
End Function


---

2. 更新 Email 內容

我們現在 每個 PO1 都會顯示對應的 SCH05（排程數量）。

Sub SendEdiConfirmationEmail(order As EdiOrderInfo)
    Dim subject As String
    Dim body As New StringBuilder()

    If order.OrderType = "850" Then
        subject = $"[新訂單通知] 訂單 {order.PurchaseOrderNumber} 已收到"
        body.AppendLine($"訂單號碼: {order.PurchaseOrderNumber}")
        body.AppendLine($"訂單類型: 850 (新訂單)")
        body.AppendLine($"合約編號: {order.ContractNumber}")
        body.AppendLine()
        body.AppendLine("【產品清單】")

    ElseIf order.OrderType = "860" Then
        subject = $"[訂單變更通知] 訂單 {order.PurchaseOrderNumber} 已變更"
        body.AppendLine($"訂單號碼: {order.PurchaseOrderNumber}")
        body.AppendLine($"訂單類型: 860 (變更訂單)")
        body.AppendLine()
        body.AppendLine("【變更後產品清單】")
    Else
        Exit Sub ' 不是 850 或 860 就不發送
    End If

    ' **新增 PO1 產品明細**
    For Each item As EdiOrderItem In order.ItemList
        body.AppendLine($"- 產品編號: {item.ProductCode}")
        body.AppendLine($"  數量: {item.Quantity}")
        body.AppendLine($"  單價: {item.UnitPrice}")
        body.AppendLine($"  總金額: {item.Amount}")
        body.AppendLine($"  排程數量 (SCH05): {item.ScheduleQuantity}")
        body.AppendLine()
    Next

    ' **發送郵件**
    SendEmail("sales@example.com", subject, body.ToString())
End Sub


---

3. 交易 (COMMIT 之後)

這段程式碼確保：

1. 解析並存入 EDI 數據


2. 成功後 COMMIT


3. 再查詢訂單資訊並發送 Email



Try
    Using connection As New SqlConnection(connectionString)
        connection.Open()
        Using transaction As SqlTransaction = connection.BeginTransaction()

            ' 解析 EDI 並存入資料庫
            Dim mainId As Integer = SaveEdiData(transaction, ediData)

            ' 提交交易
            transaction.Commit()

            ' **查詢完整訂單資訊 (含多個 PO1 與 SCH05)**
            Dim orderInfo As EdiOrderInfo = GetEdiOrderDetailsWithItems(mainId, connection)

            ' **發送通知信**
            SendEdiConfirmationEmail(orderInfo)
        End Using
    End Using
Catch ex As Exception
    ' 交易失敗處理
    LogError(ex.Message)
End Try


---

4. 更新 EdiOrderInfo 與 EdiOrderItem 資料結構

Class EdiOrderInfo
    Public Property MainId As Integer
    Public Property OrderType As String
    Public Property PurchaseOrderNumber As String
    Public Property ContractNumber As String
    Public Property ItemList As List(Of EdiOrderItem)
End Class

Class EdiOrderItem
    Public Property ProductCode As String
    Public Property Quantity As String
    Public Property UnitPrice As String
    Public Property Amount As String
    Public Property ScheduleQuantity As String ' SCH05 (排程數量)
End Class


---

5. 最終 Email 格式

850 (新訂單)

[新訂單通知] 訂單 PO12345 已收到

訂單號碼: PO12345
訂單類型: 850 (新訂單)
合約編號: ABC123

【產品清單】
- 產品編號: ITEM001
  數量: 100
  單價: 50
  總金額: 5000
  排程數量 (SCH05): 30, 50

- 產品編號: ITEM002
  數量: 200
  單價: 30
  總金額: 6000
  排程數量 (SCH05): 100

860 (訂單變更)

[訂單變更通知] 訂單 PO12345 已變更

訂單號碼: PO12345
訂單類型: 860 (變更訂單)

【變更後產品清單】
- 產品編號: ITEM001
  數量: 150
  單價: 45
  總金額: 6750
  排程數量 (SCH05): 50, 50, 20

- 產品編號: ITEM003
  數量: 50
  單價: 25
  總金額: 1250
  排程數量 (SCH05): 無排程


---

結論

✅ 查詢 LoopType = 'PO1' 的產品資訊
✅ 查詢 LoopType = 'SCH' 的 SCH05 並與對應 PO1 綁定
✅ 850 與 860 分開處理，確保通知內容正確
✅ COMMIT 之後發送 Email，確保數據成功入庫
✅ 未來可擴展，例如 CC 給多個人、Webhook、SMS 通知

這樣確保 所有 PO1 產品都會顯示對應的 SCH05 排程數量，確保 Email 完整、準確、不影響交易效能！



你的 PO1 是迴圈，表示一張訂單可能有 多個商品項目，因此需要調整查詢方式，確保每封郵件都能列出所有 PO1（產品明細）。


---

最佳解法

1. COMMIT 之後查詢 EDI 訂單資訊


2. 查詢所有 PO1 明細


3. 組合成 Email 內容


4. 發送 Email


5. 錯誤處理




---

1. 查詢 EDI 訂單（包含多個 PO1）

我們要 將多個 PO1 組合在一起，以 迴圈方式處理每個 PO1，然後將它們加到 Email 內容中。

Function GetEdiOrderDetailsWithItems(mainId As Integer, connection As SqlConnection) As EdiOrderInfo
    Dim query As String = "
        SELECT 
            e.MainId, e.St01 AS OrderType, e.Beg03 AS PurchaseOrderNumber, 
            e.Beg05 AS ContractNumber, i.Po102 AS Quantity, 
            i.Po104 AS UnitPrice, i.Po107 AS ProductCode, 
            i.Po109 AS Amount, s.Ref02 AS Discount
        FROM EdiMain e
        LEFT JOIN EdiDetailItem i ON e.MainId = i.MainId
        LEFT JOIN EdiSegment s ON e.MainId = s.MainId AND s.SegmentType = 'SAC'
        WHERE e.MainId = @MainId"
    
    Dim order As New EdiOrderInfo()
    order.ItemList = New List(Of EdiOrderItem)()

    Using cmd As New SqlCommand(query, connection)
        cmd.Parameters.AddWithValue("@MainId", mainId)
        Using reader As SqlDataReader = cmd.ExecuteReader()
            While reader.Read()
                If order.MainId = 0 Then
                    order.MainId = reader("MainId")
                    order.OrderType = reader("OrderType").ToString()
                    order.PurchaseOrderNumber = reader("PurchaseOrderNumber").ToString()
                    order.ContractNumber = reader("ContractNumber").ToString()
                    order.Discount = If(IsDBNull(reader("Discount")), "", reader("Discount").ToString())
                End If

                ' 讀取 PO1 明細
                Dim item As New EdiOrderItem()
                item.ProductCode = reader("ProductCode").ToString()
                item.Quantity = reader("Quantity").ToString()
                item.UnitPrice = reader("UnitPrice").ToString()
                item.Amount = reader("Amount").ToString()

                order.ItemList.Add(item)
            End While
        End Using
    End Using
    
    Return order
End Function


---

2. 發送 Email，顯示所有 PO1

Sub SendEdiConfirmationEmail(order As EdiOrderInfo)
    Dim subject As String
    Dim body As New StringBuilder()

    If order.OrderType = "850" Then
        subject = $"[新訂單通知] 訂單 {order.PurchaseOrderNumber} 已收到"
        body.AppendLine($"訂單號碼: {order.PurchaseOrderNumber}")
        body.AppendLine($"訂單類型: 850 (新訂單)")
        body.AppendLine($"合約編號: {order.ContractNumber}")
        body.AppendLine($"折扣: {order.Discount}")
        body.AppendLine()
        body.AppendLine("【產品清單】")

    ElseIf order.OrderType = "860" Then
        subject = $"[訂單變更通知] 訂單 {order.PurchaseOrderNumber} 已變更"
        body.AppendLine($"訂單號碼: {order.PurchaseOrderNumber}")
        body.AppendLine($"訂單類型: 860 (變更訂單)")
        body.AppendLine($"折扣: {order.Discount}")
        body.AppendLine()
        body.AppendLine("【變更後產品清單】")
    Else
        Exit Sub ' 不是 850 或 860 就不發送
    End If

    ' 迴圈加入所有 PO1 產品明細
    For Each item As EdiOrderItem In order.ItemList
        body.AppendLine($"- 產品編號: {item.ProductCode}")
        body.AppendLine($"  數量: {item.Quantity}")
        body.AppendLine($"  單價: {item.UnitPrice}")
        body.AppendLine($"  總金額: {item.Amount}")
        body.AppendLine()
    Next

    ' 發送郵件
    SendEmail("sales@example.com", subject, body.ToString())
End Sub


---

3. EdiOrderInfo 與 EdiOrderItem 資料結構

Class EdiOrderInfo
    Public Property MainId As Integer
    Public Property OrderType As String
    Public Property PurchaseOrderNumber As String
    Public Property ContractNumber As String
    Public Property Discount As String
    Public Property ItemList As List(Of EdiOrderItem)
End Class

Class EdiOrderItem
    Public Property ProductCode As String
    Public Property Quantity As String
    Public Property UnitPrice As String
    Public Property Amount As String
End Class


---

4. 交易 (COMMIT 之後)

Try
    Using connection As New SqlConnection(connectionString)
        connection.Open()
        Using transaction As SqlTransaction = connection.BeginTransaction()

            ' 解析 EDI 並存入資料庫
            Dim mainId As Integer = SaveEdiData(transaction, ediData)

            ' 提交交易
            transaction.Commit()

            ' 查詢完整訂單資訊 (含多個 PO1)
            Dim orderInfo As EdiOrderInfo = GetEdiOrderDetailsWithItems(mainId, connection)

            ' 發送通知信
            SendEdiConfirmationEmail(orderInfo)
        End Using
    End Using
Catch ex As Exception
    ' 交易失敗處理
    LogError(ex.Message)
End Try


---

5. 最終 Email 格式

850 (新訂單)

[新訂單通知] 訂單 PO12345 已收到

訂單號碼: PO12345
訂單類型: 850 (新訂單)
合約編號: ABC123
折扣: 10%

【產品清單】
- 產品編號: ITEM001
  數量: 100
  單價: 50
  總金額: 5000

- 產品編號: ITEM002
  數量: 200
  單價: 30
  總金額: 6000

860 (訂單變更)

[訂單變更通知] 訂單 PO12345 已變更

訂單號碼: PO12345
訂單類型: 860 (變更訂單)
折扣: 5%

【變更後產品清單】
- 產品編號: ITEM001
  數量: 150
  單價: 45
  總金額: 6750

- 產品編號: ITEM003
  數量: 50
  單價: 25
  總金額: 1250


---

結論

✅ 正確查詢多個 PO1，確保所有產品明細都列出
✅ 850 與 860 不同處理，顯示正確的通知內容
✅ COMMIT 之後發送信件，確保數據入庫成功才通知
✅ 系統未來可擴展，例如加入 CC 給其他人、發送 SMS、Webhook

這樣的方式確保 Email 完整、準確、不影響交易效能！

