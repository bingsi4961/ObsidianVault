---
title: JavaScript Regular Expression
tags: [JavaScript]

---

# 1. 字元匹配

## `.`（點）：匹配任何單個字元（除了換行符 `\n`）

這是最基本的通配符，它能匹配除換行符以外的任何單個字符。

```javascript
let pattern = /h.t/;
console.log(pattern.test("hat")); // 輸出: true (h + a + t)
console.log(pattern.test("h t")); // 輸出: true (h + 空格 + t)
console.log(pattern.test("ht")); // 輸出: false (缺少中間字元)
console.log(pattern.test("heat")); // 輸出: false (中間有兩個字元)
```

這個例子中，`h.t` 匹配「h」後面跟著任何一個字符，再跟著「t」。

## `\d`：匹配任何數字字元

匹配 0-9 之間的任何一個數字。

```javascript
let pattern = /\d\d\d/; // 匹配三個連續數字
console.log(pattern.test("123")); // 輸出: true
console.log(pattern.test("abc")); // 輸出: false
console.log(pattern.test("1a3")); // 輸出: false (第二個字符不是數字)
```

## `\D`：匹配任何非數字字元

匹配任何不是 0-9 之間數字的字符。

```javascript
let pattern = /\D\D\D/; // 匹配三個連續非數字
console.log(pattern.test("abc")); // 輸出: true
console.log(pattern.test("123")); // 輸出: false
console.log(pattern.test("a1c")); // 輸出: false (第二個字符是數字)
```

## `\w`：匹配字母、數字或底線

匹配任何字母、數字或底線字符（相當於 `[A-Za-z0-9_]`）。

```javascript
let pattern = /\w\w\w/; // 匹配三個連續字母、數字或底線
console.log(pattern.test("abc")); // 輸出: true
console.log(pattern.test("123")); // 輸出: true
console.log(pattern.test("a_c")); // 輸出: true
console.log(pattern.test("a$c")); // 輸出: false ($ 不是字母、數字或底線)
console.log(pattern.test("a12c111d")); // 輸出: false (第二個字符不是數字)
```

## `\W`：匹配非字母、數字或底線

匹配任何不是字母、數字或底線的字符。

```javascript
let pattern = /\W\W\W/; // 匹配一個非字母、數字或底線
console.log(pattern.test("abc")); // 輸出: false (全是字母)
console.log(pattern.test("a$c")); // 輸出: false ($ 是非字母、數字或底線)
console.log(pattern.test("@$^")); // 輸出: true (三個字母都是非字母、數字或底線)
```

## `\s`：匹配任何空白字元

匹配 ==空格、製表符(\t)、換行符等空白字符==。

```javascript
let pattern = /hello\sworld/; // 匹配 "hello" 後跟一個空白，再跟 "world"
console.log(pattern.test("hello world")); // 輸出: true
console.log(pattern.test("hello	world")); // 輸出: true (用製表符分隔)
console.log(pattern.test("helloworld")); // 輸出: false (沒有空白字元)
```

## `\S`：匹配任何非空白字元

匹配任何不是==空格、製表符(\t)、換行符的字符==。

```javascript
let pattern = /\S\S\S/; // 匹配三個連續非空白字元
console.log(pattern.test("abc")); // 輸出: true
console.log(pattern.test("a c")); // 輸出: false (第二個字元是空格)
```

## `\n`、`\t`、`\0`：特殊字元

這些匹配特定的控制字符：
- `\n`：換行符
- `\t`：製表符 (Tab)
- `\0`：null 字元

```javascript
let str = "Hello
            nWorld";
let pattern = /\n/;
console.log(pattern.test(str)); // 輸出: true (字串包含換行符)

let tabStr = "Hello	World";
let tabPattern = /\t/;
console.log(tabPattern.test(tabStr)); // 輸出: true (字串包含製表符)

let tabStr = "Hello
WorldHello	World";
let tabPattern = /\n\w+\t/;
console.log(tabPattern.test(tabStr)); // 輸出: true (字串包含換行及製表符)

```

# 2. 量詞

量詞讓我們可以指定一個模式需要匹配的次數。

## `*`：匹配前面的字元 0 次或多次

