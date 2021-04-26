## Content-Disposition


### 作为消息主题中的消息头

在常规的 HTTP 应答中，`Content-Disposition` 响应头指示回复的内容该以何种形式展示，是以内联的形式（即网页或者页面的一部分），还是以附件的形式下载并保存到本地。

* `inline` 则浏览器会尝试打开文件并展示 需要添加正确的content-type头， 如果是不支持显示的文件类型则会强制下载比如`.zip`格式等。
* `attachment` 浏览器始终会直接下载文件。

### 作为multipart body中的消息头

在 `multipart/form-data` 类型的应答消息体中，`Content-Disposition` 消息头可以被用在 `multipart` 消息体的子部分中，用来给出其对应字段的相关信息。各个子部分由在`Content-Type` 中定义的分隔符分隔。用在消息体自身则无实际意义。

### 指令

`Content-Disposition` 消息头最初是在 MIME 标准中定义的，HTTP 表单及 POST 请求只用到了其所有参数的一个子集。只有 `form-data` 以及可选的 `name` 和 `filename` 三个参数可以应用在HTTP场景中。

#### name

后面是一个表单字段名的字符串，每一个字段名会对应一个子部分。在同一个字段名对应多个文件的情况下（例如，带有 multiple 属性的 <input type=file> 元素），则多个子部分共用同一个字段名。如果 name 参数的值为 '_charset_' ，意味着这个子部分表示的不是一个 HTML 字段，而是在未明确指定字符集信息的情况下各部分使用的默认字符集

### filename

默认下载的文件名

后面是要传送的文件的初始名称的字符串。这个参数总是可选的，而且不能盲目使用：路径信息必须舍掉，同时要进行一定的转换以符合服务器文件系统规则。这个参数主要用来提供展示性信息。当与 Content-Disposition: attachment 一同使用的时候，它被用作"保存为"对话框中呈现给用户的默认文件名。

### filename*

filename" 和 "filename*" 两个参数的唯一区别在于，"filename*" 采用了  RFC 5987 中规定的编码方式。当 "filename" 和 "filename*" 同时出现的时候，应该优先采用 "filename*"，假如二者都支持的话。

上面大概意思是`filename*`支持中文

## Referer 

Referer 请求头包含了当前请求页面的来源页面的地址，即表示当前页面是通过此来源页面里的链接进入的。服务端一般使用 Referer 请求头识别访问来源，可能会以此进行统计分析、日志记录以及缓存优化等。

需要注意的是 referer 实际上是 "referrer" 误拼写。参见 HTTP referer on Wikipedia （HTTP referer 在维基百科上的条目）来获取更详细的信息。

Referer 请求头可能暴露用户的浏览历史，涉及到用户的隐私问题。

在以下两种情况下，Referer 不会被发送：

* 来源页面采用的协议为表示本地文件的 "file" 或者 "data" URI；
* 当前请求页面采用的是非安全协议，而来源页面采用的是安全协议（HTTPS）

### 语法

