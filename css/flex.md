# 弹性容器

## 声明弹性容器

~~~css
div {
  display: flex
}
~~~
## 弹性容器的`css`属性

* `flex-flow` `flex-direction`和`flex-wrap`的缩写
  * `flex-direction`会根据书写方向自动转换，`flex-wrap`的`wrap`和`wrap-reverse`分别是换行沿着纵轴的`起边`和`终边`。
* `justify-content` 控制主轴元素的对齐方式有六个值`flex-start 、flex-end、 center、 space-between、 space-around、space-evenly`
* `align-items` 控制纵轴的对齐方式有五个值`flex-start、 flex-end、 center、 baseline、 stretch` 默认值是 `stretch`意思是所有可拉伸的元素将和所在行中最高或最宽的元素一样高一样宽。
* `align-content` 控制纵轴上多行的显示效果， 对单行没有效果有七个值`flex-start 、flex-end、 center、 space-between、 space-around、space-evenly stretch` 除了`stretch`其他值就类比`justify-content`, `stretch`纵轴上的多余空间平分给每一行也就所有行填满纵轴。


# 弹性元

弹性容器的直接后代称为`弹性元素`包含子元素和元素间的非空文本节点

## 弹性元素的特点

* `float`和`clear`对弹性元素无效。
* `vertical-align`对弹性元素本身没有影响只会影响弹性元素内的文本。
* `position: absolute`的弹性元素会从文档流移除， 不在参与弹性布局， 但是弹性容器上属性和一些弹性元素的属性会影响绝对定位的元素的定位远点
   * 比如 `justify-content`和`align-self`设为`center`则元素会居中但是如果你设置了绝对定位的`top、bottom、left、right`等则会相对于真实的定位元素偏移。


## 弹性元素的属性

* `flex`是`flex-grow 、 flex-shrink、 flex-basic的简写`
  * 如果声明了`flex` 却没有明确声明`flex-grow`则 `flex-grow`的默认值是`1`
  * 如果`flex`和`flex-grow`都没有声明则`flex`默认为`0`

  
## `flex`的默认值

* `flex: inital` 相当于`flex: 0 1 auto`
* `flex: auto` 相当于 `flex: 1 1 auto`
* `flex: none` 相当于 `flex: 0 0 auto`
* `flex:: number` 相当于 `flex: number 0 0`

## 注意的点

* `flex-basic: 0` 会覆盖`width`, 
* `flex: 0`元素宽度并不是，而是能够容纳子元素的最小宽度感觉就相当于`width: auto;`

