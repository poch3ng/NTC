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

