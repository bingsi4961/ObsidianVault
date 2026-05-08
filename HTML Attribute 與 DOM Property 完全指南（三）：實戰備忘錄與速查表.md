---
date: 2026-05-08 08:55
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

## 一、F12 親手測試驗證結果整理

### 1. `input type="text"`（文字輸入框）

以 `<input type="text" value="初始文字" id="myInput">` 為例。

**讀取初始狀態：**

- `el.getAttribute('value')` → 從 HTML 藍圖讀取，永遠是 `"初始文字"`
- `el.defaultValue` → 從 DOM 讀取初始快照，同樣是 `"初始文字"`

**修改初始狀態（修改藍圖）：**

- 使用 `el.setAttribute('value', '新值')` 或 jQuery `.attr('value', '新值')` 後，會連動更新 HTML Attribute **和** `defaultValue`，但此後 `.value`（畫面）的行為取決於「是否已弄髒」

**修改當前狀態（修改畫面）：**

- 使用 `el.value = '新值'` 或 jQuery `.val('新值')`，會連動更新畫面

**手動編輯：**

- 使用者在畫面上打字後，會連動更新 `.value` / `.val()`，同時觸發弄髒機制，使 `.value` 與 `defaultValue` 脫鉤

---

### 2. `textarea`（多行文字輸入框）

以 `<textarea id="memo">初始文字</textarea>` 為例。

**初始狀態三位一體**（修改其中一個，其他兩個連動）：

- `el.defaultValue` ↔ `el.innerHTML` ↔ `el.textContent`，三者互相連動，都代表「HTML 藍圖上的初始文字」

**修改當前狀態（修改畫面）：**

- 使用 `el.value = '新值'` 或 jQuery `.val('新值')` → 連動更新畫面

**手動編輯：**

- 使用者在 textarea 裡打字後 → 連動更新 `.value` / `.val()`

**特別注意**：`el.innerHTML = ''` 或 jQuery `.empty()` 只能清掉初始浮水印，**無法清空使用者的輸入**，要清空畫面請用 `el.value = ''`

---

### 3. `input type="checkbox"`

以 `<input type="checkbox" id="agree" value="yes" checked>` 為例。

**讀取勾選狀態：**

- `el.checked` → DOM Property，取得當前是否打勾（`true` / `false`）
- `el.defaultChecked` → DOM Property，取得初始預設是否打勾

**關於送出表單的值（注意：`.value` 不代表勾選狀態）：**

- `el.value` → checkbox 勾選並送出時傳給伺服器的值，這裡是 `"yes"`
- `el.defaultValue` → 初始的送出值

**修改藍圖（初始設定）：**

- `el.setAttribute('value', '新值')` 或 `.attr('value', '新值')` → 連動更新 HTML Attribute 和 `defaultValue`
- `el.setAttribute('checked', 'checked')` 或 `.attr('checked', 'checked')` → 連動更新 HTML Attribute 和 `defaultChecked`

**修改當前勾選狀態（修改畫面）：**

- `el.checked = true` / `el.checked = false` 或 jQuery `.prop('checked', true)` → 連動更新畫面

**手動編輯：**

- 使用者點擊後 → 連動更新 `.checked` / `.prop('checked')`

**瀏覽器 Fallback 行為**：執行 `el.removeAttribute('value')` 後，`el.value` 會變成 `'on'`（而非空字串）。這是 HTML 規範的防呆機制——如果你沒設定 `value`，瀏覽器預設送出 `'on'` 表示「此欄位有被觸發」。**實務上一定要幫 checkbox 明確設定 `value`，否則後端收到的全是意義不明的 `'on'`。**

---

### 4. `input type="radio"`

行為與 `input type="checkbox"` 相同，同樣區分 `.checked` / `.defaultChecked` 以及 `.value` / `.defaultValue`，同樣具備 `'on'` 的 Fallback 行為。

---

### 5. `select`（下拉選單）

以以下 HTML 為例：

```html
<select id="meal">
  <option value="beef" selected>牛肉麵</option>
  <option value="pork">排骨飯</option>
</select>
```

**讀取當前選擇：**

- `el.value` 或 `$.val()` → 取得當前已選的 option 的 value 值
- `el.selectedIndex`（或 `el.options.selectedIndex`）→ 取得當前已選的 option 的 index（從 0 開始）

**修改當前選擇（連動更新畫面）：**

- `el.value = 'pork'` 或 `$.val('pork')` → 畫面跟著切換到「排骨飯」

**讀取結構內容：**

- `el.innerHTML` → 存放所有 `option` 的 HTML 結構（切換選項不會改變這裡）
- `el.textContent` → 存放所有 `option` 的純文字（不含 HTML 標籤）

