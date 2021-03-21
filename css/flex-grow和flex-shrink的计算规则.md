## flex-grow

`flex-grow` 定义当主轴有多余空间时，`flex-item`的增长系数， 默认值为`0`即不增长。

`flex-grow`的计算规则也比较简单假如又下面结构

~~~html
<style>
  .test-flex-grow {
    display: flex;
    width: 600px;
    height: 100px;
    align-items: center;
    background-color: rosybrown;
  }
  .test-flex-grow .item-1 {
    flex: 1 2 300px;
    height: 50px;
    background-color: seagreen;
  }
  .test-flex-grow .item-2 {
    flex: 2 2 200px;
    height: 50px;
    background-color: royalblue;
  }
</style>
<div class="test-flex-grow">
  <div class="item-1"></div>
  <div class="item-2"></div>
</div>
~~~

已知主轴有`100px`多余空间，则有如下计算规则

~~~js
item1 = 300 + 100 * 1/(1+2) = 333.33
item2 = 200 + 100 * 2/(1+2) = 266.67
~~~

## flex-shrink

`flex-shrink`的计算规则就没有那么直观了。

修改样式为

~~~html
<style>
  .test-flex-grow {
    display: flex;
    width: 400px;
    height: 100px;
    align-items: center;
    background-color: rosybrown;
  }
  .test-flex-grow .item-1 {
    flex: 1 2 300px;
    height: 50px;
    background-color: seagreen;
  }
  .test-flex-grow .item-2 {
    flex: 2 1 200px;
    height: 50px;
    background-color: royalblue;
  }
</style>
<div class="test-flex-grow">
  <div class="item-1"></div>
  <div class="item-2"></div>
</div>
~~~

已知主轴缺少`100px`空间则有如下计算规则

~~~js
// 计算总权重
// 各自的比例系数乘各自的宽度 
total = 2 * 300 + 1 * 200 = 800
缩小比例 = 100 / total
item1 = 300 - (300 * （2 * 缩小比例)) = 225
item2 = 200 - （200 * (1 * 缩小比例)） = 175
~~~

我也不知道为啥`flex-shrink`不和`flex-grow`一样按照比例计算。
