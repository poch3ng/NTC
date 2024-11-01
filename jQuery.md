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


