## 相同点

### 都可以用来描述对象和函数

interface

~~~ts
interface A {
  x: string;
}

interface B { 
  (s: string) => boolean
}
~~~

type

~~~ts
type A = { x: string; }
type B = (s: string) => boolean
~~~

### 都可以被扩展

借用官方例子

~~~ts
interface Animal {
  name: string
}

interface Bear extends Animal {
  honey: boolean
}

const bear = getBear() 
bear.name
bear.honey
~~~

type

~~~ts
type Animal = {
  name: string
}

type Bear = Animal & { 
  honey: Boolean 
}

const bear = getBear();
bear.name;
bear.honey;
~~~

## 不同点

### interface 支持对现有的接口修改(支持声明合并) type则不可以

interface

~~~ts
interface Window {
  title: string
}

interface Window {
  ts: TypeScriptAPI
}

const src = 'const a = "Hello World"';
window.ts.transpileModule(src, {});
~~~

type

~~~ts
type Window = {
  title: string
}
 // Error: Duplicate identifier 'Window'.
type Window = {
  ts: TypeScriptAPI
}
~~~


### type可以重命名原始类型, 定义联合类型、元组等, interface没这个能力

 type 语句中还可以使用 typeof 获取实例的 类型进行赋值

~~~ts
// 重命名原始类型
type newstring = string;
// 联合类型
type stringandnumber = string | number;

let s: stringandnumber
s = '1'
s = 2
// 元组
type tuple = [string, number]
let t: tuple = ['1', 1]
// 使用typeof 获取实例类型赋值
type c = typeof t
let d: c = t
 
~~~


## 总结

一般能用`interface`实现就用`interface`, 当需要使用到`type`的特性时可以使用`type`