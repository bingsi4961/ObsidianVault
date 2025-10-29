---
title: parseInt、parseFloat、Number、isNaN
tags: [JavaScript]

---

1. parseInt():
```javascript
// 從左到右解析，直到遇到非數字字元就停止
// 只返回整數部分，不會四捨五入
parseInt(undefined)        // NaN
parseInt(null)             // NaN
parseInt("")               // NaN
parseInt(" ")              // NaN
parseInt(true)		       // NaN
parseInt("0")              // 0
parseInt("123.456")        // 123
parseInt("  123.456  ")    // 123
parseInt("123.456abc")     // 123
parseInt("abc123")         // NaN
```

`parseInt` 的核心工作方式是「**嘗試從一個字串的開頭解析出一個整數**」。
理解它的關鍵規則就能明白上述所有結果：

#### 規則 1：先將輸入值轉換為「字串」

`parseInt` 會先將**非字串**的第一個參數（例如 `undefined`, `null`, `true`）轉換為字串。

- `parseInt(undefined)` → `parseInt("undefined")` → "u" 不是數字 → `NaN`    
- `parseInt(null)` → `parseInt("null")` → "n" 不是數字 → `NaN`    
- `parseInt(true)` → `parseInt("true")` → "t" 不是數字 → `NaN`    

#### 規則 2：忽略「開頭」的空白

`parseInt` 會自動修剪 (trim) 字串**開頭**的任何空白字元。

- `parseInt(" ")` → (修剪空白) → `parseInt("")` → 空字串無法解析 → `NaN`    
- `parseInt(" 123.456 ")` → (修剪開頭空白) → `parseInt("123.456 ")` → (開始解析)    

#### 規則 3：從頭解析，直到「第一個非數字字元」為止

`parseInt` 會從字串的第一個字元開始讀取，直到它遇到一個**不屬於**10進位整數的字元（包含小數點 `.`）為止，
然後停止解析並回傳已解析到的部分。

- `parseInt("123.456")` → 讀取 "1", "2", "3"。遇到 "."，這不是整數的一部分，停止解析。 → 回傳 `123`。    
- `parseInt("123.456abc")` → 同上，讀取 "1", "2", "3"。遇到 "."，停止解析。（後面的 "abc" 完全被忽略）。 → 回傳 `123`。    
- `parseInt(" 123.456 ")` → 先忽略開頭空白，然後讀取 "1", "2", "3"。遇到 "."，停止解析。（後面的 ".456 " 完全被忽略）。 → 回傳 `123`。    

#### 規則 4：如果「第一個非空白字元」就不是數字，則回傳 `NaN`

如果 `parseInt` 在修剪完開頭空白後，遇到的第一個字元就無法被解析為數字，它會立即停止並回傳 `NaN` (Not a Number)。

- `parseInt("abc123")` → 讀取 "a"。 "a" 不是數字，停止解析。 → 回傳 `NaN`。    
- `parseInt("")` → 沒有任何字元可以解析。 → 回傳 `NaN`。    
- `parseInt("undefined")` → 讀取 "u"。 "u" 不是數字，停止解析。 → 回傳 `NaN`。    


2. parseFloat():
```javascript
// 從左到右解析，直到遇到非數字字元就停止
// 可處理小數
parseFloat(undefined)        // NaN
parseFloat(null              // NaN
parseFloat("")               // NaN
parseFloat(" ")              // NaN
parseFloat(true)             // NaN
parseFloat("0")              // 0
parseFloat("123.456")        // 123.456
parseFloat("  123.456  ")    // 123.456
parseFloat("123.456abc")     // 123.456
parseFloat("abc123.456")     // NaN
```

3. Number():
```javascript
// 最嚴格的轉換
Number(undefined)     // NaN
Number(null)          // 0
Number("")            // 0
Number("  ")          // 0
Number(true)          // 1
Number("0")           // 0
Number("123.456")     // 123.456
Number("  123.456  ") // 123.456
Number("123.456abc")  // NaN
Number("abc123.456")  // NaN
```

