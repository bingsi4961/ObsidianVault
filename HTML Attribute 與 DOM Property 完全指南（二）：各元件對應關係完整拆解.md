---
date: 2026-05-07 14:03
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

## 前言：為什麼要逐元件拆解？

在第一篇，我們建立了「藍圖（Attribute）vs. 現況（Property）」的基本框架。但知道概念是一回事，在實際開發時能快速找到正確的 Property 名稱又是另一回事。

開發表單時，最容易踩坑的場景就是「**重設（Reset）**」和「**髒資料檢查（Dirty Checking，判斷使用者有沒有改過表單）**」。如果你搞不清楚哪個 Property 代表初始狀態、哪個代表當前狀態，你的重設邏輯就會失效，你的髒資料判斷就會錯誤。

這篇文章的目標，就是讓你在碰到每個表單元件時，都能清楚知道對應的 Attribute 和 Property 分別叫什麼名字、該怎麼操作。

---

## 一、元件分類：先建立大架構

在逐一拆解之前，我們先把元件分成兩大類，建立一個清晰的思考框架：

**第一類——有「狀態/值」的表單元件**（`input`、`textarea`、`checkbox`、`radio`、`select`）：它們有明確的「初始預設」和「當下現狀」，是這篇的重點。

**第二類——純展示或觸發的元件**（`div`、`span`、`label`、`button`）：它們主要負責顯示結構或文字，不讓使用者直接輸入資料。對這類元件，我們通常只關注它們的「內容（Content）」，使用 `innerHTML` / `textContent`，不需要特別區分初始與當前狀態。

---

## 二、總覽對應表：表單元件篇

先給你一張全局地圖，後面會逐一深入解釋：

| 元件種類                 | HTML 藍圖（初始設定 Attribute）            | DOM Property（讀取初始狀態） | DOM Property（讀/寫當前現狀） |
| -------------------- | ---------------------------------- | -------------------- | --------------------- |
| `input type="text"`  | `value="預設值"`                      | `.defaultValue`      | `.value`              |
| `textarea`           | 寫在標籤中間 `<textarea>預設文字</textarea>` | `.defaultValue`      | `.value`              |
| `checkbox` / `radio` | `checked="checked"`                | `.defaultChecked`    | `.checked`            |
| `select`             | 無                                  | 無                    | `.value`              |
| `option`             | `selected`（布林）、`value="xxx"`       | `.defaultSelected`   | `.selected`、`.value`  |

> 因 option 不允許使用者修改，故僅有 value Property，沒有 defaultValue Property

---

## 三、`input type="text"`：文字輸入框

### HTML Attribute → DOM Property 對應

當你寫下：

```html
<input type="text" value="華碩" id="myInput">
```

瀏覽器建立 DOM 物件後：

- `value="華碩"` 這個 Attribute → 同時存放在 DOM 的 `.defaultValue`（初始）和 `.value`（當前）
- 一旦使用者開始打字，或你用程式修改了 `.value`，就觸發「弄髒（Dirty）」機制，`.value` 開始獨立運作，跟 `.defaultValue` 脫鉤

### 操作語法對照

```javascript
const el = document.getElementById('myInput');

// --- 讀取初始狀態 ---
el.getAttribute('value')   // 從 HTML 藍圖讀取，回傳 "華碩"
el.defaultValue            // 從 DOM 讀取初始快照，回傳 "華碩"（效能較好）

// --- 讀取當前狀態 ---
el.value                   // 原生 JS：使用者打字後的即時內容
$('#myInput').val()        // jQuery 等效寫法

// --- 修改初始狀態（極少使用）---
el.setAttribute('value', '新預設值')  // 修改藍圖，同時連動更新 defaultValue
$('#myInput').attr('value', '新預設值')

// --- 修改當前狀態（99% 的情況用這個）---
el.value = '新內容'        // 直接更新畫面
$('#myInput').val('新內容')

// --- 均為空值，沒有作用 ---
el.innerHTML
el.textContent
```

