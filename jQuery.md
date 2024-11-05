簡單直接介紹 Ajax 及範例程式碼


---

Ajax（Asynchronous JavaScript and XML）是一種在不重新載入整個網頁的情況下，與伺服器進行資料交換的技術。這使得網頁能夠更加動態和互動，提供更好的使用者體驗。


---

1. 什麼是 Ajax？

非同步資料請求： 不需要重新載入整個頁面，就可以從伺服器獲取資料或將資料發送到伺服器。

組成技術：

JavaScript：用於發送請求和處理回應。

XMLHttpRequest（XHR）：瀏覽器提供的物件，用於與伺服器進行非同步通訊。

JSON 或 XML：常用的資料格式。




---

2. 為什麼使用 Ajax？

提升使用者體驗： 減少頁面重新載入的次數，提供更流暢的互動。

減少伺服器負載： 只傳輸必要的資料，節省帶寬和資源。

即時更新內容： 能夠即時顯示最新的資料，例如即時聊天、即時通知等。



---

3. 基本的 Ajax 實作方式

3.1 使用原生 JavaScript

範例：

// 建立一個 XMLHttpRequest 物件
var xhr = new XMLHttpRequest();

// 設定請求的方法和 URL
xhr.open('GET', 'data.json', true);

// 設定回應的處理函數
xhr.onreadystatechange = function() {
    if (xhr.readyState === 4 && xhr.status === 200) {
        // 解析回應的 JSON 資料
        var data = JSON.parse(xhr.responseText);
        console.log(data);
    }
};

// 發送請求
xhr.send();

解釋：

new XMLHttpRequest()：建立一個新的 XHR 物件。

xhr.open(method, url, async)：初始化請求。

xhr.onreadystatechange：監聽狀態改變事件。

xhr.send()：發送請求。



---

4. 使用 jQuery 簡化 Ajax

jQuery 提供了簡單易用的方法來執行 Ajax 請求，例如 $.ajax()、$.get()、$.post()。

4.1 使用 $.get()

範例：

$.get('data.txt', function(data) {
    // 處理回應的資料
    $('#result').html(data);
});

解釋：

$.get(url, success)：對指定的 URL 執行 GET 請求。

success：請求成功後的回調函數，data 是回應的資料。



---

4.2 使用 $.post()

範例：

$.post('submit.php', { name: 'John', age: 30 }, function(response) {
    // 處理回應的資料
    alert('伺服器回應：' + response);
});

解釋：

$.post(url, data, success)：對指定的 URL 執行 POST 請求。

data：要發送的資料物件。

success：請求成功後的回調函數。



---

4.3 使用 $.ajax()

範例：

$.ajax({
    url: 'data.json',
    type: 'GET',
    dataType: 'json',
    success: function(data) {
        // 處理回應的 JSON 資料
        console.log(data);
    },
    error: function(xhr, status, error) {
        console.error('發生錯誤：' + error);
    }
});

解釋：

url：請求的 URL。

type：請求的方法（GET、POST 等）。

dataType：預期的回應資料類型（json、xml、text 等）。

success：請求成功後的回調函數。

error：請求失敗時的回調函數。



---

5. 範例：建立一個簡單的 Ajax 應用

目標： 點擊按鈕，從伺服器獲取一段文字並顯示在頁面上。

5.1 HTML

<button id="loadData">載入資料</button>
<div id="content"></div>

5.2 伺服器端（data.txt）

這是從伺服器載入的資料。

5.3 jQuery 代碼

$(document).ready(function(){
    $('#loadData').click(function(){
        $.get('data.txt', function(data){
            $('#content').html(data);
        });
    });
});

解釋：

當按鈕被點擊時，執行 $.get() 請求。

成功獲取資料後，將其顯示在 #content 區域。



---

6. 傳送資料到伺服器

範例：提交表單資料

6.1 HTML

<form id="myForm">
    名稱：<input type="text" name="name"><br>
    年齡：<input type="text" name="age"><br>
    <input type="submit" value="提交">
</form>
<div id="response"></div>

6.2 jQuery 代碼

$(document).ready(function(){
    $('#myForm').submit(function(event){
        event.preventDefault(); // 防止表單的預設提交行為

        var formData = $(this).serialize(); // 序列化表單資料

        $.post('submit.php', formData, function(response){
            $('#response').html(response);
        });
    });
});

解釋：

event.preventDefault()：阻止表單的預設提交。

