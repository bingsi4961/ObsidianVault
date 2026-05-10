---
title: ES Module 範例
tags: [JavaScript]

---

## SharedUtils.js
```javascript=
// ★ 因設定 const，故是一個單例(Singleton)模式的對象
export const SharedUtils = {
    
    // 集中管理所有DOM相關的選擇器和設定值
    config: {
        selectors: {
            textInputs: 'input[type="text"]',
            numberInputs: 'input[type="number"]',
            textAreas: 'textarea'
        }
    },

    // 清除元件的值
    resetElements: function ($elements) {
        // doSomething
    },

    // 複製元件的值
    copyElements: function ($oldElements, $newElements) {
        // doSomething
    },

    // 初始化模組
    Init: function () {

        // 綁定文字輸入框的Blur事件
        $(document).on('blur', this.config.selectors.textInputs, function () {
            this.value = this.value.trim();
        });

        // 限制數值輸入框僅能輸入至小數第2位
        $(document).on('input', this.config.selectors.numberInputs, function () {
            let value = $(this).val();
            if (value.includes('.') && value.split('.')[1].length > 2) {
                $(this).val(Number(value).toFixed(2));
            }
        });

        // 綁定textarea輸入框的Blur事件
        $(document).on('blur', this.config.selectors.textAreas, function () {
            this.value = this.value.trim();
        });
    }

};
```

## ProjectEdit.js
```javascript=
import { SharedUtils } from '../SharedUtils.js';

export const ProjectEdit = {
	
    // 集中管理所有DOM相關的選擇器和設定值
    config: {        
        selectors: {
            plmsTable: 'table#PlmsTable',
            seriesSelect: 'select#ddlSeries',
            projectIdInput: 'input#txtProjectId'            
        },
        // data-* 名稱設定
        dataAttrNames: {
            optionCost: 'option-cost',
            listId: 'list-id',
            projectId: 'project-id'
        },
        // URL 相關設定
        urls: {
            action: '/SpmCostProject/EditConfirmed',     // 表單提交的目標 URL
            success: '/SpmCostProject/Index',            // 成功後導向的 URL
            failure: ''                                 // 失敗時導向的 URL
        }
    },

    // 取消勾選其他的 CheckBox
    handleSingleCheckBox: function (element) {
        // doSomething
    },

    // KeyPart DropDownList onChange 時，將成本帶入 Cost 欄位
    updateKeyPartCost: function (element) {
        // doSomething
    },    

    triggerHorizontalButtonsClick: function (element) {
        const $row = $(element).closest('tr');
        const $costSumButtons = $row.find(this._selectors.costSumButton);
        $costSumButtons.trigger('click');
    },

    viewModel: {
        Projects: [],
        ProjectLists: [],
        Series: [],
        DeletedProjects: []
    },

    collectData: function () {
        this.viewModel.Projects = [];
        this.viewModel.ProjectLists = [];
        this.viewModel.Series = [];
		// doSomething		
        self.viewModel.Series.push(series);
    },

    Save: function () {

        if (!this.validateSingleCheckBox()) {
            ShowModal('最低規SKU (主要銷售SKU) 不可重複勾選', '提示訊息');
            return false;
        }

        this.collectData();

        // 發送 AJAX 請求
        this.ajx_sendSaveRequest(this.viewModel)
            .done((result) => this.ajx_handleSaveSuccess(result))
            .fail((xhr, status, error) => this.ajx_handleSaveError(xhr))
            .always(() => console.log('請求結束'));
    },

    // 將 selectors 提取為物件屬性
    _userInterfaces: null,
    _selectors: null,
    _dataAttrNames: null,    

    // 初始化模組
    Init: function () {
        this._userInterfaces = this.config.userInterfaces;
        this._selectors = this.config.selectors;
        this._dataAttrNames = this.config.dataAttrNames;        
    }
};
```

## Edit.cshtml
```htmlembedded=
<script type="module">
    @{
        var sharedUtilsPath = Url.Content("~/js/SharedUtils.js");
        var projectEditPath = Url.Content("~/js/SpmCost/ProjectEdit.js");

        sharedUtilsPath += $"?v={File.GetLastWriteTime(Context.Request.PathBase + sharedUtilsPath).Ticks}";
        projectEditPath += $"?v={File.GetLastWriteTime(Context.Request.PathBase + projectEditPath).Ticks}";
    }
    
    // ★ 在匯入時，建立實例
    // ★ 後續再 import 這個模組，都會得到同一個對象實例
    import { SharedUtils } from '@sharedUtilsPath';
    import { ProjectEdit } from '@projectEditPath';

    $(() => {
        SharedUtils.Init();
        // 模組作用域變數，不是全域變數
        // const(var/let) ProjectEdit = ProjectEdit;
        window.ProjectEdit = ProjectEdit;
        ProjectEdit.Init();
        ProjectEdit.config.urls.action = '@Url.Action("EditConfirmed")';
        ProjectEdit.config.urls.success = '@Url.Action("Index")';
        $('script[type="module"]').after(`<style type="text/css">${ProjectEdit.commonCSS}</style>`);
    });

</script>
```

## Main.js
```javascript=
// ★ 這些只是 export 聲明,並不會造成實例化。
// ★ 告訴系統 "這些模組可以被使用"，但實際上要等到 import 才會實例化。
export { SharedUtils } from '../SharedUtils.js';
export { KpColsCreate } from './KpColsCreate.js';
export { KpDataEdit } from './KpDataEdit.js';
export { KpDataIndex } from './KpDataIndex.js';
export { ProjectEdit } from './ProjectEdit.js';
export { ProjectIndex } from './ProjectIndex.js';
```

## Edit.cshtml
```html
// 預設會有 defer 屬性延遲執行
<script type="module">
    @{
        var mainPath = Url.Content("~/js/SpmCost/Main.js");
        mainPath += $"?v={File.GetLastWriteTime(Context.Request.PathBase + mainPath).Ticks}";
    }

    import {SharedUtils, ProjectEdit} from '@mainPath';

    $(() => {
        SharedUtils.Init();
        window.ProjectEdit = ProjectEdit;
        ProjectEdit.Init();
        ProjectEdit.config.urls.action = '@Url.Action("EditConfirmed")';
        ProjectEdit.config.urls.success = '@Url.Action("Index")';
        $('script[type="module"]').after(`<style type="text/css">${ProjectEdit.commonCSS}</style>`);
    });

</script>
```