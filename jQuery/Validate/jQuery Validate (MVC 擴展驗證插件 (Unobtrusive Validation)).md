---
title: jQuery Validate (MVC 擴展驗證插件 (Unobtrusive Validation))
tags: [jQuery]

---

Field-validation-Valid、Field-validation-Error 是 jQuery Validation Plugin 用來標示驗證狀態的 CSS 類別。當欄位驗證通過時，會加上 Field-validation-Valid；驗證失敗時則加上 Field-validation-Error。

1. `data-val = "true"`
● 這是用來啟用 jQuery Validation 的驗證功能
● 必須加在要驗證的表單欄位上
● 沒有這個屬性的欄位不會被驗證

2. `data-val-required = "請輸入使用者名稱"`
● 設定當欄位為空值時要顯示的錯誤訊息
● 訊息內容可以自訂
● 只有在 data-val="true" 時才會生效
● 通常用於必填欄位的驗證

3. `data-valmsg-for = "username"`
● 這個屬性用來指定錯誤訊息要對應到哪個欄位
● 值必須對應到表單欄位的 name 屬性
● 通常用在顯示錯誤訊息的 span 或 div 元素上

4. `data-valmsg-replace = "true"`
● 控制錯誤訊息如何顯示
● 設為 "true" 時，會完全替換該元素的內容
● 設為 "false" 時，會將錯誤訊息附加到現有內容後面

5. `data-val-length-min/max = "數字"`
● 用於設定欄位的最小/最大長度限制
● min 設定最少要輸入幾個字
● max 設定最多可以輸入幾個字
● 可以搭配 data-val-length-msg 來自訂錯誤訊息

6. `data-val-regex-pattern = "正則表達式"`
● 使用正則表達式驗證輸入格式
● 可以驗證特定的字元組合模式
● 搭配 data-val-regex 顯示不符合格式時的錯誤訊息

7. `data-val-email = "true"`
● 驗證是否為有效的 Email 格式
● 會檢查是否包含 @ 符號及網域名稱
● 搭配 data-val-email-msg 顯示格式錯誤訊息

8. `data-val-range-min/max = "數字"`
● 設定數值的範圍限制
● min 設定最小允許值
● max 設定最大允許值
● 搭配 data-val-range-msg 顯示超出範圍的錯誤訊息

9. `data-val-equalto = "其他欄位的名稱"`
● 驗證此欄位的值是否與指定的欄位相同
● 常用於密碼確認的情境
● 搭配 data-val-equalto-msg 顯示不相符的錯誤訊息
```htmlembedded=
<!-- 最常見的使用情境 - 確認密碼必須與密碼相同 -->
<div class="form-group">
    <label>密碼</label>
    <input type="password" 
           name="password"
           id="password"
           data-val="true"
           data-val-required="請輸入密碼" />
           
    <label>確認密碼</label>
    <input type="password"
           name="confirmPassword"
           data-val="true"
           data-val-equalto="#password"
           data-val-equalto-msg="兩次輸入的密碼不相同" />
</div>
```

10. `data-val-remote = "URL"`
● 透過 AJAX 呼叫遠端驗證
● 可以檢查資料是否已存在於資料庫
● 常用於檢查使用者名稱或 Email 是否重複
● 搭配 data-val-remote-msg 顯示驗證失敗訊息
```htmlembedded=
<!-- 遠端驗證 -->
<input type="text"
       name="username"
       data-val="true"
       data-val-remote="/api/checkUsername"
       data-val-remote-msg="此使用者名稱已被使用" />
```

11. `data-val-custom = "自訂驗證函數名稱"`
● 可以定義自己的驗證邏輯
● 透過 JavaScript 函數實現複雜的驗證規則
● 搭配 data-val-custom-msg 顯示驗證失敗訊息

12. `data-val-grouprequired = "true"`
● 群組必填驗證
● 確保群組中至少有一個欄位有值
● 常用於選項按鈕或核取方塊群組
```htmlembedded=
<!-- 手機和市話至少需填寫一個 -->
<div class="contact-group">
    <input type="text" 
           name="mobile"
           data-val="true"
           data-val-grouprequired="true"
           data-val-grouprequired-msg="手機或市話至少填寫一個" />

    <input type="text"
           name="telephone"
           data-val="true"
           data-val-grouprequired="true" />
</div>

<!-- 興趣至少選擇一項 -->
<div class="hobby-group">
    <input type="checkbox" 
           name="hobby" 
           value="reading"
           data-val="true"
           data-val-grouprequired="true"
           data-val-grouprequired-msg="請至少選擇一項興趣" />
    <label>閱讀</label>

    <input type="checkbox" 
           name="hobby" 
           value="sports"
           data-val="true"
           data-val-grouprequired="true" />
    <label>運動</label>

    <input type="checkbox" 
           name="hobby" 
           value="music"
           data-val="true"
           data-val-grouprequired="true" />
    <label>音樂</label>
</div>

<!-- 性別必選一項 -->
<div class="gender-group">
    <input type="radio" 
           name="gender" 
           value="male"
           data-val="true"
           data-val-grouprequired="true"
           data-val-grouprequired-msg="請選擇性別" />
    <label>男</label>

    <input type="radio" 
           name="gender" 
           value="female"
           data-val="true"
           data-val-grouprequired="true" />
    <label>女</label>
</div>

```