$(this).serialize()：將表單資料序列化為字串。

$.post()：將資料以 POST 方法發送到伺服器。



---

7. 處理 JSON 資料

7.1 伺服器端（data.json）

{
    "users": [
        { "name": "John", "age": 30 },
        { "name": "Mary", "age": 25 }
    ]
}

7.2 jQuery 代碼

$(document).ready(function(){
    $.getJSON('data.json', function(data){
        var users = data.users;
        var output = '<ul>';
        $.each(users, function(index, user){
            output += '<li>' + user.name + '，年齡：' + user.age + '</li>';
        });
        output += '</ul>';
        $('#content').html(output);
    });
});

解釋：

$.getJSON()：從伺服器獲取 JSON 資料。

$.each()：遍歷 JSON 數組，動態生成列表。



---

8. Ajax 請求錯誤處理

範例：

$.ajax({
    url: 'data.json',
    type: 'GET',
    dataType: 'json',
    success: function(data){
        // 處理成功的回應
    },
    error: function(xhr, status, error){
        alert('發生錯誤：' + error);
    },
    complete: function(xhr, status){
        console.log('請求完成。');
    }
});

解釋：

error：處理請求失敗的情況。

complete：不論成功或失敗，請求完成後都會執行。



---

9. 常見的 Ajax 選項

data：要發送到伺服器的資料。

dataType：預期的回應資料類型。

timeout：設定請求的超時時間（毫秒）。

beforeSend：在請求發送之前執行的函數。


範例：

$.ajax({
    url: 'submit.php',
    type: 'POST',
    data: { name: 'John', age: 30 },
    dataType: 'text',
    timeout: 5000, // 5 秒超時
    beforeSend: function(){
        $('#loading').show();
    },
    success: function(response){
        $('#result').html(response);
    },
    error: function(){
        alert('請求失敗！');
    },
    complete: function(){
        $('#loading').hide();
    }
});


---

10. Ajax 跨域請求

限制： 出於安全考慮，瀏覽器限制從一個域向另一個域發送 Ajax 請求。

解決方案：

JSONP（已過時）： 使用 <script> 標籤來載入資料。

CORS（推薦）： 伺服器設定允許跨域資源共享。



注意： 跨域請求需要伺服器的配合設定。


---

總結

Ajax 是一種在不重新載入整個頁面的情況下，與伺服器進行資料交換的技術。

jQuery 提供了簡化的 Ajax 方法，使開發更容易。

$.ajax()：最通用的 Ajax 方法，可設定各種選項。

$.get() 和 $.post()：用於簡化 GET 和 POST 請求。

$.getJSON()：用於獲取 JSON 資料。


注意事項：

處理錯誤和超時情況，增強應用的穩定性。

跨域請求需要伺服器端的支持。




---

建議

實踐練習： 嘗試在實際項目中運用 Ajax，增強理解。

安全性考量： 注意防範 XSS 和 CSRF 等安全問題。

查閱文件： jQuery 官方網站和相關教程提供了詳細的資訊。



---

參考資源

jQuery Ajax API 文件

jQuery 教程（繁體中文）

MDN XMLHttpRequest



---

希望這些簡單直接的介紹和範例程式碼能夠幫助您快速理解和運用 Ajax！



jQuery 事件處理的簡單介紹與範例


---

jQuery 提供了簡單易用的方法來處理事件，使我們能夠輕鬆地為網頁元素添加互動功能。本指南將以簡單直接的方式介紹 jQuery 的事件處理，並提供範例程式碼。


---

1. 什麼是事件？

事件是用來描述瀏覽器或使用者與網頁互動時發生的行為，例如：

使用者點擊按鈕

滑鼠移動到元素上

表單提交

鍵盤按鍵被按下



---

2. jQuery 事件處理的基本語法

2.1 使用簡單事件方法

語法： $('selector').eventName(handlerFunction);

說明：

selector：要綁定事件的元素選擇器。

eventName：事件名稱，如 click、mouseover 等。

handlerFunction：事件觸發時要執行的函數。



範例：

<!-- HTML -->
<button id="myButton">點擊我</button>

<!-- jQuery -->
<script>
$(document).ready(function(){
    $('#myButton').click(function(){
        alert('按鈕被點擊了！');
    });
});
</script>


---

2.2 使用 on() 方法

語法： $('selector').on(event, handlerFunction);

說明：

event：事件名稱，可以是單個事件或多個事件（以空格分隔）。