```javascript
let pattern = /ab*c/; // 匹配 'a'，後跟 0 個或多個 'b'，再跟 'c'
console.log(pattern.test("ac")); // 輸出: true (0 個 'b')
console.log(pattern.test("abc")); // 輸出: true (1 個 'b')
console.log(pattern.test("abbc")); // 輸出: true (2 個 'b')
```

在這個例子中，`ab*c` 匹配「a」後跟著零個或多個「b」，再跟著「c」。所以「ac」、「abc」和「abbc」都能匹配。

## `+`：匹配前面的字元 1 次或多次

```javascript
let pattern = /ab+c/; // 匹配 'a'，後跟 1 個或多個 'b'，再跟 'c'
console.log(pattern.test("ac")); // 輸出: false (沒有 'b')
console.log(pattern.test("abc")); // 輸出: true (1 個 'b')
console.log(pattern.test("abbc")); // 輸出: true (2 個 'b')
```

與 `*` 的不同之處在於 `+` 要求至少有一個前面的字符。

## `?`：匹配前面的字元 0 次或 1 次

```javascript
let pattern = /colou?r/; // 匹配 'color' 或 'colour'
console.log(pattern.test("color")); // 輸出: true (0 個 'u')
console.log(pattern.test("colour")); // 輸出: true (1 個 'u')
console.log(pattern.test("colouur")); // 輸出: false (2 個 'u')
```

這在處理英式/美式英語拼寫差異時很有用。

## `{n}`：匹配前面的字元恰好 n 次

```javascript
let pattern = /\d{3}/; // 匹配恰好 3 個數字
console.log(pattern.test("123")); // 輸出: true
console.log(pattern.test("12")); // 輸出: false (只有 2 個數字)
console.log(pattern.test("1234")); // 輸出: true (包含恰好 3 個數字的部分)
```

## `{n,}`：匹配前面的字元至少 n 次

```javascript
let pattern = /\d{2,}/; // 匹配至少 2 個數字
console.log(pattern.test("12")); // 輸出: true (2 個數字)
console.log(pattern.test("123")); // 輸出: true (3 個數字)
console.log(pattern.test("1")); // 輸出: false (只有 1 個數字)
```

## `{n,m}`：匹配前面的字元至少 n 次但不超過 m 次

```javascript
let pattern = /\d{2,4}/; // 匹配 2 到 4 個數字
console.log(pattern.test("12")); // 輸出: true (2 個數字)
console.log(pattern.test("123")); // 輸出: true (3 個數字)
console.log(pattern.test("12345")); // 輸出: true (字串中包含 2 到 4 個數字的部分)
console.log(pattern.test("1")); // 輸出: false (只有 1 個數字)
```

# 3. 位置匹配

位置匹配符讓我們能夠指定模式必須出現在字串的特定位置。

## `^`：匹配輸入字串的開始位置

```javascript
let pattern = /^Hello/; // 匹配以 'Hello' 開頭的字串
console.log(pattern.test("Hello World")); // 輸出: true
console.log(pattern.test("World Hello")); // 輸出: false (不是開頭)
```

## `$`：匹配輸入字串的結束位置

```javascript
let pattern = /World$/; // 匹配以 'World' 結尾的字串
console.log(pattern.test("Hello World")); // 輸出: true
console.log(pattern.test("World Hello")); // 輸出: false (不是結尾)
```

結合 `^` 和 `$` 可以匹配完整的字串：

```javascript
let pattern = /^Hello World$/; // 匹配恰好是 'Hello World' 的完整字串
console.log(pattern.test("Hello World")); // 輸出: true
console.log(pattern.test("Hello World!")); // 輸出: false (有額外字元)
```

## `\b`：匹配一個字詞邊界

字詞邊界是指 ==「字母、數字或底線 (\w)」== 與 ==「其他字符、字串開頭/結尾」== 之間的位置。

```javascript
let pattern = /\bcat\b/; // 匹配 'cat' 作為一個獨立的詞
console.log(pattern.test("The cat is black")); // 輸出: true
console.log(pattern.test("category")); // 輸出: false ('cat' 不是獨立的詞)
console.log(pattern.test("tomcat")); // 輸出: false ('cat' 不是獨立的詞)
console.log(pattern.test("The cat#s name is Kitty#cat")); // 輸出: true (兩個 cat 都符合)
```

