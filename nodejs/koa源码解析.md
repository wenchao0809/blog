## Application

### 构造函数

~~~js
  constructor(options) {
    super();
    options = options || {};
    this.proxy = options.proxy || false;
    this.subdomainOffset = options.subdomainOffset || 2;
    this.proxyIpHeader = options.proxyIpHeader || 'X-Forwarded-For';
    this.maxIpsCount = options.maxIpsCount || 0;
    this.env = options.env || process.env.NODE_ENV || 'development';
    if (options.keys) this.keys = options.keys;
    this.middleware = [];
    this.context = Object.create(context);
    this.request = Object.create(request);
    this.response = Object.create(response);
    // util.inspect.custom support for node 6+
    /* istanbul ignore else */
    if (util.inspect.custom) {
      this[util.inspect.custom] = this.inspect;
    }
  }
~~~

### use

~~~js
use(fn) {
  if (typeof fn !== 'function') throw new TypeError('middleware must be a function!');
  if (isGeneratorFunction(fn)) {
    deprecate('Support for generators will be removed in v3. ' +
              'See the documentation for examples of how to convert old middleware ' +
              'https://github.com/koajs/koa/blob/master/docs/migration.md');
    fn = convert(fn);
  }
  debug('use %s', fn._name || fn.name || '-');
  this.middleware.push(fn);
  return this;
}
~~~

`use`方法首先判断中间件是否是生成器如果是则打印警告信息，并且调用`convert`转为`async await`最后`push`进`middleware`数组。
### listen

~~~js
  listen(...args) {
    debug('listen');
    const server = http.createServer(this.callback());
    return server.listen(...args);
  }

callback() {
    // componse组装中间件
    const fn = compose(this.middleware);

    if (!this.listenerCount('error')) this.on('error', this.onerror);

    const handleRequest = (req, res) => {
      // 每次请求都创建一个新的上下文
      const ctx = this.createContext(req, res);
      return this.handleRequest(ctx, fn);
    };

    return handleRequest;
  }

  /**
   * Handle request in callback.
   *
   * @api private
   */

  handleRequest(ctx, fnMiddleware) {
    const res = ctx.res;
    // 默认状态码404
    res.statusCode = 404;
    // 处理错误
    const onerror = err => ctx.onerror(err);
    // 处理响应
    const handleResponse = () => respond(ctx);
    onFinished(res, onerror);
    // 执行完中间件后没有错误调用handleResponse
    return fnMiddleware(ctx).then(handleResponse).catch(onerror);
  }
~~~

`lsiten`调用`http.createServer` 创建`server`传入`callBack`

这里`request`事件真实的回调函数实际上在`callback`进一步包装后返回的

### respond 处理响应

~~~js
function respond(ctx) {
  // allow bypassing koa
  if (false === ctx.respond) return;

  if (!ctx.writable) return;

  const res = ctx.res;
  let body = ctx.body;
  const code = ctx.status;

  // ignore body
  if (statuses.empty[code]) {
    // strip headers
    ctx.body = null;
    return res.end();
  }

  if ('HEAD' === ctx.method) {
    if (!res.headersSent && !ctx.response.has('Content-Length')) {
      const { length } = ctx.response;
      if (Number.isInteger(length)) ctx.length = length;
    }
    return res.end();
  }

  // status body
  if (null == body) {
    if (ctx.response._explicitNullBody) {
      ctx.response.remove('Content-Type');
      ctx.response.remove('Transfer-Encoding');
      return res.end();
    }
    if (ctx.req.httpVersionMajor >= 2) {
      body = String(code);
    } else {
      body = ctx.message || String(code);
    }
    if (!res.headersSent) {
      ctx.type = 'text';
      ctx.length = Buffer.byteLength(body);
    }
    return res.end(body);
  }

  // responses
  // 二进制数据
  if (Buffer.isBuffer(body)) return res.end(body);
  // 字符串
  if ('string' === typeof body) return res.end(body);
  // 流
  if (body instanceof Stream) return body.pipe(res);

  // body: json
  // json
  body = JSON.stringify(body);
  if (!res.headersSent) {
    ctx.length = Buffer.byteLength(body);
  }
  res.end(body);
}
~~~

当我们执行`ctx.body = xxx`之后， `xxx`的格式支持一下几种

