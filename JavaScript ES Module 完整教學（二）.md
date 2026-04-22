---
date: 2026-04-12 11:16
title:
aliases:
  - 別名測試1
  - 別名測試2
tags:
  - 標籤測試1
  - 標籤測試2
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[]]

---
# Part 2：JavaScript 狀態管理與作用域深化

> **閱讀順序**：這是第二篇，請先閱讀 Part 1《ES Modules 完整指南》後再閱讀本篇。
> 
> **適合對象**：已理解 `import / export` 基本語法，想進一步搞懂「模組共用狀態」、「工廠函式獨立實例」，以及「跨分頁資料同步」的開發者。特別適合有 C# 背景的讀者，本篇大量使用 C# 概念作為對照橋樑。

---

## 第一章：你在哪個「宇宙」？——釐清記憶體的邊界

在開始之前，我們先建立一個最重要的心理模型：**JavaScript 的變數記憶體，存在於明確的邊界之內**。搞清楚這個邊界，接下來的一切都會豁然開朗。

### 1.1 同一個分頁內（Tab）

在 Part 1 我們學過，一個模組無論被 `import` 幾次，瀏覽器只會執行它一次。這代表在同一個分頁裡，**所有引入同一個模組的 `.js` 檔案，共用的是同一個記憶體空間**。

```javascript
// counterModule.js
export let sharedCount = 0;
export function add() { sharedCount++; }
```

```javascript
// fileA.js
import { sharedCount, add } from './counterModule.js';
add(); // sharedCount 在記憶體中變成 1
```

```javascript
// fileB.js
import { sharedCount } from './counterModule.js';
console.log(sharedCount); // 印出 1，因為大家看的是同一個變數
```

這就像公司裡大家共用同一間茶水間——A 部門的人喝了一杯咖啡，B 部門的人進來時看到的就是已經少了一杯豆子的咖啡機。

### 1.2 不同分頁之間（Tab A vs Tab B）

這是很多後端轉前端開發者最容易踩的坑：**每一個瀏覽器分頁（Tab）都是完全獨立的執行緒與記憶體空間**。

就算 A 分頁和 B 分頁開的是同一個網址、引用同一個模組，它們的記憶體也是絕對隔離的，像是平行宇宙一樣。A 分頁裡任何變數的改動，B 分頁完全感知不到。

> 💡 這個「分頁隔離」的規則適用於所有依賴 JavaScript 變數的技術，包含模組、工廠函式、Callback，甚至 React / Vue 的狀態管理。只要資料活在「變數」裡，它就不可能跨分頁共用。

---

## 第二章：共用 vs 獨立——ES Module 與工廠函式的差異

現在我們來面對一個核心問題：**有時候我們要共用狀態，有時候我們要獨立狀態，怎麼做到？**

### 2.1 模組變數天生是共用的

在 Part 1 我們知道，模組只執行一次。這代表模組頂層宣告的變數，是整個分頁層級共用的「單例（Singleton）」。

這很好用，但也很危險。來看一個常見的陷阱：

```javascript
// ❌ 踩坑示範 (config.js)
let userRole = "guest"; // 放在模組頂層

export function setUser(role) {
    userRole = role;
}
export function getRole() {
    return userRole;
}
```

坑在哪裡？假設 A 畫面呼叫了 `setUser('admin')`，當使用者切換到 B 畫面時，B 呼叫 `getRole()` 拿到的會是 `'admin'`！因為 `userRole` 是整個應用程式共享的。如果你希望每個畫面有各自獨立的狀態，絕對不能把變數放在模組頂層。