**重要邊界**：以上所有操作都在 DOM 實例中進行，不會變更 HTML 原始碼（HTML Attribute / HTML 結構），這跟 `input text` 的行為是一致的。

---

### 6. `select` 的 `option`（選項）

**讀取個別選項狀態：**

- `el.options[x].selected` → 取得第 x 個 option 當前是否被選中（`true` / `false`）
- `el.options[x].defaultSelected` → 取得第 x 個 option 一開始是否被預設選中

**`option` 的文字內容：**

- `el.options[x].innerHTML` 和 `el.options[x].textContent` 都存放 option 的顯示文字（因為 option 通常不放 HTML 標籤，兩者結果相同）

**修改個別選項狀態：**

- `el.options[x].selected = true` → 把第 x 個 option 設為選中（同時連動 `select.value` 和 `select.selectedIndex`）
- `el.options[x].defaultSelected = true` → 修改初始預設選中狀態
- jQuery 等效：`$(el).find('option').eq(x).prop('selected', true)` → 效果同上，連動 `element.value` 和 `selectedIndex`

**Attribute 與 Property 雙向連動（option 的特殊行為）：**

- `el.options[0].setAttribute('value', '155')` 後，`el.options[0].value` 會連動變成 `155`（不同於 input text 的脫鉤行為，因為 option 不能被使用者直接「弄髒」）
- `el.options[0].removeAttribute('value')` 後，`el.options[0].value` 會直接抓 option 的文字內容（Fallback 機制）

**Attribute 操作連動 `defaultSelected`：**

- `el.options[x].removeAttribute('selected')` → `el.options[x].defaultSelected` 連動變成 `false`
- `el.options[x].setAttribute('selected', 'selected')` → `el.options[x].defaultSelected` 連動變成 `true`

---

## 二、三大工具的使用時機速查

這是整個系列最精煉的一張表，貼在桌上隨時查閱：

|工具|操作的是|適用場景|注意事項|
|---|---|---|---|
|`.attr()` / `getAttribute()`|HTML 藍圖 🏗️|讀寫 `data-*` 等自訂屬性；要影響 CSS 選擇器的狀態屬性；讀取 `id`、`src`、`href` 等靜態設定|永遠回傳字串|
|`.prop()` / 直接點屬性|DOM 實體 🏠|讀寫 `checked`、`disabled`、`selected` 等布林狀態；讀寫 `value`（當前輸入值）|可回傳布林值、數字等正確型別|
|`.data()`|jQuery 內部快取 📓|純粹在 JS 邏輯裡暫存複雜資料（JSON、陣列等），不影響外觀|不會更新 DOM 或 HTML，CSS 選擇器看不到變化，與原生 JS 脫鉤|

---

## 三、`innerHTML` vs `textContent` 速查

|屬性|jQuery 對應|對 HTML 標籤的態度|適用場景|
|---|---|---|---|
|`.textContent`|`.text()`|視為普通文字（不解析）|顯示使用者輸入的任何內容，防 XSS|
|`.innerHTML`|`.html()`|解析並渲染|自己動態生成的安全排版（表格、清單等）|

---

## 四、清空元素內容速查

不同元件的清空方式不同，請看清楚再動手：

|元件類型|清空目標|正確做法|錯誤做法|
|---|---|---|---|
|`input type="text"`|輸入框的當前值|`el.value = ''` 或 `$(el).val('')`|❌ `innerHTML = ''`（無效）|
|`textarea`|使用者輸入的內容|`el.value = ''` 或 `$(el).val('')`|❌ `innerHTML = ''` 或 `.empty()`（只清掉初始文字，使用者輸入不受影響）|
|`select`|清空所有選項|`el.innerHTML = ''` 或 `$(el).empty()`|—|
|`div` / `span` / 容器|清空顯示內容|`el.innerHTML = ''` 或 `$(el).empty()`|—|

---

## 五、表單送出時的 Fallback 行為整理

這個知識點在實際開發中非常重要，雖然不常見但一旦踩坑就很難 debug：

|元件|情況|Fallback 行為|
|---|---|---|
|`checkbox`|沒有設定 `value` attribute|送出時傳 `'on'` 給伺服器|
|`checkbox`|沒有打勾就送出|這個欄位**完全不出現**在送出的資料裡（伺服器端收不到這個 key）|
|`option`|沒有設定 `value` attribute|`option.value` 自動使用 `textContent` 的文字|

關於 checkbox 未勾選的送出行為，許多開發者可能不知道：後端收不到這個欄位，不是收到 `false` 或空字串，而是根本沒有這個 key。這意味著後端需要用「欄位存不存在」來判斷是否勾選，而不是判斷值是 `true` 還是 `false`。

