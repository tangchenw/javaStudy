# 容器进阶知识

本节将讨论容器的进阶知识，包括管理多个docker host、跨主机的容器网络、监控、日志管理、数据管理等高级主题。

- multi-host
- 容器网络
- 监控
- 数据管理
- 日志管理

这些是将容器技术真正应用到生产环境必须要考虑的主题，我们将分别详细讨论。

## 1、多主机管理

之前我们的实验环境中只有一个docker host，所有的容器都是运行在这一个host上的。但在真正的环境中会有多个host，容器在这些host中启动、运行、停止和销毁，相关容器会通过网络相互通信，无论它们是否位于相同的host。

对于这样一个multi-host环境，我们将如何高效地进行管理呢？

我们面临的第一个问题是：为所有的host安装和配置docker。

在前面我们手工安装了第一个docker host，步骤包括：

1. 安装https CA证书
2. 添加GPG key
3. 添加docker apt源。
4. 安装docker。

可见步骤还是挺多的，对于多主机环境手工方式效率低且不容易保证一致性针对这个问题，docker给出的解决方案是docker machine。

用docker machine可以批量安装和配置docker host，这个host可以是本地的虚拟机、物理机，也可以是公有云中的云主机。

docker machine支持在不同的环境下安装配置docker host，包括：

1. 常规的linux操作系统。
2. 虚拟化平台——virtualBox、VMWare、Hype-V3.Openstack。
3. 公有云——Amazon Web Services、Microsoft Azure、Google Compute Engine、Ditital Ocean等。

Docker Machine为这些环境起了一个统一的名字：provider。对于某个特定的provider，docker machine使用相应的driver安装和配置docker host。

### 1.1、实验环境描述

实验环境有三个运行的ubuntu的host：192.168.56.101、192.168.56.104、192.168.56.105

我们将在192.168.56.101上安装docker machine，然后通过docker-machine命令在其它两个host上部署docker。

### 1.2、 安装docker machine

官方安装文档在https://docs.docker.com/machine/install-machine/

安装方法很简单，执行如下命令：

```shell
$ base=https://github.com/docker/machine/releases/download/v0.16.0 &&
  curl -L $base/docker-machine-$(uname -s)-$(uname -m) >/tmp/docker-machine &&
  sudo install /tmp/docker-machine /usr/local/bin/docker-machine
```

下载的文件被放到/usr/local/bin中，执行docker-mahine version验证命令是否可用。

```dockerfile
docker mahine version
```

注：当你看到这篇文章的时候，Docker Machine 应该有了更新的版本，可参考官方文档进行安装。

为了得到更好的体验，我们可以安装 bash completion script，这样在 bash 能够通过 `tab` 键补全 `docker-mahine` 的子命令和参数。安装方法是从https://github.com/docker/machine/tree/master/contrib/completion/bash下载 completion script：

将其放置到 `/etc/bash_completion.d` 目录下。然后将如下代码添加到`$HOME/.bashrc`

PS1='[\u@\h \W$(__docker_machine_ps1)]\$ '

其作用是设置 docker-machine 的命令行提示符，不过要等到部署完其他两个 host 才能看出效果。

### 1.3、 创建Machine

对于 Docker Machine 来说，术语 `Machine` 就是运行 docker daemon 的主机。“创建 Machine” 指的就是在 host 上安装和部署 docker。先执行 `docker-machine ls` 查看一下当前的 machine：

```dockerfile
docker-machine ls
```

如我们所料，当前还没有 machine，接下来我们创建第一个 machine： host1 - 192.168.56.104。

创建 machine 要求能够无密码登录远程主机，所以需要先通过如下命令将 ssh key 拷贝到192.168.56.104:

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604081925638-1799651926.png)

一切准备就绪，执行 `docker-machine create` 命令创建 host1：

```ABAP
docker-machine create --driver generic --generic-ip-address=192.168.56.104 host1
```

因为我们是往普通的 Linux 中部署 docker，所以使用 `generic` driver，其他 driver 可以参考文档 https://docs.docker.com/machine/drivers/。

`--generic-ip-address` 指定目标系统的 IP，并命名为 `host1`。命令执行过程如下：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604085558996-1208728727.png)

> 被安装docker主机必须能连外网

1. 通过 ssh 登录到远程主机。
2. 安装 docker。
3. 拷贝证书。
4. 配置 docker daemon。
5. 启动 docker。

