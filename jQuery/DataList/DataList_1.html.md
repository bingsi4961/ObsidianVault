---
title: DataList_1.html
tags: [jQuery]

---

```html
<link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.2/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdnjs.cloudflare.om/ajax/libs/jquery/3.7.1/jquery.min.js"></script>

<script type="text/javascript">
    $(function() {
        // 點擊 deptInput 並且 deptInput 有值時，dataList 只會顯示符合 deptInput value 的選項
        // 所以可以先清掉 deptInput value，讓 dataList 顯示全部的選項 
        $('#deptInput').on('focus',function(){
            //$('#deptInput').val('').removeAttr('value');		
        });	
    });
</script>

<div class="container mt-3">
    <select id="schoolSelect" class="form-control mb-3">
        <option value="">請選擇學校</option>
    </select>
    <div class="form-group">
        <input list="deptList" 
                id="deptInput"  
                class="form-control" 
                placeholder="請輸入或選擇科系名稱" 
                autocomplete="off" />
        <!-- 點擊 option 後，會將 option value 帶入 deptInput -->
        <datalist id="deptList">
            <option value="資訊工程學系">Department of Computer Science and Engineering</option>
            <option value="企業管理學系">Department of Business Administration</option>
            <option value="心理學系">Department of Psychology</option>
            <option value="生物科技學系">Department of Biotechnology</option>
            <option value="建築學系">Department of Architecture</option>
        </datalist>
        <!-- 
            因為是將 dataList option value 帶入 deptInput (等同在 deptInput 輸入文字)
            所以如果需要紀錄值的話，要另外自已紀錄
        -->
        <input type="hidden" id="deptCode">
    </div>
</div>
```