## `\B`：匹配非字詞邊界

```javascript
let pattern = /\Bcat\B/; // 匹配 'cat' 但前後都不是字詞邊界
console.log(pattern.test("category")); // 輸出: false ('cat' 前面是字詞邊界)
console.log(pattern.test("tomcats")); // 輸出: true ('cat' 前後都不是字詞邊界)
```

# 4. 字元集

字元集允許我們指定多個可能的字元，只要匹配其中一個即可。

## `[abc]`：匹配方括號中列出的任何一個字元

```javascript
let pattern = /[aeiou]/; // 匹配任何一個英文元音字母
console.log(pattern.test("apple")); // 輸出: true (包含 'a')
console.log(pattern.test("sky")); // 輸出: false (不包含元音)
```

可以使用連字符表示範圍：

```javascript
let pattern = /[a-z]/; // 匹配任何一個小寫字母
console.log(pattern.test("Apple")); // 輸出: true (包含小寫字母 'p')
console.log(pattern.test("123")); // 輸出: false (不包含字母)

let digitPattern = /[0-5]/; // 匹配 0 到 5 之間的任何數字
console.log(digitPattern.test("123")); // 輸出: true (包含 1, 2, 3)
console.log(digitPattern.test("789")); // 輸出: false (沒有 0-5 之間的數字)
```

## `[^abc]`：匹配任何不在方括號中的字元

這是字元集的否定形式。

```javascript
let pattern = /[^aeiou]/; // 匹配任何不是元音的字元
console.log(pattern.test("apple")); // 輸出: true (包含非元音字母 'p', 'l')
                                    // [^aeiou] => ^a ^e ^i ^o ^u 
console.log(pattern.test("aei")); // 輸出: false (全是元音)
```

# 5. 分組和引用

分組讓我們可以將==多個字元作為一個單位處理==，並且可以在後面的模式中==引用這些匹配的內容==。

## `(abc)`：匹配並捕獲括號內的表達式

```javascript
let pattern = /(ab)+/; // 匹配 'ab' 一次或多次
console.log(pattern.test("ab")); // 輸出: true
console.log(pattern.test("abab")); // 輸出: true
console.log(pattern.test("abc")); // 輸出: true (包含 'ab')
```

```javascript
let regex = /(ab)+/;
let txtTest = 'cababcddabab';
const result = txtTest.match(regex);
console.log(result);

0: "abab"    // 完整匹配的結果
1: "ab"      // 捕獲最后一次匹配的 "ab"(index:3)(變量 $1(第1個))
// 非全局匹配返回第一个匹配的详细信息，包括捕獲组
groups: undefined
index: 1
input: "cababcddabab"
length: 2
```

```javascript
let regex = /(ab)+/g;
let txtTest = 'cababcddabab';
const result = txtTest.match(regex);
console.log(result);

0: "abab"  // 完整匹配的結果1
1: "abab"  // 完整匹配的結果2
// 全局匹配返回所有匹配的字符串，但不包含「groups、index、input、捕獲组信息」
length: 2
```

捕獲組的強大之處在於我們可以稍後引用它們：

```javascript
let str = "Hello World Hello World";
let result = str.replace(/(Hello) (World)/, "$2 $1");
console.log(result); // 輸出: "World Hello Hello World"
```

這裡 `$1` 引用第一個捕獲組 `(Hello)`，`$2` 引用第二個捕獲組 `(World)`。

```javascript
let str = "Hello WorldHello World";
// 不是 Hello World Hello World (這只會匹配到 Hello World)
let result = str.replace(/((Hello) (World))+/, "$2 $1");
console.log(result);    // Hello Hello World
```

1. 正則表達式 ``/((Hello) (World))+/`` 的含義：
* `(Hello)` 是第 2 個捕獲組，匹配文本 "Hello"
* `(World)` 是第 3 個捕獲組，匹配文本 "World"
* 這兩個組組合成 `((Hello) (World))` 作為第 1 個捕獲組

2. 替換部分 `"$2 $1"` 的含義：
* `$2` 引用第 2 個捕獲組的內容，即 "Hello"
* `$1` 引用第 1 個捕獲組的內容，即 "Hello World"
* `$1、$2 ...` 僅能使用在 replace()、replaceAll()