# 完整範例
```htmlmixed=
<style>
    .field-validation-error {
        color: red;
    }
    .input-validation-error {
        border-color: red;
    }	
</style>

<link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.3/css/bootstrap.min.css" rel="stylesheet"> 
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.20.0/jquery.validate.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validate/1.20.0/additional-methods.min.js"></script>
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery-validation-unobtrusive/4.0.0/jquery.validate.unobtrusive.min.js"></script>

<body class="bg-light container mt-5">
    <form id="registrationForm" class="card col-md-6 mx-auto p-4" novalidate>
        <h2 class="text-center mb-4">用戶註冊</h2>	
        <div class="mb-3">
            <label for="username" class="form-label">用戶名稱</label>
            <input type="text" class="form-control" id="username" name="username"
                data-val="true"
                data-val-required="請輸入用戶名稱"
                data-val-length="用戶名稱長度需要在2-20個字符之間"
                data-val-length-min="2"
                data-val-length-max="20" />
            <span class="text-danger field-validation-valid" 
                data-valmsg-for="username" 
                data-valmsg-replace="true"></span>
        </div>	
        <div class="mb-3">
            <label for="email" class="form-label">電子郵件</label>
            <input type="email" class="form-control" id="email" name="email"
                data-val="true"
                data-val-required="請輸入電子郵件"
                data-val-email="請輸入有效的電子郵件地址" />
            <span class="text-danger field-validation-valid" 
                data-valmsg-for="email" 
                data-valmsg-replace="true"></span>
        </div>	
        <div class="mb-3">
            <label for="phone" class="form-label">電話號碼</label>
            <input type="tel" class="form-control" id="phone" name="phone"
                data-val="true"
                data-val-required="請輸入電話號碼"
                data-val-regex="請輸入10位數字的電話號碼"
                data-val-regex-pattern="^[0-9]{10}$" />
            <span class="text-danger field-validation-valid" 
                data-valmsg-for="phone" 
                data-valmsg-replace="true"></span>
        </div>
        <div class="mb-3">
            <label for="mobile" class="form-label">手機號碼：</label>
            <input type="tel" class="form-control" id="mobile" name="mobile"
                data-val="true"
                data-val-required="手機號碼為必填欄位"
                data-val-taiwanmobile="請輸入有效的台灣手機號碼" />
            <span data-valmsg-for="mobile" data-valmsg-replace="true"></span>
        </div>	
        <div class="mb-3">
            <label for="age" class="form-label">年齡</label>
            <input type="number" class="form-control" id="age" name="age"
                data-val="true"
                data-val-required="請輸入年齡"
                data-val-range="年齡必須在18-120歲之間"
                data-val-range-min="18"
                data-val-range-max="120" />
            <span class="text-danger field-validation-valid" 
                data-valmsg-for="age" 
                data-valmsg-replace="true"></span>
        </div>	
        <div class="mb-3">
            <label for="password" class="form-label">密碼</label>
            <input type="password" class="form-control" id="password" name="password"
                data-val="true"
                data-val-required="請輸入密碼"
                data-val-length="密碼長度至少8個字符"
                data-val-length-min="8" />
            <span class="text-danger field-validation-valid" 
                data-valmsg-for="password" 
                data-valmsg-replace="true"></span>
        </div>	
        <div class="mb-3">
            <label for="confirmPassword" class="form-label">確認密碼</label>
            <input type="password" class="form-control" id="confirmPassword" name="confirmPassword"
                data-val="true"
                data-val-required="請確認密碼"
                data-val-equalto="兩次輸入的密碼不一致"
                data-val-equalto-other="*.password" />
            <span class="text-danger field-validation-valid" 
                data-valmsg-for="confirmPassword" 
                data-valmsg-replace="true"></span>
        </div>	
        <button type="submit" class="btn btn-primary">註冊</button>
    </form>

    <script>
        $(document).ready(function() {
            var $form = $("#registrationForm");

            // === 定義自訂驗證方法 ===========================================
            $.validator.addMethod("taiwanmobile", function(value, element) {
                return this.optional(element) || /^09\d{8}$/.test(value);
            }, "請輸入有效的台灣手機號碼");

            // taiwanmobile：驗證方法的名稱
            // value：要驗證的值
            // element：要驗證的輸入欄位之DOM
            // this.optional(element)：如果欄位是選填的且為空值，則返回 true

            // 將自訂驗證方法與 unobtrusive validation 整合
            $.validator.unobtrusive.adapters.addBool("taiwanmobile");

            // ===============================================================

            // 重置並重新初始化驗證
            $form.removeData('validator').removeData('unobtrusiveValidation');
            $.validator.unobtrusive.parse($form);
            $form.validate().resetForm();

            // 處理表單提交
            $form.on("submit", function(e) {
                return $(this).valid();
            });
        });
    </script>
</body>
```