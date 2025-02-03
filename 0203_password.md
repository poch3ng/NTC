你的說明文字可以這樣設計，既清楚告知用戶密碼過期的情況，也強調是為了符合資安規範：


---

標題 (Title)

🔹 "Your Password Has Expired"
🔹 "Password Expired – Security Update Required"

說明文字 (Instructional Text)

🔹 "For security reasons, your password has expired and must be updated to continue using your account."
🔹 "To comply with security policies, your password has expired. Please set a new password to regain access."
🔹 "Your password has expired as part of our security policy. Please update your password to continue."

輸入欄位 (Form Labels)

🔹 "New Password"（新密碼）
🔹 "Confirm New Password"（確認新密碼）

按鈕 (Button)

🔹 "Update Password"（更新密碼）

成功訊息 (Success Message)

🔹 "Your password has been successfully updated. Redirecting to your account..."
🔹 "Password updated successfully. You will now be logged in."


---

這樣的設計既能明確傳達密碼過期的必要性，也讓客戶理解這是為了安全考量，而不是額外的麻煩。



以下是一個依據您提供 UI 文案設計的強制修改密碼範例，僅包含「New Password」與「Confirm New Password」兩個欄位，並提供疊在文字框右上角的 Show/Hide 切換按鈕。您可直接放在 Master Page 的 Content 區塊中使用：

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
  <style type="text/css">
      .password-panel {
          max-width: 400px;
          margin: 50px auto;
          padding: 20px;
          background-color: #333;
          border: 1px solid #444;
          border-radius: 8px;
          box-shadow: 0 2px 6px rgba(0,0,0,0.5);
          font-family: Calibri, "微軟正黑體", "Courier New", "Ms Pmincho", sans-serif;
          color: #fff;
          text-align: center;
      }
      .password-panel h1 {
          margin-bottom: 20px;
          color: #fff;
      }
      .instruction {
          margin-bottom: 20px;
          font-size: 14px;
          line-height: 1.4;
      }
      .instruction p {
          margin: 5px 0;
      }
      .form-group {
          margin-bottom: 15px;
          text-align: left;
      }
      .form-group label {
          display: block;
          margin-bottom: 5px;
          font-weight: bold;
          color: #fff;
      }
      .input-container {
          position: relative;
      }
      .input-container .password-field {
          width: 100%;
          padding: 8px;
          border: 1px solid #ccc;
          border-radius: 4px;
          background-color: #555;
          color: #fff;
          font-family: inherit;
      }
      /* 將按鈕疊在文字框上方右上角 */
      .toggle-link {
          position: absolute;
          top: 0;
          right: 0;
          background: rgba(0, 0, 0, 0.6);
          padding: 4px 8px;
          border-radius: 0 4px 0 0;
          cursor: pointer;
          color: #fff;
          text-decoration: none;
          font-size: 12px;
      }
      .btn-custom {
          width: 100%;
          padding: 10px;
          background-color: #007bff;
          border: none;
          color: #fff;
          border-radius: 4px;
          cursor: pointer;
          font-family: inherit;
          font-size: 16px;
      }
      .btn-custom:hover {
          background-color: #0056b3;
      }
      .message {
          margin-top: 15px;
          color: #fff;
          font-size: 14px;
      }
  </style>

  <script type="text/javascript">
      function toggleFieldVisibility(fieldId, link) {
          var field = document.getElementById(fieldId);
          if (field) {
              if (field.type === 'password') {
                  field.type = 'text';
                  link.textContent = 'Hide';
              } else {
                  field.type = 'password';
                  link.textContent = 'Show';
              }
          }
      }
  </script>

  <asp:Panel ID="pnlPassword" runat="server" CssClass="password-panel">
      <!-- 標題 -->
      <h1>Password Expired – Update Required</h1>
      
      <!-- 說明文字 -->
      <div class="instruction">
          <p>To continue using your account, please set a new password.</p>
          <p>For security reasons, you must update your password before accessing your account.</p>
      </div>
      
      <!-- New Password -->
      <div class="form-group">
          <label for="txtNewPassword">New Password</label>
          <div class="input-container">
              <asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
              <a href="javascript:void(0);" 
                 onclick="toggleFieldVisibility('<%= txtNewPassword.ClientID %>', this);" 
                 class="toggle-link">Show</a>
          </div>
          <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server"
              ControlToValidate="txtNewPassword" 
              ErrorMessage="Please enter a new password" 
              Display="Dynamic" ForeColor="Red" />
          <asp:RegularExpressionValidator ID="revNewPassword" runat="server"
              ControlToValidate="txtNewPassword"
              ErrorMessage="Password must be greater than 12 characters and include uppercase, lowercase, number, and special symbol."
              ValidationExpression="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{13,}$"
              Display="Dynamic" ForeColor="Red" />
      </div>
      
      <!-- Confirm New Password -->
      <div class="form-group">
          <label for="txtConfirmPassword">Confirm New Password</label>
          <div class="input-container">
              <asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
              <a href="javascript:void(0);" 
                 onclick="toggleFieldVisibility('<%= txtConfirmPassword.ClientID %>', this);" 
                 class="toggle-link">Show</a>
          </div>
          <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server"
              ControlToValidate="txtConfirmPassword" 
              ErrorMessage="Please confirm your new password" 
              Display="Dynamic" ForeColor="Red" />
      </div>
      
      <!-- 更新密碼按鈕 -->
      <asp:Button ID="btnUpdate" runat="server" Text="Update Password" OnClick="btnUpdate_Click" CssClass="btn-custom" />
      
      <!-- 成功訊息 (初始隱藏) -->
      <asp:Label ID="lblSuccess" runat="server" CssClass="message" Visible="false"></asp:Label>
  </asp:Panel>
