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
parseInt(true)		    // NaN
parseInt("0")              // 0
parseInt("123.456")        // 123
parseInt("  123.456  ")    // 123
parseInt("123.456abc")     // 123
parseInt("abc123")         // NaN
```

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
Number(null)          // NaN
Number("")            // 0
Number("  ")          // 0
Number(true)          // NaN
Number("0")           // 0
Number("123.456")     // 123.456
Number("  123.456  ") // 123.456
Number("123.456abc")  // NaN
Number("abc123.456")  // NaN
```

4. isNaN():
```javascript
isNaN(undefined)     // true
isNaN(null)          // true
isNaN("")            // false
isNaN("  ")          // false
isNaN(true)          // true
isNaN("0")           // false
isNaN("123.456")     // false
isNaN("  123.456  ") // false
isNaN("123.456abc")  // true
isNaN("abc123.456")  // true
```

---
```javascript=
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