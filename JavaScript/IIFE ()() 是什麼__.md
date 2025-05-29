---
title: IIFE ()() 是什麼??
tags: [JavaScript, jQuery]

---

在 JavaScript 中，`()()` 這種語法稱為「立即調用函數表達式」(Immediately Invoked Function Expression，簡稱 IIFE)。讓我來詳細解釋這個概念。

首先，我們可以把它拆解成兩個部分來看：
==1. 第一個括號 `()` 包含了函數的定義==
==2. 第二個括號 `()` 用來立即呼叫這個函數==

來看一個具體的例子：

```javascript
(function() {
    console.log("Hello!");
})();
```

這段程式碼會立即執行並輸出 "Hello!"。

IIFE 的主要用途包括：

1. **創建私有作用域**：
```javascript
(function() {
    var privateVar = "我是私有變數";
    // 這裡的變數不會污染全局作用域
})();
// 在這裡無法訪問 privateVar
```

2. **避免命名衝突**：
```javascript
(function() {
    var name = "Alice";
    // 這裡的 name 不會與全局的 name 變數衝突
})();
```

```javascript
// 不好的寫法
function setup() {
    // ... 初始化代碼
}
setup();

// 更好的寫法
(function() {
    // ... 初始化代碼
})();
```

IIFE 還有一個變體寫法，可以傳入參數：

```javascript
(function(name) {
    console.log("Hello, " + name + "!");
})("Alice");  // 輸出：Hello, Alice!
```

在現代 JavaScript 中，由於有了塊級作用域（let 和 const）和模組系統，IIFE 的使用已經沒有以前那麼普遍。但理解這個概念仍然很重要。

# 不同的 IIFE 之間呼叫函數

```javascript=
// 第一個 IIFE
var module1 = (function($) {
    return {
        someFunction: function() {
            module2.doSomething();
        }
    };
})(jQuery);

// 第二個 IIFE
var module2 = (function($) {
    return {
        doSomething: function() {
            console.log('這是第二個 IIFE 的函數');
        }
    };
})(jQuery);
```

每個 IIFE 會創建自己的作用域，即使 module1 和 module2 被設為全域變數，它們內部的私有變數也不會互相影響。讓我用例子說明：

```javascript
var module1 = (function($) {
    // 私有變數
    var name = "module1";
    var data = { count: 0 };
    
    // 公開的方法和變數
    return {
        name: name,
        getData: function() {
            return data.count;
        },
        increment: function() {
            data.count++;
        }
    };
})(jQuery);

var module2 = (function($) {
    // 同樣名稱的私有變數
    var name = "module2";
    var data = { count: 100 };
    
    return {
        name: name,
        getData: function() {
            return data.count;
        },
        increment: function() {
            data.count++;
        }
    };
})(jQuery);

// 測試
console.log(module1.name);  // 輸出: "module1"
console.log(module2.name);  // 輸出: "module2"

module1.increment();
console.log(module1.getData());  // 輸出: 1
console.log(module2.getData());  // 輸出: 100
```

這裡有幾個重要的概念：

1. **閉包 (Closure)**: 每個 IIFE 創建一個閉包，使內部私有變數（如 `name` 和 `data`）保持獨立。

2. **私有變數**: IIFE 內部宣告的變數（如 `name` 和 `data`）對外部是不可見的，只能通過返回的公開方法訪問。

3. **公開介面**: 通過 return 的物件定義模組的公開介面，只有這些被返回的方法和屬性才能被外部訪問。

為了更清楚地展示變數的獨立性，我們可以這樣測試：

```javascript
var module1 = (function($) {
    var privateVar = "module1 private";
    var sharedName = "Alice";
    
    return {
        getPrivate: function() {
            return privateVar;
        },
        getSharedName: function() {
            return sharedName;
        },
        setSharedName: function(name) {
            sharedName = name;
        }
    };
})(jQuery);

var module2 = (function($) {
    var privateVar = "module2 private";
    var sharedName = "Bob";
    
    return {
        getPrivate: function() {
            return privateVar;
        },
        getSharedName: function() {
            return sharedName;
        },
        setSharedName: function(name) {
            sharedName = name;
        }
    };
})(jQuery);

// 測試變數獨立性
console.log(module1.getSharedName());  // 輸出: "Alice"
console.log(module2.getSharedName());  // 輸出: "Bob"

module1.setSharedName("Charlie");
console.log(module1.getSharedName());  // 輸出: "Charlie"
console.log(module2.getSharedName());  // 輸出: "Bob" (不受影響)
```

即使兩個模組都有相同名稱的變數（如 `privateVar` 和 `sharedName`），它們也是完全獨立的。修改一個模組的變數不會影響另一個模組的變數。

唯一需要注意的是模組名稱本身（`module1` 和 `module2`）不要重複，因為它們是全域變數。你可以使用命名空間來進一步組織這些模組，例如：

```javascript
var MyApp = MyApp || {};
MyApp.module1 = (function($) {
    // 模組內容
})(jQuery);

MyApp.module2 = (function($) {
    // 模組內容
})(jQuery);
```

這樣可以進一步減少全域命名空間的污染。