再次执行 `docker-machine ls` ：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604085808550-333394377.png)

已经能看到 host1 了。 我们可以登录到 host1 查看 docker daemon 的具体配置 /etc/systemd/system/docker.service。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604090115600-407700848.png)

1. `-H tcp://0.0.0.0:2376` 使 docker daemon 接受远程连接。
2. `--tls*` 对远程连接启用安全认证和加密。

同时我们也看到 hostname 已经设置为 `host1`：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604090252168-658779263.png)

使用同样的方法创建 host2：

```ABAP
docker-machine create --driver generic --generic-ip-address=192.168.0.45 host2
```

创建成功后 `docker-machine ls` 可以看到 host1 和 host2 都已经就绪：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604090658385-1792931980.png)

### 1.4、 管理Machine

用 `docker-machine` [创建 machine](http://mp.weixin.qq.com/s?__biz=MzIwMTM5MjUwMg==&mid=2653587742&idx=1&sn=8404889612dbca8e759d4697de69fafe&chksm=8d308107ba470811a761e96032786f1da8fdd5248d91a3b6eca0b1faf6ac05b0b16a8a84fe3e&scene=21#wechat_redirect) 的过程很简洁，非常适合多主机环境。除此之外，Docker Machine 也提供了一些子命令方便对 machine 进行管理。其中最常用的就是无需登录到 machine 就能执行 docker 相关操作。

我们前面学过，要执行远程 docker 命令我们需要通过 `-H` 指定目标主机的连接字符串，比如：

```dockerfile
docker -H tcp://192.168.56.105:2376 ps
```

Docker Machine 则让这个过程更简单。`docker-machine env host1`显示访问 host1 需要的所有环境变量：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604092436672-183306598.png)

根据提示，执行 `eval $(docker-machine env host1)`：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604100105901-310803775.png)

然后，就可以看到命令行提示符已经变了，其原因是我们之前在`$HOME/.bashrc` 中配置了 `PS1='[\u@\h \W$(__docker_machine_ps1)]\$ '`，用于显示当前 docker host。

在此状态下执行的所有 docker 命令其效果都相当于在 host1 上执行，例如启动一个 busybox 容器：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604100328828-49210897.png)

执行 `eval $(docker-machine env host2)` 切换到 host2：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604100437315-211428305.png)

下面再介绍几个有用的 docker-machine 子命令：

`docker-machine upgrade` 更新 machine 的 docker 到最新版本，可以批量执行：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604100657400-1073426905.png)

`docker-machine config` 查看 machine 的 docker daemon 配置：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604100805042-1529657396.png)

`stop/start/restart` 是对 machine 的操作系统操作，而 **不是** stop/start/restart docker daemon。

`docker-machine scp` 可以在不同 machine 之间拷贝文件，比如：

docker-machine scp host1:/tmp/a host2:/tmp/b

可见，在多主机环境下 Docker Machine 可以大大提高效率，而且操作也很简单，希望大家都能掌握。

## 2、容器网络

前面已经学习了docker的几种网络方案：none、host、bridge和joined容器，他们解决了单个docker host内容器通信的问题。

本节重点讨论跨主机容器之间通信的方案：

1. 容器网络
   1. docker network
      1. none
      2. host
      3. bridge
      4. joined container
      5. overlay
      6. macvlan
   2. flannel
   3. weave
   4. calico

跨主机网络方案包括：

1. docker 原生的 overlay 和 macvlan。
2. 第三方方案：常用的包括 flannel、weave 和 calico。

docker 网络是一个非常活跃的技术领域，不断有新的方案开发出来，那么要问个非常重要的问题了：

如此众多的方案是如何与 docker 集成在一起的？

答案是：libnetwork 以及 CNM。

### 2.1、 libnetwork & CNM

libnetwork 是 docker 容器网络库，最核心的内容是其定义的 Container Network Model (CNM)，这个模型对容器网络进行了抽象，由以下三类组件组成：

**1. Sanbox**

Sandbox 是容器的网络栈，包含容器的 interface、路由表和 DNS 设置。 Linux Network Namespace 是 Sandbox 的标准实现。Sandbox 可以包含来自不同 Network 的 Endpoint。

**2. Endpoint**

Endpoint 的作用是将 Sandbox 接入 Network。Endpoint 的典型实现是 veth pair，后面我们会举例。一个 Endpoint 只能属于一个网络，也只能属于一个 Sandbox。

