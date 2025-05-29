---
title: xxx.js 檔案，語法結構參考
tags: [JavaScript]

---

```javascript=
(function ($) {
    'use strict';

    const createKeyPartColDef = function () {
        return {
            /**
            * 集中管理所有DOM相關的選擇器和設定值
            */
            config: {
                selectors: {
                    templateRow: '#template-row',
                    formRow: '.form-row',
                    nameInput: 'input[id^="KpColumns_"][id$="_Name"]',
                    costInput: 'input[id="KpColumns_Cost__Name"]',
                    form: 'form',
                    textInputs: 'input[type="text"]'
                },
                attributes: ['name', 'data-valmsg-for', 'id'],
                validationMessages: {
                    duplicateColumn: '<ul style="color:red"><li>「主欄位名稱、成本欄位名稱、欄位名稱」不允許重複。</li></ul>'
                }
            },
            /**
            * 重置新增元素的狀態
            */
            _resetElementValue: function ($element) {
                const elementType = $element.prop('tagName').toLowerCase();
                const inputType = $element.attr('type');

                if (elementType === 'input') {
                    switch (inputType) {
                        case 'text':
                        case 'number':
                            $element.val('').attr('value', '');
                            break;
                        case 'checkbox':
                            $element.prop('checked', false)
                                .removeAttr('checked')
                                .removeAttr('value');
                            break;
                    }
                } else if (elementType === 'span') {
                    $element.empty();
                }
            },
            /**
            * 變更新增元素的屬性值
            */
            _updateElementAttributes: function ($element, nextIndex) {
                this.config.attributes.forEach(attrName => {
                    const attrValue = $element.attr(attrName);
                    if (attrValue) {
                        const newValue = attrName === 'id'
                            ? attrValue.replace('_1_', `_${nextIndex}_`)
                            : attrValue.replace('[1]', `[${nextIndex}]`);
                        $element.attr(attrName, newValue);
                    }
                });
            },
            /**
            * 重置表單驗證
            */
            _resetFormValidation: function ($form) {
                $form.removeData('validator').removeData('unobtrusiveValidation');
                $.validator.unobtrusive.parse($form);
                $form.validate().resetForm();
            },
            /**
            * 新增欄位定義列
            */
            Add: function (button) {
                const $formRows = $(this._selectors.templateRow + ' ' + this._selectors.formRow);
                const nextIndex = $formRows.length + 1;

                // 複製第一個範本列
                const $template = $formRows.first().clone();
                $template.find('button:last').prop('disabled', false);

                $template.find('input, span').each((_, element) => {
                    const $element = $(element);
                    this._resetElementValue($element);
                    this._updateElementAttributes($element, nextIndex);
                });

                $(button).closest(this._selectors.formRow).after($template);
                this._resetFormValidation($(button).closest('form'));
            },
            /**
            * 移除欄位定義列
            */
            Remove: function (button) {
                const $formRow = $(button).closest(this.config.selectors.formRow);
                $formRow.find('input[type="checkbox"]')
                    .prop('checked', true)
                    .attr('checked', 'checked')
                    .attr('value', 'true');
                $formRow.hide();
            },
            /**
            * 驗證欄位值的唯一性
            */
            ValidateUniqColName: function () {
                const names = [];
                let hasError = false;

                // 檢查所有可見的欄位
                $(this._selectors.nameInput + ', ' + this._selectors.costInput)
                    .each((_, element) => {
                        const value = $(element).val().toLowerCase().trim();

                        if (!value || $(element).closest(this._selectors.formRow).css('display') === 'none') {
                            return true;
                        }

                        if (names.includes(value)) {
                            hasError = true;
                        } else {
                            names.push(value);
                        }
                    });

                return !hasError;
            },
            /**
            * 將 selectors 提取為物件屬性
            */
            _selectors: null,
            /**
            * 初始化模組
            */
            Init: function () {
                this._selectors = this.config.selectors;

                $(document).on('blur', this._selectors.textInputs, function () {
                    this.value = this.value.trim();
                });

                $(this._selectors.form).on('submit', (e) => {
                    if (!this.ValidateUniqColName()) {
                        ShowModal(
                            this.config.validationMessages.duplicateColumn,
                            '驗證錯誤',
                            null
                        );
                        return false;
                    }
                });
            }
        }
    };

    // DOM準備好時初始化模組
    $(() => {
        window.KeyPartColDef = createKeyPartColDef();
        window.KeyPartColDef.Init();
    });

})(jQuery)
```