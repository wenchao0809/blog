## 几种方法

`JavaScript`判断变量类型一直是比较头疼的问题总结下总共有一下几种方法

### typeof 

基本数据类型用`typeof`

~~~js
let a = '1'
typeof a === 'string' // true
~~~

缺点
* 无法判断对象对于对象会一直返回`object`
* `typeof null` 也会返回`Object`

### instanceof

`instanceof` 运算符用于检测构造函数的 `prototype` 属性是否出现在某个实例对象的原型链上。


~~~js
function A() {
  this.a = 'a'
}

let a = new A
a instanceof A // true
a instanceof Object // true

function B() {}
B.prototype = new A()
B.prototype.constructor = B
let b = new B
b instanceof B // true
b instanceof A // true
~~~

缺点
* 只要是构造函数的`prototype`在对象的原型链上都会返回true。

### constructor

~~~js
function A() {
  this.a = 'a'
}

let a = new A
a.constructor === A // true
~~~

缺点

* `null` 和 `undefined` 是无效的对象，因此是不会有 `constructor` 存在的，这两种类型的数据需要通过其他方式来判断。

* 函数的 `constructor` 是不稳定的，这个主要体现在自定义对象上，当开发者重写 `prototype` 后，原有的 `constructor` 引用会丢失，`constructor` 会默认为 `Object`

### Object.Prototype.toString

一定要用`Object.Prototype.toString.call()`调用， 不要直接用对象上的`toString`因为很多对象都重写了`toString`方法。

`Object.Prototype.toString`主要用于判断`JavaScript`内置类型像`Date`, `Array`， `RegExp`等

~~~js
let a = /abc/
Object.prototype.toString.call(a) // "[object RegExp]"
let b = new Date()
Object.prototype.toString.call(b) // "[object Date]"
~~~

## lodash

* `lodash` 对于基本数据类型默认使用`typeof`
* 对于对象则使用 `Object.prototype.string`, 包括基本数据类型的的包装对象。