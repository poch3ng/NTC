jQuery 選擇器簡介與範例


---

jQuery 選擇器用於選取 HTML 文件中的元素，以便對它們進行操作。選擇器的語法類似於 CSS 選擇器，使用起來簡單直觀。


---

1. 基本選擇器

1.1 元素選擇器

語法： $('元素名稱')

功能： 選取所有指定的 HTML 元素。


範例：

<!-- HTML -->
<p>段落一</p>
<p>段落二</p>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('p').css('color', 'blue'); // 將所有段落文字設為藍色
});
</script>


---

1.2 ID 選擇器

語法： $('#id值')

功能： 選取具有特定 ID 的元素。


範例：

<!-- HTML -->
<p id="intro">這是一個介紹段落。</p>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('#intro').hide(); // 隱藏 ID 為 intro 的段落
});
</script>


---

1.3 類別選擇器

語法： $('.類別名稱')

功能： 選取具有特定類別的元素。


範例：

<!-- HTML -->
<p class="highlight">重要資訊一</p>
<p class="highlight">重要資訊二</p>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('.highlight').css('background-color', 'yellow'); // 將類別為 highlight 的段落背景設為黃色
});
</script>


---

2. 層級選擇器

2.1 後代選擇器

語法： $('父元素 後代元素')

功能： 選取在某個父元素內的所有指定後代元素。


範例：

<!-- HTML -->
<ul id="menu">
    <li>首頁</li>
    <li>關於我們</li>
    <li>聯絡方式</li>
</ul>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('#menu li').css('display', 'inline'); // 將清單項目設為行內顯示
});
</script>


---

2.2 子選擇器

語法： $('父元素 > 子元素')

功能： 選取某個父元素的直接子元素。


範例：

<!-- HTML -->
<div class="container">
    <p>這是第一個段落。</p>
    <div>
        <p>這是嵌套在 div 裡的段落。</p>
    </div>
</div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('.container > p').css('font-weight', 'bold'); // 只加粗直接子元素 p
});
</script>


---

3. 屬性選擇器

語法： $('[屬性名稱]')、$('[屬性名稱="值"]')

功能： 選取具有特定屬性的元素，或屬性值符合條件的元素。


範例：

<!-- HTML -->
<input type="text" name="username">
<input type="password" name="password">

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('input[type="password"]').css('background-color', '#fdd'); // 將密碼輸入框背景設為淡紅色
});
</script>


---

4. 內容過濾選擇器

4.1 :first 和 :last

語法： $('元素:first')、$('元素:last')

功能： 選取第一個或最後一個匹配的元素。


範例：

<!-- HTML -->
<ul>
    <li>項目一</li>
    <li>項目二</li>
    <li>項目三</li>
</ul>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('li:first').css('color', 'green'); // 將第一個清單項目文字設為綠色
    $('li:last').css('color', 'red');    // 將最後一個清單項目文字設為紅色
});
</script>


---

4.2 :even 和 :odd

語法： $('元素:even')、$('元素:odd')

功能： 選取索引為偶數或奇數的元素（索引從 0 開始）。


範例：

<!-- HTML -->
<ul>
    <li>項目一</li> <!-- 索引 0 -->
    <li>項目二</li> <!-- 索引 1 -->
    <li>項目三</li> <!-- 索引 2 -->
    <li>項目四</li> <!-- 索引 3 -->
</ul>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('li:even').css('background-color', '#eee'); // 偶數項背景設為灰色
    $('li:odd').css('background-color', '#ddd');  // 奇數項背景設為深灰色
});
</script>


---

4.3 :eq(index)

語法： $('元素:eq(index)')

功能： 選取具有特定索引的元素。


範例：

<!-- HTML -->
<p>段落一</p>
<p>段落二</p>
<p>段落三</p>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('p:eq(1)').css('font-size', '20px'); // 將第二個段落文字放大
});
</script>


---

5. 表單選擇器

5.1 :input、:text、:checkbox、:radio

語法： $(':input')、$(':text') 等

功能： 選取特定類型的表單元素。


範例：

<!-- HTML -->
<form>
    <input type="text" name="username">
    <input type="password" name="password">
    <input type="checkbox" name="remember">
    <input type="submit" value="登入">
</form>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $(':text').css('border', '1px solid blue');        // 文本框加上藍色邊框
    $(':checkbox').css('margin', '10px');              // 勾選框增加外距
    $(':submit').val('立即登入');                      // 修改提交按鈕的文字
});
</script>


---

6. 多重選擇器

語法： $('選擇器1, 選擇器2, ...')

功能： 同時選取多種元素。


範例：

<!-- HTML -->
<h1>標題</h1>
<p>段落文字。</p>
<div>區塊內容。</div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('h1, p').css('color', 'purple'); // 將 h1 和 p 元素文字設為紫色
});
</script>


---

7. 動態選擇器

7.1 :animated

語法： $(':animated')

功能： 選取正在執行動畫的元素。


範例：

