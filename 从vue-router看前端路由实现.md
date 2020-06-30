

## 如何使用

```javascript
import Vue from 'vue'
import routes from './router'
import Router from 'vue-router'

const router = new VueRouter({
  routes
})

Vue.use(Router)

new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
```

总共分为三步

> 1. 创建一个`Router`实例，传入我们定义好的路由。
> 2. **调用** `Vue.use(Router)` 传`Router`类。
> 3. 创建`Vue`跟实例传入第一步创建的`Router`实例。

## Router实例如何创建 ？

### Router类

定义在`index.js`

~~~javascript
export default class VueRouter {
  static install() {}
  app: any;
  apps: Array<any>;
  // 导航是否准备好
  ready: boolean;
  readyCbs: Array<Function>;
  options: RouterOptions;
  mode: string;
  history: HashHistory | HTML5History | AbstractHistory;
  matcher: Matcher;
  fallback: boolean;
  beforeHooks: Array<?NavigationGuard>;
  resolveHooks: Array<?NavigationGuard>;
  afterHooks: Array<?AfterNavigationHook>;
  // 构造函数
  constructor () {}
	// 用于匹配路由
	match () {}
	// 获取当前匹配路由 
  get currentRoute (): ?Route {
    return this.history && this.history.current
  }
	// 初始化方法 重要下文详解
	init () {}
	// 用于注册beforeEach钩子函数 router.beforeEach((form, to, next) => {})调用的就是这个方法
	beforeEach (fn: Function): Function {}
	// 用于注册全局beforeResolve钩子函数
	beforeResolve (fn: Function): Function {}
	// 用于注册afterEach钩子函数
	afterEach (fn: Function): Function {}
	// onReady 
	onReady (cb: Function, errorCb?: Function) {}
	//
	onError (errorCb: Function) {}
	// 路由导航
	push (location: RawLocation, onComplete?: Function, onAbort?: Function){}
	replace (location: RawLocation, onComplete?: Function, onAbort?: Function) {}
	go (n: number) {}
	back () {}
	forward () {}
	// 
	getMatchedComponents (to?: RawLocation | Route): Array<any> {}
  // 
  resolve (
    to: RawLocation,
    current?: Route,
    append?: boolean
  ): {
    location: Location,
    route: Route,
    href: string,
    // for backwards compat
    normalizedTo: Location,
    resolved: Route
  }  {}
	//  动态添加路由
	addRoutes (routes: Array<RouteConfig>) {}
}
~~~



###   Router构造函数

```javascript
  constructor (options: RouterOptions = {}) {
    this.app = null
    this.apps = []
    // 我们传入的路由选项
    this.options = options
    // 保存注册的全局beforeEach钩子
    this.beforeHooks = []
    // 保存注册的全局beforeResolve钩子
    this.resolveHooks = []
    // 保存注册的全局afterEach钩子
    this.afterHooks = []
    /*
     * !!!重要 路由匹配的核心实现
     * 返回一个对象包含下列属性 { match, addRoutes } 
     * match 用于路由匹配很重要下文详解
     * addRoutes 用于动态添加路由
     */
    this.matcher = createMatcher(options.routes || [], this)
		// 路由模式 history | abstract | hash 默认是hash路由
    let mode = options.mode || 'hash'
    // 当前运行环境不支持history模式退回到hash模式
 		// 这里要注意在前两个条件满足的情况下如果不明确指定fallback为false则默认为true
    this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false
    if (this.fallback) {
      mode = 'hash'
    }
    // 当前环境不再浏览器模式为abstract
    if (!inBrowser) {
      mode = 'abstract'
    }
    this.mode = mode
		// 根据mode创建一个History实例
    switch (mode) {
      case 'history':
        this.history = new HTML5History(this, options.base)
        break
      case 'hash':
        this.history = new HashHistory(this, options.base, this.fallback)
        break
      case 'abstract':
        this.history = new AbstractHistory(this, options.base)
        break
      default:
        if (process.env.NODE_ENV !== 'production') {
          assert(false, `invalid mode: ${mode}`)
        }
    }
  }
```

###  createMatcher方法