handlerFunction：事件觸發時要執行的函數。



範例：

$('#myButton').on('click', function(){
    alert('按鈕被點擊了！');
});


---

3. 常用事件類型

3.1 滑鼠事件

click：單擊事件

dblclick：雙擊事件

mouseenter：滑鼠進入元素區域

mouseleave：滑鼠離開元素區域

mousemove：滑鼠在元素上移動


範例：

<div id="box" style="width:100px; height:100px; background-color:gray;"></div>

<script>
$(document).ready(function(){
    $('#box').mouseenter(function(){
        $(this).css('background-color', 'blue');
    });
    $('#box').mouseleave(function(){
        $(this).css('background-color', 'gray');
    });
});
</script>


---

3.2 鍵盤事件

keydown：按下鍵盤按鍵

keyup：釋放鍵盤按鍵

keypress：按下並持續按住鍵盤按鍵


範例：

<input type="text" id="inputBox">

<script>
$(document).ready(function(){
    $('#inputBox').keydown(function(event){
        console.log('按下了鍵：' + event.key);
    });
});
</script>


---

3.3 表單事件

submit：表單提交

change：表單元素的值改變

focus：元素獲得焦點

blur：元素失去焦點


範例：

<form id="myForm">
    <input type="text" name="username">
    <input type="submit" value="提交">
</form>

<script>
$(document).ready(function(){
    $('#myForm').submit(function(event){
        event.preventDefault(); // 防止表單實際提交
        alert('表單已提交！');
    });
});
</script>


---

4. 事件物件

事件處理函數可以接收一個事件物件，提供有關事件的詳細資訊。

語法： function(event){ ... }


範例：

$('#myButton').click(function(event){
    console.log('事件類型：' + event.type);
    console.log('觸發事件的元素：', event.target);
});


---

5. 事件委派

當你需要為動態生成的元素綁定事件時，可以使用事件委派。

5.1 為未來的元素綁定事件

語法： $(parentSelector).on(event, childSelector, handlerFunction);


範例：

<ul id="itemList">
    <li>項目一</li>
    <li>項目二</li>
</ul>
<button id="addItem">新增項目</button>

<script>
$(document).ready(function(){
    // 為未來的 li 元素綁定點擊事件
    $('#itemList').on('click', 'li', function(){
        $(this).toggleClass('selected');
    });

    // 動態新增項目
    $('#addItem').click(function(){
        $('#itemList').append('<li>新項目</li>');
    });
});
</script>

<!-- CSS -->
<style>
.selected {
    color: red;
    text-decoration: underline;
}
</style>


---

6. 一次性事件

6.1 使用 one() 方法

功能： 事件處理器只執行一次，執行後自動解除綁定。


範例：

$('#myButton').one('click', function(){
    alert('這個訊息只會顯示一次。');
});


---

7. 解除事件處理器

7.1 使用 off() 方法

語法： $('selector').off(event, handlerFunction);


範例：

function clickHandler(){
    alert('按鈕被點擊了！');
}

// 綁定事件
$('#myButton').on('click', clickHandler);

// 解除事件
$('#removeHandler').click(function(){
    $('#myButton').off('click', clickHandler);
});


---

8. 常用事件輔助方法

8.1 hover() 方法

功能： 簡化了 mouseenter 和 mouseleave 的寫法。

語法： $('selector').hover(handlerIn, handlerOut);


範例：

$('#box').hover(
    function(){
        $(this).css('background-color', 'green');
    },
    function(){
        $(this).css('background-color', 'gray');
    }
);


---

8.2 focus() 和 blur() 方法

功能： 簡化了對 focus 和 blur 事件的處理。


範例：

$('input').focus(function(){
    $(this).css('background-color', '#ffffcc');
});

$('input').blur(function(){
    $(this).css('background-color', '#ffffff');
});


---

9. 事件預防與傳播控制

9.1 阻止事件的預設行為

使用 event.preventDefault()


範例：

$('a').click(function(event){
    event.preventDefault(); // 阻止連結的預設行為（跳轉）
    alert('連結被點擊，但不會跳轉。');
});


---

9.2 停止事件傳播

使用 event.stopPropagation()


範例：

<div id="parent">
    <button id="childButton">點擊我</button>
</div>

<script>
$(document).ready(function(){
    $('#parent').click(function(){
        alert('父元素被點擊');
    });

    $('#childButton').click(function(event){
        event.stopPropagation(); // 阻止事件冒泡
        alert('子按鈕被點擊');
    });
});
</script>