<!-- HTML -->
<div id="box" style="width:100px; height:100px; background:red;"></div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('#box').animate({left: '250px'}, 1000);
    // 在動畫進行時改變元素背景色
    $(":animated").css('background-color', 'green');
});
</script>


---

8. 子元素與內容選擇器

8.1 :has(selector)

語法： $('元素:has(子選擇器)')

功能： 選取包含特定子元素的元素。


範例：

<!-- HTML -->
<ul>
    <li>無子清單項目</li>
    <li>
        有子清單項目
        <ul>
            <li>子項目一</li>
        </ul>
    </li>
</ul>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('li:has(ul)').css('font-weight', 'bold'); // 將包含子清單的項目加粗
});
</script>


---

9. 屬性值以某些字串開頭或結尾的選擇器

9.1 [attribute^="value"]（以 value 開頭）

語法： $('[屬性名稱^="值"]')

功能： 選取屬性值以指定字串開頭的元素。


範例：

<!-- HTML -->
<a href="https://www.google.com">Google</a>
<a href="https://www.github.com">GitHub</a>
<a href="http://www.example.com">Example</a>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('a[href^="https"]').css('color', 'green'); // 將 href 以 https 開頭的連結設為綠色
});
</script>


---

9.2 [attribute$="value"]（以 value 結尾）

語法： $('[屬性名稱$="值"]')

功能： 選取屬性值以指定字串結尾的元素。


範例：

<!-- HTML -->
<img src="image1.png" alt="圖片一">
<img src="photo.jpg" alt="照片">
<img src="graphic.svg" alt="圖形">

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('img[src$=".png"]').css('border', '1px solid red'); // 給 src 以 .png 結尾的圖片加紅色邊框
});
</script>


---

10. 組合範例

範例：

<!-- HTML -->
<div class="content">
    <p class="intro">這是介紹段落。</p>
    <p>這是一般段落。</p>
    <a href="https://www.example.com" target="_blank">外部連結</a>
    <a href="/about">內部連結</a>
</div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 選取類別為 content 下的所有段落
    $('.content p').css('margin', '20px');
    
    // 選取類別為 intro 的段落
    $('p.intro').css('font-size', '18px');
    
    // 選取所有帶有 target 屬性的連結
    $('a[target]').css('text-decoration', 'underline');
    
    // 選取 href 屬性值包含 "example" 的連結
    $('a[href*="example"]').css('color', 'red');
});
</script>


---

總結

jQuery 選擇器讓我們能夠輕鬆選取並操作網頁中的元素。

熟練掌握各種選擇器，能夠提高開發效率和代碼可讀性。

結合不同的選擇器，可以選取更精確的元素，滿足複雜的需求。



---

建議：

多練習：嘗試在實際項目中應用這些選擇器，加深理解。

查閱文件：jQuery 官方網站提供了詳細的選擇器參考。


參考資源：

jQuery 選擇器官方文件

jQuery 學習手冊（繁體中文）



---

希望這些簡單直接的介紹和範例能夠幫助您快速上手 jQuery 選擇器！



jQuery 是一個 快速、簡潔 的 JavaScript 函式庫，專為簡化 HTML 文檔操作、事件處理、動畫效果 和 Ajax 互動 而設計。它的口號是「寫更少的代碼，做更多的事情」，使得開發者能夠以更簡單的方式操控網頁元素。


---

主要特點：

簡化 DOM 操作： 使用簡單的語法選擇和操作 HTML 元素。

跨瀏覽器兼容性： 解決了不同瀏覽器之間的兼容問題，提供一致的開發體驗。

事件處理： 提供方便的方法來綁定和觸發事件。

動畫效果： 內建多種動畫方法，輕鬆製作動態效果。

Ajax 支持： 簡化與服務器之間的非同步數據交互。



---

簡單範例：

1. 引入 jQuery 函式庫：

在 HTML 文件的 <head> 部分引入 jQuery：

<script src="https://code.jquery.com/jquery-3.6.0.min.js"></script>

2. 基本用法：

HTML：

<button id="hideButton">隱藏段落</button>
<p id="text">這是一個段落。</p>

JavaScript：

$(document).ready(function(){
    $("#hideButton").click(function(){
        $("#text").hide();
    });
});

解釋：

$(document).ready(function(){ ... }); 確保在文檔完全加載後執行內部代碼。

$("#hideButton").click(function(){ ... }); 綁定點擊事件到按鈕。

$("#text").hide(); 隱藏具有 id="text" 的元素。



---

優點：

學習曲線平緩： 對初學者友好，容易上手。

社區支持： 擁有龐大的用戶社群和豐富的插件資源。

提高開發效率： 簡化了常見的 JavaScript 任務。



---

總結：

jQuery 大大簡化了 JavaScript 編程，使得操作 DOM、處理事件和執行 Ajax 請求變得更加容易。雖然近年來有新的框架和函式庫興起（如 React、Vue 等），但 jQuery 仍然是許多項目中不可或缺的工具。


---

參考資源：

jQuery 官方網站

jQuery 教程（繁體中文）


