---
title: _jQuery練習的Html範本
tags: [jQuery, 技術文件]

---

```htmlembedded=
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>jQuery Practice</title>
    <script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>
    <style>
        .box {
            width: 100px;
            height: 100px;
            background-color: #f0f0f0;
            margin: 10px;
            display: inline-block;
        }
        .highlight {
            background-color: yellow;
        }
        .hidden {
            display: none;
        }
        .checkbox-group {
            margin: 15px 0;
        }
    </style>
</head>
<body>
    <div class="container">
        <!-- 按鈕區 -->
        <div class="buttons">
            <button id="btn1">按鈕 1</button>
            <button id="btn2">按鈕 2</button>
            <button class="btn-class">Class按鈕 1</button>
            <button class="btn-class">Class按鈕 2</button>
        </div>

        <!-- Checkbox 區 -->
        <div class="checkbox-group">
            <div>
                <input type="checkbox" id="checkbox1" name="single">
                <label for="checkbox1">單個 Checkbox</label>
            </div>
            
            <div>
                <input type="checkbox" class="group-checkbox" name="group[]" value="1">
                <label>群組 Checkbox 1</label>
                
                <input type="checkbox" class="group-checkbox" name="group[]" value="2">
                <label>群組 Checkbox 2</label>
                
                <input type="checkbox" class="group-checkbox" name="group[]" value="3">
                <label>群組 Checkbox 3</label>
            </div>

            <div>
                <input type="checkbox" id="selectAll">
                <label for="selectAll">全選</label>
                
                <input type="checkbox" class="sub-checkbox" name="sub[]" value="A">
                <label>子項目 A</label>
                
                <input type="checkbox" class="sub-checkbox" name="sub[]" value="B">
                <label>子項目 B</label>
                
                <input type="checkbox" class="sub-checkbox" name="sub[]" value="C">
                <label>子項目 C</label>
            </div>
        </div>

        <!-- 文字輸入區 -->
        <div class="inputs">
            <input type="text" id="input1" placeholder="輸入框 1">
            <input type="text" class="input-class" placeholder="輸入框 2">
            <select id="select1">
                <option value="1">選項 1</option>
                <option value="2">選項 2</option>
            </select>
        </div>

        <!-- 測試區塊 -->
        <div class="boxes">
            <div class="box" id="box1">Box 1</div>
            <div class="box" id="box2">Box 2</div>
            <div class="box" id="box3">Box 3</div>
        </div>

        <!-- 列表區 -->
        <ul id="list1">
            <li>項目 1</li>
            <li>項目 2</li>
            <li>項目 3</li>
        </ul>
        
        <ol>
            <li class="one">ol 項目 1</li>
            <li class="two">ol 項目 2</li>
            <li class="three">ol 項目 3</li>
        </ol>

        <!-- 表格 -->
        <table border="1">
            <tr>
                <td>Cell 1</td>
                <td>Cell 2</td>
            </tr>
            <tr>
                <td>Cell 3</td>
                <td>Cell 4</td>
            </tr>
        </table>

        <!-- 結果顯示區 -->
        <div id="result"></div>
    </div>

    <script>
        $(document).ready(function() {
            // 在這裡寫你的 jQuery 程式碼
            
            // 全選/取消全選功能範例
            $("#selectAll").change(function() {
                $(".sub-checkbox").prop('checked', $(this).prop('checked'));
            });
        });
    </script>
</body>
</html>
```