## `(?:abc)`：非捕獲組 - 匹配括號內的表達式但不捕獲

```javascript
let pattern = /(?:ab)+c/;
console.log(pattern.test("abc")); // 輸出: true
console.log(pattern.test("ababc")); // 輸出: true
```

這與捕獲組類似，但不能在後面引用匹配的內容。當你只需要分組而不需要引用時，使用非捕獲組可以提高效率。

## `\1`：引用第一個捕獲組的匹配內容

```javascript
let pattern = /(a|b)\1/; // 匹配 'aa' 或 'bb'
console.log(pattern.test("aa")); // 輸出: true
console.log(pattern.test("bb")); // 輸出: true
console.log(pattern.test("ab")); // 輸出: false (第二個字符與第一個不同)
```

```javascrpt
let regex = /(abc|cde)\1/;
let txtTest = 'cabcabccde11cdecde22';
let result = txtTest.match(regex);
console.log(result);

0: "abcabc"
1: "abc"
groups: undefined
index: 1
input: "cabcabccde11cdecde22"
length: 2
```

這裡的 `\1` 表示==引用「第一個捕獲組匹配的內容」==。常用於查找重複的字符或字詞。

```javascript
// 匹配重複的詞
let repeatWordPattern = /\b(\w+)\s+\1\b/;
console.log(repeatWordPattern.test("hello hello")); // 輸出: true
console.log(repeatWordPattern.test("hello world")); // 輸出: false
```

```javascript
let regex = /\b(\w+)\s\1\b/;
let txtTest = 'Hello_ Hello_';
let result = txtTest.match(regex);
console.log(result);

0: "Hello_ Hello_"
1: "Hello_"    // 捕獲組匹配的內容
groups: undefined
index: 0
input: "Hello_ Hello_"
length: 2
```
# 6. 或運算

## `|`：匹配左側或右側的==表達式==

```javascript
let pattern = /cat|dog/; // 匹配 'cat' 或 'dog'
console.log(pattern.test("I have a cat")); // 輸出: true
console.log(pattern.test("I have a dog")); // 輸出: true
console.log(pattern.test("I have a bird")); // 輸出: false
```

```javascript
let regex = /I have a cat|dog/;
let txtTest = 'I have a dog';
let result = txtTest.match(regex);
console.log(result);

0: "dog"
groups: undefined
index: 9
input: "I have a dog"
length: 1
```

```javascript
let regex = /I have a cat|dog/;
let txtTest = 'I have a cat';
let result = txtTest.match(regex);
console.log(result);

0: "I have a cat"
groups: undefined
index: 0
input: "I have a cat"
length: 1
```

結合括號可以創建更複雜的或模式：

```javascript
let pattern = /I have a (cat|dog)/;
console.log(pattern.test("I have a cat")); // 輸出: true
console.log(pattern.test("I have a bird")); // 輸出: false
```

# 7. 前瞻後顧（Lookahead and Lookbehind）

這些是特殊的零寬度斷言，用於匹配某個位置，而不消耗字符。

## `x(?=y)`：正向前瞻，匹配 x，但僅當 x 後面跟著 y 時

```javascript
let str = "Java JavaScript JavaBeans";
let pattern = /Java(?=Script)/;
let matches = str.match(pattern);
console.log(matches); // 輸出: ["Java"] (只匹配 JavaScript 中的 Java)

0: "Java"
groups: undefined
index: 5
input: "Java JavaScript JavaBeans"
length: 1
```

在這個例子中，它只匹配後面緊跟著「Script」的「Java」，所以只匹配到「JavaScript」中的「Java」。

## `x(?!y)`：負向前瞻，匹配 x，但僅當 x 後面不是 y 時

```javascript
let str = "Java JavaScript JavaBeans";
let pattern = /Java(?!Script)/g;
let matches = str.match(pattern);
console.log(matches); // 輸出: ["Java", "Java"] (從 Java 和 JavaBeans 中匹配)

0: "Java"
1: "Java"
length: 2
```