~~~js
export function createMatcher (
  routes: Array<RouteConfig>,
  router: VueRouter
): Matcher {
  // 创建routeMap
  const { pathList, pathMap, nameMap } = createRouteMap(routes)

  function addRoutes (routes) {
    createRouteMap(routes, pathList, pathMap, nameMap)
  }
	// 最终调用的match方法
  function match (
    raw: RawLocation,
    currentRoute?: Route,
    redirectedFrom?: Location
  ): Route {
    // 格式化location 主要是提取query
    const location = normalizeLocation(raw, currentRoute, false, router)
    const { name } = location

    if (name) {
      // 存在name 优先命名导航
      const record = nameMap[name]
      if (process.env.NODE_ENV !== 'production') {
        warn(record, `Route with name '${name}' does not exist`)
      }
      // 未匹配
      if (!record) return _createRoute(null, location)
      const paramNames = record.regex.keys
        .filter(key => !key.optional)
        .map(key => key.name)

      if (typeof location.params !== 'object') {
        location.params = {}
      }
			// 如果没有传递params允许用当前路由的key相同的参数
      if (currentRoute && typeof currentRoute.params === 'object') {
        for (const key in currentRoute.params) {
          if (!(key in location.params) && paramNames.indexOf(key) > -1) {
            location.params[key] = currentRoute.params[key]
          }
        }
      }
			// 将参数填充进路径
      location.path = fillParams(record.path, location.params, `named route "${name}"`)
      // 调用route创建一个route对象
      return _createRoute(record, location, redirectedFrom)
    } else if (location.path) {
      // 路径匹配
      // 和命名匹配相反， 路径匹配要从路径中解析出参数
      location.params = {}
      for (let i = 0; i < pathList.length; i++) {
        const path = pathList[i]
        const record = pathMap[path]
        // matchRoute会把参数解析到params
        if (matchRoute(record.regex, location.path, location.params)) {
          return _createRoute(record, location, redirectedFrom)
        }
      }
    }
    // no match
    return _createRoute(null, location)
  }
  // _createRoute
  function _createRoute (
    record: ?RouteRecord,
    location: Location,
    redirectedFrom?: Location
  ): Route {
    // 重定向
    if (record && record.redirect) {
      return redirect(record, redirectedFrom || location)
    }
    // 路由别名
    if (record && record.matchAs) {
      return alias(record, location, record.matchAs)
    }
    // 创建route对象
    return createRoute(record, location, redirectedFrom, router)
  }

  return {
    match,
    addRoutes
  }
}
// matchRoute
function matchRoute (
  regex: RouteRegExp,
  path: string,
  params: Object
): boolean {
  const m = path.match(regex)

  if (!m) {
    return false
  } else if (!params) {
    return true
  }
	// 解析出参数
  for (let i = 1, len = m.length; i < len; ++i) {
    const key = regex.keys[i - 1]
    const val = typeof m[i] === 'string' ? decodeURIComponent(m[i]) : m[i]
    if (key) {
      // Fix #1994: using * with props: true generates a param named 0
      params[key.name || 'pathMatch'] = val
    }
  }

  return true
}
~~~

### createRouteMap方法

~~~js
export function createRouteMap (
  routes: Array<RouteConfig>,
  oldPathList?: Array<string>,
  oldPathMap?: Dictionary<RouteRecord>,
  oldNameMap?: Dictionary<RouteRecord>
): {
  pathList: Array<string>,
  pathMap: Dictionary<RouteRecord>,
  nameMap: Dictionary<RouteRecord>
} {
  // 这里routes就是我们定义的路由对象
  // 这里有传入`oldPathList` oldPathMap oldPathMap, 在首次调用不传， 主要用于动态添加路由
  // the path list is used to control path matching priority
  const pathList: Array<string> = oldPathList || []
  const pathMap: Dictionary<RouteRecord> = oldPathMap || Object.create(null)
  const nameMap: Dictionary<RouteRecord> = oldNameMap || Object.create(null)
	// 循环添加Record
  routes.forEach(route => {
    addRouteRecord(pathList, pathMap, nameMap, route)
  })

  // ensure wildcard routes are always at the end
  // 确保通用匹配在路由末尾 允许有多个通配符路由？
  for (let i = 0, l = pathList.length; i < l; i++) {
    if (pathList[i] === '*') {
      pathList.push(pathList.splice(i, 1)[0])
      // 修正指针
      l--
      i--
    }
  }

  if (process.env.NODE_ENV === 'development') {
    // 检查是否有不是‘/’开头的路径
    // warn if routes do not include leading slashes
    const found = pathList
    // check for missing leading slash
      .filter(path => path && path.charAt(0) !== '*' && path.charAt(0) !== '/')

    if (found.length > 0) {
      const pathNames = found.map(path => `- ${path}`).join('\n')
      warn(false, `Non-nested routes must include a leading slash character. Fix the following routes: \n${pathNames}`)
    }
  }

  return {
    pathList,
    pathMap,
    nameMap
  }
}