</asp:Content>


---

UI 設計說明

1. 標題與說明文字

標題使用 "Password Expired – Update Required"。

說明文字分為兩行，分別顯示「To continue using your account, please set a new password.」與「For security reasons, you must update your password before accessing your account.」



2. 輸入欄位

僅包含「New Password」與「Confirm New Password」欄位。

每個欄位皆採用疊在右上角的 Show/Hide 按鈕，可切換密碼顯示狀態，不影響文字框的寬度與置中效果。



3. 按鈕

按鈕文字為 "Update Password"，符合強制修改密碼的要求，並未提供取消或稍後提醒選項。



4. 成功訊息

成功訊息區塊 (lblSuccess) 可根據後端處理結果顯示如下訊息之一：

"Your password has been successfully updated. Redirecting to your account..."

"Password updated successfully. You will now be logged in."





此設計保持簡潔、清楚，並符合安全要求，避免出現「Cancel」與「Remind Me Later」等選項。您可依專案需求進一步調整樣式與後端邏輯。



以下提供新版範例，將每個密碼欄位旁加上顯示/隱藏切換連結，並與文字框保持在同一水平線上：

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
  <style type="text/css">
      .password-panel {
          max-width: 400px;
          margin: 50px auto;
          padding: 20px;
          background-color: #333;
          border: 1px solid #444;
          border-radius: 8px;
          box-shadow: 0 2px 6px rgba(0,0,0,0.5);
          font-family: Calibri, "微軟正黑體", "Courier New", "Ms Pmincho", sans-serif;
          color: #fff;
      }
      .password-panel h2 {
          text-align: center;
          margin-bottom: 20px;
          color: #fff;
      }
      .form-group {
          margin-bottom: 15px;
      }
      .form-group label {
          font-weight: bold;
          margin-bottom: 5px;
          display: block;
          color: #fff;
      }
      .input-group {
          display: flex;
          align-items: center;
      }
      .input-group .password-field {
          flex: 1;
          padding: 8px;
          border: 1px solid #ccc;
          border-radius: 4px;
          background-color: #555;
          color: #fff;
          font-family: inherit;
      }
      .toggle-link {
          margin-left: 10px;
          cursor: pointer;
          color: #fff;
          text-decoration: underline;
      }
      .btn-custom {
          width: 100%;
          padding: 10px;
          background-color: #007bff;
          border: none;
          color: #fff;
          border-radius: 4px;
          cursor: pointer;
          font-family: inherit;
      }
      .btn-custom:hover {
          background-color: #0056b3;
      }
      .message {
          text-align: center;
          margin-bottom: 15px;
          color: #fff;
      }
  </style>

  <script type="text/javascript">
      function toggleFieldVisibility(fieldId, link) {
          var field = document.getElementById(fieldId);
          if (field) {
              if (field.type === 'password') {
                  field.type = 'text';
                  link.textContent = 'Hide';
              } else {
                  field.type = 'password';
                  link.textContent = 'Show';
              }
          }
      }
  </script>

  <asp:Panel ID="pnlPassword" runat="server" CssClass="password-panel">
      <h2>Change Password</h2>
      <asp:Label ID="lblMessage" runat="server" CssClass="message" ForeColor="Red"></asp:Label>
      
      <!-- Old Password -->
      <div class="form-group">
          <label for="txtOldPassword">Old Password</label>
          <div class="input-group">
              <asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
              <a href="javascript:void(0);" onclick="toggleFieldVisibility('<%= txtOldPassword.ClientID %>', this);" class="toggle-link">Show</a>
          </div>
          <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server"
              ControlToValidate="txtOldPassword" 
              ErrorMessage="Please enter your old password" 
              Display="Dynamic" ForeColor="Red" />
      </div>
      
      <!-- New Password -->
      <div class="form-group">
          <label for="txtNewPassword">New Password</label>
          <div class="input-group">
              <asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
              <a href="javascript:void(0);" onclick="toggleFieldVisibility('<%= txtNewPassword.ClientID %>', this);" class="toggle-link">Show</a>
          </div>
          <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server"
              ControlToValidate="txtNewPassword" 
              ErrorMessage="Please enter a new password" 
              Display="Dynamic" ForeColor="Red" />
          <asp:RegularExpressionValidator ID="revNewPassword" runat="server"
              ControlToValidate="txtNewPassword"
              ErrorMessage="Password must be greater than 12 characters and include uppercase, lowercase, number, and special symbol."
              ValidationExpression="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{13,}$"
              Display="Dynamic" ForeColor="Red" />
      </div>
      
      <!-- Confirm New Password -->
      <div class="form-group">
          <label for="txtConfirmPassword">Confirm New Password</label>
          <div class="input-group">
              <asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
              <a href="javascript:void(0);" onclick="toggleFieldVisibility('<%= txtConfirmPassword.ClientID %>', this);" class="toggle-link">Show</a>
          </div>
          <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server"
              ControlToValidate="txtConfirmPassword" 
              ErrorMessage="Please confirm your new password" 
              Display="Dynamic" ForeColor="Red" />
      </div>
      
      <asp:Button ID="btnChange" runat="server" Text="Change Password" OnClick="btnChange_Click" CssClass="btn-custom" />
  </asp:Panel>