> * 二进制， 直接返回二进制数据
> * 字符串， 直接返回字符串
> * 文件流， `pipe`，因为`res`本身也是一个可写流。
>* `json` 返回json


## Context

### 创建上下文

~~~js
 createContext(req, res) {
    // this.context 定义的原始上下文， 以它为原型创建一个新对象
    const context = Object.create(this.context);
    const request = context.request = Object.create(this.request);
    const response = context.response = Object.create(this.response);
    context.app = request.app = response.app = this;
    context.req = request.req = response.req = req;
    context.res = request.res = response.res = res;
    request.ctx = response.ctx = context;
    request.response = response;
    response.request = request;
    context.originalUrl = request.originalUrl = req.url;
    context.state = {};
    return context;
  }
~~~

### throw

抛出一个错误

~~~js
ctx.throw('erro')
ctx.throw(400, 'Bad Request')
~~~

### assert

提供一个断言试的抛出错误， 类似`ctx.throw`

~~~js
ctx.assert(ctx.state.user, 401, 'un auth')
// 等价
if(!ctx.state.user) {
  ctx.throw(401, 'un auth')
}
~~~

### onerror

处理 throw抛出的错误

~~~js
  onerror(err) {
    // don't do anything if there is no error.
    // this allows you to pass `this.onerror`
    // to node-style callbacks.
    if (null == err) return;

    // When dealing with cross-globals a normal `instanceof` check doesn't work properly.
    // See https://github.com/koajs/koa/issues/1466
    // We can probably remove it once jest fixes https://github.com/facebook/jest/issues/2549.
    const isNativeError =
      Object.prototype.toString.call(err) === '[object Error]' ||
      err instanceof Error;
    if (!isNativeError) err = new Error(util.format('non-error thrown: %j', err));

    let headerSent = false;
    if (this.headerSent || !this.writable) {
      headerSent = err.headerSent = true;
    }

    // delegate
    this.app.emit('error', err, this);

    // nothing we can do here other
    // than delegate to the app-level
    // handler and log.
    if (headerSent) {
      return;
    }

    const { res } = this;

    // first unset all headers
    /* istanbul ignore else */
    if (typeof res.getHeaderNames === 'function') {
      res.getHeaderNames().forEach(name => res.removeHeader(name));
    } else {
      res._headers = {}; // Node < 7.7
    }

    // then set those specified
    this.set(err.headers);

    // force text/plain
    this.type = 'text';

    let statusCode = err.status || err.statusCode;

    // ENOENT support
    if ('ENOENT' === err.code) statusCode = 404;

    // default to 500
    if ('number' !== typeof statusCode || !statuses[statusCode]) statusCode = 500;

    // respond
    const code = statuses[statusCode];
    const msg = err.expose ? err.message : code;
    this.status = err.status = statusCode;
    this.length = Buffer.byteLength(msg);
    res.end(msg);
  },
~~~

默认500错误， 使用`http-errors`根据状态码创建错误

## response

这些方法和属性都被`ctx代理`
### redirect

重定向  默认会修改状态码未 `302`, 你也可以自己手动设置其他重定向状态码

~~~js
redirect(url, alt) {
    // location
    if ('back' === url) url = this.ctx.get('Referrer') || alt || '/';
    this.set('Location', encodeUrl(url));

    // status
    if (!statuses.redirect[this.status]) this.status = 302;

    // html
    if (this.ctx.accepts('html')) {
      // 状态码设为300时会使用body
      url = escape(url);
      this.type = 'text/html; charset=utf-8';
      this.body = `Redirecting to <a href="${url}">${url}</a>.`;
      return;
    }

    // text
    this.type = 'text/plain; charset=utf-8';
    this.body = `Redirecting to ${url}.`;
  },
~~~


~~~js
ctx.redirect('/test')

ctx.status = 301
ctx.redirect('/test')

ctx.status = 307
ctx.redirect('/test')
~~~

### remove

移除未发送的响应头

~~~js
ctx.remove('Content-Encoding')
~~~

### vary

设置`Vary`头

~~~js
ctx.vary('Content-Encoding')
~~~

### has

判断`headers`中是否包含指定头

~~~js
ctx.has('Content-Enoding')
~~~

## set 

设置响应头

~~~js
  ctx.set('Foo', ['bar', 'baz']);
  ctx.set('Accept', 'application/json');
  ctx.set({ Accept: 'text/plain', 'X-API-Key': 'tobi' });
~~~

## append

