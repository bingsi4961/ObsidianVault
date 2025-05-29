---
title: SweetAlert2 使用筆記
tags: [JavaScript]

---

# SweetAlert2 使用筆記

## 基本設置

首先，我們需要在專案中引入 SweetAlert2：

```html
<!-- CSS 引入 (放在 <head> 中) -->
<link href="https://cdn.jsdelivr.net/npm/sweetalert2@11/dist/sweetalert2.min.css" rel="stylesheet">

<!-- JavaScript 引入 (放在 </body> 前) -->
<script src="https://cdn.jsdelivr.net/npm/sweetalert2@11"></script>
```

## 從原生 Alert 轉換到 SweetAlert2 

讓我們看看如何將原本的程式碼改寫成使用 SweetAlert2 的版本：

原始版本（使用原生 alert）：
```javascript
function checkKpCategory() {
    if ($('#@nameof(Model.QueryCategory)').val() == 0) { 
        alert('請先選擇 @QueryCategoryTip');
        return false;        
    }    
}
```

改寫後的 SweetAlert2 版本：
```javascript
function checkKpCategory() {
    const $category = $('#@nameof(Model.QueryCategory)');
    const value = $category.val();
    
    if (!value || value === '0') {
        Swal.fire({
            title: '提示',              // 標題
            text: '請先選擇 @QueryCategoryTip',  // 內容
            icon: 'warning',            // 圖示類型
            confirmButtonText: '確定'    // 按鈕文字
        });
        $category.focus();
        return false;
    }
    return true;
}
```

## SweetAlert2 常用設定選項

在表單驗證時，我們常用的設定選項包括：

```javascript
Swal.fire({
    // 基本外觀
    title: '標題文字',
    text: '提示內容',
    html: '<p>可以使用 HTML 格式的內容</p>',
    icon: 'warning',  // 可用: 'success', 'error', 'warning', 'info', 'question'
    
    // 按鈕相關
    confirmButtonText: '確定',
    cancelButtonText: '取消',
    showCancelButton: true,  // 是否顯示取消按鈕
    
    // 樣式相關
    customClass: {
        container: 'my-swal-container',
        popup: 'my-swal-popup',
        confirmButton: 'my-swal-confirm-button'
    },
    
    // 其他實用選項
    timer: 3000,           // 自動關閉時間（毫秒）
    timerProgressBar: true, // 顯示倒數進度條
    allowOutsideClick: false // 防止點擊外部關閉
});
```

## 進階使用情境

### 1. 有確認和取消按鈕的對話框：

```javascript
function confirmDelete() {
    return Swal.fire({
        title: '確認刪除',
        text: '確定要刪除此項目嗎？',
        icon: 'warning',
        showCancelButton: true,
        confirmButtonText: '確定刪除',
        cancelButtonText: '取消',
        confirmButtonColor: '#d33',
        cancelButtonColor: '#3085d6'
    }).then((result) => {
        return result.isConfirmed;
    });
}
```

### 2. 連鎖提示（一個接一個）：

```javascript
function showSuccessMessage() {
    Swal.fire({
        title: '儲存成功',
        icon: 'success',
        timer: 1500
    }).then(() => {
        Swal.fire({
            title: '是否繼續編輯？',
            icon: 'question',
            showCancelButton: true,
            confirmButtonText: '繼續編輯',
            cancelButtonText: '返回列表'
        });
    });
}
```

## 實用的程式碼片段

以下是一些常用的程式碼片段：

### 簡單的成功提示：
```javascript
function showSuccess(message) {
    Swal.fire({
        title: '成功',
        text: message,
        icon: 'success',
        timer: 1500,
        showConfirmButton: false
    });
}
```

### 錯誤提示：
```javascript
function showError(message) {
    Swal.fire({
        title: '錯誤',
        text: message,
        icon: 'error',
        confirmButtonText: '確定'
    });
}
```

這些程式碼片段和說明應該能幫助你更好地使用 SweetAlert2，特別是在處理表單驗證和使用者互動的情境中。你可以根據專案需求調整這些設定，創造更好的使用者體驗。