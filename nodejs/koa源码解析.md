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

## req

## res