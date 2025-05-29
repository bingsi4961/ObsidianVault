---
title: DataList_3.html
tags: [jQuery]

---

```html
<link href="https://cdnjs.cloudflare.com/ajax/libs/bootstrap/5.3.2/css/bootstrap.min.css" rel="stylesheet">
<script src="https://cdnjs.cloudflare.com/ajax/libs/jquery/3.7.1/jquery.min.js"></script>

<script type="text/javascript">
    $(function() {

        // 定義學校資料結構
        const schoolData = [
            {
                code: 'SCH001',
                name: '台灣大學'
            },
            {
                code: 'SCH002',
                name: '清華大學'
            }
        ];

        // 定義科系資料結構
        const deptCodeMap = [
            {
                code: 'CS001',
                name: '資訊工程學系',
                engName: 'Department of Computer Science and Engineering',
                schoolCode: 'SCH001'
            },
            {
                code: 'BA001',
                name: '企業管理學系',
                engName: 'Department of Business Administration',
                schoolCode: 'SCH001'
            },
            {
                code: 'PSY001',
                name: '心理學系',
                engName: 'Department of Psychology',
                schoolCode: 'SCH002'
            },
            {
                code: 'BIO001',
                name: '生物科技學系',
                engName: 'Department of Biotechnology',
                schoolCode: 'SCH002'
            },
            {
                code: 'ARCH001',
                name: '建築學系',
                engName: 'Department of Architecture',
                schoolCode: 'SCH002'
            }
        ];

        // 動態生成學校選項	
        function generateSchoolOptions() {

            $('#schoolSelect').empty().append($('<option>', {
                value: '',
                text: '請選擇學校'
            }));

            $.each(schoolData, function(index, school) {
                const option = $('<option>', {
                    value: school.code,
                    text: school.name
                });
                $('#schoolSelect').append(option);
            });		
        }

        // 根據選擇的學校動態生成科系選項
        function generateDeptOptions(schoolCode) {
            $('#deptList').empty(); // 清空現有選項

            // 過濾出該學校的科系
            const filteredDepts = schoolCode ? 
                deptCodeMap.filter(dept => dept.schoolCode === schoolCode) :
                deptCodeMap;

            $.each(filteredDepts, function(index, dept) {
                const option = $('<option>', {
                    value: dept.name,
                    text: dept.engName,
                    'data-code': dept.code
                });
                $('#deptList').append(option);
            });

            // 清空科系輸入框和代碼
            $('#deptInput').val('').removeAttr('value');
            $('#deptCode').val('').removeAttr('value');
        }

        // 監聽學校選擇變化
        $('#schoolSelect').on('change', function() {
            const selectedSchoolCode = $(this).val();
            generateDeptOptions(selectedSchoolCode);
        });

        // 監聽 input 的變化
        $('#deptInput').on('input', function() {
            const selectedDeptName = $(this).val();
            // .find() 只會回傳第一個符合的項目，所以 selectedDept 是物件型態，不是陣列型態
            const selectedDept = deptCodeMap.find(dept => dept.name === selectedDeptName);

            if (selectedDept) {
                $('#deptCode').val(selectedDept.code);
                console.log('Selected Department:', selectedDept.name);
                console.log('Department Code:', selectedDept.code);
            } else {
                $('#deptCode').val('').removeAttr('value');
            }
        });

        // 點擊 deptInput 並且 deptInput 有值時，dataList 只會顯示符合 deptInput value 的選項
        // 所以可以先清掉 deptInput value，讓 dataList 顯示全部的選項 
        $('#deptInput').on('focus',function(){
            //$('#deptInput').val('').removeAttr('value');	// 不會觸發 deptInput 的 onInput 事件
            $('#deptInput').val('').removeAttr('value').trigger('input');
        });

        generateSchoolOptions();	// 動態生成學校選項	
        generateDeptOptions();	// 根據選擇的學校動態生成科系選項
    });
</script>

<div class="container mt-3">
    <select id="schoolSelect" class="form-control mb-3">
        <!--<option value="">請選擇學校</option>-->
    </select>
    <div class="form-group">
        <input list="deptList" 
                id="deptInput"  
                class="form-control" 
                placeholder="請輸入或選擇科系名稱" 
                autocomplete="off" />
        <!-- 點擊 option 後，會將 option value 帶入 deptInput -->
        <datalist id="deptList">
            <!--
                <option value="資訊工程學系">Department of Computer Science and Engineering</option>
                <option value="企業管理學系">Department of Business Administration</option>
                <option value="心理學系">Department of Psychology</option>
                <option value="生物科技學系">Department of Biotechnology</option>
                <option value="建築學系">Department of Architecture</option>
            -->
        </datalist>
        <!-- 
            因為是將 dataList option value 帶入 deptInput (等同在 deptInput 輸入文字)
            所以如果需要紀錄值的話，要另外自已紀錄
        -->
        <input type="hidden" id="deptCode">
    </div>
</div>
```