# var

## 重复声明

~~~js
var a = 1
var a = 2
~~~

这样不会报错， 后者会取代前者

## 声明提升

~~~js
function foo() {
 console.log(age);
 var age = 26;
} 
~~~

上面代码实际相当于

~~~js
function foo() {
  var age
 console.log(age);
 age = 26
}
~~~

也就是我们可以在变量声明前访问变量

## var 声明的作用域

var 如果在全局作用域声明， 则会声明的变量会成为`globalThis`的属性

~~~js
var a = 1
console.log(1)
~~~

var 如果在函数内声明， 时函数内的局部变量只能在函数内部访问，函数执行完毕会随之销毁。

~~~js
function foo() {
  var age
 console.log(age);
 age = 26
}
console.log(age) // Uncaught ReferenceError: age is not defined
~~~

## for 循环

~~~js
 const callbacks = []
for (var i = 0 ; i < 5; i++) {
  callbacks.push(() => console.log(i)) 
}
callbacks.map(callback => callback()) // 5 5 5 5 5
~~~

由于 `var`声明的函数作用域，所以所有`callback`函数引用的都是同一个变量最终会递增到`5`

# let

## 快作用域

`let` 声明的是快作用域

~~~js
function fool() {
  if (true) {
    let x = 1
    console.log(x)
  }

  console.log(x) // 报错 不通于var
}
~~~
   
## 暂时性死区

`let`声明的变量不会再作用域中被提升， 因此要想访问变量 必须先声明

~~~js
console.log(x) // 报错
let x = 1

console.log(y) // 不报错
var y = 2

~~~

在 let 声明之前的执行瞬间被称为“暂时性死区”（temporal dead zone），在此
阶段引用任何后面才声明的变量都会抛出 ReferenceError。

## 全局变量

`let` 声明的全局变量不会变成`globalThis`的属性。

~~~js
let x = 1
console.log(globalThis.x) // undefined
~~~

## 条件声明

`let`由于不能在声明前访问变量,所以就不可能判断变量不存在的情况下声明变量。

另外因为条件声明是一种反模式，它让程序变得更难理解。如果你发现自己在使用这个模式，那一定有更好的替代方式。

## for 循环中的 let 声明

不同于`var`在`for`循环中的由于快作用域每次函数引用的都是一个不同的变量。

~~~js
 const callbacks = []
for (let i = 0 ; i < 5; i++) {
  callbacks.push(() => console.log(i)) 
}
callbacks.map(callback => callback()) // 0 1 2 3 4
~~~

上面代码等同于

~~~js
const callbacks = []
{
  let k
  for (k = 0 ; k < 5; k++) {
    let i = k
    callbacks.push(() => console.log(i)) 
  }
}

callbacks.map(callback => callback()) // 0 1 2 3 4
~~~

[关于`for`中的`let`可以参考](https://segmentfault.com/q/1010000007541743)

# const

用法和`let`相同，区别是`const`声明的变量要同时初始化， 否则声明之后就不能再进行赋值操作了。

~~~js
const x = 1
x = 2 // 报错
~~~

但是声明的引用类型，可以修改其属性， 如果修改引用类型自身就会报错。

~~~js
const x = []
const y = {}
x.push(1)  // 正常
y.fool = 1 // 正常

x = [] // 报错，
y = {} // 报错

~~~