---
date: 2025-11-06 11:00
aliases:
tags:
  - Bootstrap_4_3_1
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---
# 連結筆記
#### 📑 [[]]

---

- **主要功能**: Bootstrap 提供
- **依賴**: jQuery (Bootstrap 4 需要 jQuery 支援)
- **Bootstrap 4 特別注意**: 還需要 Popper.js 才能讓 modal 正常運作

```html
<!-- Message Modal -->
<div class="modal fade" id="messageModal" tabindex="-1" role="dialog" aria-labelledby="messageModalLabel" aria-hidden="true">
    <div class="modal-dialog modal-dialog-centered" role="document">
        <div class="modal-content">
            <div class="modal-header">
                <h5 class="modal-title" id="messageModalLabel">來源群組</h5>
                <button type="button" class="close" data-dismiss="modal" aria-label="Close">
                    <span aria-hidden="true">&times;</span>
                </button>
            </div>
            <div class="modal-body" id="messageModalBody">
                <!-- 訊息內容 -->
                <partial name='~/Views/Shared/SampleAllocation/_GroupSelect.cshtml' 
                    model='new GroupSelectVM() {
                        YearCtrlName = "modalYear",
                        SeriesCtrlName = "modalSeries",
                        GroupCtrlName = "modalGroupId"
					};' />
            </div>
            <div class="modal-footer">
                <button type="button" class="btn btn-primary" data-dismiss="modal" id="btnModalConfirm">確定</button>
                <button type="button" class="btn btn-secondary" data-dismiss="modal">取消</button>
            </div>
        </div>
    </div>
</div>
```

- `data-dismiss="modal"` - Bootstrap 的自訂屬性,點擊時自動關閉 modal
- `modal fade` - Bootstrap 的 CSS class
- `modal-dialog`、`modal-content` 等 - Bootstrap 的結構 class

```javascript
// 使用 Bootstrap 的 modal 方法 
$('#messageModal').modal('show'); 

// 關閉 modal 
$('#messageModal').modal('hide');
```


```javascript
// messageModal 確定按鈕
$('#btnModalConfirm').on('click', function() {
    
    var value = $('#modalGroupId').val();
    if (value <= 0) {
        ShowModal('請先選擇來源群組');
        return false;
    }
    
    $.get({
        url: '@Url.Action("GetCtrlData")',
        dataType: 'json',
        data: { groupId: value },		// 因為是 $.get()，所以會在網址後加上 ?[groupId]=[value]
        beforeSend: function() {
            BlockUI.show('載入中...');
        },
        complete: function() {
            BlockUI.hide();
        }
    })
    .done(function(result) {
        if (result.status == 200) {
            $('input[type="text"][data-field="baseCount"]:visible').val(result.data.baseCount);
            $('input[type="text"][data-field="cpuCount"]:visible').val(result.data.cpuCount);
            $('input[type="text"][data-field="gpuCount"]:visible').val(result.data.gpuCount);
            $('input[type="text"][data-field="modulus"]:visible').val(result.data.modulus);
            $('input[type="text"][data-field="cqMiniBuilt"]:visible').val(result.data.cqMiniBuilt);
            $('input[type="text"][data-field="extraCount"]:visible').val(result.data.extraCount);
            $('input[type="text"][data-field="odmAsusWater"]:visible').val(result.data.odmAsusWater);
            $('input[type="text"][data-field="odmIrtCount"]:visible').val(result.data.odmIrtCount);
            $('input[type="text"][data-field="certification"]:visible').val(result.data.certification);
            $('input[type="text"][data-field="spmRequest"]:visible').val(result.data.spmrequest);
            $('input[type="text"][data-field="odmExcludeEms"]:visible').val(result.data.odmExcludeEms);            
        } else {
            ShowModal('載入失敗：' + result.message);
        }
    })
    .fail(function(xhr) {
        ShowModal(`作業失敗 (HTTP ${xhr.status})，發生非預期性錯誤，請通知系統人員。`);
    })
    .always(function() {
        console.log('請求結束');
    });
});
```