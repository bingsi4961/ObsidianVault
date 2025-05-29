---
title: 複製 HTML 元素的輸入狀態
tags: [jQuery, HTML]

---

```javascript=
function copyElements($oldElements, $newElements) {
    $oldElements.each(function (index) {

        const $oldElement = $(this);
        const $newElement = $newElements.eq(index);

        const elementType = $oldElement.prop('tagName').toLowerCase();
        const inputType = $oldElement.attr('type');

        // 根據不同元件類型複製值
        if (elementType === 'input') {
            switch (inputType) {
                case 'text':
                case 'hidden':
                case 'number':
                case 'date':
                case 'datetime-local':
                case 'email':
                case 'file':
                    const value = $oldElement.val();
                    $newElement.val(value).attr('value', value);
                    break;
                case 'radio':
                case 'checkbox':
                    const isChecked = $oldElement.prop('checked');
                    $newElement.prop('checked', isChecked);

                    if (isChecked) {
                        $newElement.attr('checked', 'checked');
                    } else {
                        $newElement.removeAttr('checked');
                    }

                    break;
            }

        } else if (elementType === 'textarea') {
            $newElement.val($oldElement.val());
            $newElement.html($oldElement.html());

        } else if (elementType === 'select') {
            const selectedValue = $oldElement.val();
            $newElement.val(selectedValue);

            $newElement.find('option').each(function () {
                const $option = $(this);
                if ($option.val() === selectedValue) {
                    $option.attr('selected', 'selected');
                } else {
                    $option.removeAttr('selected');
                }
            });

        } else if (elementType === 'span' || elementType === 'div' || elementType === 'label') {
            $newElement.val($oldElement.val());
            $newElement.html($oldElement.html());
        }
    });
}
```