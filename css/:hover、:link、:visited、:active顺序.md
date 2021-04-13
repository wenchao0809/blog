> * 在 CSS 定义中，a:hover 必须被置于 a:link 和 a:visited 之后，才是有效的。
> * 在 CSS 定义中，a:active 必须被置于 a:hover 之后，才是有效的。

正确的顺序是

`link-visited-hover-active`

1、:hover 被置于 :link 和 :visited 之前了；或被置于:active之后了

:link：选择未被访问的链接，并设置其样式；即：定义正常（未被访问）链接的样式。

:hover：选择鼠标指针浮动在其上的元素，并设置其样式；即：定义鼠标悬浮在链接上时的样式。

:active：选择活动链接，并设置其样式；即：定义鼠标点击链接时的样式。

:visited：选择已访问的链接，并设置其样式；即：定义已访问过链接的样式。