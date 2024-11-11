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

