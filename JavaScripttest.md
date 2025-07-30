以下是針對 JavaScript、CSS 和 jQuery 的十道簡單選擇題，每題包含四個選項（一個正確答案和三個錯誤選項）。這些題目涵蓋基礎概念，適合初學者或複習使用。每題附有正確答案和簡短說明。

---

### **JavaScript 選擇題 (4 題)**

1. **問題：如何在 JavaScript 中宣告一個變數並賦值為數字 42？**
   - A. `let number = 42;`
   - B. `number = 42;`
   - C. `let 42 = number;`
   - D. `const number == 42;`
   - **正確答案**：A
   - **說明**：`let` 用於宣告變數，後接變數名和賦值。B 缺少宣告關鍵字，C 語法錯誤（數字不能作為變數名），D 使用比較運算符 `==` 而非賦值。

2. **問題：以下哪個函數能正確回傳兩個參數的和？**
   - A. 
     ```javascript
     function sum(a, b) {
       return a + b;
     }
     ```
   - B. 
     ```javascript
     function sum(a, b) {
       a + b;
     }
     ```
   - C. 
     ```javascript
     function sum(a, b) {
       return a - b;
     }
     ```
   - D. 
     ```javascript
     sum(a, b) {
       return a + b;
     }
     ```
   - **正確答案**：A
   - **說明**：A 正確使用 `return` 回傳結果。B 缺少 `return`，不會回傳值。C 執行減法而非加法。D 缺少 `function` 關鍵字，語法錯誤。

3. **問題：如何檢查一個變數 `x` 是否為陣列？**
   - A. `Array.isArray(x);`
   - B. `x.isArray();`
   - C. `typeof x == 'array';`
   - D. `x instanceof Array.isArray;`
   - **正確答案**：A
   - **說明**：`Array.isArray(x)` 是標準方法檢查陣列。B 的 `isArray` 不是變數方法。C 的 `typeof` 對陣列回傳 'object'，不正確。D 語法錯誤，`instanceof` 用法應為 `x instanceof Array`。

4. **問題：以下哪段程式碼能將陣列 `[1, 2, 3]` 的每個元素加倍？**
   - A. 
     ```javascript
     let arr = [1, 2, 3];
     let doubled = arr.map(num => num * 2);
     ```
   - B. 
     ```javascript
     let arr = [1, 2, 3];
     let doubled = arr.forEach(num => num * 2);
     ```
   - C. 
     ```javascript
     let arr = [1, 2, 3];
     let doubled = arr.map(num => num + 2);
     ```
   - D. 
     ```javascript
     let arr = [1, 2, 3];
     let doubled = arr * 2;
     ```
   - **正確答案**：A
   - **說明**：`map()` 創建新陣列並對每個元素操作，A 正確加倍。B 的 `forEach` 不回傳新陣列。C 執行加法而非乘法。D 對陣列直接乘法無效。

---

### **CSS 選擇題 (3 題)**

5. **問題：如何使用 CSS 將一個元素的背景顏色設為藍色？**
   - A. 
     ```css
     .element {
       background-color: blue;
     }
     ```
   - B. 
     ```css
     .element {
       background: color-blue;
     }
     ```
   - C. 
     ```css
     .element {
       color: blue;
     }
     ```
   - D. 
     ```css
     .element {
       bg-color: blue;
     }
     ```
   - **正確答案**：A
   - **說明**：`background-color` 是正確屬性。B 的 `color-blue` 無效。C 的 `color` 影響文字顏色。D 的 `bg-color` 不是有效屬性。

6. **問題：如何讓一個塊級元素水平置中？**
   - A. 
     ```css
     .element {
       margin: 0 auto;
       width: 100px;
     }
     ```
   - B. 
     ```css
     .element {
       align: center;
     }
     ```
   - C. 
     ```css
     .element {
       margin: auto 0;
     }
     ```
   - D. 
     ```css
     .element {
       text-align: center;
     }
     ```
   - **正確答案**：A
   - **說明**：`margin: 0 auto` 搭配寬度使塊級元素置中。B 的 `align` 無效。C 的 `margin: auto 0` 不會水平置中。D 的 `text-align` 只影響行內元素或文字。

7. **問題：以下哪個 CSS 屬性可完全隱藏元素且不佔空間？**
   - A. `display: none;`
   - B. `visibility: hidden;`
   - C. `opacity: 0;`
   - D. `display: hidden;`
   - **正確答案**：A
   - **說明**：`display: none` 隱藏元素且不佔空間。B 隱藏但保留空間。C 透明但仍可交互。D 的 `display: hidden` 無效。

---

### **jQuery 選擇題 (3 題)**

8. **問題：如何使用 jQuery 選擇所有 class 為 `box` 的元素並改變其文字內容為 "Hello"？**
   - A. `$('.box').text('Hello');`
   - B. `$('.box').html('Hello');`
   - C. `$('box').text('Hello');`
   - D. `$('.box').value('Hello');`
   - **正確答案**：A
   - **說明**：`$('.box')` 選擇 class，`.text()` 設置文字。B 的 `.html()` 用於 HTML 內容。C 缺少 `.` 選擇 class。D 的 `.value()` 無效。

9. **問題：以下哪段 jQuery 程式碼能為按鈕添加點擊事件，點擊後顯示 alert？**
   - A. 
     ```javascript
     $('button').click(function() {
       alert('Button clicked!');
     });
     ```
   - B. 
     ```javascript
     $('button').onClick(function() {
       alert('Button clicked!');
     });
     ```
   - C. 
     ```javascript
     $('button').click('Button clicked!');
     ```
   - D. 
     ```javascript
     $(button).click(function() {
       alert('Button clicked!');
     });
     ```
   - **正確答案**：A
   - **說明**：`.click()` 綁定點擊事件，A 正確。B 的 `onClick` 無效。C 傳入字串而非函數。D 缺少選擇器引號。

10. **問題：如何使用 jQuery 將一個元素的 CSS 背景顏色設為紅色？**
    - A. `$('.element').css('background-color', 'red');`
    - B. `$('.element').style('background-color', 'red');`
    - C. `$('.element').css('background', 'red-color');`
    - D. `$('element').css('background-color', 'red');`
    - **正確答案**：A
    - **說明**：`.css()` 設置 CSS 屬性，A 正確。B 的 `.style()` 無效。C 的 `'red-color'` 無效值。D 缺少 `.` 選擇 class。

---

### **備註**
- 每題的錯誤選項設計為常見錯誤或混淆點，幫助學習者辨識正確語法。
- 如果需要更進階題目（例如閉包、CSS Grid、jQuery AJAX）或不同題型（例如程式碼填空），請告知！
- 若需更詳細解釋或實作範例，請告訴我。