function addRouteRecord (
  pathList: Array<string>,
  pathMap: Dictionary<RouteRecord>,
  nameMap: Dictionary<RouteRecord>,
  route: RouteConfig,
  parent?: RouteRecord,
  matchAs?: string
) {
  // pathList pathMap nameMap 直接传递引用到函数内
  // 我们定义的 path 和name
  const { path, name } = route
  if (process.env.NODE_ENV !== 'production') {
    // 检查path未定义的情况
    assert(path != null, `"path" is required in a route configuration.`)
    assert(
      typeof route.component !== 'string',
      `route config "component" for path: ${String(
        path || name
      )} cannot be a ` + `string id. Use an actual component instead.`
    )
  }
	// 编译正则的选项 { strict: true | false, sensitive: false | true, end: false | true }
  // 详见Path-to-RegExp用法
  const pathToRegexpOptions: PathToRegexpOptions =
    route.pathToRegexpOptions || {}
  // 格式化path
  const normalizedPath = normalizePath(path, parent, pathToRegexpOptions.strict)

  if (typeof route.caseSensitive === 'boolean') {
    // 大小写敏感选项
    pathToRegexpOptions.sensitive = route.caseSensitive
  }

  const record: RouteRecord = {
    path: normalizedPath,
    regex: compileRouteRegex(normalizedPath, pathToRegexpOptions),
    // 路由组件
    // 命名视图 默认命名default
    components: route.components || { default: route.component },
    // 命名到组件的实例的映射表
    instances: {},
    // 路由名称
    name,
    // RouteRecord 父级路由
    parent,
    // 路由别名的原始路由路径
    matchAs,
    // 路由重定向
    redirect: route.redirect,
    // 路由独享beforeEnter钩子
    beforeEnter: route.beforeEnter,
    // 元数据
    meta: route.meta || {},
    // 针对不同的命名视图提供不同props
    props:
      route.props == null
        ? {}
        : route.components
          ? route.props
          : { default: route.props }
  }

  if (route.children) {
    // Warn if route is named, does not redirect and has a default child route.
    // If users navigate to this route by name, the default child will
    // not be rendered (GH Issue #629)
    if (process.env.NODE_ENV !== 'production') {
      if (
        route.name &&
        !route.redirect &&
        route.children.some(child => /^\/?$/.test(child.path))
      ) {
        warn(
          false,
          `Named Route '${route.name}' has a default child route. ` +
            `When navigating to this named route (:to="{name: '${
              route.name
            }'"), ` +
            `the default child route will not be rendered. Remove the name from ` +
            `this route and use the name of the default child route for named ` +
            `links instead.`
        )
      }
    }
    route.children.forEach(child => {
      // 递归addRouteRecord
      const childMatchAs = matchAs
        ? cleanPath(`${matchAs}/${child.path}`)
        : undefined
      addRouteRecord(pathList, pathMap, nameMap, child, record, childMatchAs)
    })
  }

  if (!pathMap[record.path]) {
    // 将路由记录添加进pathMap
    pathList.push(record.path)
    pathMap[record.path] = record
  }

  if (route.alias !== undefined) {
    // 处理别名
    const aliases = Array.isArray(route.alias) ? route.alias : [route.alias]
    for (let i = 0; i < aliases.length; ++i) {
      // 路由别名可以是个数组
      const alias = aliases[i]
      if (process.env.NODE_ENV !== 'production' && alias === path) {
        warn(
          false,
          `Found an alias with the same value as the path: "${path}". You have to remove that alias. It will be ignored in development.`
        )
        // skip in dev to make it work
        continue
      }
			// 存在路由别名 则为路由别名也创建一个路由记录
      // 注意这个记录并没有保存 components等一部分路由信息， 当匹配到这个记录时， 还是会通过
      // matchAs 找到原始路由获取这些信息
      const aliasRoute = {
        path: alias,
        children: route.children
      }
      addRouteRecord(
        pathList,
        pathMap,
        nameMap,
        aliasRoute,
        parent,
        record.path || '/' // 这个是原始路由的路径
      )
    }
  }

  if (name) {
    if (!nameMap[name]) {
      // 将路由记录添加进nameMap
      nameMap[name] = record
    } else if (process.env.NODE_ENV !== 'production' && !matchAs) {
      // 检查是否有重复的命名路由
      warn(
        false,
        `Duplicate named routes definition: ` +
          `{ name: "${name}", path: "${record.path}" }`
      )
    }
  }
}

function compileRouteRegex (
  path: string,
  pathToRegexpOptions: PathToRegexpOptions
): RouteRegExp {
  const regex = Regexp(path, [], pathToRegexpOptions)
  if (process.env.NODE_ENV !== 'production') {
    // 避免路径中出现重复的路径参数
    const keys: any = Object.create(null)
    regex.keys.forEach(key => {
      warn(
        !keys[key.name],
        `Duplicate param keys in route with path: "${path}"`
      )
      keys[key.name] = true
    })
  }
  return regex
}

