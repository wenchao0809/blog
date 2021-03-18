## ES6 `class`继承的实现原理


### 实例

~~~js
class Super {
  x = 1
  static testArray = [1]
  constructor() {
    debugger
  }
  static log(msg) {
    console.log('message', msg)
  }
  test(msg) {
    console.log('test message', msg)
  }
}

class Sub extends Super {
  y = 2
  static testArray = [3, 1]
  constructor() {
    super()
  }
  static testSuperOnClass() {
    super.log('test super on class')
  }
  testSuperOnInstance() {
    console.log(super.x)
    // super()
    super.test('test suepr on instance')
  }
}
~~~
  ES6的继承链有三条
  
 > 1. 静态属性的继承也就是以父类自身为原型创建父类 即Sub = Objet.create(Super)
 > 2. 方法的继承， 方法的继承是通过原型继承的也就是Sub.prototype = Object.create(Super.prototype)
 > 3. 实例属性的继承，通过盗用父类构造函数创建`this`实现实例属性的继承， 每个子类都必须调用`super`父类的构造函数， 如果子类没有定义构造函数，则会默认调用父类的构造函数。
 
 通过以上三点可以得出
 所以得出结论ES6的继承只不过是`ES5`寄生式组合继承的语法糖

以上代码用`ES5`重写 大概如下

~~~js
function Super() {
  this.x = 1
}

Super.testArray = [1]
// 静态方法
Super.log =  function(msg) {
  console.log('message', msg)
}
// 实例方法
Super.prototype.test = function(msg) {
  console.log('test message', msg)
}

function Sub() {
  // 盗用构造函数继承实例属性
  Super.apply(this)
  this.y = 2
}

// 继承静态属性
Sub.__proto__ = Super

Sub.testSuperOnClass = function() {
  super.log('test super on class')
}

// 原型继承实现方法的继承
var p = Object.create(Super.prototype)
p.constructor = Sub
p.testSuperOnInstance = function() {
  console.log(super.x)
  // super()
  super.test('test suepr on instance')
}
Sub.prototype = p
~~~

## `super`的注意事项

> 1.  在子类的构造函数中`super`作为函数调用时，代表父类的构造函数
> 2. `super`作为对象时，在普通方法中，指向父类的原型对象；在静态方法中，指向父类。
> 3. 在 `JavaScript高级编程第四版中有说`super 只能在派生类构造函数和静态方法中使用。
> 4. 不能单独引用`super`关键字， 只能当作构造函数访问或者访问它的属性。
> 5. `super`被当作父类的构造函数调用时， 可以传递参数。
> 6. 如果没有定义类构造函数，在实例化派生类时会调用 super()，而且会传入所有传给派生类的参数。
> 7. 在类构造函数中，不能在调用 super()之前引用 this。
> 8. 如果在派生类中显式定义了构造函数，则要么必须在其中调用 super()，要么必须在其中返回一个对象。