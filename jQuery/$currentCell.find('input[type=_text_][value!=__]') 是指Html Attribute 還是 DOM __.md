---
title: '$currentCell.find(''input[type="text"][value!=""]'') 是指Html Attribute 還是 DOM ??'
tags: [jQuery]

---

## $currentCell.find('input[type="text"][value!=""]') 的 value 是指 Html Attribute，還是手動輸入的 Value？？

您提出了一個很好的問題！這裡確實涉及到 jQuery 和 HTML 表單元素值的一個重要區別。

`value != ""` 這個選擇器是根據 HTML 元素的 `value` 屬性來篩選的，而不是根據當前元素的實際值。這裡有一個關鍵區別：

1. **HTML 屬性 value**：這是在 HTML 中直接設定的屬性值，例如 `<input type="text" value="初始值">`。選擇器 `[value!=""]` 只會檢查這個 HTML 屬性。

2. **元素當前值**：當使用者在瀏覽器中輸入新的值後，元素的當前值會改變，但 HTML 的 `value` 屬性不會自動更新。

因此：
- `$('input[value!=""]')` 只會選取那些在 HTML 中設定了非空 `value` 屬性的輸入框
- `.val()` 方法則會獲取元素的當前值，包括使用者輸入的內容

所以在您的代碼中：
```javascript
let val = $currentCell.find('input[type="text"][value!=""]').val();
```

這行只會找到那些在 HTML 中已預設有非空值的輸入框，然後獲取它們的當前值。如果使用者清空了一個原本有預設值的輸入框，`val()` 可能會返回空字串，這時 `if (val == "")` 就可能成立。

如果您想檢查使用者當前輸入的值，更合適的寫法是：
```javascript
let $input = $currentCell.find('input[type="text"]');
let val = $input.val();

if (!val) {  // 檢查當前值是否為空
    // 處理邏輯
}
```

## 網頁載入完成時 <input type="text" class="form-control">，然後手動輸入值，但 $currentCell.find('input[type="text"][value!=""]').val() 卻還取得到值？？

您發現了一個有趣的現象！這確實需要特別解釋一下。

當您描述的情況是：
1. HTML 中的元素是 `<input type="text" class="form-control">` （沒有預設的 value 屬性）
2. 使用者手動輸入了值
3. 然後執行 `console.log($currentCell.find('input[type="text"][value!=""]').val())`
4. 您發現它仍然能獲取到使用者輸入的值

這樣的行為可能讓人感到困惑，但有一個重要的原因：

在某些版本的 jQuery 中，`[value!=""]` 選擇器在運行時會檢查元素的當前值（也就是 `.val()`），而不僅僅是 HTML 屬性中的原始值。這是 jQuery 的一個特性（有些人可能會認為是不一致性），特別是對於表單元素。

在 jQuery 的選擇器引擎中，某些屬性（特別是表單元素的 value）會被特殊處理，這使得選擇器在某些情況下會考慮使用者修改後的值，而不僅僅是 HTML 中的原始屬性。

如果您想要更可靠、更一致的行為，建議避免在選擇器中使用 `[value!=""]` 來檢測使用者輸入的值，而是使用更明確的方法：

```javascript
// 找到所有文字輸入框
let $inputs = $currentCell.find('input[type="text"]');

// 過濾出有值的輸入框
let $nonEmptyInputs = $inputs.filter(function() {
    return $(this).val() !== "";
});

if ($nonEmptyInputs.length > 0) {
    // 有非空值的輸入框
    let val = $nonEmptyInputs.val();
    console.log(val);
} else {
    // 處理沒有值的情況
}
```

這樣的寫法更加明確，並且不依賴於 jQuery 選擇器對 value 屬性的特殊處理。