**3. Network**

Network 包含一组 Endpoint，同一 Network 的 Endpoint 可以直接通信。Network 的实现可以是 Linux Bridge、VLAN 等。

![CNM示例](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\CNM示例.png)

如图所示两个容器，一个容器一个 Sandbox，每个 Sandbox 都有一个 Endpoint 连接到 Network 1，第二个 Sandbox 还有一个 Endpoint 将其接入 Network 2.

libnetwork CNM 定义了 docker 容器的网络模型，按照该模型开发出的 driver 就能与 docker daemon 协同工作，实现容器网络。docker 原生的 driver 包括 none、bridge、overlay 和 macvlan，第三方 driver 包括 flannel、weave、calico 等。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604105705175-1120892821.png)

下面我们以 docker bridge driver 为例讨论 libnetwork CNM 是如何被实现的。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604105757792-22428017.png)

这是前面我们讨论过的一个容器环境：

1. 两个 Network：默认网络 “bridge” 和自定义网络 “my_net2”。实现方式是 Linux Bridge：“docker0” 和 “br-5d863e9f78b6”。
2. 三个 Enpoint，由 veth pair 实现，一端(vethxxx)挂在 Linux Bridge 上，另一端(eth0)挂在容器内。
3. 三个 Sandbox，由 Network Namespace 实现，每个容器有自己的 Sanbox。

接下来我们将详细讨论各种跨主机网络方案，首先学习 Overlay。

### 2.2、 overlay

为支持容器跨主机通信，Docker 提供了 overlay driver，使用户可以创建基于 VxLAN 的 overlay 网络。VxLAN 可将二层数据封装到 UDP 进行传输，VxLAN 提供与 VLAN 相同的以太网二层服务，但是拥有更强的扩展性和灵活性。

Docerk overlay 网络需要一个 key-value 数据库用于保存网络状态信息，包括 Network、Endpoint、IP 等。Consul、Etcd 和 ZooKeeper 都是 Docker 支持的 key-vlaue 软件，我们这里使用 Consul。

#### 2.2.1、 实验环境描述

我们会直接使用上一章 docker-machine 创建的实验环境。在 docker 主机 host1（192.168.0.44）和 host2（192.168.0.45）上实践各种跨主机网络方案，在 192.168.0.43 上部署支持的组件，比如 Consul。

最简单的方式是以容器方式运行 Consul：

```dockerfile
docker run -d -p 8500:8500 -h consul --name consul progrium/consul -server -bootstrap
```

容器启动后，可以通过 http://192.168.56.101:8500 访问 Consul。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604112156742-1506170698.png)

容器启动后，可以通过 http://192.168.0.43:8500 访问 Consul。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604112337786-1914486703.png)

接下来修改 host1 和 host2 的 docker daemon 的配置文件/etc/systemd/system/docker.service.d/10-machine.conf。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604144336825-37769397.png)

`--cluster-store` 指定 consul 的地址。
`--cluster-advertise` 告知 consul 自己的连接地址。

重启 docker daemon。

systemctl daemon-reload  

systemctl restart docker.service

host1 和 host2 将自动注册到 Consul 数据库中。

http://192.168.0.43:8500/ui/#/dc1/kv/docker/nodes/

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604152127407-422559944.png)

#### 2.2.2、创建overlay网络

在 host1 中创建 overlay 网络 ov_net1：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604153017136-831058294.png)

`-d overlay` 指定 driver 为 overaly。

`docker network ls` 查看当前网络：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604155505724-695287999.png)

注意到 `ov_net1` 的 SCOPE 为 global，而其他网络为 local。在 host2 上查看存在的网络：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604155914569-191137.png)

host2 上也能看到 ov_net1。这是因为创建 ov_net1 时 host1 将 overlay 网络信息存入了 consul，host2 从 consul 读取到了新网络的数据。之后 ov_net 的任何变化都会同步到 host1 和 host2。

`docker network inspect` 查看 ov_net1 的详细信息：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604160235317-1163936577.png)

IPAM 是指 IP Address Management，docker 自动为 ov_net1 分配的 IP 空间为 10.0.0.0/24。

#### 2.2.3、 在overlay中运行容器