function normalizePath (
  path: string,
  parent?: RouteRecord,
  strict?: boolean
): string {
   // 是否允许末尾`/` /bar/foo/
  if (!strict) path = path.replace(/\/$/, '')
  if (path[0] === '/') return path
  if (parent == null) return path
   // cleanPath 避免出现两个连续的 //
   // 拼接上父级路由的路径
  return cleanPath(`${parent.path}/${path}`)
}
~~~
这里调用`createRouteMap`返回` 

* `pathList` 数组保存所有`path`集合
* `pathMap` `path` 到 `record`的映射
* `nameMap` `name` 到 `record` 的映射

数据类型 `RouteRecored`

~~~js
declare type RouteRecord = {
  // 路径
  path: string;
  // 路径正则
  regex: RouteRegExp;
  // 组件 对象key视图命名默认为default value为组件定义
  components: Dictionary<any>;
	//  对象key视图命名默认为default value组件实例
  instances: Dictionary<any>;
  // 路由命名
  name: ?string;
  // 父RouteRecord
  parent: ?RouteRecord;
  // 重定向
  redirect: ?RedirectOption;
  // 别名路由时保存原始路由的路径， 别名路由并不保存路由导航所需信息， 
  matchAs: ?string;
  // 路由独享beforeEnter钩子
  beforeEnter: ?NavigationGuard;
  // 原数据
  meta: any;
  // 往组件传递 props
  props: boolean | Object | Function | Dictionary<boolean | Object | Function>;
}

~~~

## vue.use(Router)

`vue-router`作为`Vue`插件用于实现路由功能，看过`Vue`源码的应该知道`vue.use`方法实质调用插件自身的`install`方法，上面介绍`Router`类时有提到一个静态的`install`方法。

```javascript
// 定义在src/install.js
export function install (Vue) {
  // 只执行一次
  if (install.installed && _Vue === Vue) return
  install.installed = true

  _Vue = Vue

  const isDef = v => v !== undefined

  const registerInstance = (vm, callVal) => {
    let i = vm.$options._parentVnode
    if (isDef(i) && isDef(i = i.data) && isDef(i = i.registerRouteInstance)) {
      i(vm, callVal)
    }
  }
	// 混入全局beforeCreate函数
  // 每个实例调用前都会执行
  Vue.mixin({
    beforeCreate () {
      if (isDef(this.$options.router)) {
        // 只有Vue跟实例会传入
        this._routerRoot = this
        this._router = this.$options.router
        // 调用router实例方法
        this._router.init(this)
        // 根实例上的_route定义为响应式
        Vue.util.defineReactive(this, '_route', this._router.history.current)
      } else {
        // 子组件
        this._routerRoot = (this.$parent && this.$parent._routerRoot) || this
      }
      // 注册
      registerInstance(this, this)
    },
    destroyed () {
      // 第二个参数传空 unregister
      registerInstance(this)
    }
  })

  Object.defineProperty(Vue.prototype, '$router', {
    get () { return this._routerRoot._router }
  })

  Object.defineProperty(Vue.prototype, '$route', {
    get () { return this._routerRoot._route }
  })

  Vue.component('RouterView', View)
  Vue.component('RouterLink', Link)

  const strats = Vue.config.optionMergeStrategies
  // use the same hook merging strategy for route hooks
  strats.beforeRouteEnter = strats.beforeRouteLeave = strats.beforeRouteUpdate = strats.created
}

