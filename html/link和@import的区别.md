# 区别

> 1. 从属关系的区别：link属于XHTML标签，而@import是CSS提供的语法规则，link除了加载CSS，还可以定义RSS，定义rel连接属性等，@import就只能加载CSS。
> 2. 加载顺序的区别：页面加载时，link会同时被加载，而@import引用的CSS会等页面被加载完后再加载。
> 3. 兼容性的区别：@import只有IE5以上才能被识别，而link是XHTML标签，无兼容问题。
> 4. DOM可控性区别：通过js操作DOM,可以插入link标签来改变样式；由于DOM方法是基于文档的，无法使用@import方式插入样式