> ⚠️ **這個坑的前提是 SPA（單頁應用程式）架構，一定要先搞清楚**
> 
> 你可能會直覺覺得：「不同畫面應該會重新載入 JS，B 畫面拿到的 `userRole` 難道不是重置過的 `"guest"` 嗎？」——這個直覺在**傳統多頁網站**是正確的，但在 **SPA** 裡就不是了。
> 
> **SPA（Single Page Application）** 的特點是：整個網站從頭到尾只有一份 HTML，使用者點選選單切換「畫面」時，瀏覽器並沒有跳頁、也沒有重新載入 JS——JavaScript 只是把「A 畫面的 HTML 區塊」隱藏起來，再把「B 畫面的 HTML 區塊」顯示出來而已。
> 
> 比較一下兩種架構的差異：
> 
> |操作|傳統多頁網站（MPA）|SPA 單頁應用程式|
> |:--|:--|:--|
> |使用者點擊「切換畫面」|瀏覽器發出新請求，伺服器回傳新 HTML|JavaScript 用顯示/隱藏切換畫面內容|
> |JS 是否重新載入？|**是**，全部重新執行|**否**，從頭到尾只載入一次|
> |模組頂層變數是否重置？|**是**，歸零|**否**，保留上一個畫面留下的值|
> 
> 因此，在 SPA 架構裡，A 畫面改動了 `userRole`，切換到 B 畫面時 `userRole` 完全沒有被重置，就還是 A 畫面留下來的值。這個殘留狀態，就是這個坑的根源。
> 
> **如何判斷你的系統是不是 SPA？** 最簡單的方式：切換畫面時，觀察瀏覽器的網址列是否有短暫的白屏或重新整理的感覺。如果切換得很流暢、完全感覺不到頁面重載，那很可能就是 SPA。

### 2.2 工廠函式——創造獨立的實例

如果說「模組」是一間共用的茶水間，那**工廠函式（Factory Function）**就像是一台「保險箱製造機」。每次呼叫它，就會打造一個全新的保險箱（私有變數），並給你幾把專屬的鑰匙（操作方法），別人絕對打不開你的保險箱。

```javascript
// 這就是一個工廠函式——回傳一個含有私有狀態的物件
function createCounter() {
    // 每次呼叫，都會產生一個全新的、獨立的 count
    let count = 0;

    // 回傳的物件是外部唯一的操作管道
    return {
        add: function() {
            count++;
            return count;
        },
        show: function() {
            return count;
        }
    };
}

const myCounter = createCounter();   // 打造第一個保險箱
const yourCounter = createCounter(); // 打造第二個保險箱

myCounter.add();
console.log(myCounter.show());   // 1
console.log(yourCounter.show()); // 0——完全不受影響，是兩個獨立的實例
```

### 2.3 架構選擇指南

|資料特性|應使用的技術|範例情境|
|:--|:--|:--|
|全站唯一、需要共用|**ES Module 頂層變數**|購物車總數、使用者登入狀態、系統設定|
|每個物件各自獨立|**工廠函式**|每張商品卡片的計數器、遊戲裡每隻怪物的生命值|

以電商網站為例：右上角的「購物車圖示顯示的總數」是全站共用的，適合放在 Module；但每張商品卡片「這個商品要買幾個」是各自獨立的，適合用工廠函式。

---

## 第三章：工廠函式與閉包——C# 開發者的橋樑

身為 C# 開發者，你對「建立物件」最熟悉的方式就是 `new ClassName()`。現在讓我幫你把 JavaScript 工廠函式的機制，完整對照到你已知的 C# 知識。

### 3.1 每次呼叫函式，就是在「配置新的記憶體空間」

每次你呼叫 `createCounter()`，JavaScript 引擎就會在記憶體中開啟一個**全新的、專屬的執行空間**。在這個空間裡，`let count = 0` 被創造出來。如果呼叫了 10 次函式，就會開啟 10 個獨立的空間，產生 10 個互不干擾的 `count`。

這就像 C# 中你 `new` 一個物件時，系統為 `private int count` 配置了記憶體——每次 `new`，都是全新的一塊。

### 3.2 閉包——讓記憶體空間不消失的關鍵

函式執行完畢後，如果什麼都沒有 `return`，那個專屬空間跟裡面的 `count` 就會被 JavaScript 的垃圾回收機制（Garbage Collector，簡稱 GC）清掉——就像在 C# 的 Method 裡宣告了一個 `int temp = 0` 但完全沒用到，Method 執行完就消失了。

