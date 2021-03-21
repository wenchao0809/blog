## 环境准备

首先参考(Creating a Base Box)[https://www.vagrantup.com/docs/boxes/base] 创建一台虚拟机，为了节省工作量我们把基本的环境配置和软件安装全部打包进`Box`

### 禁用`swap`分区

K8s的要求，在每个宿主机上执行：
~~~sh
sudo swapoff -a
# 永久关闭 关闭到swap那行
sudo vi /etc/fstab
~~~

这个我看很多教程都说要关闭，但是在官方教程没看到，先关了吧

### 确保时区时间正确。

每个宿主机上都要确保时区和时间是正确的。

如果时区不正确，请使用下面的命令来修改：
~~~sh
sudo timedatectl set-timezone Asia/Shanghai
sudo systemctl restart rsyslog 
~~~

### 允许 iptables 检查桥接流量 
确保 `br_netfilter` 模块被加载。这一操作可以通过运行 `lsmod | grep br_netfilter` 来完成。若要显式加载该模块，可执行 `sudo modprobe br_netfilter`。

为了让你的 Linux 节点上的 iptables 能够正确地查看桥接流量，你需要确保在你的 sysctl 配置中将 `net.bridge.bridge-nf-call-iptables` 设置为 1。例如：

在`Ubuntu 20.04 Server`上，这个值就是1。如果你的系统上不一致，使用下面的命令来修改：

~~~sh
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
sudo sysctl --system
~~~
### 设置rp_filter的值

因为`Gemfield`的`K8s`集群即将部署的是`calico`网络插件，而`calico`需要这个内核参数是0或者1，但是`Ubuntu20.04`上默认是`2`，这样就会导致`calico`插件报下面的错误（这是个fatal级别的error）：
```
int_dataplane.go 1035: Kernel's RPF check is set to 'loose'.  \
This would allow endpoints to spoof their IP address.  \
Calico requires net.ipv4.conf.all.rp_filter to be set to 0 or 1. \
If you require loose RPF and you are not concerned about spoofing, \
this check can be disabled by setting the IgnoreLooseRPF configuration parameter to 'true'.
```
使用下面的命令来修改这个参数的值：
~~~sh
#修改/etc/sysctl.d/10-network-security.conf
sudo vi /etc/sysctl.d/10-network-security.conf

#将下面两个参数的值从2修改为1
#net.ipv4.conf.default.rp_filter=1
#net.ipv4.conf.all.rp_filter=1

#然后使之生效
sudo sysctl --system
~~~

### 安装`docker`

~~~sh
sudo apt-get remove docker docker-engine docker.io containerd runc
sudo apt-get update
sudo apt-get install \
    apt-transport-https \
    ca-certificates \
    curl \
    gnupg-agent \
    software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository \
   "deb [arch=amd64] https://download.docker.com/linux/ubuntu \
   $(lsb_release -cs) \
   stable"
sudo apt-get update
sudo apt-get install docker-ce docker-ce-cli containerd.io
sudo systemctl start docker
sudo systemctl enable docker
sudo systemctl status docker
~~~

### 安装 kubeadm、kubelet 和 kubectl 

直接用`google`源需要配置`apt`的代理

~~~sh
sudo apt-get update && sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
~~~

用`阿里`的源

~~~sh
sudo apt-get update && sudo apt-get install -y ca-certificates curl software-properties-common apt-transport-https curl
curl -s https://mirrors.aliyun.com/kubernetes/apt/doc/apt-key.gpg | sudo apt-key add -
sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF 
sudo tee /etc/apt/sources.list.d/kubernetes.list <<EOF 
deb https://mirrors.aliyun.com/kubernetes/apt/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
~~~

以上基础环境配置好后， 用一下命令打包成`box`

~~~sh
vagrant package --base ubuntu20.04 -o k8s-base-ubuntu2004
~~~

## 初始化 master

### Vagrantfile
~~~ruby
Vagrant.configure("2") do |config|
  config.ssh.insert_key = false
  # config.ssh.private_key_path = 'id_rsa'
  nodenumber=1
  config.ssh.username = "vagrant"
  config.ssh.password = "vagrant"
  config.vm.define "k8s-master" do | k8smaster |
    k8smaster.vm.box = "k8s-base-ubuntu2004"
    k8smaster.vm.hostname = "k8s-master"
    k8smaster.vm.network "private_network", ip: "192.168.10.100"
    k8smaster.vm.provider "virtualbox" do | v |
      v.name = "k8s-master"
      v.memory = "2048"
      v.cpus = 2
    end
  end
  (1..nodenumber).each do | i |
    config.vm.define "k8s-node-#{i}" do | node |
      node.vm.box = "k8s-base-ubuntu2004"
      node.vm.hostname = "k8s-node-#{i}"
      node.vm.network "private_network", ip: "192.168.10.#{100+i}"
      node.vm.provider "virtualbox" do | v |
        v.name = "k8s-node-#{i}"
        v.memory = "1024"
        v.cpus = 1
      end
    end
  end
end
~~~

### 初始化 master

~~~sh
sudo kubeadm init --apiserver-advertise-address 192.168.10.100 --pod-network-cidr 172.16.0.0/16 --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers
~~~

非`root`用户执行`kubectl`命令需要

~~~sh
mkdir -p $HOME/.kube
sudo cp -i mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
sudo chmod 666 /etc/kubernetes/admin.conf
~~~

### 安装`calico`网络插件

~~~sh
wget  https://docs.projectcalico.org/v3.18/manifests/calico.yaml
~~~