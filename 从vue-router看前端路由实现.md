# vue

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

### `new VueRouter({ routers })` 做了什么

```javascript
  constructor (options: RouterOptions = {}) {
    this.app = null
    this.apps = []
    this.options = options
    this.beforeHooks = []
    this.resolveHooks = []
    this.afterHooks = []
    this.matcher = createMatcher(options.routes || [], this)

    let mode = options.mode || 'hash'
    this.fallback = mode === 'history' && !supportsPushState && options.fallback !== false
    if (this.fallback) {
      mode = 'hash'
    }
    if (!inBrowser) {
      mode = 'abstract'
    }
    this.mode = mode

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

* `pathList` 数组保存所有`path`集合
* `pathMap` `path` 和 `record`的映射
* `nameMap` `name` 到 `record` 的映射

###  路由别名

