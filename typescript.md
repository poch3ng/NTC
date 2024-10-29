JavaScript 有三種變量宣告方式：`var`、`let`、`const`。

### 1. `var`
   - **早期宣告方式**，可以重複宣告，範圍為**整個函數**或全域。
   - 不建議使用，容易出錯。

### 2. `let`
   - 用於**可變的變量**，範圍為**區塊**（例如 if, for 內）。
   - 適合需要改變值的情況。

### 3. `const`
   - 用於**不可改變的變量**，範圍為**區塊**。
   - 宣告後的值不能修改，適合固定數值。 

### 總結

- **`var`**：範圍大，不建議使用。
- **`let`**：區塊範圍，值可以變。
- **`const`**：區塊範圍，值不能變。

要向沒接觸過的工程師介紹 TypeScript，可以這樣簡單說明，讓他們快速理解：

---

### 1. **什麼是 TypeScript？**
   TypeScript 就是加強版的 JavaScript，它是 JavaScript 的「超集」，意思是它包含了所有 JavaScript 的功能，同時還增加了一些特別的「靜態類型」功能，讓我們可以更安全、更方便地寫代碼。

---

### 2. **TypeScript 能幫助你什麼？**
   - **減少錯誤**：在你寫代碼時，TypeScript 就會檢查你的代碼，幫你找到可能的錯誤。這樣在正式執行代碼之前，很多小問題就能被發現。
   - **提升可讀性**：因為 TypeScript 可以標註每個變量的類型，其他開發者看你的代碼時，不用猜測就能知道這些變量是什麼用途，代碼會更容易理解。
   - **更強大的自動補全**：因為 TypeScript 有靜態類型支持，當你寫代碼時，編輯器可以更精準地提供建議，像變數和方法的自動補全，開發效率更高。

---

### 3. **為什麼我們要學 TypeScript？**
   - **大部分公司和專案都在用**：現在很多大公司（如 Microsoft、Google）都在用 TypeScript，它成為了一種業界標準，特別是對於大型專案來說，TypeScript 讓代碼更穩定，團隊合作更順暢。
   - **不需要捨棄 JavaScript**：因為 TypeScript 最後會被「編譯」成 JavaScript，在你現有的 JavaScript 知識上稍微學一下 TypeScript，你可以更安全、更輕鬆地寫代碼，不會有太多額外學習負擔。

---

### 4. **簡單舉例**

   - **JavaScript**：
     ```javascript
     function greet(name) {
       return "Hello, " + name;
     }
     greet(123); // 沒問題，但會出現意想不到的結果
     ```
   
   - **TypeScript**：
     ```typescript
     function greet(name: string): string {
       return "Hello, " + name;
     }
     greet(123); // 錯誤，因為傳入的不是 string
     ```
     
   在 JavaScript 中，你可以傳入任意類型，但 TypeScript 會幫你檢查，確保代碼更精確。

---

### 總結

TypeScript 就像是幫 JavaScript 裝上了安全帶，不僅讓代碼更安全，還能幫助團隊開發更複雜的專案。你可以把它當成「安全版的 JavaScript」。