```javascript
let str = "Java JavaScript JavaBeans";
let pattern = /Java(?!Script)/;
let matches = str.match(pattern);
console.log(matches);

0: "Java"
groups: undefined
index: 0
input: "Java JavaScript JavaBeans"
length: 1
```

這裡匹配所有後面不是接「Script」的「Java」。

## `(?<=y)x`：正向後顧，匹配 x，但僅當 x 前面是 y 時

```javascript
let str = "Script JavaScript TypeScript";
let pattern = /(?<=Java)Script/;
let matches = str.match(pattern);
console.log(matches); // 輸出: ["Script"] (只匹配 JavaScript 中的 Script)
```

這個例子匹配前面是「Java」的「Script」，所以只匹配到「JavaScript」中的「Script」。

## `(?<!y)x`：負向後顧，匹配 x，但僅當 x 前面不是 y 時

```javascript
let str = "Script JavaScript TypeScript";
let pattern = /(?<!Java)Script/g;
let matches = str.match(pattern);
console.log(matches); // 輸出: ["Script", "Script"] (從 Script 和 TypeScript 中匹配)
```

這裡匹配所有前面不是「Java」的「Script」。

# 8. 正則表達式方法

JavaScript 提供了幾種使用正則表達式的方法：

## `RegExp.prototype.test()`：檢查是否有匹配

```javascript
let pattern = /hello/;
console.log(pattern.test("hello world")); // 輸出: true
console.log(pattern.test("hi there")); // 輸出: false
```

這是最簡單的方法，僅返回布爾值表示是否找到匹配。

## `RegExp.prototype.exec()`：返回匹配的詳細信息

**無使用全局匹配**
```javascript
let regex = /(\d{3})-(\d{4})/;
let txtTest = 'call 555-1234 for help, otherwise call 963-8296';
let result = regex.exec(txtTest);
console.log(result);

// 輸出類似於:
// [ 
//    0: "555-1234",    // 完整匹配
//    1: "555",         // 第一個捕獲組
//    2: "1234",        // 第二個捕獲組
//    groups: undefined,
//    index: 5,        // 匹配在字串中的起始位置
//    input: "call 555-1234 for help, otherwise call 963-8296",    // 原始字串
//    length: 3
// ]

// 進行第二次 exec，結果與第一次一樣
result = regex.exec(txtTest);
console.log(result);

// 輸出類似於:
// [ 
//    0: "555-1234",    // 完整匹配
//    1: "555",         // 第一個捕獲組
//    2: "1234",        // 第二個捕獲組
//    groups: undefined,
//    index: 5,        // 匹配在字串中的起始位置
//    input: "call 555-1234 for help, otherwise call 963-8296",    // 原始字串
//    length: 3
// ]
```
**使用全局匹配**
```javascript
let regex = /(\d{3})-(\d{4})/g;
let txtTest = 'call 555-1234 for help, otherwise call 963-8296';
let result = regex.exec(txtTest);
console.log(result);

// 輸出類似於:
// [ 
//    0: "555-1234",    // 完整匹配
//    1: "555",         // 第一個捕獲組
//    2: "1234",        // 第二個捕獲組
//    groups: undefined,
//    index: 5,        // 匹配在字串中的起始位置
//    input: "call 555-1234 for help, otherwise call 963-8296",    // 原始字串
//    length: 3
// ]

// 進行第二次 exec
result = regex.exec(txtTest);
console.log(result);

// 輸出類似於:
// [ 
//    0: "963-8296",    // 完整匹配
//    1: "963",         // 第一個捕獲組
//    2: "8296",        // 第二個捕獲組
//    groups: undefined,
//    index: 39,        // 匹配在字串中的起始位置
//    input: "call 555-1234 for help, otherwise call 963-8296",    // 原始字串
//    length: 3
// ]
```

## `String.prototype.match()`

**無使用全局匹配**
```javascript
let regex = /(\d{3})-(\d{4})/;
let txtTest = 'call 555-1234 for help, otherwise call 963-8296';
let result = txtTest.match(regex);
console.log(result);

// 輸出類似於:
// [ 
//    0: "555-1234",    // 完整匹配
//    1: "555",         // 第一個捕獲組
//    2: "1234",        // 第二個捕獲組
//    groups: undefined,
//    index: 5,        // 匹配在字串中的起始位置
//    input: "call 555-1234 for help, otherwise call 963-8296",    // 原始字串
//    length: 3
// ]
```
* 如果沒有 `g` 標誌，`match()` 方法的行為與 `exec()` 類似。