---

10. 綜合範例：建立一個簡單的互動效果

範例：點擊按鈕，切換顯示一段文字

<button id="toggleButton">顯示/隱藏文字</button>
<p id="text" style="display:none;">這是一段可顯示或隱藏的文字。</p>

<script>
$(document).ready(function(){
    $('#toggleButton').click(function(){
        $('#text').toggle();
    });
});
</script>


---

總結

事件綁定方法： 使用簡單事件方法或 on() 方法來綁定事件。

常用事件類型： 滑鼠事件、鍵盤事件、表單事件等。

事件物件： 提供有關事件的詳細資訊，可在事件處理函數中使用。

事件委派： 使用 on() 方法為動態生成的元素綁定事件。

一次性事件： 使用 one() 方法讓事件處理器只執行一次。

解除事件： 使用 off() 方法解除事件處理器。

預防與傳播控制： 使用 preventDefault() 和 stopPropagation()。



---

建議：

多練習： 嘗試在實際項目中運用這些事件處理方法。

查閱文件： jQuery 官方網站提供了詳細的事件 API 參考。


參考資源：

jQuery 事件 API 文件

jQuery 教程（繁體中文）



---

希望這些簡單直接的介紹和範例程式碼能夠幫助您快速掌握 jQuery 的事件處理！



Attribute 和 Property 的主要差別在於：

1. 靜態 vs 動態：

Attribute 是 HTML 中的靜態屬性，定義了元素的初始值或配置。例如：<input type="text" value="Hello"> 中的 value="Hello" 是 attribute，這是元素的初始值。

Property 是 JavaScript 中的 DOM 屬性，反映了元素在頁面當前的狀態或值。當 JavaScript 代碼改變 input 的值時，這只會影響 DOM 的 property 而不會更新 HTML 中的 attribute。



2. 同步行為：

在元素載入時，attribute 的值會被同步到 property。例如 <input value="Hello"> 在載入後，input.value 也會是 "Hello"。

不過，之後的變化並非雙向同步。如果修改 property（如 input.value = "Goodbye"），HTML 的 attribute 值（value="Hello") 不會變更。而直接改變 HTML 的 attribute（如用 JavaScript 設定 input.setAttribute("value", "Hi")），此時 DOM 的 property 也會更新為新的值。



3. 使用時機：

Attribute 主要用來設定元素的預設狀態（在 HTML 文件中）。

Property 主要用於操作或讀取元素的當前狀態（在 JavaScript 中）。




簡單範例

<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Attribute vs Property</title>
</head>
<body>
  <input id="myInput" type="text" value="Hello">

  <script>
    const inputElement = document.getElementById("myInput");

    // 讀取 attribute 和 property
    console.log(inputElement.getAttribute("value")); // 輸出: "Hello"（HTML 的靜態屬性）
    console.log(inputElement.value);                 // 輸出: "Hello"（DOM 的初始屬性）

    // 修改 property（不會改變 HTML 的 attribute）
    inputElement.value = "Goodbye";
    console.log(inputElement.getAttribute("value")); // 仍輸出: "Hello"
    console.log(inputElement.value);                 // 輸出: "Goodbye"（DOM 當前值）

    // 修改 attribute（會同步更新 DOM 的 property）
    inputElement.setAttribute("value", "Hi");
    console.log(inputElement.getAttribute("value")); // 輸出: "Hi"
    console.log(inputElement.value);                 // 輸出: "Hi"
  </script>
</body>
</html>

總結

修改 Property 只影響當前 DOM 的狀態，不會改變原始 HTML 的 Attribute。

修改 Attribute 時會同步更新 Property，但反過來不會。




jQuery 操作元素的簡單介紹與範例


---

jQuery 提供了強大的方法來操作網頁中的元素，使我們能夠輕鬆地修改內容、屬性、樣式，甚至是結構。本指南將以簡單直接的方式介紹常用的操作元素的方法，並提供範例程式碼。


---

1. 操作元素內容

1.1 text() 方法

功能： 獲取或設定元素的文字內容（不包含 HTML 標籤）。

語法：

獲取文字內容： var text = $('selector').text();

設定文字內容： $('selector').text('新文字內容');



範例：

<!-- HTML -->
<p id="paragraph">這是一段文字。</p>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 獲取文字內容
    var content = $('#paragraph').text();
    console.log(content); // 輸出：這是一段文字。

    // 設定新的文字內容
    $('#paragraph').text('文字已被更新。');
});
</script>


