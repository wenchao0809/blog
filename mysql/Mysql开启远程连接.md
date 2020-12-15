CentOS系统安装好MySQL后，默认情况下不支持用户通过非本机连接上数据库服务器，下面是解决方法：

1、在控制台执行
~~~bash
mysql -uroot -p
~~~
系统提示输入数据库root用户的密码，输入完成后即进入mysql控制台

2、选择数据库
~~~bash
use mysql;
~~~

开启远程连接
root 用户名
% 所有人都可以访问
password 密码

~~~mysql
GRANT ALL PRIVILEGES ON *.* TO 'root'@'%' IDENTIFIED BY 'password' WITH GRANT OPTION;

FLUSH PRIVILEGES; 

3、重起mysql服务

service mysqld restart
~~~

如果执行完以上步骤，还是不能远程连接，那么我们需要查看服务器的防火墙是否开启


```bash
service iptables status
```
如果防火墙开启，请关闭

```bash
service iptables stop
```

到此就可以远程连接了！