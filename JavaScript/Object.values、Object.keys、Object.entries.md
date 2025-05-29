---
title: Object.values、Object.keys、Object.entries
tags: [JavaScript]

---

```javascript
const person = {
    name: "小明",
    age: 25,
    city: "台北"
};
```

### Object.keys()
這個方法會返回物件所有的鍵（key）作為陣列：

```javascript
const keys = Object.keys(person);
console.log(keys); // ['name', 'age', 'city']
```

這個方法在我們需要遍歷物件的所有屬性名稱時特別有用。例如，我們可以這樣使用：

```javascript
Object.keys(person).forEach(key => {
    console.log(`屬性名稱: ${key}`);
});
```

### Object.values()
這個方法會返回物件所有的值（value）作為陣列：

```javascript
const values = Object.values(person);
console.log(values); // ['小明', 25, '台北']
```

當我們只關心物件中存儲的值而不需要知道鍵名時，這個方法特別實用。比如計算數值的總和：

```javascript
const scores = {
    math: 90,
    english: 85,
    science: 95
};

const total = Object.values(scores).reduce((sum, score) => sum + score, 0);
console.log(`總分: ${total}`); // 總分: 270

const allScores = Object.values(scores);
const average = allScores.reduce((sum, score) => sum + score, 0) / allScores.length;
console.log(`平均分數: ${average}`);
```

```javascript
// 包含不同資料類型的物件
const mix = {
    a: 123,
    b: '你好',
    c: true,
    d: { x: 1, y: 2 },
    e: [1, 2, 3]
};

console.log(Object.values(mix));
// [123, '你好', true, { x: 1, y: 2 }, [1, 2, 3]]
```

```javascript
// 檢查所有商品是否都有庫存
const inventory = {
    apple: 5,
    banana: 0,
    orange: 3
};

const hasStock = Object.values(inventory).every(stock => stock > 0);
console.log(`所有商品都有庫存：${hasStock}`); // false
```

### Object.entries()
這個方法會將物件轉換為 [key, value] 形式的陣列：

```javascript
const entries = Object.entries(person);
console.log(entries);
// 將物件轉換成二維陣列
// [
//   ['name', '小明'],
//   ['age', 25],
//   ['city', '台北']
// ]
```

Object.entries() 特別適合當我們需要同時處理鍵和值的情況。例如，我們可以很容易地將物件轉換為 Map：

```javascript
const personMap = new Map(Object.entries(person));
console.log(personMap.get('name')); // '小明'
```

### 這三個方法的實際應用場景：

```javascript
// 假設我們有一個購物車物件
const cart = {
    apple: 5,
    banana: 3,
    orange: 2
};

// 1. 使用 Object.keys() 檢查購物車是否為空
console.log(Object.keys(cart).length === 0); // false

// 2. 使用 Object.values() 計算購物車中的總商品數量
const totalItems = Object.values(cart).reduce((sum, count) => sum + count, 0);
console.log(`購物車總數量: ${totalItems}`); // 購物車總數量: 10

// 3. 使用 Object.entries() 格式化購物車內容
Object.entries(cart).forEach(([item, count]) => {
    console.log(`${item}: ${count}個`);
    // apple: 5個
    // banana: 3個
    // orange: 2個
});
```