但當你 `return` 了一個物件，而且這個物件裡面的方法還**引用著**內部的 `count` 時，GC 就會判定：「外面還有人拿著這個 `count` 的遙控器，這個空間不能刪除！」

這個「被保留下來的專屬空間」，在 JavaScript 裡叫做**「閉包（Closure）」**。

```javascript
function createCounter() {
    let count = 0; // 這個變數因為被下面的方法引用，所以會被閉包保留

    return {
        add: function() { count++; },   // 引用了 count，形成閉包
        show: function() { return count; } // 同上
    };
}
```

如果你 `return` 的物件裡的方法根本沒有用到 `count`，那 `count` 就真的會被 GC 回收，不存在了。被 `return` 出去的物件本身會繼續存在（因為外部變數接住了它），但裡面沒有被引用的私有變數就不會存在了。

### 3.3 C# Class vs JS 工廠函式完整對照表

|概念|C# 物件導向寫法|JavaScript 工廠函式寫法|
|:--|:--|:--|
|**定義藍圖**|`class Counter { ... }`|`function createCounter() { ... }`|
|**私有變數（狀態）**|`private int count = 0;`|`let count = 0;`|
|**公開方法（介面）**|`public int Add() { ... }`|`return { add: function() { ... } }`|
|**建構子（初始化）**|`public Counter(int start) { this.count = start; }`|函式參數：`function createCounter(argCount) { let count = argCount; }`|
|**創造實例**|`Counter myCount = new Counter(10);`|`const myCount = createCounter(10);`|
|**封裝（私有屬性）**|`public int Count { get; private set; }`|只在 `return` 的物件裡放讀取方法，不放修改方法|

在 JavaScript 的這個寫法中，雖然沒有寫 `class` 和 `new`，但透過「每次呼叫函式產生新變數」加上「回傳物件把變數鎖住（閉包）」，我們完美模擬出了 C# 產生物件實例的行為。

### 3.4 帶參數的工廠函式——對應 C# 的建構子

就像 C# 的建構子可以接收初始值，工廠函式也可以透過參數來設定實例的初始狀態：

```javascript
function createPlayer(playerName, initialHp) {
    // 傳入的參數本身也會成為被閉包保護的私有變數
    let currentHp = initialHp;

    return {
        getName: function() {
            return playerName;
        },
        takeDamage: function(damage) {
            currentHp -= damage;
        },
        getHp: function() {
            return currentHp;
        }
    };
}

const player1 = createPlayer("騎士", 100);
const player2 = createPlayer("法師", 60);

player1.takeDamage(30);
console.log(player1.getHp()); // 70
console.log(player2.getHp()); // 60——player2 完全不受影響
```

---

## 第四章：雙劍合璧——Module + 工廠函式的組合架構

當我們把模組的「全域共用」與工廠函式的「獨立實例」結合起來，就能設計出非常清晰的架構。以電商購物車為例：

```javascript
// cartStore.js（全域 Module，負責管理全站唯一的購物車總數）
export let totalCartCount = 0;

export function addTotal() {
    totalCartCount++;
    console.log(`目前購物車總數：${totalCartCount}`);
}
```

```javascript
// productFactory.js（工廠函式，負責每張商品卡片的獨立計數）
import { addTotal } from './cartStore.js'; // 🔑 把模組的方法引入進來

export function createProduct(productName) {
    let itemCount = 0; // 私有變數：只屬於這個商品實例

    return {
        add: function() {
            itemCount++;
            console.log(`${productName} 現在有 ${itemCount} 個`);

            // 呼叫模組的方法，同步更新全站的總數
            addTotal();
        }
    };
}
```

```javascript
// main.js
import { createProduct } from './productFactory.js';

const appleCard = createProduct("蘋果");
const bananaCard = createProduct("香蕉");

appleCard.add(); // 蘋果: 1，購物車總數: 1
bananaCard.add(); // 香蕉: 1，購物車總數: 2
appleCard.add(); // 蘋果: 2，購物車總數: 3
```