追加响应头， 如果响应头还未被设置，等价于`set`

~~~js
ctx.append('Link', ['<http://localhost/>', '<http://localhost:3000/>']);
ctx.append('Set-Cookie', 'foo=bar; Path=/; HttpOnly');
ctx.append('Warning', '199 Miscellaneous warning');
~~~

### flushHeaders

刷新响应头， 调用此方法后`ctx.headerSent`将为`true`此后再设置响应头不会有效果，

~~~js
ctx.set('Set-Cookie', 'foo=bar') // 有效
console.log('header sent', ctx.headerSent) // false
ctx.flushHeaders()
ctx.set('Set-Cookie', 'foo=bar') // 无效
console.log('header sent', ctx.headerSent)// true
~~~

这里的`flushHeaders`会调用`http.ServerResponse`的`flushHeaders`方法，这又相当于调用了`writeHead()`方法，官方文档是这样解释的

`writeHead`方法必须被调用而且只能调用一次， 必须在`res.end`和`res.write`之前调用，如果在没有调用`flushHeaders` 调用了`res.write`和`res.end`
则`res.end`会默认调用`res.flushHeaders`方法。

另外`res.setHeader()`并不会立即写入响应头，而是会在内部缓存，调用了`flushHeaders` 就会立即将响应头写入网络通道，之后就无法在设置响应头了。

## staus

获取和设置状态的响应码

~~~js
const status = ctx.status // 相当于ctx.res.status
ctx.status = 200 // 相当于 ctx.res.status = 200 
~~~

## message

获取或设置 `response.statusMessage`

~~~js
const message = status.message
status.message = 'ok'
~~~

当使用隐式的响应头时（没有显式地调用 response.writeHead()），此属性控制在刷新响应头时将发送到客户端的状态消息。 如果保留为 undefined，则将使用状态码的标准消息。

## body

获取或设置响应体，支持 `string|Buffer|Object|Stream`

~~~js
ctx.body = 'ok'
ctx.body = new Buffer([1, 2, 3])
ctx.body = { a: 1, b: 2 }
ctx.body = fsPromise.readFile(path.resolve(__dirname, '.', 'AerialTamul_ZH-CN3164679201_1920x1080.jpg')) // stream
~~~

## length

获取 `content-length`

~~~js
const length = ctx.length
~~~

## type

设置或移除`content-type`

~~~js
ctx.type = 'json' // 设置位 appliaction/json
ctx.type = '' // 移除content-type
~~~

## **lastModified**

获取或设置 `last-modified`

~~~js
const last = ctx.lastModified // 获取
ctx.lastModified = '2021-09-08 21:00:00' // 设置
~~~

## etag

获取或设置 `ETag`

~~~js
const etag = ctx.etag 
ctx.etag = 'W/"1b91c78e261444b204d6c9f0349d9a56"'
~~~

## headerSent

获取 `response.headerSent`

~~~js
const headerSent = ctx.headerSent
~~~

如果这个属性为`true`说明
## writable

获取 `writable`此标志基于`response.writableEnded`, 为`false`则说明不能继续再向响应写入数据了。

调用`res.end()`会将此标识置为`true`

## request

同 `respone`对象这些方法和属性也被`context`代理


### accepts

检查指定类型是否是客户端可接受的根据请求的`accept-type`判断, 不可接受返回`false`， 可接受返回可接受的类型

~~~js
ctx.accepts('html') // html
ctx.accepts(['html', 'json']) // json 如果   Accept: text/*;q=.5, application/json

  // Accept: text/*, application/json
ctx.accepts('image/png');
ctx.accepts('png'); // => false
~~~
### acceptsLanguages

获取可接受的语言根据`Accept-Language`

~~~js
ctx.acceptsLanguages("en") // false
ctx.acceptsLanguages(['es', 'pt', 'en']) // ['es', 'en']
~~~
### acceptsEncodings

获取最适合的压缩算法根据`Accept-Encoding`

~~~js
ctx.acceptsEncodings(['gzip', 'deflate', 'br']) // ['gzip', 'br']
~~~
### acceptsCharsets

获取最适合的字符编码， 根据`Accept-Charset`

~~~js
ctx.acceptsCharsets( ['utf-8', 'utf-7', 'iso-8859-1'])
~~~

### url

获取或设置请求路径

~~~js
const url = ctx.url // 内部使用request.url
ctx.url = '/test'
~~~

