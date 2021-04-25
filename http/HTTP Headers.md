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