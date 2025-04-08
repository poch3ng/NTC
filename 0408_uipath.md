在 UiPath 的「HTTP Request Wizard」中，您需要根據提供的 ASMX Web 服務信息來配置參數。假設您的 Web 服務有一個方法 `ProcessImageAndOCR`，它接受一個名為 `base64image` 的參數（Base64 編碼的圖片數據），並且不需要身份驗證（匿名訪問）。以下是如何填寫「HTTP Request Wizard」的步驟：

---

### 配置「HTTP Request Wizard」
1. **打開 HTTP Request Wizard**：
   - 在 UiPath Studio 中，將「HTTP Request」活動拖到工作流中。
   - 點擊活動右側的「Configure」按鈕，打開「HTTP Request Wizard」。

2. **填寫基本信息**：
   - **URL**: 輸入您的 ASMX Web 服務的完整端點地址，例如：
     ```
     http://yourserver/WebService.asmx
     ```
     - 確保這是服務的正確地址（可以從 WSDL 文件或服務文檔中獲取）。
   - **Method**: 選擇 `POST`，因為 SOAP 請求通常使用 POST 方法。

3. **設置請求類型**：
   - 在「Request Type」中選擇 `SOAP`，因為 ASMX 是基於 SOAP 協議的 Web 服務。

4. **SOAP 配置**：
   - **SOAP Action**: 輸入 `ProcessImageAndOCR` 方法的完整 SOAP Action。這通常可以在 WSDL 文件中找到，格式類似：
     ```
     http://yournamespace/ProcessImageAndOCR
     ```
     - 如果不確定具體命名空間，可以聯繫服務提供者或檢查 WSDL。
   - **SOAP Envelope**: 構造 SOAP 請求的 XML 模板。根據您的需求（`base64image` 作為參數），示例模板如下：
     ```xml
     <?xml version="1.0" encoding="utf-8"?>
     <soap:Envelope xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance" 
                    xmlns:xsd="http://www.w3.org/2001/XMLSchema" 
                    xmlns:soap="http://schemas.xmlsoap.org/soap/envelope/">
       <soap:Body>
         <ProcessImageAndOCR xmlns="http://yournamespace/">
           <base64image>{base64image}</base64image>
         </ProcessImageAndOCR>
       </soap:Body>
     </soap:Envelope>
     ```
     - `{base64image}` 是占位符，表示稍後會動態替換成實際的 Base64 字符串。
     - 將 `http://yournamespace/` 替換為服務的實際命名空間（從 WSDL 中獲取）。

5. **參數配置**：
   - 在「Parameters」部分，添加參數：
     - **Name**: `base64image`
     - **Type**: 選擇 `String`
     - **Value**: 輸入您的 Base64 字符串變數（例如 `base64String`），這應該是之前通過讀取圖片文件並轉換為 Base64 的結果。
   - 點擊「Add」添加這個參數。
   - Wizard 會自動將這個變數值替換到 SOAP Envelope 中的 `{base64image}` 位置。

6. **身份驗證**：
   - 既然您提到「匿名」，在「Authentication」部分選擇 `None`，表示不需要用戶名、密碼或其他憑據。

7. **Headers**（可選）：
   - 通常不需要額外手動添加頭信息，因為 Wizard 會自動設置 `Content-Type: text/xml; charset=utf-8`。
   - 如果服務有特殊要求（例如自定義頭），可以在「Headers」部分手動添加。

8. **測試請求**：
   - 點擊「Test Request」按鈕，檢查是否能成功調用 Web 服務。
   - 如果成功，您會看到響應數據；如果失敗，根據錯誤信息調整配置（例如 URL、SOAP Action 或 XML 格式）。

9. **完成配置**：
   - 點擊「OK」保存配置，Wizard 會生成一個完整的「HTTP Request」活動。
   - 將響應存儲到變數（例如 `response`），以便後續處理。

---

### 完整流程示例
假設您已經有一個 Base64 字符串變數 `base64String`，以下是完整流程：
1. **準備 Base64 字符串**：
   - 使用「Read File」讀取圖片文件。
   - 使用「Invoke Code」將其轉換為 Base64，例如：
     ```vb
     Dim imageBytes As Byte() = System.IO.File.ReadAllBytes("C:\path\to\image.jpg")
     base64String = Convert.ToBase64String(imageBytes)
     ```

2. **配置 HTTP Request**：
   - URL: `http://yourserver/WebService.asmx`
   - Method: POST
   - SOAP Action: `http://yournamespace/ProcessImageAndOCR`
   - SOAP Envelope: 如上所述
   - Parameter: `base64image` = `base64String`
   - Authentication: None

3. **處理響應**：
   - 使用「Deserialize XML」活動解析返回的 XML 響應，提取 OCR 結果或其他數據。

---

### 注意事項
- **確認服務細節**：如果您有 WSDL 文件，可以提供給我，我可以幫您更精確地構造 SOAP Envelope。
- **錯誤處理**：建議在流程中添加「Try Catch」，捕獲可能的網絡錯誤或服務端拒絕。
- **測試環境**：先用簡單的 Base64 字符串（例如小圖片）測試，確保配置正確。

如果您有更多具體信息（例如 WSDL URL 或服務端要求），請告訴我，我可以進一步優化答案！

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

