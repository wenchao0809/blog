### 元数据类型

* 如果设置了 name 属性，meta 元素提供的是文档级别（document-level）的元数据，应用于整个页面。
* 如果设置了 http-equiv 属性，meta 元素则是编译指令，提供的信息与类似命名的HTTP头部相同。
* 如果设置了 charset 属性，meta 元素是一个字符集声明，告诉文档使用哪种字符编码。
* 如果设置了 itemprop 属性，meta 元素提供用户定义的元数据。


###  content

此属性包含http-equiv 或name 属性的值，具体取决于所使用的值。

### http-equiv

属性定义了一个编译指示指令。这个属性叫做 http-equiv(alent) 是因为所有允许的值都是特定HTTP头部的名称，如下：

#### content-security-policy

定义页面内容安全策略

#### x-ua-compatible

如果指定，则 content 属性必须具有值 "IE=edge"。用户代理必须忽略此指示。

`X-UA-Compatible` 是自从IE8新加的一个设置，对于`IE8`以下的浏览器是不识别的。
通过在`meta`中设置`X-UA-Compatible`的值，可以指定网页的兼容性模式设置


#### refresh

这个属性指定:
如果 content 只包含一个正整数，则为重新载入页面的时间间隔(秒)；
如果 content 包含一个正整数，并且后面跟着字符串 ';url=' 和一个合法的 URL，则是重定向到指定链接的时间间隔(秒)
### charset

这个属性声明了文档的字符编码。如果使用了这个属性，其值必须是与ASCII大小写无关（ASCII case-insensitive）的"utf-8"。

### name 

`name` 和 `content` 属性可以一起使用，以名-值对的方式给文档提供元数据，其中 `name` 作为元数据的名称，`content` 作为元数据的值。

#### referrer

控制`referrer`头的发送

~~~html
<!-- 不发送referrer头 -->
  <meta name="referrer" content="no-referrer">
~~~

#### viewport

为`viewport`（视口）的初始大小提供指示`（hint）`。目前仅用于移动设备。

~~~html
   <!-- 定义初始宽度为device-width 定义缩放比例为1即不缩放 不允许用户缩放-->
    <meta name="viewport" content="width=device-width,initial-scale=1.0, user-scalable=no">
~~~
### 示例

~~~html
<!DOCTYPE html>
<html>
  <head>
    <meta charset="utf-8">
    <!-- 定义初始宽度为device-width 定义缩放比例为1即不缩放 -->
    <meta name="viewport" content="width=device-width,initial-scale=1.0">
    <!-- 告诉IE浏览器，IE8/9及以后的版本都会以当前浏览器支持的最高版本IE来渲染页面 chrome=1表示可以激活Chrome Frame 参考https://docs.microsoft.com/en-us/openspecs/ie_standards/ms-iedoco/380e2488-f5eb-4457-a07a-0cb1b6e4b4b5 -->
    <meta http-equiv="X-UA-Compatible" content="IE=edge,chrome=1">
    <!-- 将http访问升级为https这个需要服务器支持https访问才行 -->
    <meta http-equiv="Content-Security-Policy" content="upgrade-insecure-requests">
    <title>changsha-portal-manager-frontend</title>
  </head>
  <body>
    <div id="app"></div>
  </body>
</html>
~~~

### 参考

[标准元数据名称](https://developer.mozilla.org/zh-CN/docs/Web/HTML/Element/meta/name)