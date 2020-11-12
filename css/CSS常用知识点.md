# `CSS`常用知识点

## 隐藏元素

### `diaplay: none`

* 不占据额外的额空间，产生重排和重绘
* 不会被子元素继承
* 无法进行事件绑定
* transition对display无效

### `visibility: hidden`

* 会占据额外的空间，只会产生重绘
* 会被子元素继承，可使用`visibility: visible`使子元素显示出来
* 绑定事件无效
* transition对其有效

### `opacity: 0`

* 会占据额外的空间，只会产生重绘
* 会被子元素继承，无法使子元素重显
* 绑定事件有效
* transition对其有效