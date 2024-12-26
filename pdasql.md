如果你使用 GetSqlStringCommand 來執行 SQL，並且希望根據使用者輸入動態查詢特定月份的資料，可以按照以下步驟進行操作。


---

完整程式碼範例

Imports System.Data
Imports System.Data.Common
Imports Microsoft.Practices.EnterpriseLibrary.Data ' 如果你使用 Enterprise Library

Public Class Example
    Public Sub QueryMonthlyData()
        ' 使用者輸入，例如 "202411"
        Dim userInput As String = "202411"
        Dim year As String = userInput.Substring(0, 4) ' 提取年份，例如 "2024"
        Dim month As String = userInput.Substring(4, 2) ' 提取月份，例如 "11"

        ' 生成 SQL 查詢字串
        Dim sqlQuery As String = $"SELECT * FROM YourTable WHERE YEAR(DateColumn) = {year} AND MONTH(DateColumn) = {month}"

        ' 建立資料庫連線 (假設你使用 Enterprise Library)
        Dim database As Database = DatabaseFactory.CreateDatabase("YourConnectionStringName")
        Dim sqlCommand As DbCommand = database.GetSqlStringCommand(sqlQuery)

        Try
            ' 執行查詢並取得 DataSet
            Dim dataSet As DataSet = database.ExecuteDataSet(sqlCommand)

            ' 處理結果
            If dataSet.Tables.Count > 0 AndAlso dataSet.Tables(0).Rows.Count > 0 Then
                For Each row As DataRow In dataSet.Tables(0).Rows
                    ' 打印每一列的資料
                    For Each column As DataColumn In dataSet.Tables(0).Columns
                        Console.WriteLine($"{column.ColumnName}: {row(column)}")
                    Next
                    Console.WriteLine("------------")
                Next
            Else
                Console.WriteLine("No data found for the selected month.")
            End If
        Catch ex As Exception
            Console.WriteLine($"Error: {ex.Message}")
        End Try
    End Sub
End Class


---

主要邏輯拆解

1. 解析使用者輸入

從輸入值 202411 中解析年份和月份。


Dim year As String = userInput.Substring(0, 4) ' 提取年份，例如 "2024"
Dim month As String = userInput.Substring(4, 2) ' 提取月份，例如 "11"


2. 建立 SQL 查詢

使用 YEAR() 和 MONTH() 函數進行日期篩選。


Dim sqlQuery As String = $"SELECT * FROM YourTable WHERE YEAR(DateColumn) = {year} AND MONTH(DateColumn) = {month}"


3. 執行查詢

使用 GetSqlStringCommand 方法將 SQL 指令傳入資料庫連線，並取得查詢結果。


Dim sqlCommand As DbCommand = database.GetSqlStringCommand(sqlQuery)
Dim dataSet As DataSet = database.ExecuteDataSet(sqlCommand)


4. 處理結果

遍歷查詢結果並打印每一列的內容。


For Each row As DataRow In dataSet.Tables(0).Rows
    For Each column As DataColumn In dataSet.Tables(0).Columns
        Console.WriteLine($"{column.ColumnName}: {row(column)}")
    Next
Next




---

注意事項

1. 輸入驗證

確保使用者輸入符合 YYYYMM 格式，避免 SQL 錯誤或無效查詢。



2. SQL 注入

雖然這裡直接使用字串插值處理，但推薦使用參數化查詢以防止 SQL 注入。


Dim sqlQuery As String = "SELECT * FROM YourTable WHERE YEAR(DateColumn) = @Year AND MONTH(DateColumn) = @Month"
Dim sqlCommand As DbCommand = database.GetSqlStringCommand(sqlQuery)
database.AddInParameter(sqlCommand, "@Year", DbType.Int32, year)
database.AddInParameter(sqlCommand, "@Month", DbType.Int32, month)


3. 日期型別

確保資料庫中的 DateColumn 是標準的 DATE 或 DATETIME 型別，以便使用 YEAR() 和 MONTH() 函數。





---

如有其他需求，請提供更多細節，我可以進一步協助！

