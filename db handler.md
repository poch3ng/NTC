在 Singleton 模式中，資料庫連接 (Connection) 通常會作為 DbHandler 的一部分進行管理。以下是如何在 DbHandler 中管理資料庫連接的方法：


---

完整實現：包含 Connection 的 Singleton

假設使用 ADO.NET，以下是一個包含 SqlConnection 的 DbHandler Singleton 實現：

Imports System.Data.SqlClient

Public Class DbHandler
    ' 靜態變數保存唯一實例
    Private Shared _instance As DbHandler
    Private Shared ReadOnly lockObj As New Object()

    ' 私有成員，保存資料庫連接
    Private connection As SqlConnection

    ' 私有建構函數
    Private Sub New(connectionString As String)
        connection = New SqlConnection(connectionString)
        OpenConnection() ' 初始化時嘗試打開連接
    End Sub

    ' 公共方法提供單例訪問
    Public Shared Function GetInstance(connectionString As String) As DbHandler
        If _instance Is Nothing Then
            SyncLock lockObj
                If _instance Is Nothing Then
                    _instance = New DbHandler(connectionString)
                End If
            End SyncLock
        End If
        Return _instance
    End Function

    ' 打開資料庫連接
    Private Sub OpenConnection()
        If connection.State = ConnectionState.Closed Then
            connection.Open()
        End If
    End Sub

    ' 獲取資料庫連接 (如果需要直接使用)
    Public Function GetConnection() As SqlConnection
        Return connection
    End Function

    ' 執行查詢，返回 DataTable
    Public Function ExecuteQuery(query As String) As DataTable
        Dim dt As New DataTable()
        Using command As New SqlCommand(query, connection)
            Using reader As SqlDataReader = command.ExecuteReader()
                dt.Load(reader)
            End Using
        End Using
        Return dt
    End Function

    ' 執行非查詢命令 (INSERT/UPDATE/DELETE)
    Public Sub ExecuteNonQuery(query As String)
        Using command As New SqlCommand(query, connection)
            command.ExecuteNonQuery()
        End Using
    End Sub

    ' 釋放資源
    Public Sub Dispose()
        If connection IsNot Nothing AndAlso connection.State <> ConnectionState.Closed Then
            connection.Close()
        End If
    End Sub
End Class


---

關鍵點說明

1. Connection 的初始化

在 Singleton 的建構函數中，傳入連接字串，並初始化 SqlConnection：

connection = New SqlConnection(connectionString)


2. Connection 的管理

OpenConnection() 方法確保連接處於開啟狀態。

如果需要直接使用連接物件（例如透過交易處理），可以透過 GetConnection() 方法獲取。


3. 查詢與非查詢執行

查詢返回 DataTable，使用 SqlCommand 和 SqlDataReader：

Using command As New SqlCommand(query, connection)
    Using reader As SqlDataReader = command.ExecuteReader()
        dt.Load(reader)
    End Using
End Using

非查詢執行 (如 INSERT、UPDATE、DELETE)：

Using command As New SqlCommand(query, connection)
    command.ExecuteNonQuery()
End Using


4. Dispose 方法

為避免連接持續占用資源，提供 Dispose 方法，在應用結束時釋放連接資源。



---

如何使用這個 Singleton

以下是一個範例，展示如何使用這個 DbHandler：

Sub Main()
    ' 初始化 Singleton 實例
    Dim db As DbHandler = DbHandler.GetInstance("YourConnectionString")

    ' 執行查詢
    Dim result As DataTable = db.ExecuteQuery("SELECT * FROM Users")

    ' 執行非查詢
    db.ExecuteNonQuery("INSERT INTO Users (Name, Age) VALUES ('John', 30)")

    ' 釋放資源
    db.Dispose()
End Sub


---

最佳實踐

1. 確保連接的唯一性：

使用 Singleton 確保應用中只有一個資料庫連接。



2. 資源管理：

使用 Dispose 方法或 Using 語法確保資料庫連接及時釋放。



3. 多執行緒環境：

使用 SyncLock 確保執行緒安全性，防止多個實例創建。





---

這樣的設計確保了資料庫連接的高效使用，同時避免了重複初始化和資源浪費問題。

