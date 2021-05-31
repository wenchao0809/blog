### nextick的作用

在下次 DOM 更新循环结束之后执行延迟回调。在修改数据之后立即使用这个方法，获取更新后的 DOM。

### 全局`Vue.nextTick`

~~~js
export function nextTick (cb?: Function, ctx?: Object) {
  let _resolve
  callbacks.push(() => {
    if (cb) {
      try {
        // 调用回调函数 并绑定`this` 只有this.$next或者手动传入ctx绑定回调函数的this
        cb.call(ctx)
      } catch (e) {
        handleError(e, ctx, 'nextTick')
      }
    } else if (_resolve) {
      // 不传递回调函数直接返回resove的Promise
      _resolve(ctx)
    }
  })
  if (!pending) {
    pending = true
    timerFunc()
  }
  // $flow-disable-line
  if (!cb && typeof Promise !== 'undefined') {
    return new Promise(resolve => {
      _resolve = resolve
    })
  }
}

~~~

因为`vue`的`dom`更新是异步的数据更新后我们不能立即拿到更新后的`dom`使用`nexttick`我们可以拿到更新后的`dom`, `nextick`将回调函数`push`进`callbacks`, 在`dom`更新后刷新`callbacks`

### this.$nextTick

~~~js
  Vue.prototype.$nextTick = function (fn: Function) {
    return nextTick(fn, this)
  }
~~~

自动绑定`this`