</asp:Content>


---

設計重點說明：

1. 同水平排列

使用 .input-group 的 Flex 排版，讓文字框與切換連結保持同一水平線。



2. 顯示/隱藏切換

每個密碼欄位右側放置一個 <a> 連結，點擊後呼叫 toggleFieldVisibility 函式。

函式根據目前狀態變更輸入欄位的 type 屬性，並更新連結文字為 "Show" 或 "Hide"。



3. 其他功能

新密碼欄位包含正規表示式驗證，確保密碼大於 12 字元且包含大小寫、數字及特殊符號。

所有元件均使用深色背景與白色文字，並指定所需字體。




依需求再調整樣式或驗證內容即可。



以下提供新版範例，將每個密碼輸入框旁邊各自加上切換顯示/隱藏功能，請參考：

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <style type="text/css">
        .password-panel {
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            background-color: #333;
            border: 1px solid #444;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.5);
            font-family: Calibri, "微軟正黑體", "Courier New", "Ms Pmincho", sans-serif;
            color: #fff;
        }
        .password-panel h2 {
            text-align: center;
            margin-bottom: 20px;
            color: #fff;
        }
        .form-group {
            margin-bottom: 15px;
        }
        .form-group label {
            font-weight: bold;
            margin-bottom: 5px;
            display: block;
            color: #fff;
        }
        .input-group {
            display: flex;
            align-items: center;
        }
        .input-group .password-field {
            flex: 1;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            background-color: #555;
            color: #fff;
            font-family: inherit;
        }
        .input-group input[type="checkbox"] {
            margin-left: 10px;
        }
        .btn-custom {
            width: 100%;
            padding: 10px;
            background-color: #007bff;
            border: none;
            color: #fff;
            border-radius: 4px;
            cursor: pointer;
            font-family: inherit;
        }
        .btn-custom:hover {
            background-color: #0056b3;
        }
        .message {
            text-align: center;
            margin-bottom: 15px;
            color: #fff;
        }
    </style>

    <script type="text/javascript">
        function toggleFieldVisibility(fieldId, checkbox) {
            var field = document.getElementById(fieldId);
            if (field) {
                field.type = checkbox.checked ? 'text' : 'password';
            }
        }
    </script>

    <asp:Panel ID="pnlPassword" runat="server" CssClass="password-panel">
        <h2>Change Password</h2>
        <asp:Label ID="lblMessage" runat="server" CssClass="message" ForeColor="Red"></asp:Label>
        
        <!-- Old Password -->
        <div class="form-group">
            <label for="txtOldPassword">Old Password</label>
            <div class="input-group">
                <asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
                <input type="checkbox" id="chkShowOld" onclick="toggleFieldVisibility('<%= txtOldPassword.ClientID %>', this);" />
                <label for="chkShowOld" style="color: #fff; margin-left: 5px;">Show</label>
            </div>
            <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server"
                ControlToValidate="txtOldPassword" 
                ErrorMessage="Please enter your old password" 
                Display="Dynamic" ForeColor="Red" />
        </div>
        
        <!-- New Password -->
        <div class="form-group">
            <label for="txtNewPassword">New Password</label>
            <div class="input-group">
                <asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
                <input type="checkbox" id="chkShowNew" onclick="toggleFieldVisibility('<%= txtNewPassword.ClientID %>', this);" />
                <label for="chkShowNew" style="color: #fff; margin-left: 5px;">Show</label>
            </div>
            <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server"
                ControlToValidate="txtNewPassword" 
                ErrorMessage="Please enter a new password" 
                Display="Dynamic" ForeColor="Red" />
            <asp:RegularExpressionValidator ID="revNewPassword" runat="server"
                ControlToValidate="txtNewPassword"
                ErrorMessage="Password must be greater than 12 characters and include uppercase, lowercase, number, and special symbol."
                ValidationExpression="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{13,}$"
                Display="Dynamic" ForeColor="Red" />
        </div>
        
        <!-- Confirm New Password -->
        <div class="form-group">
            <label for="txtConfirmPassword">Confirm New Password</label>
            <div class="input-group">
                <asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
                <input type="checkbox" id="chkShowConfirm" onclick="toggleFieldVisibility('<%= txtConfirmPassword.ClientID %>', this);" />
                <label for="chkShowConfirm" style="color: #fff; margin-left: 5px;">Show</label>
            </div>
            <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server"
                ControlToValidate="txtConfirmPassword" 
                ErrorMessage="Please confirm your new password" 
                Display="Dynamic" ForeColor="Red" />
        </div>
        
        <asp:Button ID="btnChange" runat="server" Text="Change Password" OnClick="btnChange_Click" CssClass="btn-custom" />
    </asp:Panel>