一节我们[创建了 overlay 网络](http://mp.weixin.qq.com/s?__biz=MzIwMTM5MjUwMg==&mid=2653587761&idx=1&sn=54d284b9fe1c48872c5d8521635b135e&chksm=8d308128ba47083e813de53887ec7e15f698ce1f61b4e5a7a45461cf8b45078384b668d465f6&scene=21#wechat_redirect) ov_net1，今天将运行一个 busybox 容器并连接到 ov_net1：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604161507663-1816302355.png)

查看容器的网络配置：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604162233512-331048302.png)

bbox1 有两个网络接口 eth0 和 eth1。eth0 IP 为 10.0.0.2，连接的是 overlay 网络 ov_net1。eth1 IP 172.18.0.2，容器的默认路由是走 eth1，eth1 是哪儿来的呢？

其实，docker 会创建一个 bridge 网络 “docker_gwbridge”，为所有连接到 overlay 网络的容器提供访问外网的能力。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604164008517-565247486.png)

从 `docker network inspect docker_gwbridge` 输出可确认 docker_gwbridge 的 IP 地址范围是 172.18.0.0/16，当前连接的容器就是 bbox1（172.18.0.2）。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604164151257-1190928964.png)

而且此网络的网关就是网桥 docker_gwbridge 的 IP 172.18.0.1。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604164417842-388916990.png)

这样容器 bbox1 就可以通过 docker_gwbridge 访问外网。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604164633108-406011157.png)

如果外网要访问容器，可通过主机端口映射，比如：

docker run -p 80:80 -d --net ov_net1 --name web1 httpd

验证完外网的连通性，下一节验证 overlay 网络跨主机通信。

#### 2.2.4、 overlay网络连通性

在 host2 中运行容器 bbox2：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604170906922-625623025.png)

bbox2 IP 为 10.0.0.3，可以直接 ping bbox1：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604171522583-1321355839.png)

可见 overlay 网络中的容器可以直接通信，同时 docker 也实现了 DNS 服务。

下面我们讨论一下 overlay 网络的具体实现：

docker 会为每个 overlay 网络创建一个独立的 network namespace，其中会有一个 linux bridge br0，endpoint 还是由 veth pair 实现，一端连接到容器中（即 eth0），另一端连接到 namespace 的 br0 上。

br0 除了连接所有的 endpoint，还会连接一个 vxlan 设备，用于与其他 host 建立 vxlan tunnel。容器之间的数据就是通过这个 tunnel 通信的。逻辑网络拓扑结构如图所示：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604172249708-1964509152.png)

要查看 overlay 网络的 namespace 可以在 host1 和 host2 上执行 `ip netns`（请确保在此之前执行过 `ln -s /var/run/docker/netns /var/run/netns`），可以看到两个 host 上有一个相同的 namespace “1-3e14e93b3e”：

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604180130005-424038225.png)

这就是 ov_net1 的 namespace，查看 namespace 中的 br0 上的设备。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604182853589-876373184.png)

查看 vxlan0 设备的具体配置信息可知此 overlay 使用的 VNI（VxLAN ID）为 256。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190604183050280-356409049.png)

理解了 overlay 网络的连通性，下一节我们继续讨论 overlay 的网络隔离特性。

#### 2.2.5、 网络隔离

不同的 overlay 网络是相互隔离的。我们创建第二个 overlay 网络 ov_net2 并运行容器 bbox3。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190613102801603-1981244277.png)

bbox3 分配到的 IP 是 10.0.1.2，尝试 ping bbox1（10.0.0.2）。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190613103528577-1150291291.png)

ping 失败，可见不同 overlay 网络之间是隔离的。即便是通过 docker_gwbridge 也不能通信。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190613104309388-785878252.png)

如果要实现 bbox3 与 bbox1 通信，可以将 bbox3 也连接到 ov_net1。

![img](https://img2018.cnblogs.com/blog/1563741/201906/1563741-20190613104515215-337237367.png)

#### 2.2.6、 overlay IPAM

docker 默认为 overlay 网络分配 24 位掩码的子网（10.0.X.0/24），所有主机共享这个 subnet，容器启动时会顺序从此空间分配 IP。当然我们也可以通过 `--subnet` 指定 IP 空间。

docker network create -d overlay --subnet 10.22.1.0/24 ov_net3

至此，overlay 网络已经讨论完了，下一节我们开始学习 macvlan 网络。

### 2.3、 macvlan

