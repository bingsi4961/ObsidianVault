---
title: ES Modules 與 IIFE 的比較
tags: [JavaScript]

---

## 為什麼 ES Modules 不需要 IIFE

ES Modules 提供了變數作用域的隔離，這與 IIFE（立即調用函數表達式）的主要目的相同。當你使用 `import` 和 `export` 時，每個模組都有自己的獨立作用域。這意味著：

1. **模組作用域**：在一個模組內部宣告的變數、函數和類，預設情況下是模組私有的，除非你明確地使用 `export` 將它們導出。
2. **避免全域污染**：與 IIFE 類似，模組中的變數不會自動變成全域變數，也不會污染全域命名空間。
3. **明確的依賴關係**：使用 `import` 可以明確地表示一個模組依賴於另一個模組，這比 IIFE 中通過參數傳遞依賴關係更加清晰。


```javascript
// moduleA.js
import $ from 'jquery';
const privateVar = "I'm private";  // 這個變數只在此模組內可見

export const moduleA = {
    publicMethod() {
        console.log(privateVar);
    }
};

// moduleB.js
import $ from 'jquery';
const privateVar = "Different private value";  // 不會與 moduleA 中的 privateVar 衝突

export const moduleB = {
    anotherMethod() {
        console.log(privateVar);
    }
};
```

1. `privateVar` 變數不會相互衝突。
2. **語法更簡潔**：不需要 IIFE 的包裹和參數傳遞。
3. **更明確的導出**：使用 `export` 關鍵字明確指定哪些是公開的。

# ES Modules 作用域詳解
ES Modules 實現了真正的模組隔離。當你在 HTML 中導入多個模組時，每個模組都維持著自己的獨立作用域和記憶體空間。這意味著當你執行不同模組中的方法時，它們會各自訪問自己模組內部的變數。

## 深入理解模組作用域

讓我具體解釋發生的事情：

```javascript
// moduleA.js
const privateVar = "I'm from module A";

export function publicMethod() {
    console.log(privateVar);  // 永遠存取 moduleA 的 privateVar
}

// moduleB.js
const privateVar = "I'm from module B";  

export function anotherMethod() {
    console.log(privateVar);  // 永遠存取 moduleB 的 privateVar
}

// 在 HTML 中導入
<script type="module">
    import { publicMethod } from './moduleA.js';
    import { anotherMethod } from './moduleB.js';
    
    publicMethod();  // 輸出: "I'm from module A"
    anotherMethod(); // 輸出: "I'm from module B"
</script>
```

在這個例子中：

1. `publicMethod()` 執行時會回到 `moduleA.js` 的作用域，存取 moduleA 中的 `privateVar`
2. `anotherMethod()` 執行時會回到 `moduleB.js` 的作用域，存取 moduleB 中的 `privateVar`

兩個函數雖然在同一個 HTML 頁面中被調用，但它們各自保持了自己原始模組的詞法作用域 (lexical scope)。

## 技術原理

從技術上說，ES Modules 的這種行為是通過以下機制實現的：

1. **模組實例化**：當瀏覽器載入模組時，它會為每個模組創建一個獨立的環境記錄 (environment record)
2. **連接**：解析所有導入導出關係
3. **求值**：執行模組代碼並填充環境記錄
4. **保持連接**：即使模組代碼執行完畢，模組的環境記錄仍然被保留，供導出的函數訪問

當一個函數從模組被導出並在其他地方調用時，該函數仍然保持對其原始模組環境的引用（通過閉包）。這就確保了無論函數在哪裡被調用，它都能訪問到自己模組中的變數。

## 簡單類比

你可以把它想像成：
* 每個模組就像一間獨立的辦公室
* 模組內的變數是辦公室中的文件和設備
* 模組導出的函數就像是派出去工作的員工
* 這些員工無論去到哪裡工作，都會回到自己的辦公室取需要的文件

這就是為什麼在你的情境中，`publicMethod()` 和 `anotherMethod()` 會各自回到自己的"辦公室"（模組）去存取自己的 `privateVar`。

## 不同物件實例之間的隔離

```javascript
const moduleA = createModule();
const moduleB = createModule();

moduleA.property1 = "moduleA value";
moduleB.property1 = "moduleB value";

console.log(moduleA.property1); // "moduleA value"
console.log(moduleB.property1); // "moduleB value"
```

在這個例子中，`moduleA` 和 `moduleB` 是完全獨立的物件實例，它們的屬性互不影響。

## 工廠函數中的私有變數

進一步說，如果你在工廠函數（factory function）內部宣告變數，然後返回物件，那麼這些變數對物件的方法來說形成了閉包，成為「私有」變數：

```javascript
function createCounter() {
    // 這是一個私有變數，不同的 counter 實例會有各自獨立的 count
    let count = 0;
    
    return {
        increment: function() {
            count++;
            return count;
        },
        decrement: function() {
            count--;
            return count;
        },
        getCount: function() {
            return count;
        }
    };
}

const counterA = createCounter();
const counterB = createCounter();

counterA.increment(); // 1
counterA.increment(); // 2
console.log(counterB.getCount()); // 0，不受 counterA 影響
```

在這個例子中：
1. 每次調用 `createCounter()` 都會創建一個新的 `count` 變數
2. 返回的物件中的方法形成了對這個 `count` 變數的閉包
3. 不同的 counter 實例有各自獨立的 `count` 變數，互不影響

## ES Modules 與工廠函數的結合

在 ES Modules 中，你可以結合模組隔離和工廠函數的優勢：

```javascript
// keyPartColDef.js
let privateModuleVar = "module level privacy";

export function createKeyPartColDef() {
    let privateInstanceVar = "instance level privacy";
    
    return {
        publicMethod() {
            console.log(privateModuleVar);    // 訪問模組級私有變數
            console.log(privateInstanceVar);  // 訪問實例級私有變數
        }
    };
}
```

在這裡：
1. `privateModuleVar` 是模組級別的私有變數，對所有從這個模組創建的實例都是相同的
2. `privateInstanceVar` 是實例級別的私有變數，每個實例都有自己獨立的副本

## 總結

綜上所述：

1. 同一個物件內的屬性和方法共享同一個「命名空間」，可以互相訪問
2. 不同物件實例之間的屬性和方法是隔離的，互不影響
3. 工廠函數內部的變數可以成為返回物件方法的私有變數
4. ES Modules 提供了額外的模組級隔離
