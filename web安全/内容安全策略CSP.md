## 内容安全策略

### 什么内容安全策略CSP

内容安全策略   (CSP) 是一个额外的安全层，用于检测并削弱某些特定类型的攻击，包括跨站脚本 (XSS) 和数据注入攻击等。无论是数据盗取、网站内容污染还是散发恶意软件，这些攻击都是主要的手段。

使用`CSP`需要配置`Content-Security-Policy` HTTP头部， 对于旧版本可能是`X-Content-Security-Policy`

## CSP解决的问题

### XSS

#### 什么是XSS

`Cross-Site Scripting`为了区分`CSS`而叫`XSS`， 是指攻击者利用前端或后端的输入漏洞，在页面注入攻击脚本， 通过执行非法的脚本达到非法的目的的一种`web`攻击手段。

CSP通过指定有效域——即浏览器认可的可执行脚本的有效来源——使服务器管理者有能力减少或消除XSS攻击所依赖的载体。一个CSP兼容的浏览器将会仅执行从白名单域获取到的脚本文件，忽略所有的其他脚本 (包括内联脚本和HTML的事件处理属性)。

作为一种终极防护形式，始终不允许执行脚本的站点可以选择全面禁止脚本执行

### 数据包嗅探攻击

除限制可以加载内容的域，服务器还可指明哪种协议允许使用；比如 (从理想化的安全角度来说)，服务器可指定所有内容必须通过HTTPS加载。一个完整的数据安全传输策略不仅强制使用HTTPS进行数据传输，也为所有的cookie标记安全标识 `cookies with the secure flag`，并且提供自动的重定向使得HTTP页面导向HTTPS版本。网站也可以使用  `Strict-Transport-Security`  HTTP头部确保连接它的浏览器只使用加密通道。

## 指定CSP策略

一个CSP策略由一系列指令组成， 比如`script-src`用于限制JavaScript的源地址， `img-src` 限制图片和图标的源地址，

指令又分为几类

* 获取指令：Fetch directives 通过获取指令来控制某些可能被加载的确切的资源类型的位置。
* 文档指令 | Document directives 文档指令管理文档属性或者worker环境应用的策略。
* 导航指令 | Navigation directives 导航指令管理用户能打开的链接或者表单可提交的链接
* 报告指令 报告指令控制 CSP 违规的报告过程

