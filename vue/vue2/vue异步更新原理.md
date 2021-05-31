## 依赖更新

通过`Vue`响应式原理知道`get`函数内会收集`依赖`， `set`某个数据时会通知响应`订阅者`执行`watcher.update`方法如下

~~~js
update () {
    /* istanbul ignore else */
    if (this.lazy) {
      // lazy 的话就设置脏标识
      this.dirty = true
    } else if (this.sync) {
      // 同步的Watcher会立即执行回调函数
      this.run()
    } else {
      // 异步的调用queueWatcher执行
      queueWatcher(this)
    }
  }
~~~

## queueWatcher


几个变量

* `has: { [key: number]: ?true } = {}` `watcher` id的键值对用于检查当前队列中是否已经存在要插入的`Watcher` 。
* `circular: { [key: number]: number } = {}` 。
* `waiting = false` 标识当前队列是否正在等待刷新 。
* `flushing = false` 标识当前更新队列是否正在刷新 。
* `queue` 保存`Watcher`队里， 即更新队列 。

`waiting`和`flushing`有迷惑性看着像重复的定义， 其实二者是不同的

`waiting` 标识当前是否已经触发刷新任务 如果为true则标识已经触发了刷新任务但是任务是否正在执行是未知的。
`flushing` 为`true`的话就标识当前任务正在执行。

直接看代码

~~~js
export function queueWatcher (watcher: Watcher) {
  // 排队一个wather
  const id = watcher.id
  // 只有当前队列中没有watcher才执行下面代码， 已经存在就不需要重复添加了
  if (has[id] == null) {
    has[id] = true
    if (!flushing) {
      // 当前队列未刷新直接加入wather
      queue.push(watcher)
    } else {
      // if already flushing, splice the watcher based on its id
      // if already past its id, it will be run next immediately.
      let i = queue.length - 1
      // 当前队列正在刷新则需要根据watcher的id加入到响应的位置
      while (i > index && queue[i].id > watcher.id) {
        i--
      }
      queue.splice(i + 1, 0, watcher)
    }
    // queue the flush
    if (!waiting) {
      // 当前没有触发更新任务则需要触发一个更新任务
      waiting = true

      if (process.env.NODE_ENV !== 'production' && !config.async) {
        // 设置了同步直接刷新
        flushSchedulerQueue()
        return
      }
      // 调用nextTick异步刷新， 这里也解释了为什么nextTick可以拿到更新后的dom
    // 因为nextTick始终将cb加入到末尾， 数据更新后首先加入了flushSchedulerQueue之后加入取dom得cb这样就保证了二者的执行顺序
      nextTick(flushSchedulerQueue)
    }
  }
}
~~~

## 刷新队列

~~~js
function flushSchedulerQueue () {
  currentFlushTimestamp = getNow()
  flushing = true
  let watcher, id

  // Sort queue before flush.
  // This ensures that:
  // 1. Components are updated from parent to child. (because parent is always
  //    created before the child)
  // 2. A component's user watchers are run before its render watcher (because
  //    user watchers are created before the render watcher)
  // 3. If a component is destroyed during a parent component's watcher run,
  //    its watchers can be skipped.
  // 首先根据id大大小升序排序这是为了保证父组件总是在子组件之前更新
  // 父组件总是先于子组件创建 所以父组件的watcher id总是小于子组件的watcher id, watcher id 是全局递增的
  // 用户自定义的watcher 总是先于组件watcher执行
  // 如果子组件在父组件更新过程中被销毁了， 可以避免执行多余的watcher
  queue.sort((a, b) => a.id - b.id)

  // do not cache length because more watchers might be pushed
  // as we run existing watchers
  for (index = 0; index < queue.length; index++) {
    watcher = queue[index]
    if (watcher.before) {
      watcher.before()
    }
    id = watcher.id
    has[id] = null
    watcher.run()
    // in dev build, check and stop circular updates.
    if (process.env.NODE_ENV !== 'production' && has[id] != null) {
      circular[id] = (circular[id] || 0) + 1
      if (circular[id] > MAX_UPDATE_COUNT) {
        warn(
          'You may have an infinite update loop ' + (
            watcher.user
              ? `in watcher with expression "${watcher.expression}"`
              : `in a component render function.`
          ),
          watcher.vm
        )
        break
      }
    }
  }

  // keep copies of post queues before resetting state
  const activatedQueue = activatedChildren.slice()
  const updatedQueue = queue.slice()
  // 刷新完成后重置状态标识
  resetSchedulerState()

  // call component updated and activated hooks
  // 调用组件激活的钩子
  callActivatedHooks(activatedQueue)
  // 调用组件updated钩子
  callUpdatedHooks(updatedQueue)

  // devtool hook
  /* istanbul ignore if */
  if (devtools && config.devtools) {
    devtools.emit('flush')
  }
}

function resetSchedulerState () {
  // 队列置零
  index = queue.length = activatedChildren.length = 0
  // has 重置
  has = {}
  // circular重置
  if (process.env.NODE_ENV !== 'production') {
    circular = {}
  }
  // waiting flushing 重置
  waiting = flushing = false
}
~~~