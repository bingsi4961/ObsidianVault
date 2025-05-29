---
title: const、var、let 的差異
tags: [JavaScript]

---

作用域（Scope）：
`var` 是函數作用域，這意味著它只在聲明它的函數內部可見。而 `let` 和 `const` 是塊級作用域，它們只在聲明它們的代碼塊（由花括號 {} 定義）內部可見。讓我用代碼來說明：

```javascript
function example() {
    if (true) {
        var x = 1;  // 函數作用域
        let y = 2;  // 塊級作用域
        const z = 3;  // 塊級作用域
    }
    console.log(x);  // 輸出 1
    console.log(y);  // 報錯：y is not defined
    console.log(z);  // 報錯：z is not defined
}
```

變量提升（Hoisting）：
`var` 聲明會被提升到其作用域的頂部，==但賦值不會==。而 `let` 和 `const` 不會被提升，它們存在"暫時性死區"（Temporal Dead Zone）：

```javascript
console.log(x);  // 輸出 undefined
console.log(y);  // 報錯：Cannot access 'y' before initialization
console.log(z);  // 報錯：Cannot access 'z' before initialization

var x = 1;
let y = 2;
const z = 3;
```

重新賦值和重新聲明：
- `var` 允許重新聲明和重新賦值
- `let` 允許重新賦值，但不允許重新聲明
- `const` 既不允許重新賦值，也不允許重新聲明

```javascript
var a = 1;
var a = 2;  // 允許
a = 3;      // 允許

let b = 1;
let b = 2;  // 報錯：不能重新聲明
b = 3;      // 允許

const c = 1;
const c = 2;  // 報錯：不能重新聲明
c = 3;        // 報錯：不能重新賦值
```

對於引用類型（如物件和數組），`const` 有一個特殊的行為需要注意：雖然不能重新賦值，但可以修改內部屬性：

```javascript
const person = { name: 'Alice' };
person.name = 'Bob';  // 這是允許的
person = { name: 'Charlie' };  // 這會報錯
```

實際使用建議：
1. 優先使用 `const`：當你知道變量不會被重新賦值時
2. 其次使用 `let`：當你需要重新賦值時
3. 避免使用 `var`：因為它可能導致作用域相關的問題

---

讓我深入解釋 JavaScript 中的作用域概念。作用域本質上決定了變量在代碼中的可訪問範圍。

首先，讓我們來看函數作用域（Function Scope）：
```javascript
function outerFunction() {
    var x = 1;  // 函數作用域變量
    
    function innerFunction() {
        var y = 2;  // 另一個函數作用域變量
        console.log(x);  // 可以訪問外部函數的 x
        console.log(y);  // 可以訪問自己的 y
    }
    
    innerFunction();
    console.log(x);  // 可以訪問 x
    console.log(y);  // 錯誤：y 在這裡不可見
}
```

在這個例子中，`var` 聲明的變量 x 在整個 `outerFunction` 中都可以訪問，而 y 只在 `innerFunction` 中可見。這就是函數作用域的特點。

現在讓我們看看塊級作用域（Block Scope）：
```javascript
function example() {
    let count = 0;  // 函數開始的塊級作用域
    
    if (true) {
        let innerCount = 1;  // if 塊的作用域
        const MAX_VALUE = 100;  // 也是 if 塊的作用域
        
        console.log(count);      // 可以訪問外部的 count
        console.log(innerCount); // 可以訪問
        console.log(MAX_VALUE);  // 可以訪問
    }
    
    console.log(count);      // 可以訪問
    console.log(innerCount); // 錯誤：不能訪問
    console.log(MAX_VALUE);  // 錯誤：不能訪問
}
```

這裡的 `let` 和 `const` 遵循塊級作用域規則，它們只在聲明它們的代碼塊（由 {} 定義）內可見。

讓我們再看一個更複雜的例子，包含循環：
```javascript
function loopExample() {
    for (let i = 0; i < 3; i++) {
        // i 只在這個循環塊中可見
        const message = `當前索引：${i}`;  // 每次循環都是新的 message
        console.log(message);
    }
    // console.log(i);       // 錯誤：i 在這裡不可見
    // console.log(message); // 錯誤：message 在這裡不可見
}
```

對比使用 var 的情況：
```javascript
function loopWithVar() {
    for (var i = 0; i < 3; i++) {
        var message = `當前索引：${i}`;  // var 不遵循塊級作用域
    }
    console.log(i);       // 可以訪問，值為 3
    console.log(message); // 可以訪問，值為最後一次循環的消息
}
```

特別需要注意的是閉包（Closure）與作用域的關係：
```javascript
function createCounter() {
    let count = 0;  // 這個變量被閉包捕獲
    
    return {
        increment: function() {
            count++;  // 可以訪問外部函數的 count
            return count;
        },
        getCount: function() {
            return count;  // 同樣可以訪問 count
        }
    };
}

const counter = createCounter();
console.log(counter.increment()); // 1
console.log(counter.increment()); // 2
console.log(counter.getCount()); // 2
```

這個例子展示了即使外部函數已經執行完畢，內部函數仍然可以訪問其作用域中的變量。這是因為 JavaScript 的閉包機制保留了對外部作用域的引用。