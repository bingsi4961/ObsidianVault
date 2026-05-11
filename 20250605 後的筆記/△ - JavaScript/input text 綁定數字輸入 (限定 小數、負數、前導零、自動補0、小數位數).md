---
date: 2026-05-10 20:51
title:
aliases:
tags:
  - JavaScript
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

📄 inputValidator.js

```javascript
/**
 * 確保錯誤訊息容器存在
 * - 若 input 後方尚未有 .invalid-feedback，則動態插入一個
 * - 支援靜態 HTML 欄位與 Ajax 動態產生的欄位
 */
function ensureFeedback($input) {
    if ($input.next('.invalid-feedback').length === 0) {
        $input.after('<div class="invalid-feedback"></div>');
    }
    return $input.next('.invalid-feedback');
}

/**
 * 顯示錯誤訊息
 * - 在 input 加上 Bootstrap 的 is-invalid 樣式
 * - 將錯誤文字寫入 .invalid-feedback 容器
 */
function showError($input, message) {
    $input.addClass('is-invalid');
    ensureFeedback($input).text(message);
}

/**
 * 清除錯誤訊息
 * - 移除 input 的 is-invalid 樣式
 * - 清空 .invalid-feedback 容器的文字
 */
function clearError($input) {
    $input.removeClass('is-invalid');
    ensureFeedback($input).text('');
}

/**
 * 驗證 input 的最終值（設計給 blur 事件使用）
 * - 空值：清除錯誤，不做任何處理（必填交由表單送出時驗證）
 * - 只有負號（-）：清空欄位並顯示錯誤
 * - 結尾是小數點（3.）：移除小數點並顯示錯誤
 * - 其餘合法值：清除錯誤
 */
function validate($input) {
    const v = $input.val();

    if (v === '') {
        clearError($input);
        return;
    }

    if (v === '-') {
        $input.val('');
        showError($input, '請輸入完整的數值');
        return;
    }

    if (v.endsWith('.')) {
        $input.val(v.slice(0, -1));
        showError($input, '小數點後方需輸入數字');
        return;
    }

    clearError($input);
}

/**
 * 數值輸入驗證綁定器
 * - 透過參數控制允許的輸入規則
 * - input 事件：即時攔截並移除非法字元，防止錯誤值進入欄位
 * - blur 事件：離開欄位時處理過渡狀態（如只輸入了 - 或 3.）
 * @param {string}       selector                    - jQuery 選擇器
 * @param {object}       cfg                         - 設定參數
 * @param {boolean}      cfg.allowDecimal            - 允許小數（預設 true）
 * @param {boolean}      cfg.allowNegative           - 允許負數（預設 false）
 * @param {boolean}      cfg.allowLeadingZero        - 允許前導零，如 05（預設 false）
 * @param {boolean}      cfg.autoFillLeadingZero     - .5 自動補為 0.5（預設 true）
 * @param {number|null}  cfg.maxDecimalPlaces        - 最多幾位小數，null 表示不限（預設 null）
 */
export function bindNumericInput(selector, cfg) {
    const defaults = {
        allowDecimal: true,
        allowNegative: false,
        allowLeadingZero: false,
        autoFillLeadingZero: true,
        maxDecimalPlaces: null,
    };

    const c = $.extend({}, defaults, cfg);

    $(document).on('input.numericValidator', selector, function () {
        const $input = $(this);
        const oldLength = this.value.length;
        const cursorPosition = this.selectionStart;

        // 判斷並暫存負號，避免後續 replace 誤刪
        const isNeg = c.allowNegative && this.value.startsWith('-');
        const prefix = isNeg ? '-' : '';
        let v = isNeg ? this.value.slice(1) : this.value;

        // 移除非法字元，僅保留數字；若允許小數則額外保留第一個小數點
        v = c.allowDecimal
            ? v.replace(/[^0-9.]/g, '').replace(/(\..*)\./g, '$1')
            : v.replace(/[^0-9]/g, '');

        // 移除前導零（如 005 → 5），但保留 0.5 這類合法值
        if (!c.allowLeadingZero) {
            v = v.replace(/^0+(?=\d)/, '');
        }

        // 開頭是小數點時自動補零（如 .5 → 0.5）
        if (c.autoFillLeadingZero && v.startsWith('.')) {
            v = '0' + v;
        }

        // 超過最大小數位數時，截斷多餘的小數位
        if (c.maxDecimalPlaces !== null && v.includes('.')) {
            const parts = v.split('.');
            if (parts[1].length > c.maxDecimalPlaces) {
                v = parts[0] + '.' + parts[1].slice(0, c.maxDecimalPlaces);
            }
        }

        const newValue = prefix + v;

        // 僅在值有變動時才重新賦值，並還原游標位置避免游標跳到最末端
        if (this.value !== newValue) {
            this.value = newValue;
            const newPos = Math.max(0, cursorPosition - (oldLength - newValue.length));
            this.setSelectionRange(newPos, newPos);
        }

        // 若目前欄位處於錯誤狀態，輕量檢查是否已修正
        // 不直接呼叫 validate()，避免在輸入過程中誤觸清空欄位的副作用
        if ($input.hasClass('is-invalid')) {
            const currentVal = $input.val();
            if (currentVal !== '' && currentVal !== '-' && !currentVal.endsWith('.')) {
                clearError($input);
            }
        }
    });

    // 離開欄位時，對過渡狀態做最終驗證
    $(document).on('blur.numericValidator', selector, function () {
        validate($(this));
    });
}
```