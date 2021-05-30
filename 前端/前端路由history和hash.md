## hash

`hash`路由的实现是基于`hash` 和`hashChange`事件实现


## history

`history`路由时基于`History API` 和`popstate`事件实现的
只有活动的历史条目发生变化时才会触发`popstate`, 注意这里`历史`是关键字，也就是说`pushState`并不会触发`popstate`应为`pushState`是新建了一条记录

## 区别

1. 基于`history`实现的路由更像真实的路径更美观。
2. 基于`hash`实现的路由路径是拼接在`#`号后浏览器发送请求并不会发送`hash`。
3. `hash`实现的路由兼容性更好。
4. `hash`实现的路由只能替换`#`后面的路径。
5. `history`实现路由可以替换同域的任何路径。