```



`install`方法做了一下操作

> 1. 调用`Vue.mixin`混入了两个全局生命周期函数`beforeCreate` `destroyed`，所有的`Vue`实例都会有这两个生命周期函数。
> 2. 在`Vue`原型上定义了了两个属性`$router`和`$route`，既然在原型上定义的，那么所有的实例都会继承。
> 3. 注册`RouterView`和`RouterLink`两个全局组件。
> 4. 最后一步定义了`beforeRouteEnter`、`beforeRouteLeave`、`beforeRouteUpdate`和`created`的一样



## 创建Vue跟实例

在上一步`install`方法混入了`beforeCreate`生命周期函数，因此我们在创建跟实例时就会调用这个函数做以下操作

首先如果当前创建的实例是否是跟实例， 因为只有在创建根实例时我们才会传入`router`选项所以这很容易判断，如果当前实例是跟实例那么

初始化 `_routerRoot` `_router` 两个属性， 之后执行`router`实例上的`init`方法，最后初始化`__route`属性注意这个属性是响应式属性。

如果当前实例不是跟实例，那么直接初始化`_routerRoot`为`$parent._routerRoot` 这个保存的其实就是跟实例。

###  init方法做了哪些操作

 ```javascript

  init (app: any /* Vue component instance */) {
    // 开发模式打印错误 确保在创建跟实例前调用 Vue.use方法
    process.env.NODE_ENV !== 'production' && assert(
      install.installed,
      `not installed. Make sure to call \`Vue.use(VueRouter)\` ` +
      `before creating root instance.`
    )
		// 调用init会传入跟实例 保存跟实例
    this.apps.push(app)

    // set up app destroyed handler
    // https://github.com/vuejs/vue-router/issues/2639
    app.$once('hook:destroyed', () => {
      // clean out app from this.apps array once destroyed
      const index = this.apps.indexOf(app)
      if (index > -1) this.apps.splice(index, 1)
      // ensure we still have a main app or null if no apps
      // we do not release the router so it can be reused
      if (this.app === app) this.app = this.apps[0] || null

      if (!this.app) {
        // clean up event listeners
        // https://github.com/vuejs/vue-router/issues/2341
        this.history.teardownListeners()
      }
    })

    // main app previously initialized
    // return as we don't need to set up new history listener
    if (this.app) {
      // 已经执行过初始化过程 直接return
      return
    }

    this.app = app

    const history = this.history

    if (history instanceof HTML5History || history instanceof HashHistory) {
      const setupListeners = () => {
        history.setupListeners()
      }
      // 首次初始化后 这里调用 tranitionTo方法开始导航
      history.transitionTo(history.getCurrentLocation(), setupListeners, setupListeners)
    }
		// history 注册了一个回调函数，用于之后执行
    history.listen(route => {
      this.apps.forEach((app) => {
        app._route = route
      })
    })
  }

 ```



注意这里`init`方法只在创建`Vue`跟实例的`beforeCreate`方法中执行一次，也就是在首次进入页面时执行一次，之后的导航不在执行`init`，除非你手动调用`this.$router.init`方法。

这里` history.transitionTo` 和 `history.listen`的代码顺序是个细节很有意思，稍后再将

### transitionTo 方法

这个方法定义在`history/base.js`中

```js
transitionTo (
    location: RawLocation,
    onComplete?: Function,
    onAbort?: Function
  ) {
  	// 调用 match方法匹配路由
  	// 这里的route就是我们之前创建建的`router`实例
		// 我们在新建`History`实例时将`router`传入构造函数
  	const route = this.router.match(location, this.current)
    // 确认导航
    this.confirmTransition(
      route,
      () => {
        const prev = this.current
        this.updateRoute(route)
        onComplete && onComplete(route)
        this.ensureURL()
        this.router.afterHooks.forEach(hook => {
          hook && hook(route, prev)
        })

        // fire ready cbs once
        if (!this.ready) {
          this.ready = true
          this.readyCbs.forEach(cb => {
            cb(route)
          })
        }
      },
      err => {
        if (onAbort) {
          onAbort(err)
        }
        if (err && !this.ready) {
          this.ready = true
          // Initial redirection should still trigger the onReady onSuccess
          // https://github.com/vuejs/vue-router/issues/3225
          if (!isRouterError(err, NavigationFailureType.redirected)) {
            this.readyErrorCbs.forEach(cb => {
              cb(err)
            })
          } else {
            this.readyCbs.forEach(cb => {
              cb(route)
            })
          }
        }
      }
    )
  }

 
```



`transitionTo`方法首先调用 `this.router.match`方法获取一个匹配路由，这里的route就是我们之前创建建的`router`实例， 我们在新建`History`实例时将`router`传入构造函数, `router.match`实际调用的是`this.matcher.match` 这里的`matcher`就是之前调用`createMatcher`返回的。

```js
match (
    raw: RawLocation,
    current?: Route,
    redirectedFrom?: Location
  ): Route {
    return this.matcher.match(raw, current, redirectedFrom)
  }
