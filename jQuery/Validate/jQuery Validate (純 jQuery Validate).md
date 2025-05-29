---
title: jQuery Validate (純 jQuery Validate)
tags: [jQuery]

---

```htmlmixed=
<!DOCTYPE html>
<html lang="zh-TW">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>jQuery 表單驗證範例</title>
        <link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.3/css/bootstrap.min.css" rel="stylesheet" />
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.20.0/jquery.validate.min.js"></script>
        <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.20.0/additional-methods.min.js"></script>
    </head>
    <body class="bg-light container mt-5">
        <form id="registrationForm" action="https://www.google.com.tw/" class="card col-md-6 mx-auto p-4">
            <h2 class="text-center mb-4">用戶註冊</h2>
            <div class="mb-3">
                <label for="username" class="form-label">用戶名稱</label>
                <input type="text" class="form-control" id="username" name="username"
                    required
                    minlength="2"
                    maxlength="20"
                    data-msg-required="請輸入用戶名稱"
                    data-msg-minlength="用戶名稱最少需要2個字符"
                    data-msg-maxlength="用戶名稱不能超過20個字符" />
            </div>
            <div class="mb-3">
                <label for="email" class="form-label">電子郵件</label>
                <input type="email" class="form-control" id="email" name="email" 
                    required 
                    data-msg-required="請輸入電子郵件" 
                    data-msg-email="請輸入有效的電子郵件地址" />
            </div>
            <div class="mb-3">
                <label for="phone" class="form-label">電話號碼</label>
                <input type="tel" class="form-control" id="phone" name="phone" 
                    required 
                    pattern="^[0-9]{10}$" 
                    data-msg-required="請輸入電話號碼" 
                    data-msg-pattern="請輸入10位數字的電話號碼" />
            </div>
            <div class="mb-3">
                <label for="age" class="form-label">年齡</label>
                <input type="number" class="form-control" id="age" name="age" 
                    required 
                    min="18" 
                    max="120" 
                    data-msg-required="請輸入年齡" 
                    data-msg-min="年齡必須大於或等於18歲" 
                    data-msg-max="年齡必須小於120歲" />
            </div>
            <div class="mb-3">
                <label for="password" class="form-label">密碼</label>
                <input type="password" class="form-control" id="password" name="password" 
                    required 
                    minlength="8" 
                    data-msg-required="請輸入密碼" 
                    data-msg-minlength="密碼長度至少8個字符" />
            </div>
            <div class="mb-3">
                <label for="confirmPassword" class="form-label">確認密碼</label>
                <input type="password" class="form-control" id="confirmPassword" name="confirmPassword" 
                    required 
                    equalTo="#password" 
                    data-msg-required="請確認密碼" 
                    data-msg-equalTo="兩次輸入的密碼不一致" />
            </div>
            <button type="submit" class="btn btn-primary">註冊</button>
        </form>

        <script>
            
            $(document).ready(function () {
                
                // errorPlacement 處理的是錯誤訊息的位置和樣式。
                // 如果你希望錯誤訊息直接顯示在輸入框後面（這是預設行為）
                // 而且只需要加上 Bootstrap 的 text-danger 樣式，那麼可以直接使用 errorClass 屬性
                // 簡言之就是創建Span元件，加上text-danger class，然後加入在「輸入欄(input)」後面                
                // ★ 所以 <span class="text-danger"></span> 可以不用另外寫
                $("#registrationForm").validate({
                    errorElement: 'span',    // 設定錯誤訊息要使用的HTML元素
                    errorPlacement: function(error, element) {
                        // error 要顯示錯誤訊息的元件(span)
                        // element 是發生錯誤的輸入欄位
                        error.addClass('text-danger');
                        error.insertAfter(element);
                    }
                });
            });
            
            // 簡化寫法
            $(function(){
                $("#registrationForm").validate({
                    errorElement: 'span',
                    errorClass: 'text-danger'
                });                
            });
            
        </script>
    </body>
</html>

```

讓我來解釋這些屬性：

`required`、`minlength`、`maxlength`、`pattern`、`min`、`max`、`type="email"` 這些是 HTML5 的原生屬性。

但 `data-msg-required`、`data-msg-minlength`、`data-msg-pattern` 這些是 jQuery Validate 插件的自定義屬性，不是 HTML5 的標準屬性。它們的作用是：
- `data-msg-*`: 用於自定義 jQuery Validate 的錯誤訊息
- `equalTo`: 這也是 jQuery Validate 的屬性，不是 HTML5 標準屬性

如果只用 HTML5 的原生驗證（不使用 jQuery Validate），錯誤訊息會使用瀏覽器預設的文字且無法自定義。這就是為什麼我們需要 jQuery Validate - 它讓我們能夠自定義錯誤訊息並提供更多驗證功能。

舉個例子：
```html
<!-- HTML5 原生驗證 -->
<input type="text" required minlength="2">

<!-- jQuery Validate 自定義錯誤訊息 -->
<input type="text" required minlength="2"
    data-msg-required="請輸入內容"
    data-msg-minlength="最少需要2個字">
```