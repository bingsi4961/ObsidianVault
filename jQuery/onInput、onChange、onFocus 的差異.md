---
date: 2025-09-02 14:31
aliases:
tags:
  - jQuery
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

我來詳細說明這三個 jQuery 事件的不同之處和使用場景。

input 事件：
這是最即時的輸入監聽事件。當使用者在輸入框中進行任何改變時（包括打字、貼上、刪除等），事件就會立即觸發。比如說，當使用者在搜尋框中輸入「h」，立刻就會觸發；繼續輸入「e」，又會立即觸發，非常適合需要即時回應的場景。

讓我用程式碼來示範：

```javascript
$('input').on('input', function() {
    console.log('使用者正在輸入：', $(this).val());
    // 適合用於即時搜尋建議、字數統計等功能
});
```

change 事件：
這個事件比較特別，它只在輸入「完成」時才會觸發。什麼是完成呢？就是當輸入框失去焦點（點擊其他地方）或按下 Enter 鍵時。這意味著使用者可能輸入了很多內容，但在確認完成前，change 事件都不會觸發。

```javascript
$('input').on('change', function() {
    console.log('使用者完成輸入：', $(this).val());
    // 適合用於表單驗證、數據提交前的檢查等
});
```

focus 事件：
這是最簡單的一個事件，它只關注輸入框是否獲得焦點。當使用者點擊輸入框或用 Tab 鍵切換到該輸入框時就會觸發，與輸入內容無關。

```javascript
$('input').on('focus', function() {
    console.log('輸入框獲得焦點');
    // 適合用於顯示提示信息、改變輸入框樣式等
});
```

讓我用一個實際的例子來說明這三個事件的差異：

```javascript
// 想像一個搜尋框的完整實現
$('input.search').on('input', function() {
    // 使用者每輸入一個字符都會觸發
    // 可以用來即時顯示搜尋建議
    showSearchSuggestions($(this).val());
}).on('change', function() {
    // 使用者完成輸入後觸發
    // 可以用來執行實際的搜尋操作
    performSearch($(this).val());
}).on('focus', function() {
    // 使用者點擊搜尋框時觸發
    // 可以用來顯示搜尋歷史記錄
    showSearchHistory();
});
```

這三個事件各自適用於不同的場景：
1. input 適合需要即時反應的功能，如即時搜尋建議、即時字數統計等
2. change 適合需要在輸入完成後才執行的操作，如表單驗證、數據提交等
3. focus 適合需要在使用者開始輸入前就準備好的功能，如顯示提示信息、切換樣式等

了解這些差異後，你就能根據具體需求選擇最適合的事件來使用了。