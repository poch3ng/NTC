InsertEdiMain 方法應該放在專門負責與資料庫交互的類別（例如 DatabaseHandler）中，因為這樣可以集中管理所有的資料庫操作，使程式碼符合單一責任原則並更具可維護性。


---

完整設計架構

1. 在 DatabaseHandler 類別中新增 InsertEdiMain 方法

將 InsertEdiMain 放入 DatabaseHandler 類別中，負責插入 EdiMain 資料表的紀錄。

Public Class DatabaseHandler
    Private connectionString As String

    Public Sub New()
        ' 從 App.config 讀取 Connection String
        connectionString = ConfigurationManager.ConnectionStrings("MyDatabase").ConnectionString
    End Sub

    ' 建立資料庫連線
    Private Function GetConnection() As SqlClient.SqlConnection
        Return New SqlClient.SqlConnection(connectionString)
    End Function

    ' Insert EdiMain 資料表
    Public Function InsertEdiMain(isaFields As String(), gsFields As String()) As Integer
        Dim query As String = "
            INSERT INTO EdiMain (
                ISA01, ISA02, ISA03, ISA04, ISA05, ISA06, ISA07, ISA08, ISA09, ISA10,
                ISA11, ISA12, ISA13, ISA14, ISA15, ISA16,
                GS01, GS02, GS03, GS04, GS05, GS06, GS07, GS08
            ) VALUES (
                @ISA01, @ISA02, @ISA03, @ISA04, @ISA05, @ISA06, @ISA07, @ISA08, @ISA09, @ISA10,
                @ISA11, @ISA12, @ISA13, @ISA14, @ISA15, @ISA16,
                @GS01, @GS02, @GS03, @GS04, @GS05, @GS06, @GS07, @GS08
            );
            SELECT SCOPE_IDENTITY(); -- 返回自動生成的主鍵 ID
        "

        Dim parameters As New Dictionary(Of String, Object)()
        For i As Integer = 0 To 15
            parameters.Add($"@ISA{i + 1:00}", isaFields(i))
        Next
        For i As Integer = 0 To 7
            parameters.Add($"@GS{i + 1:00}", gsFields(i))
        Next

        ' 執行查詢並返回新插入的主鍵 ID
        Using connection As SqlClient.SqlConnection = GetConnection()
            Using command As New SqlClient.SqlCommand(query, connection)
                For Each param In parameters
                    command.Parameters.AddWithValue(param.Key, param.Value)
                Next
                connection.Open()
                Dim newId As Object = command.ExecuteScalar()
                Return Convert.ToInt32(newId)
            End Using
        End Using
    End Function
End Class


---

2. 主程式調用 InsertEdiMain 方法

在主處理邏輯中（如 ProcessEDI850 函式）調用 InsertEdiMain，將解析好的段落資訊插入資料庫。

Sub ProcessEDI850(filePath As String, database As DatabaseHandler)
    Dim segments = ParseEDI850(filePath)

    Dim isaFields(15) As String
    Dim gsFields(7) As String
    Dim mainId As Integer

    For Each segment As String In segments
        Dim elements = segment.Split("*"c)
        Select Case elements(0)
            Case "ISA"
                ' 解析 ISA 段落並儲存欄位
                Array.Copy(elements, 1, isaFields, 0, 16)
            Case "GS"
                ' 解析 GS 段落並儲存欄位
                Array.Copy(elements, 1, gsFields, 0, 8)
                ' 插入 EdiMain 並取得主鍵 ID
                mainId = database.InsertEdiMain(isaFields, gsFields)
            ' 其他段落的處理邏輯...
        End Select
    Next
End Sub


---

3. 初始化 DatabaseHandler 並執行主程式

主程式負責初始化 DatabaseHandler 並調用主處理邏輯。

Sub Main()
    ' 初始化 DatabaseHandler
    Dim database As New DatabaseHandler()

    ' 處理 EDI 檔案
    ProcessEDI850("path_to_edi_file.edi", database)

    Console.WriteLine("EDI 850 處理完成，資料已寫入資料庫。")
End Sub


---

程式執行流程

1. DatabaseHandler：

集中管理所有與資料庫相關的邏輯，包括插入、查詢、更新和刪除。



2. ProcessEDI850：

專注於解析 EDI 檔案並根據段落類型調用 DatabaseHandler 提供的方法。



3. Main：

初始化並調用主處理邏輯。





---

優化建議

1. 模組化設計：

每個段落（如 ISA、GS、ST）的解析與插入邏輯可以進一步拆分為單獨的函式。



2. 錯誤處理：

增加錯誤處理，確保當資料庫操作失敗或檔案格式錯誤時，程式可以記錄詳細的錯誤資訊。



3. 日誌紀錄：

使用日誌工具（如 log4net）記錄處理狀態和錯誤。




這種設計方法可以有效分離邏輯，並讓 InsertEdiMain 的位置在 DatabaseHandler 中清晰易懂，符合良好的程式設計原則。



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

