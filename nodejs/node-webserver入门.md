## http.Server

### Event

#### Close

当服务器关闭时触发。
 
#### 'request'

每当有请求时触发。 每个连接可能有多个请求（对于 HTTP Keep-Alive 连接而言）。

回调函数会接收到两个参数`request <http.IncomingMessage>`和`response <http.ServerResponse>`对象

#### 'upgrade' 事件

* request <http.IncomingMessage> HTTP 请求的参数，与 'request' 事件中的一样。
* socket <stream.Duplex> 服务器与客户端之间的网络套接字。
* head <Buffer> 升级后的流的第一个数据包（可能为空）。
* 每次客户端请求 HTTP 升级时发出。 监听此事件是可选的，客户端无法坚持更改协议。

触发此事件后，请求的套接字将没有 'data' 事件监听器，这意味着它需要绑定才能处理发送到该套接字上的服务器的数据。

此事件保证传入 <net.Socket> 类（<stream.Duplex> 的子类）的实例，除非用户指定了 <net.Socket> 以外的套接字类型。

## http.createServer([options][, requestListener])

requestListener 是一个函数，会被自动添加到 'request' 事件。



### Method

#### server.listen()

启动 HTTP 服务器用于监听连接。 此方法与 net.Server 中的 server.listen() 相同。

#### server.close([callback])

停止服务器接收新连接