完整的指令参考[Content-Security-Policy](https://developer.mozilla.org/zh-CN/docs/Web/HTTP/Headers/Content-Security-Policy#%E6%8A%A5%E5%91%8A%E6%8C%87%E4%BB%A4)

## 示例

### 1

一个网站管理者想要所有内容均来自站点的同一个源 (不包括其子域名)

~~~js
Content-Security-Policy: default-src 'self'
~~~


### 2

一个网站管理者允许内容来自信任的域名及其子域名 (域名不必须与CSP设置所在的域名相同)

~~~js
Content-Security-Policy: default-src 'self' *.trusted.com
~~~

### 3

一个网站管理者允许网页应用的用户在他们自己的内容中包含来自任何源的图片, 但是限制音频或视频需从信任的资源提供者(获得)，所有脚本必须从特定主机服务器获取可信的代码.

~~~js
Content-Security-Policy: default-src 'self'; img-src *; media-src media1.com media2.com; script-src userscripts.example.com
~~~

## 测试策略

CSP支持`report-only`模式在此模式下，所有策略都是可选的，但是任何违规行为将会报告给一个指定的URI地址。所以你需要提供一个`report-uri`指令

~~~js
Content-Security-Policy-Report-Only: default-src 'none'; style-src cdn.example.com; report-uri /_/csp-reports
~~~

上面这个策略不会对对真实用户造成影响，但是如果有违反策略的行为将会以`post`请求的方式提交到`/_/csp-reports`

另外如果同时指定了`Content-Security-Policy`则`Content-Security-Policy-Report-Only`会合并前者的策略。

## 启用违例报告

报告(统计)将会发生的违规行为。通过统计数据去代网站的内容安全政策。观察网站的行为，查看违反报告，然后通过 Content-Security-Policy 头选择所需的政策。

有三种方式启用违例报告，

1. 通过`Content-Security-Policy-Report-Only:` `Content-Security-Policy-Report-Only: default-src https:; report-uri /csp-violation-report-endpoint/`
2. 通过`Content-Security-Policy` `Content-Security-Policy: default-src https:; report-uri /csp-violation-report-endpoint/`
3. 两者共存`Content-Security-Policy: default-src https:;` `Content-Security-Policy-Report-Only: report-uri /csp-violation-report-endpoint` 这样后者会合并前者的策略。

### 报告的数据格式

`document-uri`

发生违规的文档URI。

`referrer`

发生违规的文档referrer。

`blocked-uri`

被内容安全政策阻塞加载的资源的URI。如果被阻塞的URI与文档URI不同源，则被阻塞的URI被截断为只包含`scheme（协议），host（域名），和port（端口）`。

`violated-directive`

被违反的策略名。

`original-policy`

` Content-Security-Policy` HTTP 头部所指定的原始策略。

`disposition`

“执行”或“报告”取决于是使用`Content-Security-Policy` 头还是使用 `Content-Security-Header-Report-Only `头。

### 违例报告示例

~~~js
Content-Security-Policy-Report-Only: default-src 'none'; style-src cdn.example.com; report-uri /_/csp-reports
~~~

上面安全策略规定`style-src`只能从`cdn.example.com`站点加载， 如果存在下面的`HTML`则会提交违例报告

~~~html
<!DOCTYPE html>
<html>
  <head>
    <title>Sign Up</title>
    <link rel="stylesheet" href="css/style.css">
  </head>
  <body>
    ... Content ...
  </body>
</html>
~~~

通过`post` 请求提交的违例报告

~~~js
{
  "csp-report": {
    "document-uri": "http://example.com/signup.html",
    "referrer": "",
    "blocked-uri": "http://example.com/css/style.css",
    "violated-directive": "style-src cdn.example.com",
    "original-policy": "default-src 'none'; style-src cdn.example.com; report-uri /_/csp-reports",
    "disposition": "report"
  }
}
~~~

**!!!注意**

`正如你所看到的，报告在blocked-uri上记录了违反资源的完整路径。这并非总是如此。例如，当 signup.html 试图从 http://anothercdn.example.com/stylesheet.css加载CSS，浏览器不会包含完整路径，只包含来源。这样做是为了防止泄漏跨域资源的敏感信息。`

### 其他指令

#### block-all-mixed-content

当使用HTTPS加载页面时阻止使用HTTP加载任何资源。

#### require-sri-for

需要使用 SRI 作用于页面上的脚本或样式 用于通过服务端提供密文校验资源没有被篡改。

#### upgrade-insecure-requests

让浏览器把一个网站所有的不安全 URL（通过 HTTP 访问）当做已经被安全的 URL 链接（通过 HTTPS 访问）替代。这个指令是为了哪些有量大不安全的传统 URL 需要被重写时候准备的。
### 多内容安全策略


CSP 允许在一个资源中指定多个策略, 包括通过 `Content-Security-Policy` 头, 以及 `Content-Security-Policy-Report-Only` 头，和 `<meta>` 组件。

你可以像以下实例一样多次调用 `Content-Security-Policy` 头。 特别注意这里的 `connect-src` 指令。 尽管第二个策略允许连接, 第一个策略仍然包括了 `connect-src 'none'`。添加了附加的策略后，只会让资源保护的能力更强，也就是说不会有接口可以被允许访问，等同于最严格的策略，`connect-src 'none'` 强制开启。

~~~js
Content-Security-Policy: default-src 'self' http://example.com;
                         connect-src 'none';
Content-Security-Policy: connect-src http://example.com/;
                         script-src http://example.com/
~~~
