---
date: 2026-05-07 14:00
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

## 前言：你一定踩過這些坑

如果你曾經在寫 jQuery 時碰過以下情況，這篇文章就是為你而寫的：

- 明明用程式修改了 `input` 的值，畫面卻沒有跟著更新
- 想抓取 checkbox 有沒有被打勾，結果拿到的是奇怪的字串而不是 `true`/`false`
- 用 `.attr()` 還是 `.prop()` 傻傻分不清楚

這些問題的根源，幾乎都指向同一個觀念沒釐清：**HTML Attribute 和 DOM Property 根本是兩件不同的事**。

讓我們從頭開始，把這個地基打牢。

---

## 一、最根本的問題：HTML 與 DOM，到底是什麼關係？

當瀏覽器讀取你寫的 `.html` 檔案時，對它來說，那最初只是一串**純文字**。為了讓 JavaScript 能夠操作網頁內容，瀏覽器會把這些文字解析，轉換成記憶體裡的「物件模型」，也就是我們常聽到的 **DOM（Document Object Model）樹**。

這個過程，我們可以用「蓋房子」來理解：

### 🏗️ HTML Attribute（HTML 屬性）＝ 建築藍圖

你在 HTML 原始碼裡寫的那些屬性，例如 `value="華碩"`、`checked="checked"`，就像是建築師畫的藍圖。藍圖上記錄的是**事物的初始狀態**——大門預設是開著的、臥室預設漆成白色。

### 🏠 DOM Property（DOM 物件特徵）＝ 實際蓋好的房子

當瀏覽器把藍圖「蓋成」真實的 DOM 物件後，房子就有了自己的生命。如果有人走進去把門關上，**房子的現況改變了，但那張紙本藍圖並不會自動跟著改變**。

這個活在記憶體中、隨著使用者操作隨時變動的狀態，就是 **DOM Property**，記錄的是**當前狀態**。

> 💡 **一句話總結**：Attribute 是 HTML 原始碼裡的初始設定；Property 是瀏覽器解析後，DOM 物件在記憶體裡的最新狀態。這兩者在網頁載入後就「脫鉤」了，各自獨立存在。

---

## 二、用一個具體例子感受兩者的差異

假設有一段 HTML：

```html
<input type="text" value="華碩" id="myInput">
```

網頁載入後，使用者在輸入框裡把「華碩」刪掉，改打成「ROG」。

**請問這個當下：**

- **HTML Attribute**（藍圖）的 `value` 是什麼？→ 依然是 `"華碩"`
- **DOM Property**（現況）的 `value` 是什麼？→ 已經變成 `"ROG"`

藍圖記住的是最初的設定，Property 反映的是當下的現實。這就是整個觀念的核心。

---

## 三、為什麼搞混了會出 Bug？以 Checkbox 為例

Checkbox 是最容易讓人踩坑的地方。假設我們有一個預設打勾的同意條款：

```html
<input type="checkbox" checked="checked" id="agree">
```

使用者進入網頁後，把打勾取消了。這時候我們想用程式判斷他有沒有同意。

**❌ 用 Attribute 檢查（錯誤示範）：**

```javascript
$('#agree').attr('checked')
```

這會去翻原本的藍圖，發現藍圖上寫著 `checked`，所以回傳一個字串（非空值）。程式誤以為使用者有打勾——這就是最常見的 Bug！

**✅ 用 Property 檢查（正確做法）：**

```javascript
$('#agree').prop('checked')
```

這會去看房子的現況，發現使用者已經把勾勾取消了，精準回傳布林值 `false`。

> ⚠️ **常見誤區**：Attribute 永遠回傳的是**字串（String）**，就算你寫的是 `checked="checked"`，它也只是個字串，不是布林值。DOM Property 才能給你真正的 `true` / `false`。

---

## 四、什麼時候該用哪個工具？

這是一張你可以貼在桌上的判斷表：

|評估維度|HTML Attribute（藍圖）|DOM Property（現況）|
|---|---|---|
|**關注點**|最初始的預設狀態|當下真實的物件狀態|
|**資料型態**|永遠是字串（String）|可以是布林值、數字或字串|
|**jQuery 語法**|`.attr('屬性名')`|`.prop('屬性名')`|
|**原生 JS 語法**|`.getAttribute('屬性名')`|`.value`、`.checked` 等直接點呼叫|
|**實務應用時機**|抓取不會隨操作改變的東西（如 `data-*` 屬性、`id`、`src`）|抓取會跟著使用者操作改變的東西（如輸入框的值、打勾狀態）|

**記住這個判斷原則**：只要是**會隨著使用者在畫面上點擊、輸入而改變狀態的元素，一律看 Property**；如果是想抓取一開始設定好、拿來做標記用的靜態資料，就找 Attribute。

---

## 五、進階觀念：哪些 Property 會「反射」回 HTML Attribute？

這裡有一個很多人不知道的細節，也是你按下 F12 看 Elements 面板時會觀察到的現象。

### F12 看到的「HTML」其實不是原始碼

當你按下 F12 開啟開發者工具的 Elements 面板，你看到的標籤結構其實**已經不是**伺服器最初傳過來的那份 HTML 純文字了。那是瀏覽器把 DOM 物件「反向渲染」出來的 **Live DOM（活的視覺化結構）**。

### Reflected Properties（反射屬性）