---

## 六、常見踩坑場景整理

### 場景一：改了資料，按鈕顏色沒變

```javascript
// ❌ 問題寫法
$('#submitBtn').data('status', 'success') // 只改了 jQuery 記憶體

// 結果：CSS button[data-status="success"] 完全沒作用
// 因為 DOM 和 HTML Attribute 都沒有變化
```

```javascript
// ✅ 正確寫法
$('#submitBtn').attr('data-status', 'success') // 修改 HTML Attribute，CSS 能感應到
```

### 場景二：用 jQuery `.data()` 改資料，原生 JS 讀不到新值

這是一個在多人協作的大型專案裡非常容易出現的 Bug。假設有一個隱藏的 `div` 用來存放當前使用者的權限：

```html
<div id="userSettings" data-role="admin"></div>
```

你先用 jQuery 修改了 role：

```javascript
$('#userSettings').data('role', 'guest') // 只改了 jQuery 的秘密筆記本
```

但另一位同事寫了一段原生 JS 來讀取權限：

```javascript
// 同事的程式碼：
let role = document.getElementById('userSettings').dataset.role
// 結果：role 還是 'admin'，不是 'guest'！
```

原因就是 `.data()` 只更新了 jQuery 的內部快取，DOM 實體的 `dataset` 完全沒有被動到。jQuery 的記憶體跟 DOM 實體是兩個完全不同的世界，原生 JS 完全看不到 jQuery 的內部快取。

**正確做法**：如果需要讓其他程式（不論是原生 JS 還是 CSS）也能感知到這個狀態變化，必須用 `.attr()` 去修改 HTML Attribute：

```javascript
$('#userSettings').attr('data-role', 'guest') // ✅ 更新 DOM，所有人都看得到
```

### 場景三：清空 `textarea` 以為成功，其實沒有

```javascript
$('#memo').empty()     // ❌ 只清了初始浮水印，使用者打的字還在
$('#memo').val('')     // ✅ 才能真正清空畫面上的文字
```

### 場景四：判斷 checkbox 是否打勾，拿到奇怪的字串

```javascript
$('#agree').attr('checked') // ❌ 回傳 "checked" 字串或 undefined，不是 true/false
$('#agree').prop('checked') // ✅ 回傳 true 或 false
```

### 場景五：解鎖按鈕後，按鈕還是鎖死的

```javascript
$('#btn').removeAttr('disabled') // ❌ 舊做法，有時瀏覽器狀態判斷錯亂
$('#btn').prop('disabled', false) // ✅ 直接改 DOM 狀態，準確可靠
```

---

## 七、本系列完整學習路徑回顧

讀完三篇文章，你應該已經建立了以下完整的知識體系：

從最根本的概念出發——**HTML Attribute（藍圖）和 DOM Property（現況）是兩件不同的事，網頁載入後就脫鉤**——到理解瀏覽器的 Reflected Properties 機制（哪些 Property 的改變會同步回 Attribute）；從掌握每個表單元件的具體 Attribute 到 Property 對應關係（`defaultValue` vs. `value`，`defaultChecked` vs. `checked`），到理解 `textarea` 的三位一體弄髒機制，再到 `select` / `option` 的店長員工分工；最後理解 jQuery 的三個層次（`.attr()` 操作藍圖、`.prop()` 操作實體、`.data()` 操作內部快取）以及它們各自的使用場景與陷阱。

這些觀念在你維護使用 jQuery 1.x 或 3.x 的企業系統，甚至未來轉向原生 JavaScript 開發時，都是通用的基礎知識。

---

## 推薦延伸閱讀

- [MDN — Element.getAttribute()](https://developer.mozilla.org/en-US/docs/Web/API/Element/getAttribute)：了解 Attribute 讀取的底層規範
- [MDN — HTMLInputElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLInputElement)：看看瑞士刀身上掛了多少工具
- [MDN — HTMLElement.dataset](https://developer.mozilla.org/en-US/docs/Web/API/HTMLElement/dataset)：深入了解 `data-*` 的存取機制
- [MDN — HTMLSelectElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLSelectElement)：select 的所有 Property 清單
- [MDN — HTMLOptionElement](https://developer.mozilla.org/en-US/docs/Web/API/HTMLOptionElement)：option 的所有 Property 清單
- [jQuery 官方文件 — .prop()](https://api.jquery.com/prop/)：特別注意 Boolean attributes 的說明段落
- [jQuery 官方文件 — jQuery.data()](https://api.jquery.com/jQuery.data/)：了解內部快取機制的底層說明