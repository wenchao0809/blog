### Systemd

Systemd 是 Linux 系统工具，用来启动守护进程，已成为大多数发行版的标准配置。

历史上，Linux 的启动一直采用init进程。

下面的命令用来启动服务。

~~~shell
$ sudo /etc/init.d/apache2 start
# 或者
$ service apache2 start
~~~

这种方法有两个缺点。

一是启动时间长。init进程是串行启动，只有前一个进程启动完，才会启动下一个进程。

二是启动脚本复杂。init进程只是执行启动脚本，不管其他事情。脚本需要自己处理各cd 种情况，这往往使得脚本变得很长。

### systemctl 

systemctl是 Systemd 的主命令，用于管理系统。


~~~shell
# 重启系统
$ sudo systemctl reboot

# 关闭系统，切断电源
$ sudo systemctl poweroff

# CPU停止工作
$ sudo systemctl halt

# 暂停系统
$ sudo systemctl suspend

# 让系统进入冬眠状态
$ sudo systemctl hibernate

# 让系统进入交互式休眠状态
$ sudo systemctl hybrid-sleep

# 启动进入救援状态（单用户状态）
$ sudo systemctl rescue
~~~

### hostnamectl

hostnamectl 命令用于查看当前主机的信息。

~~~shell
# 显示当前主机的信息
$ hostnamectl

# 设置主机名。
$ sudo hostnamectl set-hostname rhel7
~~~

### timedatectl

timedatectl命令用于查看当前时区设置。


~~~shell
# 查看当前时区设置
$ timedatectl

# 显示所有可用的时区
$ timedatectl list-timezones                                                                                   

# 设置当前时区
$ sudo timedatectl set-timezone America/New_York
$ sudo timedatectl set-time YYYY-MM-DD
$ sudo timedatectl set-time HH:MM:SS
~~~

### loginctl

loginctl命令用于查看当前登录的用户。


~~~shell
# 列出当前session
$ loginctl list-sessions

# 列出当前登录用户
$ loginctl list-users

# 列出显示指定用户的信息
$ loginctl show-user ruanyf
~~~

### Unit



### 实例

#### nginx 开机启动

~~~shell
[Unit]
Description=nginx
After=network.target

[Service]
Type=forking
ExecStart=/usr/local/openresty/nginx/sbin/nginx
ExecReload=/usr/local/openresty/nginx/sbin/nginx -s reload
ExecStop=/usr/local/openresty/nginx/sbin/nginx -s quit
PrivateTmp=true

[Install]
WantedBy=multi-user.target
~~~

#### pm2 开机启动


~~~shell
[Unit]
Description=PM2 process manager
Documentation=https://pm2.keymetrics.io/
After=network.target

[Service]
Type=forking
User=gw
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
Environment=PATH=/usr/local/app/nodejs/bin:/bin:/usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin
Environment=PM2_HOME=/home/gw/.pm2
PIDFile=/home/gw/.pm2/pm2.pid

ExecStart=/usr/local/app/nodejs/lib/node_modules/pm2/bin/pm2 resurrect
ExecReload=/usr/local/app/nodejs/lib/node_modules/pm2/bin/pm2 reload all
ExecStop=/usr/local/app/nodejs/lib/node_modules/pm2/bin/pm2 kill

[Install]
WantedBy=multi-user.target
~~~