為了讓 CSS 選擇器能正確運作，瀏覽器設計了一個機制叫做 **Reflected Properties**。簡單說，有些 DOM Property 改變時，瀏覽器會強制同步更新到你在 F12 看到的 Live Attribute 上。

用房子來比喻：換掉大門的門牌號碼（`id`）、改變房子的外牆顏色（`class`），這些是「公開且影響市容」的特徵。市政府（瀏覽器）規定這類特徵一改變，就必須同步更新到門口的「公開展示牌」（F12 的 Attribute 欄位）上，讓路人（CSS 選擇器）看到最新狀態。

但房子**內部**的狀態（信箱裡有沒有信 `value`、電燈有沒有開 `checked`），改了就不需要更新到展示牌。

|屬性類別|同步機制|實務範例|說明|
|---|---|---|---|
|核心標準標識|✅ 雙向同步|`id`, `class`, `href`, `src`, `title`|主要讓 CSS 選擇器能隨時抓到最新狀態|
|表單現況 / 自訂資料|❌ 不同步|輸入框的 `value`、`checked`、`data-*`|只存在記憶體現狀，HTML Attribute 維持初始預設值|

---

## 六、自訂屬性（`data-*`）的特殊情況

如果你在按鈕上藏了商品 ID：

```html
<button data-product-id="9527" id="buyBtn">購買</button>
```

**該怎麼抓 `data-product-id`？**

應該用 `.attr('data-product-id')`，而不是 `.prop('data-product-id')`。

原因是：DOM 物件像一張「官方標準表格」，`id`、`src`、`value` 這些標準屬性在表格上都有專屬欄位，所以會直接轉換為獨立的 Property。但 `data-product-id` 這種自訂屬性，官方表格上沒有預留欄位——硬要用 `.prop()` 去找，只會拿到 `undefined`。

瀏覽器把所有 `data-*` 屬性統一收納進 DOM 物件的 **`dataset`** 資料夾裡。所以你也可以用原生 JS 這樣存取：

```javascript
// 注意：data-product-id 轉成 camelCase 變成 productId
document.getElementById('buyBtn').dataset.productId // "9527"
```

---

## 七、jQuery 的 `.data()` 是另一個世界

說到 `data-*`，你可能常看到這樣的寫法：

```javascript
$('#buyBtn').data('product-id') // 讀取
$('#buyBtn').data('product-id', 9999) // 寫入
```

⚠️ **這裡藏了一個非常多工程師不知道的陷阱**。

jQuery 的 `.data()` 既不會寫入 HTML 藍圖，也不會寫進 DOM 的 `dataset`。它在自己的系統裡建立了一本**「秘密筆記本」**（內部快取記憶體）：

- **第一次讀取**時，jQuery 去 DOM 查初始的 `data-product-id`，把值抄進筆記本，然後回傳給你。
- **寫入**時，jQuery 只把新值更新在筆記本裡，**完全不動 DOM，也不動 HTML 藍圖**。

這就導致了一個非常經典的踩坑場景：

```javascript
$('#buyBtn').data('product-id', 9999);
let result = $('#buyBtn').attr('data-product-id'); // 回傳舊的 "9527"！
```

你用 `.data()` 更新了資料，打開 F12 看 HTML 標籤，屬性還是舊的值，然後以為程式壞了——其實程式沒壞，新資料只是在 jQuery 的記憶體裡，F12 的 HTML 面板看不到。

**更嚴重的後果**：如果你用 CSS 選擇器根據 `data-*` 狀態改變外觀：

```css
button[data-status="success"] { background-color: green; }
```

然後用 `.data()` 去改狀態——按鈕**不會變綠**，因為 CSS 看的是 DOM/HTML Attribute，不是 jQuery 的筆記本。

**何時該用哪個？**

- `.attr('data-*', 值)`：當這個狀態**會影響 CSS 樣式**，或需要讓原生 JS 也能讀到最新狀態時使用。
- `.data()`：當你只是在 JavaScript 邏輯裡**暫存複雜的資料**（如 JSON 物件、陣列），不需要影響外觀時使用，效能較好。

---

## 八、Boolean Attributes 的正確處理方式

像 `disabled`、`checked`、`readonly` 這類屬性，**只要出現在標籤上就代表生效**：

```html
<input disabled>  <!-- 這樣就夠了，不需要寫 disabled="disabled" -->
```

在舊時代，大家習慣這樣操作：

```javascript
// 舊做法（容易踩坑）❌
$('#btn').attr('disabled', 'disabled') // 鎖住
$('#btn').removeAttr('disabled')       // 解開
```

問題是：我們在乎的是「現在是不是禁用狀態 true/false」，而不是「標籤上有沒有 `disabled` 這個單字」。用字串操作，有時會讓瀏覽器的真實狀態判斷錯亂，明明拔掉了屬性，輸入框卻還是鎖死的。

**現代標準做法：直接操作 Property，給明確的布林值**：

```javascript
// 正確做法 ✅
$('#btn').prop('disabled', true)  // 鎖住
$('#btn').prop('disabled', false) // 解開（不破壞藍圖）

// 或用原生 JS
document.getElementById('btn').disabled = true
document.getElementById('btn').disabled = false
```

> 💡 **原則**：只要是處理 `disabled`、`checked`、`readonly` 這類「只有是與否兩種狀態」的特徵，一律使用 `.prop()` 搭配布林值。