這就是「局部動作影響全域狀態」的標準做法：在工廠函式的實例方法內，呼叫模組導出的方法，讓兩個層級的狀態同步更新。

---

## 第五章：當變數改變，如何通知別人？——觀察者模式

到目前為止，我們知道 `totalCartCount` 改變了，但其他需要顯示這個數字的 UI 元件要怎麼「即時知道」數字變了？總不能一直去重新讀取吧？

這個問題在 C# 裡你一定遇過：當一個物件的資料改變，你會定義一個 `event`，其他物件去 `Subscribe` 它，資料一變就觸發通知。JavaScript 裡也是完全一樣的邏輯，我們稱之為**「觀察者模式（Observer Pattern）」**或**「發布/訂閱模式（Pub/Sub）」**。

### 5.1 概念：廣播電台與收音機

這套機制用廣播來理解最直覺：Module（購物車總數）是廣播電台，其他需要更新畫面的 UI 元件是收音機。收音機必須先向電台**「訂閱（Subscribe）」**：「嘿，數字改變時請呼叫我的這個方法」。當工廠函式（蘋果）觸發數字 +1，電台就廣播給所有已訂閱的收音機。

### 5.2 程式碼實作

在 Module 裡維護一個「訂閱者名單（陣列）」，並開放訂閱管道：

```javascript
// cartStore.js（升級版：加入發布/訂閱機制）
export let totalCartCount = 0;

// 用一個陣列記錄所有想被通知的回呼函式（Callbacks）
let listeners = [];

// 開放訂閱管道，讓其他檔案把「更新畫面的方法」傳進來
export function subscribe(callback) {
    listeners.push(callback);
}

export function addTotal() {
    totalCartCount++;

    // 數字一變，立刻通知所有訂閱者
    listeners.forEach(callback => {
        callback(totalCartCount);
    });
}
```

在需要更新畫面的 UI 元件裡，向模組註冊：

```javascript
// headerUI.js（負責更新右上角購物車圖示）
import { subscribe } from './cartStore.js';

// 向 Module 說：「數字改變時，請執行這個函式」
subscribe(function(newCount) {
    document.getElementById('cart-icon-text').innerText = newCount;
});
```

這就是為什麼大型網頁需要這種「註冊與廣播」機制，才能讓畫面跟資料保持同步。現代最熱門的前端框架（React、Vue、Angular）底層最核心的技術之一，就是把這套「發布/訂閱」機制自動化了——你只要改變數，框架就會自動幫你廣播並更新畫面。

> ⚠️ **Callback 和 Observer 只適用於單一分頁**
> 
> 我們剛才寫的 `let listeners = []`，是存在 JavaScript 變數記憶體中的。根據第一章說的分頁隔離原則，這個陣列在 Tab A 有一份、Tab B 有另一份，兩者是平行宇宙，絕對不能跨分頁通訊。如果你需要「A 分頁的操作要讓 B 分頁知道」，就需要突破 JavaScript 記憶體的邊界——這就是下一章的主題。

---

## 第六章：突破分頁的牆——跨分頁資料同步

### 6.1 三種儲存武器

要打破分頁的平行宇宙，我們就不能再依賴「JavaScript 變數」，必須借助獨立於 JS 記憶體之外、且所有分頁都能存取的共用空間：

|儲存方式|生命週期|能跨分頁？|實務用途|
|:--|:--|:--|:--|
|**Cookie**|可設定到期時間|✅ 可以|登入憑證（Token）、跟伺服器傳遞|
|**LocalStorage**|永久（除非手動清除）|✅ 可以|使用者偏好設定、跨分頁共用的計數|
|**SessionStorage**|跟著單一分頁走，關閉就消失|❌ 不行|同一分頁的暫存資料|