</asp:Content>


---

重點說明：

1. 獨立切換控制

每個密碼輸入框旁都放置一個 checkbox，點選後透過 JavaScript 函式 toggleFieldVisibility 切換該欄位的 type 屬性，從 password 切換到 text，反之亦然。



2. 排版樣式

使用 .input-group 以 Flex 方式排列，確保輸入框與切換控制在同一行顯示。



3. 驗證與樣式

新密碼包含必填及正規表示式驗證，確保大於 12 個字元且包含大小寫英文、數字與特殊符號。

整體頁面採用深色背景與白色字體，搭配指定字體。




依需求進一步調整樣式與功能即可。



以下提供更新後的英文版範例，除了包含密碼複雜度驗證外，還加入了切換密碼顯示/隱藏的功能。請將下列程式碼放在 Master Page 的 Content 區塊中使用：

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <style type="text/css">
        .password-panel {
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            background-color: #333;  /* Dark background */
            border: 1px solid #444;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.5);
            font-family: Calibri, "微軟正黑體", "Courier New", "Ms Pmincho", sans-serif;
            color: #fff;  /* White text */
        }
        .password-panel h2 {
            text-align: center;
            margin-bottom: 20px;
            color: #fff;
        }
        .form-group {
            margin-bottom: 15px;
        }
        .form-group label {
            display: block;
            font-weight: bold;
            margin-bottom: 5px;
            color: #fff;
        }
        .form-group input[type="password"],
        .form-group input[type="text"] {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            background-color: #555; /* Dark input background */
            color: #fff;
            font-family: inherit;
        }
        .form-group input:focus {
            border-color: #66afe9;
            outline: none;
        }
        .btn-custom {
            width: 100%;
            padding: 10px;
            background-color: #007bff;
            border: none;
            color: #fff;
            border-radius: 4px;
            cursor: pointer;
            font-family: inherit;
        }
        .btn-custom:hover {
            background-color: #0056b3;
        }
        .message {
            text-align: center;
            margin-bottom: 15px;
            color: #fff;
        }
    </style>

    <script type="text/javascript">
        function togglePasswordVisibility() {
            var passwordFields = document.querySelectorAll('.password-field');
            for (var i = 0; i < passwordFields.length; i++) {
                // Toggle between 'password' and 'text'
                passwordFields[i].type = (passwordFields[i].type === 'password') ? 'text' : 'password';
            }
        }
    </script>

    <asp:Panel ID="pnlPassword" runat="server" CssClass="password-panel">
        <h2>Change Password</h2>
        <asp:Label ID="lblMessage" runat="server" CssClass="message" ForeColor="Red"></asp:Label>
        
        <div class="form-group">
            <label for="txtOldPassword">Old Password</label>
            <asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server"
                ControlToValidate="txtOldPassword" 
                ErrorMessage="Please enter your old password" 
                Display="Dynamic" ForeColor="Red" />
        </div>
        
        <div class="form-group">
            <label for="txtNewPassword">New Password</label>
            <asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server"
                ControlToValidate="txtNewPassword" 
                ErrorMessage="Please enter a new password" 
                Display="Dynamic" ForeColor="Red" />
            <asp:RegularExpressionValidator ID="revNewPassword" runat="server"
                ControlToValidate="txtNewPassword"
                ErrorMessage="Password must be greater than 12 characters and include uppercase, lowercase, number, and special symbol."
                ValidationExpression="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{13,}$"
                Display="Dynamic" ForeColor="Red" />
        </div>
        
        <div class="form-group">
            <label for="txtConfirmPassword">Confirm New Password</label>
            <asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password" CssClass="password-field"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server"
                ControlToValidate="txtConfirmPassword" 
                ErrorMessage="Please confirm your new password" 
                Display="Dynamic" ForeColor="Red" />
        </div>

        <div class="form-group">
            <input type="checkbox" id="chkShow" onclick="togglePasswordVisibility();" />
            <label for="chkShow" style="color: #fff;">Show Password</label>
        </div>
        
        <asp:Button ID="btnChange" runat="server" Text="Change Password" OnClick="btnChange_Click" CssClass="btn-custom" />
    </asp:Panel>