4. isNaN():
```javascript
isNaN(undefined)     // true
isNaN(null)          // false
isNaN("")            // false
isNaN("  ")          // false
isNaN(true)          // false
isNaN("0")           // false
isNaN("123.456")     // false
isNaN("  123.456  ") // false
isNaN("123.456abc")  // true
isNaN("abc123.456")  // true
```

---
```javascript
<link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.2/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.2/js/bootstrap.bundle.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.20.0/jquery.validate.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.20.0/additional-methods.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validation-unobtrusive/4.0.0/jquery.validate.unobtrusive.min.js"></script>

<input type="text" id="txtTest" value="  123.456  ">
<script>	
    const txtTestVal = $('#txtTest').val();	
    switch(txtTestVal){
        case undefined:
            // 找不到 txtTest 元件時
            console.log('undefind')
            console.log(parseInt(txtTestVal));        // 結果 NaN
            console.log(parseFloat(txtTestVal));	// 結果 NaN
            console.log(Number(txtTestVal));		// 結果 NaN
            console.log(isNaN(txtTestVal));        // 結果 true
            break;
        case null:
            // value=null
            // value="null"
            console.log('null')
            console.log(parseInt(txtTestVal));        // 結果 NaN			
            console.log(parseFloat(txtTestVal));	// 結果 NaN
            console.log(Number(txtTestVal));		// 結果 NaN
            console.log(isNaN(txtTestVal));         // 結果 true
            break;
        case '':
            // value=""
            // value=
            // value
            // 沒有設定 value
            console.log('空字串')
            console.log(parseInt(txtTestVal));        // 結果 NaN
            console.log(parseFloat(txtTestVal));	// 結果 NaN
            console.log(Number(txtTestVal));		// 結果 0
            console.log(isNaN(txtTestVal));         // 結果 false
            break;
        case ' ':	//空白格
            // value=" "			
            console.log('空白格')
            console.log(parseInt(txtTestVal));        // 結果 NaN
            console.log(parseFloat(txtTestVal));	// 結果 NaN
            console.log(Number(txtTestVal));		// 結果 0
            console.log(isNaN(txtTestVal));        // 結果 false
            break;
        case true:
            // value="true"			
            // value=true
            console.log('true');
            console.log(parseInt(txtTestVal));        // 結果 NaN
            console.log(parseFloat(txtTestVal));	// 結果 NaN
            console.log(Number(txtTestVal));		// 結果 NaN
            console.log(isNaN(txtTestVal));        // 結果 true
            break;
        case false:
            // value="false"			
            // value=false		
            console.log('false');
            console.log(parseInt(txtTestVal));        // 結果 NaN
            console.log(parseFloat(txtTestVal));	// 結果 NaN
            console.log(Number(txtTestVal));		// 結果 NaN
            console.log(isNaN(txtTestVal));        // 結果 true
            break;
        default:
            // value="0"			
            console.log(txtTestVal);
            console.log(parseInt(txtTestVal));        // 結果 0
            console.log(parseFloat(txtTestVal));	// 結果 0
            console.log(Number(txtTestVal));		// 結果 0
            console.log(isNaN(txtTestVal));        // 結果 false

            // value="  123.456  "
            //console.log(txtTestVal);                // 結果   123.456  
            //console.log(parseInt(txtTestVal));	// 結果 123
            //console.log(parseFloat(txtTestVal));	// 結果 123.456
            //console.log(Number(txtTestVal));        // 結果 123.456
            //console.log(isNaN(txtTestVal));        // 結果 false

            // value="123.567AB"
            //console.log(txtTestVal);                // 結果 123.567AB
            //console.log(parseInt(txtTestVal));	// 結果 123
            //console.log(parseFloat(txtTestVal));	// 結果 123.567
            //console.log(Number(txtTestVal));        // 結果 NaN
            //console.log(isNaN(txtTestVal));        // 結果 true

            // value="AB123.567"
            //console.log(txtTestVal);                // 結果 AB123.567
            //console.log(parseInt(txtTestVal));	// 結果 NaN
            //console.log(parseFloat(txtTestVal));	// 結果 NaN
            //console.log(Number(txtTestVal));        // 結果 NaN
            //console.log(isNaN(txtTestVal));        // 結果 true

            break;
    }
	
	
</script>
```