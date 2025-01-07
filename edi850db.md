是的，將與資料庫的互動封裝在一個專門的 DatabaseHandler 類別中是更好的設計。這種設計方法符合 單一責任原則 (SRP)，使程式碼更加模組化、可讀性更高且易於維護。


---

優點

1. 集中化管理資料庫邏輯：

所有資料庫操作集中在一個類別中，減少重複程式碼。



2. 可重用性：

提供的方法可以在其他地方重用。



3. 易於修改和擴展：

若需更改資料庫邏輯（如更換資料庫類型），只需修改 DatabaseHandler 類別。



4. 易於測試：

可以為 DatabaseHandler 編寫單元測試，驗證資料庫互動的正確性。





---

DatabaseHandler 類別設計

以下是使用 DatabaseHandler 的實作範例，涵蓋與資料庫的連線、查詢和資料操作。

1. DatabaseHandler 類別

Public Class DatabaseHandler
    Private connectionString As String

    Public Sub New(connString As String)
        connectionString = connString
    End Sub

    ' 開啟資料庫連線
    Private Function GetConnection() As SqlClient.SqlConnection
        Return New SqlClient.SqlConnection(connectionString)
    End Function

    ' 執行非查詢 SQL 指令 (如 INSERT、UPDATE、DELETE)
    Public Function ExecuteNonQuery(query As String, parameters As Dictionary(Of String, Object)) As Integer
        Using connection As SqlClient.SqlConnection = GetConnection()
            Using command As New SqlClient.SqlCommand(query, connection)
                If parameters IsNot Nothing Then
                    For Each param In parameters
                        command.Parameters.AddWithValue(param.Key, param.Value)
                    Next
                End If
                connection.Open()
                Return command.ExecuteNonQuery()
            End Using
        End Using
    End Function

    ' 執行查詢並返回單一值 (如 SELECT COUNT(*))
    Public Function ExecuteScalar(query As String, parameters As Dictionary(Of String, Object)) As Object
        Using connection As SqlClient.SqlConnection = GetConnection()
            Using command As New SqlClient.SqlCommand(query, connection)
                If parameters IsNot Nothing Then
                    For Each param In parameters
                        command.Parameters.AddWithValue(param.Key, param.Value)
                    Next
                End If
                connection.Open()
                Return command.ExecuteScalar()
            End Using
        End Using
    End Function

    ' 執行查詢並返回資料表 (如 SELECT)
    Public Function ExecuteQuery(query As String, parameters As Dictionary(Of String, Object)) As DataTable
        Using connection As SqlClient.SqlConnection = GetConnection()
            Using command As New SqlClient.SqlCommand(query, connection)
                If parameters IsNot Nothing Then
                    For Each param In parameters
                        command.Parameters.AddWithValue(param.Key, param.Value)
                    Next
                End If
                Using adapter As New SqlClient.SqlDataAdapter(command)
                    Dim table As New DataTable()
                    adapter.Fill(table)
                    Return table
                End Using
            End Using
        End Using
    End Function
End Class


---

2. 主程式中的使用方法

新增資料

Public Sub InsertEdiMain(database As DatabaseHandler, isaFields As String(), gsFields As String())
    Dim query As String = "
        INSERT INTO EdiMain (
            ISA01, ISA02, ISA03, ISA04, ISA05, ISA06, ISA07, ISA08, ISA09, ISA10,
            ISA11, ISA12, ISA13, ISA14, ISA15, ISA16,
            GS01, GS02, GS03, GS04, GS05, GS06, GS07, GS08
        ) VALUES (
            @ISA01, @ISA02, @ISA03, @ISA04, @ISA05, @ISA06, @ISA07, @ISA08, @ISA09, @ISA10,
            @ISA11, @ISA12, @ISA13, @ISA14, @ISA15, @ISA16,
            @GS01, @GS02, @GS03, @GS04, @GS05, @GS06, @GS07, @GS08
        )"
    
    Dim parameters As New Dictionary(Of String, Object)()
    For i As Integer = 0 To 15
        parameters.Add($"@ISA{i + 1:00}", isaFields(i))
    Next
    For i As Integer = 0 To 7
        parameters.Add($"@GS{i + 1:00}", gsFields(i))
    Next

    database.ExecuteNonQuery(query, parameters)
End Sub

查詢資料

Public Function GetEdiMainById(database As DatabaseHandler, mainId As Integer) As DataTable
    Dim query As String = "SELECT * FROM EdiMain WHERE ID = @MainID"
    Dim parameters As New Dictionary(Of String, Object)() From {
        {"@MainID", mainId}
    }
    Return database.ExecuteQuery(query, parameters)
End Function

整合進主處理邏輯

以下是整合 DatabaseHandler 的範例：

Sub ProcessEDI850(filePath As String, database As DatabaseHandler)
    Dim segments = ParseEDI850(filePath)

    Dim isaFields(15) As String
    Dim gsFields(7) As String

    For Each segment As String In segments
        Dim elements = segment.Split("*"c)
        Select Case elements(0)
            Case "ISA"
                Array.Copy(elements, 1, isaFields, 0, 16)
            Case "GS"
                Array.Copy(elements, 1, gsFields, 0, 8)
                InsertEdiMain(database, isaFields, gsFields)
            ' 其他段落邏輯省略...
        End Select
    Next
End Sub


---

3. 主程式呼叫

Sub Main()
    Dim connString As String = "Server=YourServerName;Database=YourDatabaseName;User Id=YourUsername;Password=YourPassword;"
    Dim database As New DatabaseHandler(connString)

    ProcessEDI850("path_to_edi_file.edi", database)
End Sub


---

優化重點

1. 解耦資料庫邏輯：

資料庫相關操作完全封裝在 DatabaseHandler，其餘邏輯更專注於業務處理。



2. 可讀性高：

使用參數化查詢避免 SQL Injection，程式碼更安全。



3. 未來可擴展性：

可以輕鬆替換資料庫類型（如切換到 MySQL），只需調整 DatabaseHandler。




這樣的設計讓程式碼更乾淨且易於維護，適合複雜的應用場景。