> ⚠️ **誤區提醒**：修改初始狀態（`setAttribute`）只用在你想改變「按下 Reset 按鈕後的預設行為」時才用，平常開發直接改 `.value` 就對了。

---

## 四、`textarea`：多行文字輸入框

### `textarea` 的三位一體

`textarea` 是所有表單元件裡最容易讓人搞混的，原因是它的 HTML 寫法跟 `input` 不同：

```html
<!-- input 的預設值寫在 attribute 裡 -->
<input type="text" value="預設值">

<!-- textarea 的預設值夾在標籤中間 -->
<textarea id="feedback">請輸入您的建議</textarea>
```

雖然寫法不同，但瀏覽器建立 DOM 後，`textarea` 的初始文字會同時出現在三個地方，它們是**三位一體**的，修改其中任何一個都會連動另外兩個：

- `.defaultValue`（明確的初始值 Property）
- `.innerHTML`（容器的 HTML 內容）
- `.textContent`（容器的純文字）

但是，畫面上你實際看到的內容，永遠是由 `.value` 掌控的。

### 「弄髒機制（Dirty Flag）」詳解

![[Pasted image 20260508091053.png]]


這裡有個細膩的地方需要特別說明：

**乾淨狀態（Clean）**：網頁剛載入，使用者還沒動過這個 `textarea`。這時你用程式修改 `defaultValue`（或 `innerHTML`、`textContent`），畫面上的內容會跟著連動改變，因為 `.value` 還是跟著初始值走。

**弄髒狀態（Dirty）**：只要使用者在裡面打了任何一個字，或你用 JS 修改過 `element.value`，瀏覽器就會在這個元件上貼一張隱形的「已弄髒」標籤。從這一刻起，`.value`（當前現況）就跟 `defaultValue`（初始藍圖）正式脫鉤——你再去改 `defaultValue`，畫面上的字也不會理你了。

```javascript
const el = document.getElementById('feedback');

// --- 三位一體的初始狀態（互相連動）---
el.defaultValue  // "請輸入您的建議"
el.innerHTML     // "請輸入您的建議"（相同）
el.textContent   // "請輸入您的建議"（相同）

// --- 當前狀態（使用者輸入後才會不同）---
el.value         // 使用者實際打的字

// --- 清空畫面上的內容 ---
el.value = ''           // ✅ 正確：清空使用者輸入
el.innerHTML = ''       // ❌ 陷阱！只清掉了初始浮水印，使用者打的字不受影響
$('#feedback').val('')  // ✅ jQuery 等效正確寫法
$('#feedback').empty()  // ❌ 陷阱！效果等同 innerHTML = ''，無法清空 value
```

> 💡 **學長提示**：把 `textarea` 的 `innerHTML` 想像成「白板出廠時印上的浮水印」，而 `.value` 是使用者用麥克筆寫上去的字。你擦掉浮水印，麥克筆的字還在。要清空使用者的輸入，永遠用 `.value = ''`。

#### 一、`$.empty()` 的底層行為

對 `<textarea>` 呼叫 jQuery 的 `$.empty()`，本質上等同於原生 JS 的 `innerHTML = ''` 或 `textContent = ''`。這個操作只會修改元素的「初始預設值（`defaultValue`）」，並不會動到 `.value`。

#### 二、兩種狀態下的行為差異

##### ✅ 乾淨狀態（Clean State）——「未脫鉤」

當 `<textarea>` 剛載入、從未被使用者觸碰，也沒有被 JS 賦值過，就處於乾淨狀態。此時 `.value` 會動態映射 `defaultValue`，因此在這個狀態下執行 `$.empty()` 可以連動清空畫面文字。

##### ❌ 弄髒狀態（Dirty State）——「已脫鉤」

只要發生以下任一情況，瀏覽器就會將 Dirty Flag 標記為 `true`：

- 使用者在輸入框打過字或刪過字
- 程式碼執行過 `$.val('...')` 等賦值操作

一旦脫鉤，`.value` 就獨立運作，不再受 `defaultValue` 影響。此時執行 `$.empty()` 雖然能清除 DOM 的文字節點，但**畫面上的文字完全不會改變**。

