以下是進階常用的 CSS 技巧和概念，包括 !important 及其他高級應用，適合內部教學時強調使用場景與最佳實踐：


---

1. 使用 !important

說明：強制覆蓋其他樣式 (慎用)。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        p {
            color: red !important; /* 強制覆蓋其他樣式 */
        }
        .override {
            color: blue; /* 無法覆蓋 !important */
        }
    </style>
</head>
<body>
    <p class="override">這段文字會是紅色，因為 !important 優先級最高。</p>
</body>
</html>
```
> 注意： 避免過度使用 !important，除非需要緊急解決樣式衝突。




---

2. 繼承與優先級

說明：展示不同選擇器的優先級。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        p {
            color: black; /* 通用選擇器 */
        }
        .specific {
            color: green; /* 類別選擇器，優先級較高 */
        }
        #unique {
            color: blue; /* ID 選擇器，優先級最高 */
        }
    </style>
</head>
<body>
    <p>黑色文字</p>
    <p class="specific">綠色文字</p>
    <p id="unique">藍色文字</p>
</body>
</html>
```

---

3. ::before 與 ::after 偽元素

說明：在元素前後插入內容。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        .button::before {
            content: "👉";
            margin-right: 8px;
        }
        .button::after {
            content: "✔";
            margin-left: 8px;
        }
        .button {
            padding: 10px;
            background-color: lightblue;
        }
    </style>
</head>
<body>
    <button class="button">按鈕</button>
</body>
</html>
```

---

4. 透明度與顏色疊加

說明：利用 rgba 和 opacity。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        .overlay {
            width: 300px;
            height: 200px;
            background: rgba(0, 0, 0, 0.5); /* 半透明黑色 */
            color: white;
            text-align: center;
            line-height: 200px;
        }
    </style>
</head>
<body>
    <div class="overlay">透明度效果</div>
</body>
</html>
```

---

5. CSS 變形 (Transform)

說明：實現旋轉、縮放等特效。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        .box {
            width: 100px;
            height: 100px;
            background-color: orange;
            transition: transform 0.5s ease;
        }
        .box:hover {
            transform: rotate(45deg) scale(1.2); /* 旋轉並放大 */
        }
    </style>
</head>
<body>
    <div class="box"></div>
</body>
</html>
```

---

6. CSS 多背景

說明：同一元素上設定多個背景。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        .multi-bg {
            width: 300px;
            height: 200px;
            background: url('https://via.placeholder.com/300x200') no-repeat center,
                        linear-gradient(to right, #ff7e5f, #feb47b);
            background-blend-mode: overlay;
        }
    </style>
</head>
<body>
    <div class="multi-bg"></div>
</body>
</html>
```

---

7. 媒體查詢 (Media Query)

說明：針對不同設備設計樣式。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        body {
            font-size: 16px;
        }
        @media (max-width: 600px) {
            body {
                font-size: 12px; /* 手機裝置上字體縮小 */
            }
        }
    </style>
</head>
<body>
    <p>調整視窗寬度來測試文字大小的變化。</p>
</body>
</html>
```

---

8. 層級 (z-index)

說明：控制元素的堆疊順序。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        .box1, .box2 {
            position: absolute;
            width: 100px;
            height: 100px;
        }
        .box1 {
            background-color: red;
            z-index: 1;
            top: 50px;
            left: 50px;
        }
        .box2 {
            background-color: blue;
            z-index: 2; /* 比 box1 更高 */
            top: 70px;
            left: 70px;
        }
    </style>
</head>
<body>
    <div class="box1"></div>
    <div class="box2"></div>
</body>
</html>
```

---

9. CSS 圓角邊框 (Border Radius)

說明：創建不規則形狀。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        .circle {
            width: 150px;
            height: 150px;
            background-color: purple;
            border-radius: 50%; /* 圓形 */
        }
    </style>
</head>
<body>
    <div class="circle"></div>
</body>
</html>
```

---

10. CSS 動態樣式切換 (Hover + Transition)

說明：滑鼠懸停改變樣式，並使用動畫過渡效果。
```html
<!DOCTYPE html>
<html lang="en">
<head>
    <style>
        .btn {
            padding: 10px 20px;
            background-color: #007BFF;
            color: white;
            border: none;
            transition: background-color 0.3s ease;
        }
        .btn:hover {
            background-color: #0056b3; /* 改變顏色 */
        }
    </style>
</head>
<body>
    <button class="btn">懸停看看</button>
</body>
</html>
```

---

這些範例涵蓋進階的 CSS 使用技巧，包括樣式優先級、偽元素、變形與動畫、以及響應式設計等，幫助組員提升實際開發能力。