---

1.2 html() 方法

功能： 獲取或設定元素的 HTML 內容（包含 HTML 標籤）。

語法：

獲取 HTML 內容： var html = $('selector').html();

設定 HTML 內容： $('selector').html('<strong>新內容</strong>');



範例：

<!-- HTML -->
<div id="content">
    <p>原始內容。</p>
</div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 獲取 HTML 內容
    var htmlContent = $('#content').html();
    console.log(htmlContent); // 輸出：<p>原始內容。</p>

    // 設定新的 HTML 內容
    $('#content').html('<p><em>內容已更新。</em></p>');
});
</script>


---

1.3 val() 方法

功能： 獲取或設定表單元素的值。

語法：

獲取值： var value = $('selector').val();

設定值： $('selector').val('新值');



範例：

<!-- HTML -->
<input type="text" id="username" value="使用者名稱">

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 獲取輸入框的值
    var username = $('#username').val();
    console.log(username); // 輸出：使用者名稱

    // 設定新的值
    $('#username').val('新使用者');
});
</script>


---

2. 操作元素屬性

2.1 attr() 方法

功能： 獲取或設定元素的屬性值。

語法：

獲取屬性值： var value = $('selector').attr('屬性名稱');

設定屬性值： $('selector').attr('屬性名稱', '新值');



範例：

<!-- HTML -->
<img id="logo" src="logo.png" alt="網站標誌">

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 獲取圖片的 src 屬性
    var src = $('#logo').attr('src');
    console.log(src); // 輸出：logo.png

    // 更改圖片的 src 屬性
    $('#logo').attr('src', 'new-logo.png');
});
</script>


---

2.2 prop() 方法

功能： 獲取或設定元素的固有屬性（如選中狀態）。

語法：

獲取屬性值： var value = $('selector').prop('屬性名稱');

設定屬性值： $('selector').prop('屬性名稱', 值);



範例：

<!-- HTML -->
<input type="checkbox" id="agree" checked>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 獲取勾選框的選中狀態
    var isChecked = $('#agree').prop('checked');
    console.log(isChecked); // 輸出：true

    // 取消選中狀態
    $('#agree').prop('checked', false);
});
</script>


---

3. 操作元素的樣式

3.1 css() 方法

功能： 獲取或設定元素的 CSS 樣式。

語法：

獲取樣式值： var value = $('selector').css('屬性名稱');

設定樣式值： $('selector').css('屬性名稱', '值');



範例：

<!-- HTML -->
<div id="box" style="width:100px; height:100px; background-color:red;"></div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 獲取背景色
    var bgColor = $('#box').css('background-color');
    console.log(bgColor); // 輸出：rgb(255, 0, 0)

    // 設定新的背景色
    $('#box').css('background-color', 'blue');
});
</script>


---

3.2 addClass()、removeClass()、toggleClass() 方法

功能： 操作元素的 CSS 類別，以控制樣式。


範例：

<!-- HTML -->
<p id="message">這是一則訊息。</p>

<!-- CSS -->
<style>
.success {
    color: green;
    font-weight: bold;
}
.error {
    color: red;
    font-weight: bold;
}
</style>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 添加類別
    $('#message').addClass('success');

    // 移除類別
    $('#message').removeClass('success');

    // 切換類別
    $('#message').toggleClass('error');
});
</script>


---

4. 操作 DOM 結構

4.1 新增元素

append() 和 prepend() 方法

append()： 在選取的元素內部，最後面新增內容。

prepend()： 在選取的元素內部，最前面新增內容。


範例：

<!-- HTML -->
<ul id="list">
    <li>項目一</li>
    <li>項目二</li>
</ul>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 在清單最後新增一個項目
    $('#list').append('<li>項目三</li>');

    // 在清單最前新增一個項目
    $('#list').prepend('<li>項目零</li>');
});
</script>


---

after() 和 before() 方法

after()： 在選取的元素之後插入內容。

before()： 在選取的元素之前插入內容。


範例：

<!-- HTML -->
<p id="para1">段落一。</p>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 在段落一之後插入段落二
    $('#para1').after('<p id="para2">段落二。</p>');

    // 在段落一之前插入標題
    $('#para1').before('<h2>文章標題</h2>');
});
</script>


---

4.2 移除元素

remove() 和 empty() 方法

remove()： 移除選取的元素（包含其子元素）。

