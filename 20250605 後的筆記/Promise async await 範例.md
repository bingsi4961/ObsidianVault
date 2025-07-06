---
date: 2025-07-06 11:12
aliases: 
tags:
  - JavaScript
  - Promise
  - async/await
---
# Metadata
Status :: 🌱
Note Type :: 📰
Source URL :: {文章 URL}
Author :: {作者名稱}
Topics :: {筆記跟什麼主題有關，用 `[Topic],[Topic]` 格式}

---

![[Pasted image 20250706123147.png]]

```html
<table id="queryTable" class="table tablesorter-blue table-hover">    
    <tr>
        <th>Vendor：</th>
        <td>
            <div style="margin-bottom: 8px">
                <a href="#" onclick="checkAllVendors(true)" class="text-primary">Check all</a> /
                <a href="#" onclick="checkAllVendors(false)" class="text-primary">Uncheck all</a>
            </div>
            @foreach (var vendor in Model.VendorList)
            {
                <label for="cbxVendor_@vendor.VendorId" style="cursor: pointer;">
                    <input type="checkbox" id="cbxVendor_@vendor.VendorId" 
	                    value="@vendor.VendorId" onchange="changeGens()">
                    @vendor.VendorName
                </label>
            }
        </td>
    </tr>
    <tr id="genTR">
        <th>Gen：</th>
        <td>
            <div style="margin-bottom: 8px">
                <a href="#" onclick="checkAllGens(true)" class="text-primary">Check all</a> /
                <a href="#" onclick="checkAllGens(false)" class="text-primary">Uncheck all</a>
            </div>
            <div></div>
        </td>
    </tr>
</table>
```

-  任何標記為 `async` 的函式都會自動回傳一個 Promise
-  即使你沒有明確寫 `return`，==`async` 函式也會回傳 `Promise<void>` 型態==

```javascript
$(document).ready(function () {
    initPostBackCheckBoxes(checkedVendorIds, checkedGenIds);
});

async function initPostBackCheckBoxes(checkedVendorIds, checkedGenIds) {

    // 處理 Vendor CheckBox，並產生 Gen CheckBox
    for (const vendorId of checkedVendorIds || []) {
        const $checkBox = $(`#queryTable input[id^="cbxVendor_${vendorId}"]`);
        if ($checkBox.length > 0) {
	        // 會觸發 changeGens()
            $checkBox.trigger('click');
            // 等待本次 changeGens() 完成
			// 控制流先跳出函式(跳回呼叫端函式)，完成後才會進入下一個迴圈
            await waitCurrentChangeGens();
            // 或可以 await curtPerformChangeGensPromise || Promise.resolve();
        }
    }

    // 處理 Gen CheckBox
    $.each(checkedGenIds || [], function (index, value) {
        $(`#queryTable input[id^="cbxGen_${value}"]`).trigger('click');
    });
}

// 當前正在執行 performChangeGens 的 Promise
let curtPerformChangeGensPromise = null;

function waitCurrentChangeGens() {
    // || 符號是 C# 的 ??
    return curtPerformChangeGensPromise || Promise.resolve();
}

async function changeGens() {
    curtPerformChangeGensPromise = performChangeGens();
    await curtPerformChangeGensPromise;
    curtPerformChangeGensPromise = null;
}

// 依據所勾選的 Vendor，重新產生 Gen 選項
// ★★ async 都會回傳 Promise
async function performChangeGens() {

    const $cbxVendors = $('input:checkbox[id^="cbxVendor_"]');
    const selectedVendorIds = $cbxVendors.filter(':checked').map(function () {
        return $(this).val();
    }).toArray();

    try {
        $.blockUI({ message: '<h1>Loading...</h1>' });

		// await 等待 $.ajax() 完成後，再往下面執行
		// 控制流先跳回 changeGens()
        const result = await $.ajax({
            url: '@Url.Action("GetGenByVendor", "Performance")',
            type: 'post',
            dataType: 'json',
            data: { vendorIds: selectedVendorIds },
            traditional: true
        });
        
        const $container = $('#genTR > td > div').eq(1);
        $container.empty();

        if (result.respMsg && !result.respMsg.isSuccess) {
            ShowModal(result.respMsg.strMsg);
            return; // 等於 return Promise.resolve(undefined)
        }

        renderGens($container, result.gens);
        // 自動 return Promise.resolve(undefined)
    }
    catch (error)
    {
        ShowModal(`作業結束 (HTTP ${error.status || 'Unknown'})，發生非預期性錯誤，請通知系統人員`);
        // 已處理錯誤
        // 自動 return Promise.resolve(undefined)
	    
	    // throw error;
	    // 自動 return Promise.reject(error);

		// throw new Error('失敗');
		// 自動 return Promise.reject(new Error('失敗'));
    }
    finally
    {
        $.unblockUI();
        // finally 中的 return 會被忽略（但 throw 不會被忽略）
        // 基本不會改變 Promise 狀態（除非 finally 本身拋出錯誤）

		// throw new Error('finally 中的錯誤');
		// 自動 return Promise.reject(Error('finally 中的錯誤'));
    }
}

// 渲染 Gen 選項
function renderGens($container, gens) {
    $.each(gens, function (index, gen) {
        const $label = $('<label>', {
            for: 'cbxGen_' + gen.GenId,
            style: 'cursor: pointer;'
        });

        const $checkbox = $('<input>', {
            type: 'checkbox',
            id: 'cbxGen_' + gen.GenId,
            value: gen.GenId,
        });

        $label.append($checkbox).append(' ' + gen.GenName);
        $container.append($label);
    });
}
```