</asp:Content>


---

設計重點說明：

1. 密碼欄位驗證

使用 RegularExpressionValidator 驗證新密碼必須大於 12 個字元，並包含大寫、小寫、數字及特殊符號。

正規表達式：^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{13,}$



2. 密碼可見開關

為所有密碼欄位加上 CssClass="password-field"，便於 JavaScript 控制。

使用一個 checkbox 搭配 togglePasswordVisibility() 函式切換欄位的 type 屬性，在 password 與 text 間轉換。



3. 整體風格

以深色背景和白色文字搭配，並使用指定字體呈現，讓版面風格統一且現代感十足。




依需求進一步調整樣式或驗證內容即可。



以下是更新後的英文版範例，包含密碼至少13個字元，並必須含有大小寫英文、數字及特殊符號的正規表示式驗證器。請將下列程式碼放在 Master Page 的 Content 區塊中使用：

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <style type="text/css">
        .password-panel {
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            background-color: #333;  /* Dark background */
            border: 1px solid #444;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.5);
            font-family: Calibri, "微軟正黑體", "Courier New", "Ms Pmincho", sans-serif;
            color: #fff;  /* White text */
        }
        .password-panel h2 {
            text-align: center;
            margin-bottom: 20px;
            color: #fff;
        }
        .form-group {
            margin-bottom: 15px;
        }
        .form-group label {
            display: block;
            font-weight: bold;
            margin-bottom: 5px;
            color: #fff;
        }
        .form-group input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            background-color: #555; /* Dark input background */
            color: #fff;
            font-family: inherit;
        }
        .form-group input:focus {
            border-color: #66afe9;
            outline: none;
        }
        .btn-custom {
            width: 100%;
            padding: 10px;
            background-color: #007bff;
            border: none;
            color: #fff;
            border-radius: 4px;
            cursor: pointer;
            font-family: inherit;
        }
        .btn-custom:hover {
            background-color: #0056b3;
        }
        .message {
            text-align: center;
            margin-bottom: 15px;
            color: #fff;
        }
    </style>

    <asp:Panel ID="pnlPassword" runat="server" CssClass="password-panel">
        <h2>Change Password</h2>
        <asp:Label ID="lblMessage" runat="server" CssClass="message" ForeColor="Red"></asp:Label>
        
        <div class="form-group">
            <label for="txtOldPassword">Old Password</label>
            <asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server"
                ControlToValidate="txtOldPassword" 
                ErrorMessage="Please enter your old password" 
                Display="Dynamic" ForeColor="Red" />
        </div>
        
        <div class="form-group">
            <label for="txtNewPassword">New Password</label>
            <asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server"
                ControlToValidate="txtNewPassword" 
                ErrorMessage="Please enter a new password" 
                Display="Dynamic" ForeColor="Red" />
            <asp:RegularExpressionValidator ID="revNewPassword" runat="server"
                ControlToValidate="txtNewPassword"
                ErrorMessage="Password must be greater than 12 characters and include uppercase, lowercase, number, and special symbol."
                ValidationExpression="^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{13,}$"
                Display="Dynamic" ForeColor="Red" />
        </div>
        
        <div class="form-group">
            <label for="txtConfirmPassword">Confirm New Password</label>
            <asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server"
                ControlToValidate="txtConfirmPassword" 
                ErrorMessage="Please confirm your new password" 
                Display="Dynamic" ForeColor="Red" />
        </div>
        
        <asp:Button ID="btnChange" runat="server" Text="Change Password" OnClick="btnChange_Click" CssClass="btn-custom" />
    </asp:Panel>