empty()： 清空選取的元素內容，但保留元素本身。


範例：

<!-- HTML -->
<div id="container">
    <p>這是一個段落。</p>
    <p>這是另一個段落。</p>
</div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 移除整個容器
    $('#container').remove();

    // 或者清空容器內容，但保留容器本身
    // $('#container').empty();
});
</script>


---

4.3 複製和替換元素

clone() 方法

功能： 複製選取的元素（可選擇是否包含事件處理器）。

語法： var newElement = $('selector').clone([withDataAndEvents]);


範例：

<!-- HTML -->
<div id="original">
    <p>原始內容。</p>
</div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 複製元素
    var copy = $('#original').clone();

    // 將複製的元素添加到頁面中
    $('body').append(copy);
});
</script>


---

replaceWith() 方法

功能： 用新的內容替換選取的元素。

語法： $('selector').replaceWith('新內容');


範例：

<!-- HTML -->
<p id="old">這是舊的段落。</p>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 用新的段落替換舊的段落
    $('#old').replaceWith('<p id="new">這是新的段落。</p>');
});
</script>


---

5. 操作元素的尺寸與位置

5.1 width() 和 height() 方法

功能： 獲取或設定元素的寬度和高度（不包含內邊距、邊框和外邊距）。

語法：

獲取值： var w = $('selector').width();

設定值： $('selector').width(數值);



範例：

<!-- HTML -->
<div id="box" style="width:200px; height:100px;"></div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 獲取寬度和高度
    var w = $('#box').width();
    var h = $('#box').height();
    console.log('寬度：' + w + '，高度：' + h);

    // 設定新的尺寸
    $('#box').width(300).height(150);
});
</script>


---

5.2 position() 和 offset() 方法

position()： 獲取元素相對於父元素的位置（top 和 left）。

offset()： 獲取元素相對於文檔的位置。


範例：

<!-- HTML -->
<div id="container" style="position: relative;">
    <div id="child" style="position: absolute; top:50px; left:100px;"></div>
</div>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 獲取相對於父元素的位置
    var pos = $('#child').position();
    console.log('相對位置 - 上：' + pos.top + '，左：' + pos.left);

    // 獲取相對於文檔的位置
    var offset = $('#child').offset();
    console.log('絕對位置 - 上：' + offset.top + '，左：' + offset.left);
});
</script>


---

6. 範例：綜合運用

範例：

<!-- HTML -->
<ul id="todoList">
    <li>學習 HTML</li>
    <li>學習 CSS</li>
</ul>
<input type="text" id="newItem" placeholder="新增待辦事項">
<button id="addButton">新增</button>

<!-- jQuery -->
<script>
$(document).ready(function(){
    // 新增待辦事項
    $('#addButton').click(function(){
        var itemText = $('#newItem').val();
        if(itemText !== ''){
            $('#todoList').append('<li>' + itemText + '</li>');
            $('#newItem').val('');
        }
    });

    // 點擊項目時劃掉
    $('#todoList').on('click', 'li', function(){
        $(this).toggleClass('completed');
    });
});

</script>

<!-- CSS -->
<style>
.completed {
    text-decoration: line-through;
    color: gray;
}
</style>

解釋：

新增功能：

使用 append() 方法在清單中新增項目。

使用 val() 方法獲取和清空輸入框的值。


劃掉項目：

使用事件委派 on('click', 'li', function(){...}) 綁定事件。

使用 toggleClass() 方法切換類別。

定義 .completed 類別的樣式，達到劃掉效果。




---

總結

操作元素內容： 使用 text()、html()、val() 來獲取或設定內容。

操作元素屬性： 使用 attr()、prop() 來獲取或設定屬性值。

操作元素樣式： 使用 css()、addClass()、removeClass()、toggleClass() 來修改樣式。

操作 DOM 結構： 使用 append()、prepend()、after()、before()、remove()、empty()、clone()、replaceWith() 來新增、移除或替換元素。

操作尺寸與位置： 使用 width()、height()、position()、offset() 來獲取或設定元素的尺寸和位置。



---

建議：

多實踐： 嘗試在實際項目中運用這些方法，增強理解。

查閱文件： jQuery 官方網站提供了詳細的 API 參考。


參考資源：

jQuery API 文件

jQuery 教程（繁體中文）



---

希望這些簡單直接的介紹和範例程式碼能夠幫助您快速掌握 jQuery 操作元素的方法！




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