#### 三、最佳實踐（Best Practice）

在實際開發中，我們幾乎不可能確認 `<textarea>` 是否已被悄悄脫鉤（使用者的微小互動、瀏覽器自動回填都可能觸發）。

|方法|作用對象|脫鉤後有效？|
|---|---|---|
|`$.empty()`|`defaultValue`|❌ 否|
|`$.val('')`|`.value`（當前狀態）|✅ 是|

> **結論：永遠使用 `$.val('')`（原生為 `elem.value = ''`）來清空 `<textarea>`，不要使用 `$.empty()`。**

---

## 五、`input type="checkbox"` / `input type="radio"`

### 「狀態」與「資料」要分開看

Checkbox 和 Radio 有個特殊之處：它們身上的 `value`，不是「有沒有打勾」的意思，而是「送出表單時要傳給伺服器的值」。這兩件事要分開理解：

```html
<input type="checkbox" id="agree" value="yes" checked>
```

|要問的事|用哪個 Property|說明|
|---|---|---|
|現在有沒有打勾？|`.checked`|回傳 `true` / `false`|
|最初預設有沒有打勾？|`.defaultChecked`|對應藍圖的 `checked` attribute|
|送出表單時傳什麼值？|`.value`|回傳字串 `"yes"`|
|預設送出的值是什麼？|`.defaultValue`|對應藍圖的 `value` attribute|

### 操作語法對照

```javascript
const el = document.getElementById('agree');

// --- 讀取勾選狀態 ---
el.checked                         // 原生 JS：當前是否打勾
el.defaultChecked                  // 原生 JS：初始預設是否打勾
$('#agree').prop('checked')        // jQuery：當前是否打勾

// --- 修改勾選狀態 ---
el.checked = true                  // 原生 JS：打勾
el.checked = false                 // 原生 JS：取消打勾
$('#agree').prop('checked', true)  // jQuery 等效寫法

// --- 修改初始設定（修改藍圖）---
el.setAttribute('checked', 'checked')   // 連動更新 defaultChecked
$('#agree').attr('checked', 'checked')  // jQuery 等效
```

### ⚠️ 瀏覽器的 Fallback 機制：`value` 的預設值是 `'on'`

這是個很多人在 F12 測試時發現的奇怪現象：

```javascript
el.removeAttribute('value')
console.log(el.value) // 輸出 "on"，不是空字串！
```

原因是 HTML 規範定義了：Checkbox 送出時總得告訴伺服器「這個欄位有被勾選」。如果你沒設定 `value`，瀏覽器就預設塞入字串 `'on'` 作為退路。

這就是為什麼**一定要幫每個 checkbox 寫明確的 `value`**，否則後端收到的資料會是一堆意義不明的 `'on'`，完全無法辨識是哪個商品、哪個選項被勾選了。

---

## 六、`input type="text"` 的「瑞士刀」設計細節

在 F12 檢查 `<input type="text">` 時，你可能會看到它身上**同時有 `defaultValue` 和 `defaultChecked`**，這很容易讓人困惑——文字框跟 `checked` 有什麼關係？

原因是：在瀏覽器底層，不管 `input` 的 `type` 是什麼，它們在 DOM 裡都是**同一種物件** `HTMLInputElement`。這個物件像一把瑞士刀，上面掛著所有 `input` 類型可能用到的工具：

- 文字框用的 `.value` / `.defaultValue`（主廚刀）
- Checkbox 用的 `.checked` / `.defaultChecked`（開瓶器）
- 檔案上傳用的 `.files`（剪刀）

當你寫 `<input type="text">` 時，只是把主廚刀拉出來用，但開瓶器依然物理性地掛在刀上。所以你在 F12 看得到 `defaultChecked`，只是它對文字框來說永遠是 `false`，沒有任何實際作用。

相比之下，`<textarea>` 是完全獨立的 `HTMLTextAreaElement` 物件，身上就不會有 `checked` 或 `defaultChecked` 這類東西。