```



### createRoute函数

定义在 `utils/route.js`

~~~js
export function createRoute (
  record: ?RouteRecord,
  location: Location,
  redirectedFrom?: ?Location,
  router?: VueRouter
): Route {
  const stringifyQuery = router && router.options.stringifyQuery

  let query: any = location.query || {}
  try {
    query = clone(query)
  } catch (e) {}

  const route: Route = {
    name: location.name || (record && record.name),
    meta: (record && record.meta) || {},
    path: location.path || '/',
    hash: location.hash || '',
    query,
    params: location.params || {},
    // 拼接query参数
    fullPath: getFullPath(location, stringifyQuery),
    // 当前route路径匹配对象
    matched: record ? formatMatch(record) : []
  }
  if (redirectedFrom) {
    route.redirectedFrom = getFullPath(redirectedFrom, stringifyQuery)
  }
  return Object.freeze(route)
}
// 向上遍历将这条路径上所有的Record加入到matched中
function formatMatch (record: ?RouteRecord): Array<RouteRecord> {
  const res = []
  while (record) {
    res.unshift(record)
    record = record.parent
  }
  return res
}
~~~



### confirmTransition 函数

在`transitionTo`中又调用`confirmTransition`函数执行一系列钩子函数, 当前函数内`this`指向`History`实例

~~~js
  confirmTransition (route: Route, onComplete: Function, onAbort?: Function) {
    const current = this.current
    const abort = err => {
      // changed after adding errors with
      // https://github.com/vuejs/vue-router/pull/3047 before that change,
      // redirect and aborted navigation would produce an err == null
      if (!isRouterError(err) && isError(err)) {
        if (this.errorCbs.length) {
          this.errorCbs.forEach(cb => {
            cb(err)
          })
        } else {
          warn(false, 'uncaught error during route navigation:')
          console.error(err)
        }
      }
      onAbort && onAbort(err)
    }
    const lastRouteIndex = route.matched.length - 1
    const lastCurrentIndex = current.matched.length - 1
    if (
      isSameRoute(route, current) &&
      // in the case the route map has been dynamically appended to
      lastRouteIndex === lastCurrentIndex &&
      route.matched[lastRouteIndex] === current.matched[lastCurrentIndex]
    ) {
      // 重复导航 也就是 to === from
      this.ensureURL()
      return abort(createNavigationDuplicatedError(current, route))
    }
		/**
			* 三种队列
			* updated 复用的队列，也就是当前匹配的路由和已经匹配的路由重合的部分
			* deactivated 失活的路由
			* activated 激活的路由
			*/
    const { updated, deactivated, activated } = resolveQueue(
      this.current.matched,
      route.matched
    )

    const queue: Array<?NavigationGuard> = [].concat(
      // in-component leave guards
      // 现在失活的路由组件内执行beforeRouteLeave钩子
      extractLeaveGuards(deactivated),
      // global before hooks
      // 全局beforeEach钩子
      this.router.beforeHooks,
      // in-component update hooks
      // 在复用组件内执行 beforeRouteUpdate
      extractUpdateHooks(updated),
      // in-config enter guards
      // 路由独享的beforeEnter钩子
      activated.map(m => m.beforeEnter),
      // async components
      // 解析异步组件
      resolveAsyncComponents(activated)
    )
		
    this.pending = route
    const iterator = (hook: NavigationGuard, next) => {
      if (this.pending !== route) {
        return abort(createNavigationCancelledError(current, route))
      }
      try {
        hook(route, current, (to: any) => {
          if (to === false) {
            // next(false) -> abort navigation, ensure current URL
            this.ensureURL(true)
            abort(createNavigationAbortedError(current, route))
          } else if (isError(to)) {
            this.ensureURL(true)
            abort(to)
          } else if (
            typeof to === 'string' ||
            (typeof to === 'object' &&
              (typeof to.path === 'string' || typeof to.name === 'string'))
          ) {
            // next('/') or next({ path: '/' }) -> redirect
            abort(createNavigationRedirectedError(current, route))
            if (typeof to === 'object' && to.replace) {
              this.replace(to)
            } else {
              this.push(to)
            }
          } else {
            // confirm transition and pass on the value
            next(to)
          }
        })
      } catch (e) {
        abort(e)
      }
    }

    runQueue(queue, iterator, () => {
      const postEnterCbs = []
      const isValid = () => this.current === route
      // wait until async components are resolved before
      // extracting in-component enter guards
      // 
      const enterGuards = extractEnterGuards(activated, postEnterCbs, isValid)
      // 全局beforeResolve钩子
      const queue = enterGuards.concat(this.router.resolveHooks)
      runQueue(queue, iterator, () => {
        if (this.pending !== route) {
          return abort(createNavigationCancelledError(current, route))
        }
        this.pending = null
        onComplete(route)
        if (this.router.app) {
          this.router.app.$nextTick(() => {
            postEnterCbs.forEach(cb => {
              cb()
            })
          })
        }
      })
    })
  }

~~~

### 队列执行函数(runQueue)

~~~js

export function runQueue (queue: Array<?NavigationGuard>, fn: Function, cb: Function) {
  const step = index => {
    if (index >= queue.length) {
      cb()
    } else {
      if (queue[index]) {
        fn(queue[index], () => {
          step(index + 1)
        })
      } else {
        step(index + 1)
      }
    }
  }
  step(0)
}

~~~

可以看到在`confirmTransition`中执行了一下操作

> 1. 首先判断路由是否重复导航，如果是打印警告信息。
> 2. 解析出`beforeRouteLeave`、`beforeEach`、`beforeRouteUpdate`, `beforeEnter` 异步组件并调用`runQueue`执行。
> 3. 在第一步调用`runQueue`执行完毕的回调函数中，解析出`beforeRouteEnter`和`beforeResolve` 钩子并执行。
> 4. 在第二个`runQueue`执行完毕的回调函数中， 调用`onComplete`
> 5. 在第三步解析`beforeRouteEnter`钩子时，为实现在回调函数中那到组件实例，做了一层包装，在导航完成后调用`pol`轮序获取实例l。

第四部的`onComplete`函数是在`tranitionTo`函数中传入的如下

~~~js
   () => {
        const prev = this.current
        this.updateRoute(route)
        onComplete && onComplete(route)
        this.ensureURL()
        this.router.afterHooks.forEach(hook => {
          hook && hook(route, prev)
        })

        // fire ready cbs once
        if (!this.ready) {
          this.ready = true
          this.readyCbs.forEach(cb => {
            cb(route)
          })
        }
      }
   
    function updateRoute (route: Route) {
      this.current = route
      this.cb && this.cb(route)
    }

~~~

> 1. 首先保存之前的路由用于传入之后的钩子调用
>
>    ~~~js
>    const prev = this.current
>    ~~~
>
> 2. 调用updateRoute 更新路由
>
>    ~~~js
>    function updateRoute (route: Route) {
>      // 将导航的路由保存到 current
>      this.current = route
>      this.cb && this.cb(route)
>    }
>    ~~~
>
>    在之前`Router`的`init`函数中调用`history.listen`注册了`this.cb`因为`history.listen`的调用实在`tranitionTo`之后所以初始化的第一次导航`this.cb`并不存在，所以这里不会调用。
>
>    ~~~js
>    history.listen(route => {
>          this.apps.forEach((app) => {
>            app._route = route
>          })
>        })
>    ~~~
>
> 3. 如果`transitionTo`中有传`onComplete`函数则调用`onComplete`函数, 在`init`函数中传入了
>
>    ~~~js
>     const setupListeners = () => {
>        history.setupListeners()
>      }
>       // history.setupListeners
>      // 这里主要设置一些滚动行为的监听 用于控制滚动行为，对外接口 scrollBehavior
>     // 还有添加popstate的监听 
>      setupListeners () {
>        if (this.listeners.length > 0) {
>          return
>        }
>    
>        const router = this.router
>        const expectScroll = router.options.scrollBehavior
>        const supportsScroll = supportsPushState && expectScroll
>    
>        if (supportsScroll) {
>          this.listeners.push(setupScroll())
>        }
>    
>        const handleRoutingEvent = () => {
>          const current = this.current
>    
>          // Avoiding first `popstate` event dispatched in some browsers but first
>          // history route not updated since async guard at the same time.
>          const location = getLocation(this.base)
>          if (this.current === START && location === this._startLocation) {
>            return
>          }
>    			// 在浏览器点击前进后退只是切换了路径
>          // 调用transitionTo 更新路由和组件
>          this.transitionTo(location, route => {
>            if (supportsScroll) {
>              handleScroll(router, route, current, true)
>            }
>          })
>        }
>        window.addEventListener('popstate', handleRoutingEvent)
>        this.listeners.push(() => {
>          window.removeEventListener('popstate', handleRoutingEvent)
>        })
>      }
>    ~~~
>
>    4. 调用 `        this.ensureURL()`
>
>       ~~~js
>       // history/html5.js
>       // 
>       ensureURL (push?: boolean) {
>           if (getLocation(this.base) !== this.current.fullPath) {
>             const current = cleanPath(this.base + this.current.fullPath)
>             push ? pushState(current) : replaceState(current)
>           }
>         }
>       ~~~
>
>       

到这里路由的初始化就完成了(第一次导航或者刷新页面)

## router-view组件

在`install`方法内注册的`router-view`组件定义在`src/components/view.js`内

我们看下他的`render`函数, `router-view`定义为函数组件

~~~js
export default {
  name: 'RouterView',
  functional: true,
  props: {
    name: {
      type: String,
      default: 'default'
    }
  },
  render (_, { props, children, parent, data }) {
    // used by devtools to display a router-view badge
    data.routerView = true

    // directly use parent context's createElement() function
    // so that components rendered by router-view can resolve named slots
    const h = parent.$createElement
    const name = props.name
    const route = parent.$route
    const cache = parent._routerViewCache || (parent._routerViewCache = {})

    // determine current view depth, also check to see if the tree
    // has been toggled inactive but kept-alive.
    let depth = 0
    let inactive = false
    while (parent && parent._routerRoot !== parent) {
      const vnodeData = parent.$vnode ? parent.$vnode.data : {}
      if (vnodeData.routerView) {
        // 沿父节点向上遍历计算出当前路由深度
        depth++
      }
      // 是否是keep-alive
      if (vnodeData.keepAlive && parent._directInactive && parent._inactive) {
        inactive = true
      }
      parent = parent.$parent
    }
    data.routerViewDepth = depth

    // render previous view if the tree is inactive and kept-alive
    if (inactive) {
      // 当前渲染的树是keep-alive
      // 取出缓存数据
      const cachedData = cache[name]
      const cachedComponent = cachedData && cachedData.component
      if (cachedComponent) {
        // #2301
        // pass props
        // 填充缓存数据
        if (cachedData.configProps) {
          fillPropsinData(cachedComponent, data, cachedData.route, cachedData.configProps)
        }
        // 渲染组件
        return h(cachedComponent, data, children)
      } else {
        // render previous empty view
        return h()
      }
    }

    const matched = route.matched[depth]
    const component = matched && matched.components[name]

    // render empty node if no matched route or no config component
    if (!matched || !component) {
      cache[name] = null
      return h()
    }

    // cache component
    cache[name] = { component }

    // attach instance registration hook
    // this will be called in the instance's injected lifecycle hooks
    data.registerRouteInstance = (vm, val) => {
      // val could be undefined for unregistration
      // value为空时注销注册
      const current = matched.instances[name]
      if (
        (val && current !== vm) ||
        (!val && current === vm)
      ) {
        matched.instances[name] = val
      }
    }

    // also register instance in prepatch hook
    // in case the same component instance is reused across different routes
    ;(data.hook || (data.hook = {})).prepatch = (_, vnode) => {
      matched.instances[name] = vnode.componentInstance
    }

    // register instance in init hook
    // in case kept-alive component be actived when routes changed
   // 在初始化组件时我会调用 用于注册路由组件实例
    data.hook.init = (vnode) => {
      if (vnode.data.keepAlive &&
        vnode.componentInstance &&
        vnode.componentInstance !== matched.instances[name]
      ) {
        matched.instances[name] = vnode.componentInstance
      }
    }

    const configProps = matched.props && matched.props[name]
    // save route and configProps in cachce
    if (configProps) {
      extend(cache[name], {
        route,
        configProps
      })
      fillPropsinData(component, data, route, configProps)
    }
		// 返回匹配到路由组件的Vnode 
    return h(component, data, children)
  }
}
~~~

还记得`install`方法混入的全局钩子吗？  在执行完`router.init`方法后，在跟实例上定义了定义了一个响应式属性， `_route`, 而在`router-view`组件的`render`函数内访问了`parent.$route`这两者是等价的。

那么我们现在调用跟实例的`$mount`方法， 当创建碰到`router-view`时就会根据`route`这个响应式属性保存的组件名渲染出对应组件。

~~~js
new Vue({
  router,
  render: h => h(App)
}).$mount('#app')
~~~



##  push和replace

理解了路由的首次初始化过程，`push`和`replace`就很好理解了。只不过是重复上面的一部分步骤而已

~~~js

  push (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    this.transitionTo(location, route => {
      pushState(cleanPath(this.base + route.fullPath))
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }

  replace (location: RawLocation, onComplete?: Function, onAbort?: Function) {
    const { current: fromRoute } = this
    this.transitionTo(location, route => {
      replaceState(cleanPath(this.base + route.fullPath))
      handleScroll(this.router, route, fromRoute, false)
      onComplete && onComplete(route)
    }, onAbort)
  }
~~~

可以看到`push`和`replace`都是调用`transitionTo`方法在`onComplete`中调用`window.history`的`api`进行路径的最终切换，路径切换后那么是如何触发组件重新渲染呢？ 

理所当然 新建的路由会生成一个新的`route`对象，更新这个对象就会触发跟实例的重新渲染， 在`confirmTransition`的`onComplete`函数内调用`updateRote`更新路由对象。

~~~js
this.updateRoute(route)
function updateRoute (route: Route) {
    this.current = route
    this.cb && this.cb(route)
}
~~~

在`Router`的`init`方法中调用了`history.listen`初始化了`history.cb`， 可以看到`this.cb`就是遍历跟实例修改`_route`这个响应式属性。

~~~js
 history.listen(route => {
      this.apps.forEach((app) => {
        app._route = route
      })
    })
function listen (cb: Function) {
    this.cb = cb
  }
~~~

<img src="C:\Users\Tencent_Go\Downloads\vue-router.jpg" alt="Alt text" style="zoom:200%;" />

导航的集中场景

1. 手输入地址
2. 浏览器回退前进按钮(调用`history Api`)
3. 点击`api`。
4. 点击`router-link`
5. 调用 `vue-router`的`push`或者`replace`方法



## 其他

### 滚动行为如何实现？

###  重定向如何实现？

### 路由别名如何实现？

### 

