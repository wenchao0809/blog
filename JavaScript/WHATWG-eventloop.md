# Event loops

## 定义

为了协调事件，用户交互，脚本，渲染，网络等，用户代理必须使用本文所述的事件循环， 每一个用户代理都有一个独一无二的与之先关的事件循环。

## 事件循环分类

### 用于浏览上下文(browsing contexts)

浏览器上下文

### 用于Worker的Event loops

 包含 
 * 专用Worker`(dedicated worker agent)`。
 * 共享Worker`(shared worker agent)`。
 * ServiceWorkers

 关于 [Worker定义](https://developer.mozilla.org/zh-CN/docs/Web/API/Web_Workers_API/Using_web_workers#%E4%B8%93%E7%94%A8worker)

 ### 用于Worklet的Event looops

`Worklet`是针对特定场景的轻量级的`Web Worker`



每个`Event loop`都有一个或多个任务队列`task queue`, 一个任务队列是一系列任务的集合， 这里任务队列是集合(Set),并不是队列(queue)因为事件循环每次执行任务时，都是从当前被选择的任务队列中选取第一个可执行的任务， 而不是当前队列的队首任务，所以名义上叫任务队列， 但是这里并不是先进先出的。

以下操作会触发新建任务

* `event` 事件触发，在特定的EventTarget对象上分派Event对象，通常由专门的任务完成。
* `parse` HTML解析器token一个或多个字节，然后处理任何结果令牌，这个过程一般视为一个任务。
* `Callbacks` 调用回调通常由指定任务完成。
* `Using a resource` 当算法获取资源时，如果以非阻塞方式进行提取，那么一旦某个或全部资源可用，对资源的处理由任务执行。
* `Reacting to DOM manipulation`  对DOM操作作出反应 某些元素具有响应DOM操作而触发的任务，例如，当该元素插入到文档中时。



隐式事件循环(`implied event loop`)可以从上下文推断出， 在单个代理的情况下结果是非常明确的，由于只有一个事件循环。如果是跨窗口通信如(Window和Worker)则不能依赖隐式的事件循环必须明确的提供事件循环，在排队任务和微任务的时候。

在事件循环中隐式的文档(`implied document`)取决于下列情况。

* 如果事件循环不是窗口类型的事件循环(`window event loop`) 则返回`null`
* 如果任务是在元素的上下文中调度的，则返回元素的`node document.`
* 如果任务是在浏览上下文( `browsing context` A browsing context is an environment in which Document objects are presented to the user.)中调度的， 则返回浏览上下文中活跃的文档
* 如果任务是被一个脚本调度的则返回脚本设置对象（`settings object`）相关的文档。

隐式事件循环和隐式文档，定义模糊导致很多不确定的操作，我们希望将来从规范中移除，尤其是隐式文档

### 处理模型

事件循环必须反复的执行以下步骤

> 1. 从任务队列中(多个任务队列 多个任务源)选择一个至少包含一个可执行任务的队列， 将其设为当前执行的任务队列， 如果没有满足上述条件的队列，则跳转到下属微任务执行的步骤。
> 2. 将当前任务队列中的第一个可执行任务设为`oldestTask`,并从当前任务队列中移除此任务。
> 3. 将事件循环当前执行的任务(`currently running task`)设为`oldestTask`。
> 4. 将`taskStartTime` 设为当前毫秒级的时间。
> 5. 执行`oldestTask`的`steps`。
> 6. 将事件循环当前执行的任务(`currently running task`)设为`null`。
> 7. 执行微任务检查点(perform a microtask checkpoint)
>> 1. 如果事件循环的`performing a microtask checkpoint`的标识为`true`则跳过。
>> 2. 设置`performing a microtask checkpoint`为`true`
>> 3. 如果当前微任务队列不为空一直重复下列步骤直到微任务队列为空。
>>> 1. 依次将微任务出队，并赋值给`oldestMicrotask`。
>>> 2. 将事件循环当前执行任务设为`oldestMicrotask`。
>>> 3. 运行`oldestMicrotask`
>>> 4. 将事件循环当前执行任务设为`null`
>> 4. 对其所主管的事件循环内的每个环境设置对象( environment settings object )，通知有关该环境设置对象rejected promises。
>> 5. 清理 indexed DB 转换。
>> 6. 将 performing a microtask checkpoint 标志置为false。
> 8. 更新渲染(省略)
> 9. 下一轮循环

### 生成任务源


参考 [WHATWG Event loop](https://nodecafe.me/post/whatwg-event-loop/)