## window

浏览器环境中的全局对象 有 `windows.self === windows`

## self

用于`在 Web Workers `中获取 全局对象

## globalThis

通用的获取全局对象
可以在`Web、Worker、Node`中获取全局对象

## frames

类数组对象，保存子窗口`windows`对象

下面判断当前窗口是不是父窗口的第一个窗口
~~~js
 if (window.parent.frames[0] != window.self) {
    // 当前对象不是frames列表中的第一个时
 }
~~~

获取全局对象方法

~~~js
function getGlobal() {
  return (globalThis ? globalThis
    : ((typeof global !== 'undefined') ? global
    : ((typeof window !== 'undefined') ? window
    : ((typeof self !== 'undefined') ? self : this))))
}

function getGlobal() {
  return globalThis ? globalThis
    : global ? global
    : window ? window
    : self ? self : this
}
~~~