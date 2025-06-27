---
date: 2025-06-27 09:52
aliases: 
tags:
  - CSS
  - Flexbox
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

## 核心概念

### 軸線系統

- **主軸 (Main Axis)**: 由 ==flex-direction== 決定的主要排列方向
- **交叉軸 (Cross Axis)**: 與主軸垂直的方向

### 對齊屬性

- **justify-content**: 控制主軸方向的對齊
- **align-items**: 控制交叉軸方向的對齊

## 重要觀念

⚠️ **常見迷思**

- ❌ `justify-content` 永遠控制水平方向
- ❌ `align-items` 永遠控制垂直方向

✅ **正確理解**

- `justify-content` 控制主軸，方向會隨 `flex-direction` 改變
- `align-items` 控制交叉軸，方向會隨 `flex-direction` 改變

## 實際應用

### 情況 1: flex-direction: row (預設)

```css
.container {
    display: flex;
    flex-direction: row; /* 主軸 = 水平，交叉軸 = 垂直 */
}
```

|屬性|控制方向|說明|
|---|---|---|
|`justify-content`|水平方向|控制子元素在主軸（水平）上的對齊|
|`align-items`|垂直方向|控制子元素在交叉軸（垂直）上的對齊|

### 情況 2: flex-direction: column

```css
.container {
    display: flex;
    flex-direction: column; /* 主軸 = 垂直，交叉軸 = 水平 */
}
```

|屬性|控制方向|說明|
|---|---|---|
|`justify-content`|垂直方向|控制子元素在主軸（垂直）上的對齊|
|`align-items`|水平方向|控制子元素在交叉軸（水平）上的對齊|

## 屬性值說明

### justify-content 可用值

- `flex-start` (預設): 靠主軸起始位置
- `flex-end`: 靠主軸結束位置
- `center`: 主軸置中
- `space-between`: 兩端對齊，中間平均分布
- `space-around`: 每個元素周圍有相等空間
- `space-evenly`: 所有空間平均分布

### align-items 可用值

- `stretch` (預設): 在交叉軸上拉滿容器
- `flex-start`: 靠交叉軸起始位置
- `flex-end`: 靠交叉軸結束位置
- `center`: 交叉軸置中
- `baseline`: 以文字基線對齊

## 實用範例

### 完全置中

```css
.center-container {
    display: flex;
    justify-content: center; /* 主軸置中 */
    align-items: center;     /* 交叉軸置中 */
    height: 200px;
}
```

### 垂直排列且水平置中

```css
.vertical-center {
    display: flex;
    flex-direction: column;
    align-items: center;     /* 交叉軸（水平）置中 */
    justify-content: center; /* 主軸（垂直）置中 */
}
```

### 避免子元素拉滿

```css
.no-stretch {
    display: flex;
    flex-direction: column;
    align-items: flex-start; /* 不使用預設的 stretch */
}
```

## 記憶技巧

1. **先確定 flex-direction**，決定主軸方向
2. **justify-content** = **J**ust 主軸 (Main axis)
3. **align-items** = **A**cross 交叉軸 (Cross axis)
4. 交叉軸永遠垂直於主軸

## 除錯提示

如果發現子元素意外拉滿某個方向：

1. 檢查 `flex-direction` 確定軸線方向
2. 檢查 `align-items` 是否為預設的 `stretch`
3. 根據需求調整對齊屬性