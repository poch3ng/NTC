匿名函式（Anonymous Function）是沒有名稱的函式，在 JavaScript 中可以直接用來傳遞或執行，通常作為回呼函式（callback function）使用。匿名函式常用於一次性的操作，不需要在程式中重複引用。以下是一些匿名函式的常見應用範例：

1. 基本匿名函式

一個最簡單的匿名函式如下，它不需要名稱，可以直接執行：

(function() {
  console.log("這是一個匿名函式");
})();

在這個例子中，函式定義後立即執行（IIFE，即 Immediately Invoked Function Expression），適用於一些一次性初始化的情況。

2. 以匿名函式作為回呼函式

在 JavaScript 中，匿名函式經常用於回呼函式中，比如 map、filter、reduce 等陣列方法中：

const numbers = [1, 2, 3, 4, 5];
const doubledNumbers = numbers.map(function(number) {
  return number * 2;
});

console.log(doubledNumbers); // 輸出：[2, 4, 6, 8, 10]

這裡的 function(number) { return number * 2; } 就是匿名函式，因為不需要重複使用，所以沒有名稱。

3. 箭頭函式（Arrow Function）也是匿名函式

ES6 引入的箭頭函式語法也是一種匿名函式的形式，更加簡潔：

const numbers = [1, 2, 3, 4, 5];
const doubledNumbers = numbers.map(number => number * 2);

console.log(doubledNumbers); // 輸出：[2, 4, 6, 8, 10]

在這個例子中，number => number * 2 就是一個箭頭函式的匿名函式，用於 map 的回呼函式。

4. 事件處理中的匿名函式

匿名函式也常用於事件處理，比如按鈕的點擊事件：

document.getElementById("myButton").addEventListener("click", function() {
  console.log("按鈕被點擊了！");
});

這裡的 function() { console.log("按鈕被點擊了！"); } 就是匿名函式，用於處理點擊事件。

總結

匿名函式的優點是簡潔，可以快速編寫一次性的程式碼，尤其在回呼函式和事件處理時非常常用。不過，因為匿名函式沒有名稱，通常不適合在需要多次調用的情況下使用。



以下是 JavaScript reduce 方法的範例，展示如何使用它來累積數字和組合陣列中的物件屬性值：

1. 累加數字

假設我們有一個數字陣列，想要計算其總和：

const numbers = [1, 2, 3, 4, 5];
const sum = numbers.reduce((accumulator, currentValue) => accumulator + currentValue, 0);

console.log(sum); // 輸出：15

在這個範例中：

accumulator 是累加的結果。

currentValue 是當前的陣列元素。

0 是初始值，即 accumulator 的起始值。


2. 合併物件陣列中的屬性

假設有一個物件陣列，想要累積所有物件的 price 屬性值：

const products = [
  { name: '商品 A', price: 100 },
  { name: '商品 B', price: 200 },
  { name: '商品 C', price: 300 }
];
const totalPrice = products.reduce((accumulator, product) => accumulator + product.price, 0);

console.log(totalPrice); // 輸出：600

這樣可以輕鬆計算出所有商品的總價格。

3. 計算字母出現的次數

reduce 也可以用來計算字母出現的次數：

const letters = ['a', 'b', 'a', 'c', 'b', 'a'];
const letterCount = letters.reduce((accumulator, letter) => {
  accumulator[letter] = (accumulator[letter] || 0) + 1;
  return accumulator;
}, {});

console.log(letterCount); // 輸出：{ a: 3, b: 2, c: 1 }

在這個範例中，我們初始化了一個空物件 {} 作為初始值，並根據每個字母的出現次數累加到對應的屬性中。

這些範例展示了 reduce 在數字累加、物件屬性累加及字母次數計算上的應用，適用於許多不同的情境。

