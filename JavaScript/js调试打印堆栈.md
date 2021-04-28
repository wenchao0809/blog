## console.trace

~~~js
function foo() {
  function bar() {
    console.trace();
  }
  bar();
}

foo();
~~~

打印如下

~~~js
bar	@	VM189:3
foo	@	VM189:5
(anonymous)	@	VM189:8
~~~

## Error.protype.stack

`console.trace` 只能向控制台输出调用堆栈,我们并不能直接获取到调用堆栈的数据,但借助 Error,我们便可以直接获取当前的调用堆栈了,方法就是访问 `Error` 对象的 `stac`k 属性

另外 `Error.protype.stack` `Mozilla` 非标准特性， 我试了下`chrome`也是可以用的， 不要用在生产环境

~~~js
function trace() {
  try {
    throw new Error('myError');
  }
  catch(e) {
    console.log(e.stack);
  }
}
function b() {
  trace();
}
function a() {
  b(3, 4, '\n\n', undefined, {});
}
~~~

打印如下

~~~js
Error: myError
at trace (<anonymous>:3:11)
at b (<anonymous>:10:3)
at a (<anonymous>:13:3)
at <anonymous>:1:1
~~~

## 使用 arguments

使用`arguments.callee`可以获取当前正在执行的函数，然后通过`Functon.caller` 可以获取调用堆栈， 要注意下`callee`和`caller`在严格模式下已经废除

~~~js
function printCallStack() {
  let cur = arguments.callee.caller
  while(cur) {
    let arguments = cur.arguments
    arguments = Array.from(arguments)
    arguments = JSON.stringify(arguments)
    arguments = arguments.substring(1, arguments.length - 1)
    console.log(`${cur.name}(${arguments})`)
    cur = cur.caller
  }
}

function a(a) {
  printCallStack()
}

function b() {
  a(1, [1, 2, 3], { b: 1})
}

function main() {
  b()
}
main()
~~~