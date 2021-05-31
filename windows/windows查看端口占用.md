### 查看端口占用进程

~~~shell
netstat -aon|findstr "8080"
~~~

上面命令可以查询出进程 `PID`

### 杀掉进程


查看具体占用的是哪个进程避免误杀

~~~shell
tasklist|findstr "2668"
~~~

结束进程

~~~shell
taskkill /f /t /im 2668
~~~