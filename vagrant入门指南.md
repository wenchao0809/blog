# 前言

我工作机最近迁移到`windows`之前是`mac`众所周知在`windows`下配置各种开发环境是相当的蛋疼总会有各种莫名奇妙的问题， 另外`windows`的终端也十分难用连最基本的`alias`都难以满足我的需求，在使用`Vagrant`我尝试使用`WSL`但是用了一段时间感觉有以下两个明显的缺点。
1. 不支持`Systemd`。
2. 性能低 构建一个一般的前端项目，本机构建只需十几秒，`WSL`居然跑了一分钟左右。
`Vagrant`又是什么呢？
`Vagrant` 是个用于管理虚拟机生命周期的命令行实用程序， 通过`Vagrant`约定的配置我们可以方便的`创建`、`启动`、`关闭`、`销毁`一个虚拟机，`Vagrant`也支持集群我们可以通过一个配置文件定义多台机器，除了以上`Vagrant`最大的优点是可以方便的做分发，我们可以把已经配置好开发环境的机器打包成一个`box`这样你在何时何地都可以通过这个`box`完美的还原你的开发环境想想就爽。
****
# 创建一个基础的`Box`
****
在`Vagrant`的[Box网站](https://app.vagrantup.com/boxes/search)我们可以找到别人已经打包好并上传的`Box`, 当别人打包的`Box`比如`Ubuntu`官方`Box`通常都会指定`10G`硬盘这可能不太够，此时我们可能就需要自己动手打包一个自己的`Box`.

## 什么是基础`Box`?

一个基础的`box`通常只包含能让`Vagrant`正常工作的一些软件，对于`linux`的`box`通常如下

* 包管理器 比如`ubuntu`的`apt`
* SSH
* 一个可以让`Vagrant`连接的用户。

初次之外对于特定`provider`可能还需要安装一些额外的包。

`provider`就是`Vagrant`支持的介质类型，`Vagrant`支持一下虚拟介质

* `VirtualBox`
* `VMware`
* `Docker`
* `Hyper-V`

### 磁盘空间

在创建`虚拟机`时我们必须指定足够我们使用的的磁盘空间，因为这在`Vagrant`里时不可以配置的，为了节省磁盘空间你可以指定虚拟机磁盘为动态分配，需要相应的介质支持才行。

### 内存

这个我们可以随意指定，因为这是可配置的

### 声音、 USB等

禁用 以节省打包空间和时间

### 默认的用户设定

这里可以自定义，如果自定义需要在配置文件内指定用户名和密码， 另外如果你想公开你的`box`给任何人使用 `Vagrant`推荐默认的用户名和密码都是`vagrant`, 另外`Vagrant`默认的登录方式为`private key`你需要把[ insecure keypair](https://github.com/hashicorp/vagrant/tree/master/keys)放入到你虚拟机`~/.ssh/authorized_keys`.

#### `vagrant`的ssh连接方式

这里顺便说一下`vagrant`ssh连接的授权方式
1. 用户名密码， 在`Vagrantfile`中直接指定用户名密码， 如果安全性要求不高直接用这种就行。
  
~~~ruby
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
~~~
2. `private key` 除了上面说的放置一个`vagrant`提供的公钥`，你也可以自己生成一堆`RSA`公钥和私钥，把公钥放到服务器上， 私钥放在本地在配置文件里通过一下方式指定

~~~ruby
  config.ssh.insert_key = false
  # 路径
  config.ssh.private_key_path = 'id_rsa'
~~~

# 配置默认用户`Sudo`不需要输入密码

`vagrant`唤醒虚拟机后通过`ssh`连接到虚拟机然后会一些需要`root`权限的操作比如

* 配置网络
* 挂在共享文件
* 安装软件
* 等等

因此我们要配置默认的用户， 通过`sudo` 获取`root`权限时不需要输入密码，
# 问题

今天遇到一个坑爹的问题， `ubuntu`的文件系统莫名奇妙的编程只读了， 导致`Vagrant`报错 

~~~
await_response_state': scp: /tmp/vagrant-network-entry-1596804873: Read-only file system
~~~

需要登录到虚拟机
* sudo su 切换root用户
* mount | grep ro找到只读设备
* cat /etc/fstab 
* mount -o remount,rw --uuid 你的uuid /
* 如果失败 参考 [https://blog.csdn.net/lilywri823/article/details/86607247](https://blog.csdn.net/lilywri823/article/details/86607247)

* `vagrant` `k8s`参考 [搭建实验环境](https://kfs.ooclab.com/kfs/v1.14.1/vagrant/)