以下是 VB.NET 中的函式，用於將部門代號處理成所需的格式：

Public Function FormatDepartmentCode(originalCode As String) As String
    ' 判斷輸入是否有效
    If String.IsNullOrEmpty(originalCode) OrElse originalCode.Length < 4 Then
        Throw New ArgumentException("代號長度不正確")
    End If

    ' 根據條件變更代號
    Select Case originalCode.Substring(2, 2) ' 取代號的第3、4碼
        Case "00"
            ' 處級代號取前兩碼
            Return originalCode.Substring(0, 2)
        Case Else
            ' 部級代號取前三碼
            Return originalCode.Substring(0, 3)
    End Select
End Function

使用方式

1. 將上述函式加入至你的 VB.NET 專案中。


2. 在需要的地方呼叫此函式，並傳入部門代號，範例如下：



Dim formattedCode As String

formattedCode = FormatDepartmentCode("1000") ' 回傳 "10"
Console.WriteLine(formattedCode)

formattedCode = FormatDepartmentCode("1070") ' 回傳 "107"
Console.WriteLine(formattedCode)

formattedCode = FormatDepartmentCode("2110") ' 回傳 "211"
Console.WriteLine(formattedCode)

輸出結果

10
107
211

說明

1. 若代號的第3、4碼為 "00"（例如：1000 和 2100），則取前兩碼。


2. 其他情況（例如：1070 和 2110），則取前三碼。


3. 輸入格式需為 4 碼，否則會拋出例外以確保輸入正確性。