当前页面被链接而至的前一页面的绝对路径或者相对路径。不包含 URL fragments (例如 "#section") 和 userinfo (例如 "https://username:password@example.com/foo/bar/" 中的 "username:password"

~~~js
Referer: url

Referer: https://developer.mozilla.org/en-US/docs/Web/JavaScript
~~~

## Location


Location 首部指定的是需要将页面重新定向至的地址。一般在响应码为3xx的响应中才会有意义。

发送新请求，获取Location指向的新页面所采用的方法与初始请求使用的方法以及重定向的类型相关：

303 (See Also) 始终引致请求使用 GET 方法，而，而 307 (Temporary Redirect) 和 308 (Permanent Redirect) 则不转变初始请求中的所使用的方法；
301 (Permanent Redirect) 和 302 (Found) 在大多数情况下不会转变初始请求中的方法，不过一些比较早的用户代理可能会引发方法的变更（所以你基本上不知道这一点）。
状态码为上述之一的所有响应都会带有一个Location首部。

除了重定向响应之外， 状态码为 201 (Created) 的消息也会带有Location首部。它指向的是新创建的资源的地址。

Location 与 Content-Location是不同的，前者（Location ）指定的是一个重定向请求的目的地址（或者新创建的文件的URL），而后者（ Content-Location） 指向的是经过内容协商后的资源的直接地址，不需要进行进一步的内容协商。Location 对应的是响应，而Content-Location对应的是要返回的实


###  语法

相对地址（相对于要访问的URL）或绝对地址。

~~~js
location: /index.html
~~~

## Accept

Accept 首部列举了用户代理希望接收的媒体资源的 MIME 类型。其中不同的 MIME 类型之间用逗号分隔，同时每一种 MIME 类型会配有一个品质因数（quality factor），该参数明确了不同 MIME 类型之间的相对优先级

~~~js
Accept: text/html, application/xhtml+xml, application/xml;q=0.9, */*;q=0.8
~~~

## Accept-CH

这是被称为“客户端示意”（Client Hints）的实验性技术方案的一部分，目前仅在 Chrome 46 及以后的版本中得到了实现

有下列值

>* Device-Memory	标明客户端设备的 RAM 内存大小。该值是个估计值，设备的实际内存值会向2的次方取整，且除以1024。比如512MB的内存对应的值为0.5
>* DPR 标明客户端所在设备的像素比率。
>* Viewport-Width	标明用 CSS 像素数值表示的布局视口（layout viewport）宽度。
>* Width 标明用物理像素值表示的资源宽度（换句话说就是一张图片的固有大小）。
## Accept-Charset

用于告知服务器该客户代理可以理解何种形式的字符编码。

如今 UTF-8 编码已经得到了广泛的支持，成为首选的字符编码类型，

大多数浏览器会将 Accept-Charset 首部移除：Internet Explorer 8、Safari 5、Opera 11 以及 Firefox 10 都已经不再发送该首部。

~~~js
Accept-Charset: utf-8, iso-8859-1;q=0.5
~~~
## Accept-Encoding

可以接受的内容编码形式（所支持的压缩算法）。该首部的值是一个Q因子清单（例如 br, gzip;q=0.8），用来提示不同编码类型值的优先级顺序。默认值 identity 则优

有下列值

>* gzip表示采用 Lempel-Ziv coding (LZ77) 压缩算法，以及32位CRC校验的编码方式。
>* br 表示采用 Brotli 算法的编码方式。
>* identity 用于指代自身（例如：未经过压缩和修改）。除非特别指明，这个标记始终可以被接受。
>* `*` 匹配其他任意未在该请求头字段中列出的编码方式。假如该请求头字段不存在的话，这个值是默认值。它并不代表任意算法都支持，而仅仅表示算法之间无优先次序。

~~~js
Accept-Encoding: gzip

Accept-Encoding: br;q=1.0, gzip;q=0.8, *;q=0.1
~~~

## Accept-Language

用来提示用户期望获得的自然语言的优先顺序。该首部的值是一个Q因子清单（例如 "de, en;q=0.7"）。


~~~js
Accept-Language: fr-CH, fr;q=0.9, en;q=0.8, de;q=0.7, *;q=0.5
~~~

## use-agent

User-Agent 首部包含了一个特征字符串，用来让网络协议的对端来识别发起请求的用户代理软件的应用类型、操作系统、软件开发商以及版本号。

## Vary

Vary 是一个HTTP响应头部信息，它决定了对于未来的一个请求头，应该用一个缓存的回复(response)还是向源服务器请求一个新的回复。它被服务器用来表明在 content negotiation algorithm（内容协商算法）中选择一个资源代表的时候应该使用哪些头部信息（headers）.

在响应状态码为 304 Not Modified  的响应中，也要设置 Vary 首部，而且要与相应的 200 OK 响应设置得一模一样。


### 语法

下面是让客户端基于`Accept-Encoding` 来判断是应该使用缓存还是像服务器发起新的请求，不同`Accept-Encoding` 值会采取不同的策略，比如对于支持压缩和不支持压缩的浏览器， 如果本地缓存的是一个压缩过的文件， 对于支持压缩的浏览器在其他缓存策略命中的情况下会使用缓存，对于不支持压缩的浏览器则会发起一个新的请求，以请求未被压缩过的资源，反之是一样的。

~~~js
Vary: Accept-Encoding
~~~