</asp:Content>


---

說明：

1. RegularExpressionValidator 說明

ValidationExpression 設定為
^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[^A-Za-z0-9]).{13,}$
此正規表達式確保密碼至少有13個字元，並包含至少一個小寫字母、一個大寫字母、一個數字和一個特殊符號。



2. 其他驗證

針對每個欄位皆有必填驗證器，確保使用者填寫必要資訊。




依照需求可進一步調整樣式或驗證內容。



以下提供一個新版範例，符合以下要求：

字體：Calibri, 微軟正黑體, Courier New, Ms Pmincho

全部文字為白色


請將下列程式碼放在 Master Page 的 Content 區塊中使用：

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <style type="text/css">
        .password-panel {
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            background-color: #333;  /* 深色背景 */
            border: 1px solid #444;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.5);
            font-family: Calibri, "微軟正黑體", "Courier New", "Ms Pmincho", sans-serif;
            color: #fff;  /* 文字白色 */
        }
        .password-panel h2 {
            text-align: center;
            margin-bottom: 20px;
            color: #fff;
        }
        .form-group {
            margin-bottom: 15px;
        }
        .form-group label {
            display: block;
            font-weight: bold;
            margin-bottom: 5px;
            color: #fff;
        }
        .form-group input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
            background-color: #555; /* 較暗的輸入框背景 */
            color: #fff;
            font-family: inherit;
        }
        .form-group input:focus {
            border-color: #66afe9;
            outline: none;
        }
        .btn-custom {
            width: 100%;
            padding: 10px;
            background-color: #007bff;
            border: none;
            color: #fff;
            border-radius: 4px;
            cursor: pointer;
            font-family: inherit;
        }
        .btn-custom:hover {
            background-color: #0056b3;
        }
        .message {
            text-align: center;
            margin-bottom: 15px;
            color: #fff;
        }
    </style>

    <asp:Panel ID="pnlPassword" runat="server" CssClass="password-panel">
        <h2>修改密碼</h2>
        <asp:Label ID="lblMessage" runat="server" CssClass="message" ForeColor="Red"></asp:Label>
        
        <div class="form-group">
            <label for="txtOldPassword">舊密碼</label>
            <asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server" ControlToValidate="txtOldPassword" ErrorMessage="請輸入舊密碼" Display="Dynamic" />
        </div>
        
        <div class="form-group">
            <label for="txtNewPassword">新密碼</label>
            <asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server" ControlToValidate="txtNewPassword" ErrorMessage="請輸入新密碼" Display="Dynamic" />
        </div>
        
        <div class="form-group">
            <label for="txtConfirmPassword">確認新密碼</label>
            <asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server" ControlToValidate="txtConfirmPassword" ErrorMessage="請確認新密碼" Display="Dynamic" />
        </div>
        
        <asp:Button ID="btnChange" runat="server" Text="修改密碼" OnClick="btnChange_Click" CssClass="btn-custom" />
    </asp:Panel>
