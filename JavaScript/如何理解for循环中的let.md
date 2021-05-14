### 面试题

~~~js
var funs = []
for (var i = 0; i < 5; i++) {
  funs.push(() => {
    console.log(i)
  })
}
funs.forEach(item => item()) // 55555
~~~

改为`let`

~~~js
var funs = []
for (let i = 0; i < 5; i++) {
  funs.push(() => {
    console.log(i)
  })
}
funs.forEach(item => item()) // 0, 1 2, 3, 4
~~~

很多关于`ES6`只是简单的说`let`和`var`不同的是增加了快作用域，所以会有上面这种结果。

### `why`

阮一峰老师的`ES6`教程中是这样描述的

~~~text
上面代码中，变量i是let声明的，当前的i只在本轮循环有效，所以每一次循环的i其实都是一个新的变量，所以最后输出的是6。你可能会问，如果每一轮循环的变量i都是重新声明的，那它怎么知道上一轮循环的值，从而计算出本轮循环的值？这是因为 JavaScript 引擎内部会记住上一轮循环的值，初始化本轮的变量i时，就在上一轮循环的基础上进行计算。

另外，for循环还有一个特别之处，就是设置循环变量的那部分是一个父作用域，而循环体内部是一个单独的子作用域。
~~~

所以我们大概可以这样理解， `js`引擎大概把我们的代码编译成这个样子了

~~~js
var funs = []
// i是在父作用域声明的
for (let i = 0; i < 5; i++) {
  // 每次都声明一个新变量
  let _i = i
  funs.push(() => {
    console.log(_i)
  })
}
funs.forEach(item => item()) // 0, 1 2, 3, 4
~~~

## 参考

[怎么理解for中let每次都是新变量](https://segmentfault.com/q/1010000007541743)