### header headers

而知等价都是 获取或设置请求的`headers`

~~~js
const header  = ctx.header()
ctx.header = 'Accept: application/json' //  内部使用 ctx.req.headers = 'Accept: application/json'
~~~

### origin

获取请求站点

~~~js
const origin = ctx.origin // 请求的 协议+ host
~~~
### href

获取完整路径

~~~js
const href = ctx.href
~~~

### method

获取或设置请求方法

~~~js
const method = ctx.method // request.method
ctx.method = 'GET'
~~~

### path

### query

获取query对象或设置query通过对象

~~~js
const query = ctx.query()
ctx.query = { a: 1, b: 2} // a=1&b=2
~~~

### querystring

获取query字符串， 或设置query字符串

~~~js
const queryString = ctx.queryString
ctx.queryString = 'a=1&b=2'
~~~

### search

获取url `search`

### URL

获取 使用 `URL`创建的对象， 参加`nodej` [WHATWG-URL](http://nodejs.cn/api/url.html#url_url)

### fresh

获取请求是否新鲜， `true`新鲜`false`不新鲜， 根据`If-Modified-Since`  和`If-None-Match`分别与` Last-Modified` 和`ETag`对比其中`If-None-Match` 优先级更高遵循`HTTP`协议

~~~js
const fresh = ctx.fresh()
~~~


###  stale

检查请求是否不新鲜， 实际上就是fresh取反`!fresh`

### sock

获取请求`socket`

### idempotent

获取请求方法是否幂等

#### 术语 幂等

一个HTTP方法是幂等的，指的是同样的请求被执行一次与连续执行多次的效果是一样的，服务器的状态也是一样的。换句话说就是，幂等方法不应该具有副作用（统计用途除外）。在正确实现的条件下， GET ， HEAD ， PUT 和 DELETE  等方法都是幂等的，而  POST  方法不是。所有的 safe 方法也都是幂等的。

幂等性只与后端服务器的实际状态有关，而每一次请求接收到的状态码不一定相同。例如，第一次调用 DELETE 方法有可能返回 200 ，但是后续的请求可能会返回 404 。 DELETE 的言外之意是，开发者不应该使用 DELETE 法实现具有删除最后条目功能的 RESTful API。

需要注意的是，服务器不一定会确保请求方法的幂等性，有些应用可能会错误地打破幂等性约束。

GET /pageX HTTP/1.1 幂等的。连续调用多次，客户端接收到的结果都是一样的：


### proto

获取请求使用协议


### secure

如果当前协议是`https`返回true


~~~js
const secure = ctx.secure
~~~

### type

获取请求头中`Content-Type`
### get

根据`key`获取请求头

### is
检查传入请求是否包含 Content-Type 消息头字段， 并且包含任意的 mime type。 如果没有请求主体，返回 null。 如果没有内容类型，或者匹配失败，则返回 false。 反之则返回匹配的 content-type。


### subdomains

返回子域名数组

### ip

获取ip, 或设置ip


### ips

当 X-Forwarded-For 存在并且 app.proxy 被启用时，这些 ips 的数组被返回，从上游 - >下游排序。 禁用时返回一个空数组。

例如，如果值是 "client, proxy1, proxy2"，将会得到数组 ["client", "proxy1", "proxy2"]。

大多数反向代理（nginx）都通过 proxy_add_x_forwarded_for 设置了 x-forwarded-for，这带来了一定的安全风险。恶意攻击者可以通过伪造 X-Forwarded-For 请求头来伪造客户端的ip地址。 客户端发送的请求具有 'forged' 的 X-Forwarded-For 请求头。 在由反向代理转发之后，request.ips 将是 ['forged', 'client', 'proxy1', 'proxy2']。

Koa 提供了两种方式来避免被绕过。

如果您可以控制反向代理，则可以通过调整配置来避免绕过，或者使用 koa 提供的 app.proxyIpHeader 来避免读取 x-forwarded-for 获取 ips。

~~~js
const app = new Koa({
  proxy: true,
  proxyIpHeader: 'X-Real-IP',
});
~~~
如果您确切知道服务器前面有多少个反向代理，则可以通过配置 app.maxIpsCount 来避免读取用户的伪造的请求头：

~~~js
const app = new Koa({
  proxy: true,
  maxIpsCount: 1, // 服务器前只有一个代理
});

~~~