> ⚠️ **SessionStorage 的名字很誤導人！** 雖然名字裡有 Session，很多人以為它像 ASP.NET 的 `Session`（存在伺服器端、跨頁面共用），但瀏覽器的 SessionStorage 完全不是這回事——它只活在單一分頁，關掉就消失了，無法用來做跨分頁同步。

### 6.2 LocalStorage + storage 事件——跨分頁的即時廣播

LocalStorage 解決了「把資料存在哪裡讓大家都能讀」的問題，但「A 分頁寫了資料，B 分頁怎麼即時知道？」還需要另一個機制。

瀏覽器為此內建了一個跨分頁的廣播系統，叫做 **`storage` 事件**。當 A 分頁修改了 LocalStorage 的資料時，瀏覽器會主動向**其他所有開啟同網站的分頁**發送通知。B 分頁只要監聽這個事件，就能立刻更新畫面：

```javascript
// A 分頁：使用者點擊「加入購物車」
function addToCart() {
    const current = parseInt(localStorage.getItem('cartTotal') || '0');
    localStorage.setItem('cartTotal', current + 1);
    // 寫入 LocalStorage 這個動作，會自動觸發其他分頁的 storage 事件
}

// B 分頁：在背景默默監聽
window.addEventListener('storage', function(event) {
    if (event.key === 'cartTotal') {
        console.log(`收到其他分頁的廣播！新的總數是：${event.newValue}`);
        // 這裡更新 B 分頁畫面上的數字
        document.getElementById('cart-icon-text').innerText = event.newValue;
    }
});
```

---

## 第七章：狀態管理三層架構——全貌總結

恭喜你走到這裡！現在讓我們把所有拼圖拼起來，看到完整的全貌：

```
【資料存放位置決策框架】

需要「各物件獨立的狀態」嗎？
  → 使用 工廠函式（Factory Function）+ 閉包（Closure）
  → 例如：每張商品卡片各自的數量、遊戲裡每個玩家的生命值

需要「單一分頁內所有元件共用的狀態」嗎？
  → 使用 ES Module 頂層變數 + 發布/訂閱（Observer Pattern）
  → 例如：購物車總數、分頁內的全局設定、登入後的使用者資訊

需要「跨分頁共用，且頁面刷新後也要保留」的狀態嗎？
  → 使用 LocalStorage + storage 事件
  → 例如：在 A 分頁買東西、B 分頁的購物車圖示同步更新

需要「跟伺服器交換、或設定到期時間」的狀態嗎？
  → 使用 Cookie
  → 例如：登入 Token、記住我的功能
```

用表格的方式對比：

|技術|比喻|共享範圍|適合存放什麼|
|:--|:--|:--|:--|
|**工廠函式（閉包）**|個人專屬的密碼櫃|物件層級（各自獨立）|個別物件的私有狀態|
|**ES Module 頂層變數**|辦公室共用的茶水間|分頁層級（單一分頁內共用）|全分頁共享的設定、總計數器|
|**LocalStorage**|公司大樓裡的公告欄|瀏覽器層級（所有分頁）|跨分頁共用的資料、持久化偏好|
|**Cookie**|蓋了印章的通行證|瀏覽器層級（可傳給伺服器）|登入憑證、到期的暫存資料|

---

## 本篇完整回顧

這兩篇文章帶你走過了一條從「語法」到「架構思維」的完整旅程：

在 **Part 1** 裡，你學會了 ES Module 的所有語法細節——具名匯出、預設匯出、重命名、統包匯入、重新匯出，以及在 HTML 裡正確串接的方式。

在 **Part 2** 裡，你理解了更深層的觀念——為什麼模組變數是共用的、為什麼函式呼叫可以產生獨立的實例、閉包是什麼、GC 如何判斷哪些記憶體可以回收，以及當資料需要跨元件、跨分頁流動時，應該選擇哪種技術。

最重要的是，這些觀念並不是 JavaScript 專屬的黑魔法——它們背後的設計邏輯，在 C# 的封裝、建構子、事件系統中都能找到對應的影子。你已有的知識，是學習這些概念最好的橋樑。