---

## 七、`select` 與 `option`：店長與員工

### 職責分工

`select` 和 `option` 是合作關係，但各自負責不同的事：

**店長 `<select>`** 只負責統整結果，回答一個問題：「現在整個選單選了哪個值？」

**員工 `<option>`** 各自記錄自己的狀態：「我一開始是不是預設被選中的？（`defaultSelected`）」以及「我現在有沒有被選中？（`selected`）」

### 用一個實例追蹤狀態變化

假設我們有這樣的 HTML：

```html
<select id="meal">
  <option id="optA" value="beef" selected>牛肉麵</option>
  <option id="optB" value="pork">排骨飯</option>
</select>
```

使用者把選單從「牛肉麵」改選成「排骨飯」後，各個 Property 的變化如下：

|觀察對象（DOM Property）|屬於誰|剛載入時|改選排骨飯後|說明|
|---|---|---|---|---|
|`select.value`|店長（select）|`"beef"`|`"pork"`|店長隨時回報當下的最終決定|
|`optA.defaultSelected`|員工（optA）|`true`|`true`|看 HTML 藍圖的初始設定，藍圖沒變，永遠不變|
|`optA.selected`|員工（optA）|`true`|`false`|看當下現狀，客人變心了，牛肉麵失寵|
|`optB.selected`|員工（optB）|`false`|`true`|排骨飯現在被選中了|
|`select.innerHTML`|店長（select）|`<option...` 完整結構|`<option...` 完整結構|切換選項**不改變結構**，`innerHTML` 靜如止水|
|`select.textContent`|店長（select）|`"牛肉麵排骨飯"`|`"牛肉麵排骨飯"`|同上，肚子裡的純文字也沒有增減|

### 操作語法對照

```javascript
const sel = document.getElementById('meal');

// --- 讀取當前選擇 ---
sel.value                          // "beef" 或 "pork"，問店長最快
sel.selectedIndex                  // 0 或 1，以 index 表示
$('#meal').val()                   // jQuery 等效

// --- 修改當前選擇 ---
sel.value = 'pork'                 // 直接改 select.value，畫面跟著變
sel.selectedIndex = 1              // 或用 index
$('#meal').val('pork')             // jQuery 等效

// 也可以改單個 option 的 selected 狀態：
sel.options[1].selected = true     // 把排骨飯設為選中
$('#meal').find('option').eq(1).prop('selected', true) // jQuery 等效
$('#meal option:eq(1)').prop('selected', true)

// --- 讀取初始設定 ---
sel.options[0].defaultSelected     // true（牛肉麵是初始預設）

// --- 修改 option 的顯示文字 ---
sel.options[0].textContent = '台北 (已額滿)' // ✅ 正確：純文字，安全無虞
sel.options[0].innerHTML = '<b>台北</b>'     // ❌ HTML 規範不允許 option 內嵌標籤，
                                             //    瀏覽器會忽略 <b>，且有 XSS 風險

// --- 清空所有選項（例如：連動下拉選單要先清空再填入）---
sel.innerHTML = ''                 // ✅ 清空整個 select（正確方式）
$('#meal').empty()                 // ✅ jQuery 等效

sel.options[x].innerHTML = ''     // ⚠️ 清空單一 option 的文字，但應改用 .textContent
$('#meal option:eq(x)').empty()   // ⚠️ jQuery 等效，同上建議
```

### 為何 `select` / `option` 不需要 Dirty Flag？

`<select>` / `<option>` 是**結構選擇型元件**，使用者只能從清單中點選，無法自由輸入或編輯文字內容。

因此，瀏覽器對它們的**文字內容（`innerHTML` / `textContent`）不需要啟動弄髒機制（Dirty Flag）**——不存在「使用者輸入心血需要被保護」的情境，也就不需要區分「初始狀態」與「目前狀態」這兩層設計。

結果就是：直接修改或清空 `innerHTML` / `textContent`，畫面永遠會即時同步反映，沒有脫鉤的問題。

