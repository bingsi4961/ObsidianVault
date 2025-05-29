---
title: 重置(清空) HTML 元素的輸入狀態
tags: [jQuery, HTML]

---

```javascript=
function resetElements($elements) {	
    $elements.each(function() {		
        const $element = $(this);
        const elementType = $element.prop('tagName').toLowerCase();
        const inputType = $element.attr('type');

        if (elementType === 'input') {
            switch (inputType) {
                case 'text':
                case 'hidden':
                case 'number':
                case 'date':
                case 'datetime-local':
                case 'email':
                case 'file':
                    $element.val('').attr('value', '');
                    //$element.removeAttr('value');
                    break;
                case 'radio':
                    $element.prop('checked', false)
                        .removeAttr('checked');
                        // 如果 radio checked = false，PayLoad 就不會有 radio 參數
                        // 所以可以不必清除 value，而且也不能清除
                        //.removeAttr('value');	
                    break;
                case 'checkbox':
                    $element.prop('checked', false)
                        .removeAttr('checked')
                        .removeAttr('value');
                        // 再確認一下removeAttr('value')是否需要？
                        // 因為清除value後，後續如果又勾選，就變成沒有value了？
                    break;
            }
        } else if (elementType === 'textarea') {
            $element.val('').empty();
        } else if (elementType === 'select') {
            $element.prop('selectedIndex', 0)
                .find('option').removeAttr('selected');
        } else if (elementType === 'span' || elementType === 'div' || elementType === 'label') {
            $element.empty();
        }		
    });
}
```