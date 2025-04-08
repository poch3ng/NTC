感謝提供資訊！你要呼叫的 .asmx 方法是：

方法名稱：ProcessImageAndOCR

參數：base64image（Base64 字串）

驗證：匿名（無需帳密）



---

✅ 最穩定的做法：使用 HTTP Request + 自組 SOAP XML

這方式不依賴 UiPath 的 SOAP 活動，簡單穩定。


---

完整 UiPath 流程步驟


---

1. 建立變數


---

2. 指定圖片路徑

Assign 活動

filePath = "C:\image.jpg"


---

3. 使用 Invoke Code：讀圖片 + 轉 Base64

Dim imgBytes As Byte() = System.IO.File.ReadAllBytes(filePath)
base64Str = Convert.ToBase64String(imgBytes)

Arguments 設定


---

4. Assign：組 SOAP XML

soapXml = "<soap:Envelope xmlns:soap=""http://schemas.xmlsoap.org/soap/envelope/"">" &
            "<soap:Body>" &
              "<ProcessImageAndOCR xmlns=""http://tempuri.org/"">" &
                "<base64image>" & base64Str & "</base64image>" &
              "</ProcessImageAndOCR>" &
            "</soap:Body>" &
          "</soap:Envelope>"


---

5. 使用 HTTP Request 活動

設定如下：

Endpoint：https://ntcoat009/OCR/OCR_Data.asmx

Method：POST

Body：soapXml

Response：輸出為 responseStr

Headers：


New Dictionary(Of String, String) From {
    {"Content-Type", "text/xml; charset=utf-8"},
    {"SOAPAction", "http://tempuri.org/ProcessImageAndOCR"}
}


---

6. 顯示結果（可選）

使用 Message Box 或 Log：

responseStr


---

✅ 完成！

這樣就能穩定地用 UiPath 呼叫你的 .asmx WebService，不需要額外認證、不受 SOAP 活動限制。


---

若你需要我幫你產出 .xaml 範本檔，請說一聲，我可直接提供。是否需要？



以下是完整流程，從圖片轉 Base64 → 組 SOAP → 呼叫 .asmx WebService → 回傳處理（支援 Windows 驗證），使用 UiPath + Invoke Code。


---

流程總覽

1. 指定圖片路徑


2. 讀取圖片並轉 Base64


3. 組 SOAP XML


4. 用 Invoke Code 呼叫 .asmx


5. 輸出結果




---

步驟詳解


---

1. 建立變數（共 4 個）


---

2. Assign：讀檔路徑

filePath = "C:\test.jpg"


---

3. Invoke Code：讀取圖片並轉 Base64

Invoke Code 設定

Dim imgBytes As Byte() = System.IO.File.ReadAllBytes(filePath)
base64Str = Convert.ToBase64String(imgBytes)

Arguments


---

4. Assign：組 SOAP XML 字串

soapXml = "<soap:Envelope xmlns:soap=""http://schemas.xmlsoap.org/soap/envelope/"">" &
            "<soap:Body>" &
              "<RecognizeImage xmlns=""http://tempuri.org/"">" &
                "<imageData>" & base64Str & "</imageData>" &
              "</RecognizeImage>" &
            "</soap:Body>" &
          "</soap:Envelope>"

> 請依照你的 WebService 方法與參數名稱修改（這裡用 RecognizeImage 和 imageData 作為範例）




---

5. Invoke Code：使用 Windows 驗證呼叫 .asmx

Dim request As System.Net.HttpWebRequest = CType(System.Net.WebRequest.Create("https://ntcoat009/OCR/OCR_Data.asmx"), System.Net.HttpWebRequest)
request.Method = "POST"
request.ContentType = "text/xml; charset=utf-8"
request.Headers.Add("SOAPAction", "http://tempuri.org/RecognizeImage")
request.Credentials = System.Net.CredentialCache.DefaultCredentials

Dim bytes As Byte() = System.Text.Encoding.UTF8.GetBytes(soapXml)
request.ContentLength = bytes.Length

Using stream = request.GetRequestStream()
    stream.Write(bytes, 0, bytes.Length)
End Using

Using response = CType(request.GetResponse(), System.Net.HttpWebResponse)
    Using reader = New System.IO.StreamReader(response.GetResponseStream())
        responseStr = reader.ReadToEnd()
    End Using
End Using

Arguments


---

6. 顯示結果（可選）

使用 Message Box 或 Log Message：

responseStr


---

備註補充：

SOAPAction 要與方法名稱一致，例如：http://tempuri.org/RecognizeImage

若你不確定參數或命名，可提供 WSDL，我協助產出正確範本

可解析 responseStr（XML格式）用 Deserialize XML 或 Regex 取值



---

若你需要我幫你整理成 UiPath .xaml 範本，也可以直接提供，是否需要？