然而正因為 `option` 的文字內容由瀏覽器直接渲染、不經過使用者輸入，HTML 規範也明確**不允許在 `option` 裡包裝其他 HTML 標籤**。修改顯示文字時，一律應使用 `.textContent` 而非 `.innerHTML`，後者在 `option` 中既無效又有 XSS 風險。

### ⚠️ `option` 的 Fallback 機制與反射特性

**沒有 `value` 時，`option.value` 會抓文字**：如果你省略了 `value` 屬性，`option.value` 不會是空字串，而是直接使用 `textContent` 的內容作為退路。這是瀏覽器的便利設計，讓你在代碼和顯示文字相同時可以省略寫 `value`。

**`option` 的 `value` 是雙向反射的**：與 `input text` 不同，`option` 不允許使用者直接在畫面上修改它的值，所以瀏覽器讓它的 HTML Attribute `value` 和 DOM Property `.value` 保持雙向連動：

```javascript
sel.options[0].setAttribute('value', '155')
console.log(sel.options[0].value) // "155"，跟著連動了！
```

這跟 `input text`（使用者可打字輸入，所以會脫鉤）的設計是不同的，正是因為 `option` 沒有被使用者「弄髒」的機會。

**`option` 的 `defaultSelected` 可以被 Attribute 操作連動**：

```javascript
sel.options[0].removeAttribute('selected')
console.log(sel.options[0].defaultSelected) // false，連動了

sel.options[1].setAttribute('selected', 'selected')
console.log(sel.options[1].defaultSelected) // true，連動了
```

這跟 `input text` 的 `defaultChecked` / `defaultValue` 的同步行為是一致的。

---

## 八、`div`、`span`、`label`、`button`：容器型元件

這類元件沒有「值（value）」的概念，我們關心的是它們裡面的「內容（Content）」。操作它們主要靠兩個 Property：

### `innerHTML` vs. `textContent`

把這個 `div` 想像成一個透明展示箱：

```javascript
const box = document.getElementById('myDiv');

// --- textContent：只管純文字，對 HTML 標籤視而不見 ---
box.textContent = '<b>你好</b>'
// 畫面顯示：<b>你好</b>（把 HTML 標籤當成普通文字印出來）
// 對應 jQuery 的 .text()

// --- innerHTML：會解析並渲染 HTML 標籤 ---
box.innerHTML = '<b>你好</b>'
// 畫面顯示：**你好**（粗體）
// 對應 jQuery 的 .html()
```

|屬性|jQuery 對應|適用時機|
|---|---|---|
|`.textContent`|`.text()`|✅ 顯示使用者輸入的資料，最安全|
|`.innerHTML`|`.html()`|✅ 你要動態產生網頁排版（如生成表格、清單）|

> 🔴 **資安警告：XSS 攻擊的入口就在這裡！** 絕對不要用 `innerHTML` 直接顯示使用者輸入的內容。如果有人在留言輸入 `<script>竊取密碼()</script>`，`innerHTML` 會把它當成真的程式碼執行，這就是著名的 **XSS（跨站腳本攻擊）**。用 `textContent` 的話，這段惡意程式碼只會成為畫面上無害的一串純文字。原則就是：**自己產生的安全排版用 `innerHTML`；別人輸入的不明文字一律用 `textContent` 消毒。**

### 清空容器內容

```javascript
el.innerHTML = ''     // 最老牌常見的寫法
el.textContent = ''   // 效能稍好，不啟動 HTML 解析器
$('#myDiv').empty()   // jQuery 等效

// 【補充說明】現代瀏覽器還有一個更新的 API：
el.replaceChildren()  // 不傳參數就清空，效能極佳，未來減少 jQuery 依賴時可以考慮
```

> ⚠️ **誤區提醒**：`.empty()` 或 `innerHTML = ''` 只能用在「有肚子的容器元件」（`div`、`span`、`ul`、`tbody`、`select`）。不要對 `input` 使用——因為 `input` 是自我封閉的單一標籤，沒有肚子，它的資料存在 `.value` 裡，要清空必須用 `el.value = ''`。