**使用全局匹配**
```javascript
let regex = /(\d{3})-(\d{4})/g;
let txtTest = 'call 555-1234 for help, otherwise call 963-8296';
let result = txtTest.match(regex);
console.log(result);

// 輸出類似於:
// [ 
//    0: "555-1234",
//    1: "963-8296",
//    length: 2
// ]
```
* 使用 match() 的全局匹配（帶 g 標誌）時，您只能得到匹配的字符串，而無法獲得捕獲組信息。
* 如果您想一次性獲取所有匹配及其捕獲組，可以使用 ES2020 引入的 ==String.prototype.matchAll()== 方法：


## `String.prototype.replace()`：替換匹配的文本

```javascript
let str = "Hello world!";
let newStr = str.replace(/world/, "JavaScript");
console.log(newStr); // 輸出: "Hello JavaScript!"
```

使用捕獲組結合 `replace()`：

```javascript
let str = "John Smith";
let newStr = str.replace(/(\w+)\s(\w+)/, "$2, $1");
console.log(newStr); // 輸出: "Smith, John"
```

使用全局匹配
```javascript
let str = "Script JavaScript TypeScript";
let regex = /(?<!\b)Script/g;
let newStr = str.replace(regex,'Basic');
console.log(newStr); // 輸出：Script JavaBasic TypeBasic
```

## `String.prototype.search()`：返回匹配的索引

```javascript
let str = "Hello world!";
let position = str.search(/world/);
console.log(position); // 輸出: 6 (匹配開始的位置)
```

如果沒有找到匹配，則返回 -1。

## `String.prototype.split()`：使用正則表達式分割字符串

```javascript
let str = "apple,banana;orange.pear";
let fruits = str.split(/[,.;]/); // 按逗號、點或分號分割
console.log(fruits); // 輸出: ["apple", "banana", "orange", "pear"]
```

# 9. 易混淆指令
* (xx)：分組匹配 xx，並捕獲匹配的字符串
* (?:xx)：分組匹配 xx，但不捕獲
* (a|b)\1：\1：引用第1個捕獲組的匹配內容
* xx(?=yy)：匹配 xx，但僅當 xx 後面跟著 yy 時
* xx(?!yy)：匹配 xx，但僅當 xx 後面不是 yy 時
* (?<=yy)xx：匹配 xx，但僅當 xx 前面是 yy 時
* (?<!yy)xx：匹配 xx，但僅當 xx 前面不是 yy 時


# 10. 實用示例和進階模式

## 驗證電子郵件地址（簡單版本）

```javascript
let emailPattern = /^[a-zA-Z0-9._-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,6}$/;
console.log(emailPattern.test("user@example.com")); // 輸出: true
console.log(emailPattern.test("invalid-email")); // 輸出: false
```

## 驗證強密碼（至少 8 個字符，包含大小寫字母、數字和特殊字符）

```javascript
let strongPasswordPattern = /^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)(?=.*[!@#$%^&*]).{8,}$/;
console.log(strongPasswordPattern.test("Passw0rd!")); // 輸出: true
console.log(strongPasswordPattern.test("password")); // 輸出: false
```

這個模式使用了多個前瞻斷言來確保密碼包含各種必要的字符類型。

## 提取 URL 中的域名

```javascript
let url = "https://www.example.com/path/to/page";
let domainPattern = /^(?:https?:\/\/)?(?:www\.)?([^\/]+)/i;
let match = url.match(domainPattern);
console.log(match[1]); // 輸出: "example.com"
```

## 提取 HTML 標籤

```javascript
let html = "<div class='container'><p>Hello world!</p></div>";
let tagPattern = /<([a-z]+)[\s\S]*?>[\s\S]*?<\/\1>/gi;
let matches = html.match(tagPattern);
console.log(matches); // 輸出: ["<div class='container'><p>Hello world!</p></div>", "<p>Hello world!</p>"]
```

這個例子使用了反向引用 `\1` 來確保開始和結束標籤匹配。