</asp:Content>


---

簡要說明：

1. CSS 設定：

設定了 Panel 及所有文字的字體為指定字體，文字顏色統一使用白色。

為輸入框指定較暗背景，以便白色文字易於辨識。



2. 版面配置：

使用 Panel 包裹整體表單，並以分組方式排列標籤、輸入框與驗證器。

按鈕採用自定義樣式，符合整體風格。




依需求再進一步調整樣式即可。



以下提供一個美化範例，使用內嵌 CSS 來調整版面，方便你直接放在 Master Page 的 Content 區塊中：

<asp:Content ID="Content1" ContentPlaceHolderID="MainContent" runat="server">
    <style type="text/css">
        .password-panel {
            max-width: 400px;
            margin: 50px auto;
            padding: 20px;
            border: 1px solid #ddd;
            border-radius: 8px;
            box-shadow: 0 2px 6px rgba(0,0,0,0.1);
            background-color: #f9f9f9;
        }
        .password-panel h2 {
            text-align: center;
            margin-bottom: 20px;
        }
        .form-group {
            margin-bottom: 15px;
        }
        .form-group label {
            display: block;
            font-weight: bold;
            margin-bottom: 5px;
        }
        .form-group input {
            width: 100%;
            padding: 8px;
            border: 1px solid #ccc;
            border-radius: 4px;
        }
        .btn-custom {
            width: 100%;
            padding: 10px;
            background-color: #007bff;
            border: none;
            color: #fff;
            border-radius: 4px;
            cursor: pointer;
        }
        .btn-custom:hover {
            background-color: #0056b3;
        }
        .message {
            text-align: center;
            margin-bottom: 15px;
        }
    </style>

    <asp:Panel ID="pnlPassword" runat="server" CssClass="password-panel">
        <h2>修改密碼</h2>
        <asp:Label ID="lblMessage" runat="server" CssClass="message" ForeColor="Red"></asp:Label>
        
        <div class="form-group">
            <label for="txtOldPassword">舊密碼</label>
            <asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server" ControlToValidate="txtOldPassword" ErrorMessage="請輸入舊密碼" Display="Dynamic" />
        </div>
        
        <div class="form-group">
            <label for="txtNewPassword">新密碼</label>
            <asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server" ControlToValidate="txtNewPassword" ErrorMessage="請輸入新密碼" Display="Dynamic" />
        </div>
        
        <div class="form-group">
            <label for="txtConfirmPassword">確認新密碼</label>
            <asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password"></asp:TextBox>
            <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server" ControlToValidate="txtConfirmPassword" ErrorMessage="請確認新密碼" Display="Dynamic" />
        </div>
        
        <asp:Button ID="btnChange" runat="server" Text="修改密碼" OnClick="btnChange_Click" CssClass="btn-custom" />
    </asp:Panel>
</asp:Content>

簡要說明：

1. CSS 區塊
調整了整個 Panel 的寬度、邊距與邊框，增加陰影效果與圓角，讓版面看起來更柔和。


2. Panel 與標題
將表單內容放入 <asp:Panel>，並以 <h2> 呈現頁面標題。


3. 表單區塊
每個欄位用 .form-group 分組，包含 Label、TextBox 及驗證器，確保版面整齊。


4. 按鈕樣式
使用 .btn-custom 樣式讓按鈕更具現代感與一致的風格。



依需求調整樣式與顏色，即可達到更美觀的效果。若網站已使用 Bootstrap 等框架，也可整合框架樣式。



以下提供一個基本範例，供你參考：


---

步驟 1：建立頁面與控件

1. 新增一個 ASP.NET Web Form（例如 ChangePassword.aspx）。


2. 在頁面上放入三個 TextBox（舊密碼、新密碼、確認新密碼）、一個 Button 及相應的驗證控件。



ChangePassword.aspx 範例：

