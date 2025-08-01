---
date: 2025-06-25 17:12
aliases: 
tags:
  - BootStrap_4_3_1
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

## 1. container + row + col-md-*

這是 Bootstrap 最基本且最常用的 [[BootStrap 網格系統]]：

**特點：**

- `container` 提供固定寬度或全寬度的容器
- `row` 創建水平的行，會自動清除浮動
- `col-md-*` 定義在中等螢幕尺寸(≥768px)下的欄位寬度

**使用時機：**

- 網頁整體版面配置
- 需要響應式設計的內容區塊
- 複雜的多欄位佈局

**範例：**

```html
<div class="container">
    <div class="row">
        <div class="col-md-8">主要內容</div>
        <div class="col-md-4">側邊欄</div>
    </div>
</div>
```

## 2. [[form-row + col|form-row + col-*]]

這是專門為表單設計的佈局系統：

**特點：**

- `form-row` 比一般的 `row` 有更緊密的欄位間距
- 左右間距較小，適合表單元素
- 通常搭配 `col-*` 而非 `col-md-*`

**使用時機：**

- 表單內的欄位排列
- 需要將多個表單元素放在同一行
- 想要更緊密的元素間距

**範例：**

```html
<form>
    <div class="form-row">
        <div class="form-group col-md-6">
            <label>姓名</label>
            <input type="text" class="form-control">
        </div>
        <div class="form-group col-md-6">
            <label>電話</label>
            <input type="text" class="form-control">
        </div>
    </div>
</form>
```

## 3. form-inline

這是讓表單元素水平排列的佈局：

**特點：**

- 表單元素會在同一行內水平排列
- 自動調整元素寬度
- 在小螢幕上會自動換行

**使用時機：**

- 簡單的搜尋表單
- 登入表單
- 只有少數幾個欄位的表單

**範例：**

```html
<form class="form-inline">
    <div class="form-group mb-2">
        <label for="staticEmail2" class="sr-only">Email</label>
        <input type="text" readonly class="form-control-plaintext" value="email@example.com">
    </div>
    <div class="form-group mx-sm-3 mb-2">
        <label for="inputPassword2" class="sr-only">Password</label>
        <input type="password" class="form-control" placeholder="Password">
    </div>
    <button type="submit" class="btn btn-primary mb-2">確認</button>
</form>
```

## 總結選擇建議：

**使用 container + row + col-md-*** 當：

- 建立網頁主要版面結構
- 需要響應式的多欄位佈局
- 內容不是表單元素

**使用 form-row + col-*** 當：

- 表單內需要並排的欄位
- 希望欄位間距更緊密
- 複雜的表單佈局

**使用 form-inline 當：**

- 簡單的單行表單
- 搜尋框或登入框
- 表單元素很少且希望水平排列

#### 📑 [[form-inline、row、form-row 對孫元素的影響差異]]
#### 📑 [[col-6 與 col-md-6 的差異]]