<%@ Page Language="VB" AutoEventWireup="false" CodeBehind="ChangePassword.aspx.vb" Inherits="YourNamespace.ChangePassword" %>
<html xmlns="http://www.w3.org/1999/xhtml">
<head runat="server">
    <title>修改密碼</title>
</head>
<body>
    <form id="form1" runat="server">
        <asp:Label ID="lblMessage" runat="server" ForeColor="Red"></asp:Label><br /><br />
        舊密碼：<asp:TextBox ID="txtOldPassword" runat="server" TextMode="Password"></asp:TextBox><br /><br />
        新密碼：<asp:TextBox ID="txtNewPassword" runat="server" TextMode="Password"></asp:TextBox><br /><br />
        確認新密碼：<asp:TextBox ID="txtConfirmPassword" runat="server" TextMode="Password"></asp:TextBox><br /><br />
        <asp:Button ID="btnChange" runat="server" Text="修改密碼" OnClick="btnChange_Click" /><br /><br />
        <asp:RequiredFieldValidator ID="rfvOldPassword" runat="server" ControlToValidate="txtOldPassword" ErrorMessage="請輸入舊密碼" Display="Dynamic" /><br />
        <asp:RequiredFieldValidator ID="rfvNewPassword" runat="server" ControlToValidate="txtNewPassword" ErrorMessage="請輸入新密碼" Display="Dynamic" /><br />
        <asp:RequiredFieldValidator ID="rfvConfirmPassword" runat="server" ControlToValidate="txtConfirmPassword" ErrorMessage="請確認新密碼" Display="Dynamic" />
    </form>
</body>
</html>


---

步驟 2：實作程式邏輯

在 CodeBehind（ChangePassword.aspx.vb）中，撰寫按鈕點擊事件來執行下列步驟：

1. 檢查新密碼與確認密碼是否一致。


2. 驗證使用者輸入的舊密碼（依照你目前的密碼驗證機制，建議使用加密比對）。


3. 更新資料庫中的密碼（記得以 Hash 處理）。



ChangePassword.aspx.vb 範例：

Partial Public Class ChangePassword
    Inherits System.Web.UI.Page

    Protected Sub btnChange_Click(sender As Object, e As EventArgs) Handles btnChange.Click
        Dim oldPwd As String = txtOldPassword.Text.Trim()
        Dim newPwd As String = txtNewPassword.Text.Trim()
        Dim confirmPwd As String = txtConfirmPassword.Text.Trim()

        ' 檢查新密碼是否與確認密碼一致
        If newPwd <> confirmPwd Then
            lblMessage.Text = "新密碼與確認密碼不一致。"
            Return
        End If

        ' 驗證舊密碼（請根據實際情況取出使用者的密碼進行比對，建議比對加密後的值）
        If Not ValidateOldPassword(oldPwd) Then
            lblMessage.Text = "舊密碼錯誤。"
            Return
        End If

        ' 更新密碼（更新前請先對新密碼做 Hash 處理）
        If UpdatePassword(newPwd) Then
            lblMessage.ForeColor = Drawing.Color.Green
            lblMessage.Text = "密碼修改成功！"
        Else
            lblMessage.Text = "密碼修改失敗，請稍後再試。"
        End If
    End Sub

    ' 根據實際邏輯實作驗證舊密碼
    Private Function ValidateOldPassword(ByVal oldPwd As String) As Boolean
        ' 範例：從資料庫取出使用者的密碼 Hash，與 oldPwd 的 Hash 比較
        ' 請替換成你自己的驗證邏輯
        Return True
    End Function

    ' 根據實際邏輯實作更新密碼
    Private Function UpdatePassword(ByVal newPwd As String) As Boolean
        ' 範例：對 newPwd 做 Hash 處理後，更新資料庫
        ' 請替換成你自己的資料庫操作邏輯
        Return True
    End Function
End Class


---

注意事項

安全性：密碼應以 Hash 方式儲存，避免明碼存取。驗證時請採用相同的加密機制。

驗證：可搭配 ASP.NET 提供的驗證控件與 Membership 功能，簡化安全與驗證流程。

例外處理：根據實際需求補上例外處理機制，確保使用者體驗。



---

以上步驟與範例提供了一個簡單的 VB WebForm 密碼修改頁面實作範例，可根據你專案的實際情況進行調整。

