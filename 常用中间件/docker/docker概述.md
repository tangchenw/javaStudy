# docker概述
## 容器生态系统

容器生态系统包括：

- [x] 容器核心技术
- [x] 容器平台技术
- [x] 容器支持技术

### 容器核心技术

容器核心技术：让container在host上能够运行起来的那些技术，包括以下部分

- 容器规范
- 容器runtime
- 容器管理工具
- 容器定义工具
- Registries
- 容器OS

**容器规范**

OCI发布了两个规范：runtime spec 和 image format spec；从而使得不同的厂商开发的容器能在不同的runtime上运行。

**容器runtime**

runtime是容器真正运行的地方，runtime需要和linux操作系统的kernel紧密协作，为容器提供运行环境。

java程序好比是容器，而JVM好比是runtime。

主流的三种容器runtime:

- lxc:linux上老牌的runtime。Docker最初也是使用lxc作为runtime。
- runc: Docker自己开发的容器runtime，符合oci规范，也是现在Docker的默认runtime。
- rkt: CoreOS开发的容器runtime，符合OCI规范，因而能够运行Docker的容器

**容器管理工具**

光有runtime来运行容器还不够，用户得有工具来管理容器。容器管理工具对内与runtime交互，对外为用户提供Interface，比如：**CLI**；

就好比JVM，还得提供java命令让用户能够启停应用。

容器管理工具：

- lxd:  lxc对应的管理工具
- docker engine: runc的管理工具，docker engine 包括后台deamon和cli 两个部分。我们通常提到的docker一般指的就是docker engine。
- rkt cli: rkt的管理工具。

**容器定义工具**

容器定义工具允许用户定义容器的内容和属性，这样容器就能够被保存、共享、和重建。

容器定义工具：

- dokcer image:Docker容器的模板，runtime依据docker image 创建容器。
- dockerfile:包含若干命令（包括sh指令）的文本文件，通过这些命令创建出docker image。
- ACI(App Container Image)与docker image 类似，不过它是由coreOS开发的rkt容器的image格式。

**Registry**

容器是通过Image创建的，需要有一个仓库来统一存放image，这个仓库就叫做Registry。

企业可以用Docker Registry构建私有的Registry。

Registries

- Docker Hub：Docker为公众提供的托管Registry，上面有很多现成的image(许多中间件的image都可以直接在上面下载).
- Quay.io:另外一个公共托管Registry，和Docker Hub提供类似的服务

**容器OS**

由于有容器runtime，几乎所有的linux，MAC OS和Windows都可以运行容器，但这并不妨碍容器OS的出现。

容器OS是专门运行容器的操作系统。它与常规OS相比，容器OS通常体积更小，启动更快。因为是为容器定制的OS，所以他们运行容器的效率也会更高。

容器OS：

- CoreOS
- Atomic
- Ubuntu Core

### 容器平台技术

容器核心技术使得容器可以在单个的host上运行，而容器平台技术能够让容器作为集群在分布式环境中运行。

容器平台技术包括以下：

- 容器编排引擎
- 容器管理平台
- 基于容器的PaaS

**容器编排引擎**

基于容器的应用一般会采用微服务架构。在这种架构下，应用被划分为不同的组件，并以服务的形式运行在各自的容器中，通过API对外提供服务。为了保证应用的高可用性，每个组件可能会运行多个相同的容器。这些容器形成集群，集群中的容器会根据业务需要被动态地创建、迁移和销毁。

所以基于微服务架构的应用系统实际上是一个动态的可伸缩系统。这对我们部署环境提出了新的要求，我们需要一种高效的方法来管理容器集群。而这就是容器编排引擎要干的工作。

**编排**：通常包括容器管理、调度、集群定义和服务发现等。通过容器编排引擎，容器被有机地组成微服务应用，实现业务需求。

容器编排引擎：

- docker swarm:是Docker开发的容器编排引擎。
- kubernetes：是google领导开发的开源容器编排引擎，同时支持docker和CoreOS容器。
- mesos+marathon:mesos是一个通用的集群资源调度平台，mesos和marathon一起提供容器编排引擎的功能。

**容器管理平台**

容器管理平台是架构在容器编排引擎上的一个更为通用的平台。通常容器管理平台能够支持多种容器编排引擎，抽象了编排引擎的底层实现细节，为用户提供更方便的功能，比如application catalog 和一键应用部署等。

常见的容器管理平台有：

- Rancher
- contatinerShip

**基于容器的PaaS**

基于容器的PaaS为微服务应用开发人员和公司提供了开发、部署和管理应用的平台，使用户不必关心底层基础设施而专注于应用开发。

常见的开源容器PaaS有：

- Deis
- Flynn
- Dokku

### 容器支持技术

以下技术被用于支持基于容器的基础设备

- 容器网络
- 服务发现
- 监控
- 数据管理
- 日志管理
- 安全性

**容器网络**

容器的出现使得网络拓扑变得更加动态和复杂。因此用户需要专门的解决方案来管理容器和其它实体之间的连通性和隔离性。

常见的容器网络有以下几种：

- docker network(docker 原生的网络解决方案)
- flannel
- weave
- calico

**服务发现**

动态变化是微服务应用的一大特点。当负载增加时，集群会自动添加新的容器；负载减小，多余的容器被销毁.容器也会根据host的资源使用情况在不同host中迁移，容器的IP和端口也会随之发生改变。

在这种动态的环境下，必须要有一种机制让client能够知道如何访问容器提供的服务。这就是服务发现技术要完成的工作。

服务发现会保存容器集群中所有微服务最新的信息，比如IP和端口，并对外提供API，提供服务查询功能。

常见的服务发现解决方案有：

- etcd
- consul
- **zookeeper**(spring cloud微服务中有)

 **监控**

监控对于基础架构非常重要，而容器的动态特征对监控提出更多挑战。

常见的监控工具和方案有：

- docker ps/top/stats（docker原生的命令行监控工具。除了命令行，Docker也提供了stats API，用户可以通过HTTP请求获取容器状态信息）
- docker stats API
- sysdig
- cAdvisor/Heapster
- Weave Scope

**数据管理**

容器经常会在不同的host之间迁移，如何保证持久化数据也能够动态迁移，Rex-Ray数据管理工具提供对应的功能。

**日志管理**

- docker logs(docker 原生日志工具)
- logspout（logspout对日志提供路由功能，它可以收集不同容器的日志并转发给其它工具进行后处理）。

**安全性**

OpenSCAP是一种容器安全工具，它能够对容器镜像进行扫描，发现潜在的漏洞。

## 容器技术

### 什么是容器？

容器是一种轻量级、可移植、自包含的软件打包技术，使应用程序可以在几乎任何地方以相同的方式运行。开发人员在自己的笔记本上创建并测试好的容器，无须任何修改就能够在生产系统的虚拟机、物理服务器或共有云主机上运行。

**容器与虚拟机**

谈到容器，就不得不将它与虚拟机进行对比，因为两者都是为应用提供封装和隔离。

容器由两部分组成：（1）应用程序本身；（2）依赖：比如应用程序需要的库或者其它软件容器在host操作系统的用户空间中运行，与操作系统的其它进程隔离。这一点显著区别于虚拟机。

传统的虚拟化技术，比如VMWare、KVM、Xen，目标是创建一个完整的虚拟机。为了运行应用，除了部署应用本身及其依赖（通常几十MB），还得安装整个操作系统（几十GB）。

由于所有容器共享一个host OS,这使得容器在体积上要比虚拟机小很多。另外，启动容器不需要启动整个操作系统，所以容器部署和启动速度更快、开销更小，也更容易迁移。

### 为什么需要容器？

为什么需要容器？容器到底解决的是什么问题？简要的答案是：容器使软件具备了超强的可移植能力。

#### 容器解决的问题

如今的系统在架构上较十年前已经变得非常复杂了。以前几乎所有的应用都采用三层架构(Presentation/Application/Data)，系统部署到有限的几台物理服务器上（Web Server/Application Server/Database Server)

而今天，开发人员通常使用多种服务（比如MQ、Cache、DB）构建和组装应用，而且应用很可能会部署到不同的环境，比如虚拟服务器、私有云和公有云。

一方面应用包含了多种服务，这些服务有自己所依赖的库和软件包，另一方面存在多种部署环境，服务在运行时可能需要动态迁移到不同的环境中。这就产生了一个问题：如何让每种服务能够在所有的部署环境中顺利运行。

各种服务和环境通过排列组合产生了一个大矩阵。开发人员在编写代码时需要考虑不同的运行环境，运维人员则需要为不同的服务和平台配置环境。对他们双方来说，这都是一项困难而艰巨的任务。

如何解决这个问题？

类比于运输行业的交通问题使用集装箱进行解决。Docker将集装箱的思想运用到软件打包上，为代码提供了一个基于容器的标准化运输系统。Docker可以将任何应用及其依赖打包成一个轻量级、可移植、自包含的容器。容器可以运行在几乎所有的操作系统上。

其实，“集装箱”和“容器”对应的英文单词都是“Contatiner”

“容器”是国内约定俗成的说法，可能因为容器比集装箱更抽象，更适合软件领域的缘故。

我个人认为：在外国人的思想中，“Container”只用到了集装箱这一个意思，Docker 的Logo就是一堆集装箱。

#### Docker的特性

|   特性   |                            集装箱                            |                            Dokcer                            |
| :------: | :----------------------------------------------------------: | :----------------------------------------------------------: |
| 打包对象 |                         几乎任何货物                         |                       任何软件及其依赖                       |
| 硬件依赖 | 标准形状和接口允许集装箱被装卸到各种交通工具，整个运输过程无需打开 | 容器无需修改便可以运行在几乎所有平台上——虚拟机、物理机、公有云、私有云 |
|  隔离性  |      集装箱可以重叠起来一起运输，香蕉不会再被铁通压烂了      |          资源、网络、库都是隔离的，不会出现依赖问题          |
|  自动化  |             标准接口使集装箱很容易自动装卸和移动             |       提供run、start、stop等标准化操作，非常适合自动化       |
|  高效性  |             无须开箱，可在各种交通工具间快速搬运             |                  轻量级，能够快速启动和迁移                  |
| 职责分工 | 货主只需要考虑把什么放到集装箱里， 承运方只需要关心怎么运输集装箱 | 开发人员只需要考虑怎么写代码，运维人员只需要关心如何配置基础环境 |

#### 容器的优势

对于开发人员：Build Once、Run Anywhere。

容器意味着环境隔离和可重复性。开发人员只需为应用创建一次运行环境，然后打包成容器便可在其它机器上运行。另外，容器环境与所在的host环境是隔离的，就像虚拟机一样，但更快更简单。

对于运维人员：Configure Once、Run Anywhere。

只需要配置好标准的runtime环境，服务器就可以运行任何容器。这使得运维人员的工作变得更高效、一致和可重复。容器消除了开发、测试、生产环境的不一致性。

### 容器是如何工作的

接下来就是容器核心知识的最主要部分。

我们首先会介绍Docker的架构，然后分章节详细讨论Docker的镜像、容器、网络和存储。

#### Docker架构

Docker的核心组件包括：

- Docker 客户端：Client
- Docker 服务器：Docker daemon
- Docker 镜像：Image
- Registry
- Docker 容器：Container

Docker架构如图所示：

![Docker架构](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\Docker架构.png)

Docker采用的是Client/Server架构。客户端向服务器发送请求，服务器负责构建、运行和分发容器。客户端和服务器可以运行在同一个Host上，客户端也可以通过socket或者REST API与远程的服务器进行通信。

#### Docker客户端

最常用的Docker客户端是docker命令。通过docker我们可以方便地在Host上构建和运行容器。

docker支持很多操作（子命令），后面会逐渐用到，如下图所示：

![docker常用操作命令](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker常用操作命令.png)

除了docker命令行工具，用户也可以通过REST API与服务器通信。

#### Docker服务器

**Docker daemon是服务器组件**，以linux后台服务的方式运行。如下图所示：

![docker服务运行状态](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker服务运行状态.png)

Docker daemon运行在Docker host上，负责创建、运行、监控容器，构建、存储镜像。

默认配置下，Docker daemon只能响应来自本地host的客户端请求。如果要允许远程客户端请求，需要在配置文件中打开TCP监听，步骤如下：

（1）编辑配置文件/etc/systemd/system/multi-user.target.wants/docker.service，在环境变量ExecStart后面添加 -H tcp://0.0.0.0，允许来自任意IP的客户端连接，如图所示：

![配置docker远程客户端请求连接](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\配置docker远程客户端请求连接.png)

如果使用的是其它操作系统，配置文件的位置可能不一样。

重启Docker daemon。

```ABAP
systemctl daemon-reload
systemctl restart docker.service
```

服务器ip为192.168.56.102，客户端在命令行里加上-H 参数，即可与远程服务器通信。

```sh
docker -H 192.168.56.102 info
```

Info子命令用于查看docker 服务器的信息。

#### Docker 镜像

可将Docker镜像看成只读模板，通过它可以创建Docker容器。

例如某个镜像可能包含一个ubuntu操作系统、一个Apache HTTP Server 以及用户开发的web应用。

镜像有多种生成方法：（1）从无到有开始创建镜像；（2）下载并使用别人创建好的现成镜像；（3）在现有镜像上创建新的镜像

可以将镜像的内容和创建步骤描述在一个文本文件中，这个文件被称作dockerfile，通过执行docker bulid<docker-file>命令可以构建出Docker镜像，后面我们会讨论。

#### Docker容器

Docker容器就是Docker镜像的运行实例。

用户可以通过CLI（Docker)或者是API启动、停止、移动或删除容器。可以这么认为，对于应用软件，镜像是软件生命周期的构建和打包阶段而容器是启动和运行阶段。

#### Registry

Registry是存放Docker镜像的仓库，Registry分为私有和共有两种。

Docker Hub是默认的Registry，由Docker公司维护，上面有数以万计的镜像，用户可以自由的下载和使用。

出于对速度或者安全的考虑，用户也可以创建自己的私有Registry。后面我们会学习如何搭建私有Registry。

docker pull 命令可以从Registry下载镜像。

docker run命令则是先下载镜像（如果本地没有），然后再启动容器。

#### 一个完整的例子

容器的启动过程如下：

![docker容器的完整启动过程](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker容器的完整启动过程.png)

1. docker客户端执行docker run命令。
2. docker daemon发现本地没有httpd镜像。
3. daemon从dokcer hub 下载镜像
4. 下载完成，镜像http 被保存到本地
5. docker daemon启动容器。

查看容器镜像

```ABAP
docker images
```

查看正在运行的容器

```ABAP
docker ps/docker container ls
```

## Docker 镜像

镜像是Docker容器的基石，容器是镜像的运行实例，有了镜像才可以启动容器。

本章内容安排如下：首先研究几个经典的镜像，分析镜像的内部结构；然后学习如何构建自己的镜像；最后介绍怎样管理和分发镜像。

### 镜像的内部结构

为什么我们要讨论镜像的内部结构？

如果只是使用镜像，当然不需要了解，直接通过docker命令下载和运行就行了。

但如果我们想创建自己的镜像，或者想理解Docker为什么是轻量级的，就非常有必要学习这部分知识了。

我们从一个最小的镜像开始把。

#### Hello-world -----最小的镜像

hello-world是docker官方提供的一个镜像，通常用来验证dokcer安装是否成功的。可以先通过docker pull 从dokcer hub下载它。

如图所示：

![docker最小镜像](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker最小镜像.png)

hello-world镜像还不到20kb，通过docker run 运行，如下图所，其实我们更关心hello-world镜像包括那些内容。

Dockerfile是镜像的描述文件，定义了如何构建Docker镜像。Dockerfile的语法简洁且可读性强，后面我们会专门讨论如何编写Dockerfile。

![helloworld运行](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\helloworld运行.png)

hello-world的Dockerfile内容如下：

```dockerfile
from scratch  #镜像是白手起家，从0开始构建
COPY hello /  #将文件“hello”复制到镜像的根目录
CMD["/hello"]	#容器启动时执行/hello。
```

镜像hello-world中就只有一个可执行文件“hello”，其功能就是打印出“Hello form docker....”等信息。

/hello 就是文件系统的全部内容，连最基本的/bin、/usr、/lib、/dev都没有

hello-world虽然是一个完整的镜像，但它并没有什么实际用途。通常来说，我们希望镜像能提供一个基本的操作系统环境，用户可以根据需要安装和配置软件。

这样的镜像我们称之为base镜像。

#### base镜像

base镜像有两层含义：（1）不依赖其它镜像，从scratch构建；（2）其他镜像可以以之为基础进行扩展。

所以，能称为base镜像的通常都是各种linux发行版的docker镜像，比如ubuntu、Debian、CentOS等。

我们以CentOS为例考察base镜像包含哪些内容。

下载镜像：

```ABAP
docker pull centos
```

下载完之后可以看见centos镜像还不到200mb,平时我们安装一个CentOS至少都有几个GB，怎么可能才200MB！

![CentOS的base镜像](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\CentOS的base镜像.png)

相信这是几乎所有Docker初学者都会有的疑问，包括我自己。下面我们来解释这个问题。

linux操作系统由内核空间和用户空间组成。

**1.rootfs**

内核空间是kernel,linux刚启动时会加载bootfs文件系统，之后bootfs会被卸载掉。用户空间的文件系统是rootfs，包括我们熟悉的/dev、/proc、/bin等目录。

对于base镜像来说，底层直接用host的kernel，自己只需要提供rootfs就行了。

而对于一个精简的OS，rootfs可以很小，只需要包括最基本的命令、工具和程序库就可以了。相比其它的Linux发行版本，CentOS的rootfs已经算臃肿了，alpine还不到10MB.

我们平时安装的CenOS除了rootfs还会选装很多软件、服务、图形桌面等，需要好几个GB就不足为奇了。

**2.base镜像提供的是最小安装的Linux发行版**

CentOS镜像的Dockerfile的内容如下：

```dockerfile
FROM scratch
ADD centos-7-docker.tar.xz /  #添加到镜像的tar包就是CentOS 7的rootfs。在制作镜像时，这个tar包会自动解压到/目录下
CMD["/bin/bash"]
```

**3.支持运行多种Linux OS**

不同的LInux发行版的区别主要就是rootfs。

比如Ubuntu14.04使用upstart管理服务，apt管理软件包；而CentOS 7 使用systemd和yum。这些都是用户空间上的区别，linux kernel差别不大。

所以Dokcer可以同时支持多种Linux镜像， 模拟出多种操作系统环境。如图所示：

![kernel模型图](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\kernel模型图.png)

上图中Debian和BusyBox（一种嵌入式Linux）上层提供各自的rootfs，底层共用Docker Host的kernel.

这里需要说明的是：

> （1）base镜像只是在用户空间与发行版一致，Kernel版本与发行版是不同的。
>
> 例如：CentOS 7 使用3.x.x 的kernel，如果Docker Host 是 Ubuntu 16.04,那么在CentOS容器中使用的实际上是Host 4.x.x的kernel。
>
> （2）容器只能使用Host的kernel，并且不能修改。
>
> 所有容器都共用host的kernel，在容器中没办法对kernel升级。如果容器对kernel版本有要求，则不建议用容器，这种场景虚拟机可能更合适。

#### 镜像的分层结构

Docker支持通过扩展现有镜像，创建新的镜像。

实际上，Docker Hub中99%的镜像都是通过在base镜像中安装和配置需要的软件构建出来的。比如我们现在构建一个新的镜像，Dockerfile如下：

```dockerfile
FROM debian  #直接从Debian base镜像上创建
RUN apt-get install emacs #安装emacs编辑器
RUN apt-get install apache2 #安装apache2
CMD["/bin/bash"] #容器启动时运行bash
```

构建过程如下图所示：

![images构建过程](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\images构建过程.png)

可以看到，新镜像是从base镜像一层一层叠加生成的。每安装一个软件，就在现有镜像的基础上增加一层。

为什么Docker镜像要采用这种分层结构呢？

最大的一个好处就是：共享资源。

比如：有多个镜像都从相同的base镜像构建而来，那么Docker Host只需在磁盘上保存一份base镜像，同时内存中也只需加载一份base镜像，就可以为所有容器服务了，而且镜像的每一层都可以被共享，我们将在后面更深入地讨论这个特性。

这时可能就有人会问了：如果多个容器共享一份基础镜像，当某个容器修改了基础镜像的内容，比如/etc下的文件，这时其它容器的/etc是否会被修改？

答案是不会！

修改会被限制在单个容器内。

这就是我们接下来要学习的容器Copy-on-Write特性。

**可写的容器层**

当容器启动时，一个新的可写层被加载到镜像的顶部。

这一层通常被称作”容器层“，”容器层“之下都叫”镜像层“，如图所示：

![容器层](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\容器层.png)

所有对容器的改动，无论是添加、删除，还是修改文件都只会发生在容器层中。只有容器层是可写的，容器层下面的所有镜像层都是可读的。

下面我们深入讨论容器层的细节。

镜像层数量可能会很多，所有镜像层会联合在一起组成一个统一的文件系统。如果不同层中有一个相同路径的文件，比如/a，上层的/a会覆盖下层的/a，也就是说用户只能访问到上层中的文件/a。在容器层中，用户看到的是一个叠加之后的文件系统。

（1）添加文件。在容器中创建文件时，新文件被添加到容器层中

（2）读取文件。在容器中读取某个文件时，Docker会从上往下依次在各镜像层中查找此文件。一旦找到，打开并读入内存。

（3）修改文件。在容器中修改已存在的文件时，Docker会从上往下依次在各镜像层中查找此文件。一旦找到，立即将其复制到容器层，然后修改之。

（4）删除文件。在容器中删除文件时，Docker也是从上往下依次在镜像层中查找此文件。找到后，会在容器层中记录下此删除操作。

只有当需要修改时才复制一份数据，这种特性被称作Copy-on-Write。可见，容器层保存的是镜像变化的部分，不会对镜像本身进行任何修改。

这样就解释了前面的问题：容器层记录对镜像的修改，所有镜像层都是只读的，不会被容器修改，所以镜像可以被多个容器共享。

### 构建镜像

对于docker用户来，最好的情况是不需要自己创建镜像。几乎所有常用的数据库、中间件、应用软件等都有现成的docker官方镜像或其它人和组织创建的镜像，我们只需要稍作配置就可以一直使用。

使用现成镜像的好处是除了省去自己做镜像的工作量外，更重要的是可以利用前人的经验。特别是使用那些官方镜像，因为docker的工程师知道如何更好地在容器中运行软件。

当然，某些情况下我们也不得不自己构建镜像，比如：

（1）找不到现成的镜像，比如自己开发的应用程序。

（2）需要在镜像中加入特定的功能，比如官方镜像几乎都不提供ssh。

所以本节我们将介绍构建镜像的方法。

同时分析构建的过程也能够加深我们前面对镜像分层结构的理解。

docker提供了两种构建镜像的方法：docker commit命令和dockerfile构建文件。

#### Docker commit

docker commit命令是创建新镜像最直观的方法，其过程包含三个步骤：

- 运行容器。
- 修改容器。
- 将容器保存为新的镜像。

举个例子：在ubuntu base镜像中安装vi并保存为新镜像。

1. 运行容器

```ABAP
docker run -it ubuntu
```

![进入base镜像](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\进入base镜像.png)

-it 参数的作用是以交互模式进入容器，并打开终端。3e196bde7cf5是容器的内部id。

2. 安装vi

```ABAP
apt update
apt-get install -y vim
```

确认vi没有安装，使用apt-get install -y vim无法安装时可以使用apt update更新apt之后再执行。

![ubuntu的vi安装](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\ubuntu的vi安装.png)
3. 保存为新镜像，festive_galileo是docker为我们容器随机分配的名字。执行docker commit命令将容器保存为镜像新命名为ubuntu-with-vi。

```ABAP
docker commit festive_galileo ubuntu-with-vi
```

![ubuntu镜像](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\ubuntu镜像.png)

![新ubuntu镜像](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\新ubuntu镜像.png)

从size上看到镜像因为安装软件而变大了。

从新镜像启动容器，验证vi已经可以使用。

![验证新镜像的vi使用](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\验证新镜像的vi使用.png)

以上演示了如何使用docker commit 创建新镜像。然而，docker并不建议用户通过这种方式构建镜像。原因如下：

1. 这是一种手工创建镜像的方式，容易出错，效率低且可重复性弱。比如要在debian base镜像中也加入vi，还得重复前面的所有操作。
2. 更重要的：使用者并不知道镜像是如何创建出来的，里面是否有恶意程序。也就是说无法对镜像进行审计，存在安全隐患。

既然docker commit不是推荐的方法，我们为什么还要花时间学习呢？

原因是：即便是用dockerfile（推荐方法）构建镜像，底层也是docker commit一层一层构建新镜像的。学习docker commit能够帮助我们更加深入地理解构建过程和镜像的分层结构。

#### Dockerfile

dockerfile是一个文本文件，记录了镜像构建的所有步骤。

**1.第一个dockerfile**

用dockerfile创建上节的ubuntu-with-vi，内容如下：

```dockerfile
from ubuntu
run apt-get update && apt-get install -y vim
```

下面我们运行docker build命令构建镜像并详细分析每个细节。

![dockerfile构建镜像过程](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\dockerfile构建镜像过程.png)

![旧docker构建原理](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\旧docker构建原理.png)

```ABAP
docker build -t ubuntu-with-vi-dockerfile .  #后面这个.一定要加，不然无法正常构建
```

（3）运行docker build 命令，-t将新镜像命名为ubuntu-with-vi-dockerfile，命令末尾的.指明build  context为当前目录。docker默认会从build context 中查找dockerfile文件，我们也可以通过-f参数来指定dockerfile的位置。

（4）从这步开始就是镜像真正的构建过程。首先docker将docker build  context中所有文件发送给docker daemon。build context为镜像构建提供所需要的文件或目录。

dockerfile中的add、copy等命令可以将build context中文件添加到镜像。此例中，build context 为当前目录/root，该目录下所有的文件和子目录都会被发送给docker daemon。

所以，使用build context就得小心了，不要将多余文件放到build context，特别不要把/、/usr作为build context，否则构建过程会相当缓慢甚至失败。

（5）step 1:执行from，将ubuntu作为base镜像。

ubuntu镜像的id为上图。

（6）step 2:执行run,安装vim，具体步骤为（7）、（8）、（9）。

（7）启动id为上图中的临时容器，在容器中通过apt-get安装vim。

（8）安装成功之后将容器保存为镜像，其id为上图。

这一步底层使用的是类似docker commit 的命令。

（9）删除临时容器。

（10）容器构建成功。

在上面构建过程中，我们要特别注意指令run的执行过程（7）、（8）、（9）。docker会在启动的临时容器中执行操作，并通过commit保存为新的镜像。

**2.查看镜像分层结构**

ubuntu-with-vi-dockerfile 是通过在base镜像的顶部添加一个新的镜像层得到的，如下图所示：

![ubuntu-with-vi-dockerfile镜像结构](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\ubuntu-with-vi-dockerfile镜像结构.png)

这个新的镜像层的内容由RUN apt-get update && apt-get install -y vim 生成。这一点我们可以通过docker history命令查看，如图所示：

![dockerfile构建历史记录](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\dockerfile构建历史记录.png)

docker history会显示镜像的构建历史，也就是Dockerfile的执行过程。

ubuntu-with-vi-dockerfile与Ubuntu镜像相比，确实只是多了顶层一层顶层的镜像由apt-get命令创建，大小为97.3MB.docker history也向我们展示了镜像的分层结构，每一层由上至下排列。

> missing表示无法获取IMAGE id,通常从Docker Hub下载的镜像会有这个问题。

**3.镜像的缓存特性**

我们接下来看Docker镜像的缓存特性。

Docker会缓存已有镜像的镜像层，构建新镜像时，如果某镜像层已经存在，就直接使用，无须重新创建。

下面举例说明：

在前面的Dockerfile中添加一点新内容，往镜像中复制一个文件。

```dockerfile
from ubuntu
run apt-get update && apt-get install -y vim
copy testfile /
```

![缓存构建镜像](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\缓存构建镜像.png)

确保testfile已经存在，上图红色框所示之前执行过相同的RUN指令，这次直接使用缓存中的镜像层。执行COPY指令。

其过程是启动临时容器，复制testfile，提交新的镜像层，删除临时容器。

在ubuntu-with-vi-dokcerfile镜像上直接添加一层就得到了新的镜像ubuntu-with-vi-dockerfile-2。

如果我们希望在构建镜像时不使用缓存，可以在dokcer build命令中加上--no-cache参数。

Dockerfile中每一个指令都会创建一个镜像层，上层是依赖于下层的。无论什么时候，只要某一层发生变化，其上面所有层的缓存都会失效。

也就是说，如果我们改变dockerfile指令的执行顺序，或者修改或添加指令，都会使缓存失效。举例说明，比如交换前面的run和copy的顺序。

```dockerfile
from ubuntu
copy testfile /
run apt-get update && apt-get install -y vim
```

虽然在逻辑上这种改动对镜像的内容没有影响，但由于分层结构的结构特性，docker必须重建受影响的镜像层。

![dockerfile改变执行顺序缓存失效](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\dockerfile改变执行顺序缓存失效.png)

从上面的输出可以看到生成了新的镜像层，缓存已经失效。

除了构建时使用缓存，Docker在下载镜像时也会使用。例如我们下载httpd镜像。

docker pull命令输出显示第一层（base镜像）已经存在，不需要下载。

由Dockerfile可知httpd的base镜像为debian，正好之前已经下载过debian镜像，所以有缓存可用。通过docker history可以进一步验证。

**4.调试Dockerfile**

总结一下通过dockerfile构建镜像的过程：

（1）从base镜像运行一个容器。

（2）执行一条指令，对容器做修改。

（3）执行类似docker commit的操作，生成一个新的镜像层。

（4）Docker再基于刚刚提交的镜像运行一个新容器。

（5）重复2-4步骤，直到Dockerfile中的所有指令执行完毕。

从这个过程可以看出，如果Dockerfile由于某种原因执行到某个指令失败了，我们也将能够得到前一个指令成功执行构建出的镜像，这对调试Dockerfile非常有帮助。我们可以运行最新的这个镜像定位指令失败的原因。

我们来看一个调试的例子。Dockerfile内容如下：

```dockerfile
from busybox
run touth tmpfile
run /bin/bash -c echo "continue to build..."
copy testfile /
```

![build调试执行失败](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\build调试执行失败.png)

dockerfile在执行第三步run指令时失败。我们可以利用第二步创建的镜像进行调试，方法是通过docker run -it启动镜像的一个容器。

![docker run启动失败容器](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker run启动失败容器.png)

手工执行run指令很容易定位失败的原因是busybox镜像中没有bash。虽然这是个极其简单的例子，但它很好地展示了调试dockerfile的方法。

**5.Dockerfile常用指令**

是时候系统学习dockerfile了。

下面列出了docker中最常用的指令，完整列表和说明可参看官方文档。

|    指令    |                             作用                             |
| :--------: | :----------------------------------------------------------: |
|    FROM    |                         指定base镜像                         |
| maintainer |               设置镜像的作者，可以是任意字符串               |
|    copy    | 将文件从build context复制到镜像。copy支持两种形式：copy src dest与copy["src","dest"].<br />src只能指定build context中的文件或者目录 |
|    add     | 与copy类似，从build context复制文件到镜像。不同的是，如果src是归档文件（tar、zip、tgz、xz等），文件会被自动解压到dest |
|    env     | 设置环境变量，环境变量可被后面的指令使用。例如：env my_version 1.3 run apt-get install -y mypagckage=$my_version |
|   expose   | 指定容器中的进程会监听某个端口，docker可以将该端口暴露出来。我们会在容器网络内部详细讨论。 |
|   volume   |   将文件或目录声明为volume。我们会在容器存储部分详细讨论。   |
|  workdir   | 为后面的run、cmd、entrypoint、add或copy指令设置镜像中的当前工作目录。 |
|    run     |                   在容器中运行指定的命令。                   |
|    cmd     | 容器启动时运行指定的命令。dockerfile中可以有多个cmd指令，但只有最后一个生效。cmd可以被docer run之后的参数替替换。 |
| entrypoint | 设置容器启动时运行的命令。dockerfile中可以有多个entrypoint指令，但只有最后一个生效。cmd或docker run之后的参数会被当作参数传递给entrypoint。 |



### RUN VS CMD   VS  ENTRYPOINT

run、CMD、和ENTRYPOINT这三个Dockerfile指令看上去很类似，很容易混淆。本节将通过实践详细讨论它们的区别。

简单的说：

1. RUN:执行命令并创建新的镜像层，RUN经常用于安装软件包
2. CMD：设置容器启动后默认执行的命令以及参数，但CMD能够被docker run后面跟的命令行参数替换。
3. ENTRYPOINT：配置容器启动时运行的命令。

下面我们详细分析：

**Shell 和  Exec格式**

我们可以用两种方式指定RUN、CMD和ENTRYPOINT要运行的命令：Shell格式和Exec格式，两者在使用上有细微的区别。

Shell格式：

`<instruction><command>`

例如：

```dockerfile
RUN apt-get install python3
CMD echo "Hello World"
ENTRYPOINT echo "Hello World"
```

当指令执行时，shell格式底层会调用/bin/sh -c [command]。例如下面这段Dockerfile片段：

```dockerfile
ENV name Cloud Man ENTRYPOINT echo "Hello,$name"
```

执行docker run [image]将输出：

`Hello,Cloud Man`

注意环境变量name已经被值Clound Man替换。

下面来看Exec格式。

`<instruction>["executable","param1","param2",...]`

例如：

```dockerfile
RUN ["apt-get","install","python3"]
CMD ["/bin/echo","Hello World"]
ENTRYPOINT ["/bin/echo","Hello World"]
```

当指令执行时，会直接调用[command]，不会被shell解析。

例如下面的Dockerfile片段：

```dockerfile
ENV name Cloud Man ENTRYPOINT ["/bin/echo","Hello,$name"]
```

运行容器将输出：

`Hello,$name`

注意:环境变量name没有被替换。

如果希望使用环境变量，做如下修改Dokcerfile:

`ENV name Cloud Man ENTRYPOINT ["/bin/sh","-c","echo Hello,$name"]`

运行容器将输出：

`Hello,Cloud Man`

CMD和ENTRYPOINT推荐使用Exec格式，因为指令可读性更强，更容易理解。RUN则两种格式都可以。

**RUN**

RUN指令通常用于安装应用和软件包。

RUN在当前镜像的顶部执行命令，并创建新的镜像层。Dockerfile通常包含多个RUN指令。

RUN有两种格式：

（1）Shell格式：RUN

（2）Exec格式：RUN["executable","param1","param2"]

下面是使用RUN安装多个包的例子：

```dockerfile
RUN apt-get update && apt-get install-y\bzr\cvs\git\mercurial\subversion
```

> apt-get update和apt-get install 被放在一个run指令中执行，这样能够保证每次安装的是最新的包。如果apt-get install在单独的run中执行，则会使用apt-get update创建镜像层，而这一层可能是很久以前缓存的。

**CMD**

CMD指令允许用户指定容器的默认执行的命令。

此命令会在容器启动且docker run没有指定其它命令时运行。

- 如果docker run指定了其它命令，CMD指定的默认命令将被忽略。
- 如果dockerfile中有多个CMD指令，只有最后一个CMD有效。

CMD有三种格式：

1. Exec格式：CMD["executable","param1","param2"]  CMD推荐格式
2. CMD["param1","param2"]为ENTRYPOINT提供额外的参数，此时ENTRYPOINT必须使用Exec格式。
3. Shell格式：CMD command param1 param2

Exec格式和Shell格式前面已经介绍过了

第二种格式CMD["param1","param2"]要与Exec格式的ENTRYPOINT指令配置使用，其用途是为ENTRYPOINT设置默认的参数。我们将在后面讨论ENTRYPOINT时举例说明。

下面看看CMD是如何工作的。Dockerfile片段如下：

`CMD echo "Hello World"`

运行容器docker run -it [image]将输出；

`Hello World`

但当后面加上一个命令，比如docker run -it [image] /bin/bash,CMD会被忽略掉，命令base将被执行：

**ENTRYPOINT**

ENTRYPOINT指令可以让容器以应用程序或服务的形式运行。

ENTRYPOINT看上去与CMD很想，它们都可以指定要执行的命令及其参数。不同的地方在于ENTRYPOINT不会被忽略，一定会被执行，即使运行docker run时指定了其它命令。

ENTRYPOINT有两种格式：

1. Exec格式：ENTRYPOINT["executable","param1","param2"]这是ENTRYPOINT的推荐格式。
2. Shell格式：ENTRYPOINT command param1 param2.

在为ENTRYPOINT选择格式时必须小心，因为这两种格式的效果差别很大。

**1.Exec格式**

ENTRYPOINT的Exec格式用于设置要执行的命令及其参数，同时可通过CMD提供额外的参数。

ENTRYPOINT中的参数始终会被使用，而CMD的额外参数可以在容器启动时动态被替换掉。

比如下面的Dockerfile片段：

```dockerfile
ENTRYPOINT ["/bin/echo","Hello"] CMD ["world"]
```

当容器通过docker run -it [image]启动时，输出为：

`Hello world`

而如果通过docker run -it [image] CloudMan启动，则输出为：

`Hello CloudMan`

**2.Shell格式**

ENTRYPOINT的Shell格式会忽略任何CMD或docker run提供的参数。

**最佳实践**

1. 使用RUN指令安装应用和软件包，构建镜像。
2. 如果Docker镜像的用途是运行应用程序或服务，比如运行一个MySQL,应该优先使用Exec格式的ENTRYPOINT指令。CMD可为ENTRYPOINT提供额外的默认参数，同时可利用docker run命令行替换默认参数。
3. 如果想为容器设置默认的启动命令，可使用CMD指令。用户可在docker run命令行中替换此默认命令。

### 分发镜像

我们已经学会构建自己的镜像了。接下来的问题是如何在多个Docker Host上使用镜像。

这里有几种可用的方法：

1. 使用相同的dockerfile在其它host构建镜像。
2. 将镜像上传到公共Registry（比如Dokcer Hub），Host直接下载使用。
3. 搭建私有的Registry供本地host使用。

本节重点讨论如何使用公共和私有Registry分发镜像。

#### 为镜像命名

当我们执行docker build 命令时已经为镜像取了一个名字。例如：

```ABAP
docker build -t unbuntu-with-vi
```

这里的unbuntu-with-vi就是镜像的名字，通过dokcer images可以查看镜像信息。

![docker镜像组成](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker镜像组成.png)

这里注意到ubuntu-with-vi对应的是REPOSITORY,而且有一个叫latest的TAG。

实际上一个特定镜像的名字由两部分组成：repository和tag。

`[image name] = [repository]:[tag]`

如果执行dokcer build时没有指定tag，会使用默认值latest。其效果相当于

```ABAP
docker build -t ubuntu-with-vi:latest
```

tag常用于描述镜像的版本信息，比如httpd镜像。当然tag可以是任意字符串。

**1.小心latest tag**

千万别被latest tag给误导了。latest其实并没有什么特殊含义。当没有指明镜像tag时，dokcer会使用默认值latest而已。

虽然dokcer hub很多repository将latest作为最新稳定版本的别名，但这只是一种约定，而不是强制规定。

所以我们在使用镜像时最好还是避免使用latest，明确指定某个tag，比如：httpd:23,ubuntu:xenial。

**2.tag最佳实践**

借鉴软件版本命名方式能够让用户很好地使用镜像。

一个高效版本命名方案可以让用户清楚地知道当前使用的是哪个镜像，同时还可以保持足够的灵活性。

每个repository可以有多个tag，而多个tag可能对应的是同一个镜像。下面通过例子为大家介绍Docker社区普遍使用的tag方案。

假设我们现在发布了一个镜像myimage，版本为v1.9.1，那么我们可以给镜像打上4个tag：1.9.1、1.9、1和latest。

```ABAP
dokcer tag myimage-v1.9.1 myimage:1 docker tag myimage-v1.9.1 myimage:1.9 docker tag myimage-v1.9.1 myimage:1.9.1 docker myimage-1.9.1 myimage:latest
```

过段时间，我们发布v1.9.2。这时可以打上1.9.2的tag，并将1.9、1和latest从v1.9.1移到v1.9.2

这种tag方案使镜像的版本很直观，用户在选择时非常灵活：

1. myimage:1始终指向1这个分支中最新的镜像。
2. myimage:1.9始终指向1.9.x中最新的镜像。
3. myimage:latest始终指向所有版本中最新的镜像。
4. 如果想使用特定版本，可以选择myimage:1.9.1、myimage:1.9.2或者myimage:2.0.0

Docker Hub很多repository都采用这种方案，所以一定要熟悉。

#### 使用公共Registry

保存和分发镜像最直接方法就是使用Docker Hub。

Docker Hub是Docker 公司维护的公共Registry。用户可以将自己的镜像保存到Docker Hub免费的repository中。如果不希望别人访问自己的镜像，也可以购买私有repository。

除了Docker Hub，quay.io是另外一个Registry，提供与Docker Hub类似的服务。

下面介绍如何用Docker Hub存取我们的镜像。

（1）首先得在docker hub上注册一个账号

（2）在docker Host上登录。

（3）修改镜像的repository，使之与Docker Hub账号匹配。

`docker tag httpd tangchen/httpd:v1`

通过docker push将镜像上传至docker hub

`docker push tangchen/httpd:v1`

docker会上传镜像的每一层。因为tangchen/httpd:v1这个镜像实际上跟官方的httpd镜像一模一样，docker hub上面已经有了全部的镜像层，所以真正上传的数据很少。同样的，如果我们的镜像是基于base镜像的，也只有新增的镜像层会被上传。如果想上传同一repository中所有镜像，省略tag部分就行了。

docker push tangchen/httpd

（1）登录docker hub官方网站，在public repository中就可以看到上传的镜像。

如果要删除上传的镜像，只能在Docker Hub界面上操作。

（2）这个镜像可以被其它docker host下载使用了。

#### 搭建本地Registry

docker hub虽然方便，但是还有些限制。比如：

（1）需要Internet连接，而且下载和上传数据很慢。

（2）上传到Docker Hub的镜像任何人都能够访问，虽然可以使用私有的repository，但不是免费的。

（3）因安全原因很多组织不允许将镜像放到外网。

解决方案就是搭建本地的Registry

Docker已经将Registry开源了，同时Docker Hub上也有官方的镜像registry。下面我们就在docker中运行自己的registry。

**1.启用registry容器**

我们使用的是registry：2

```ABAP
docker run -d -p 5000:5000 -v /myregistry:/var/lib/registry registry2
```

- -d 后台启动容器
- -p 将5000端口映射到host的5000端口。5000是registry服务端口。
- -v 将容器目录/var/lib/registry目录映射到host的/myregistry，用于存放镜像数据。

通过docker tag 重命名镜像，使之与registry匹配。

```ABAP
docker tag tangchen/httpd:v1 registry.example.net:5000/tangchen/httpd:v1
```

我们在镜像的前面已经讨论了镜像名称由repository和tag两部分组成。而repository的完整格式为：

[registry-host]:port/username/xxx

只有docker hub上的镜像可以省略registry-host:[port]

**2.通过docker push上传镜像**

```ABAP
docker push registry.example.net:5000/tahngchen/httpd:v1
```

现在已经可以通过docker pull从本地registry下载镜像了。

```ABAP
docker pull registry.example.net:5000/tahngchen/httpd:v1
```

除了镜像名字长一些（包括registry host和port)，使用方式完全一样。

### 小结

本节我们学习了docker镜像，首先讨论了镜像的分层结构，然后学习了如何构建镜像，最后实践使用docker hub和本地registry。

下面是镜像的常用操作子命令：

- image:显示镜像列表
- history:显示镜像构建历史。
- commit:从容器中创建新镜像。
- build:从dokcerfile构建镜像
- tag: 给镜像打tag。
- pull:从registry下载镜像。
- push:把镜像上传到registry
- rmi:删除docker host中的镜像
- search:搜索docker hub中的镜像。

除了rmi和search其它已经都使用过了。

**1.rmi**

rmi只能删除host上的镜像，不会删除resitry的镜像。

如果一个镜像对应了多个tag，只有当最后一个tag被删除时，镜像才被真正删除。例如host中debian镜像有两个tag，删除其中debian:latest只是删除了latest tag，镜像本身没有删除；只有当debian:jessie也被删除时，整个镜像才会被删除。

**2.search**

search让我们无须打开浏览器，在命令行中就可以搜索docker hub中的镜像，当然，如果想知道镜像都有哪些tag,还是得访问docker hub。

## Docker容器

上一节我们学习了如何构建docker镜像，并通过镜像运行容器。本节将深入讨论容器：学习容器的各种操作、容器各种状态之间的转换，以及实现容器的底层技术。

### 运行容器

docker run是启动容器的方法。在讨论dockerfile时我们已经了解过，可用三种方式指定容器启动时执行的命令。

1. CMD指令
2. ENTRYPOINT指令
3. 在docker run命令行中指定。

例如下面的例子：

```dockerfile
docker run ubuntu pwd #启动容器时执行pwd，返回的/是容器中的当前目录。
```

执行docker ps或docker container ls可以查看docker host中运行的容器。发现没有运行的容器。

![docker容器运行](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker容器运行.png)

再次使用`docker ps -a`查看：-a会显示所有状态的容器，可以看到，之前的容器已经退出了，状态为Exited。

这种“一闪而过”的容器通常不是我们想要的结果，我们希望容器能保持running状态，这样才能被我们使用。

#### 让容器长期运行

如何让容器保存运行呢？

因为容器的生命周期依赖于启动时执行的命令，只要该命令不结束，容器也就不会退出。

理解了这个原理，我们就可以通过执行一个长期运行的命令来保持容器的运行状态。例如执行以下命令：

```dockerfile
docker run ubuntu /bin/bash -c "while true;do sleep 1;done"
```

while语句让bash不会退出。可以打开另一个终端查看容器的状态。发现容器仍处于运行状态。不过这种方法有个缺点：它占用了一个终端。

可以加上参数-d以后台的方式启动容器，容器启动后回到了docker host的终端。这里可以看到docker返回了一串字符，这是容器的ID。通过docker ps查看容器

![docker后台运行容器](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker后台运行容器.png)

现在我们有了两个正在运行的容器。这里注意一下容器的CONTAINER ID合NAMES这两个字段。

CONTAINER ID时容器的“短ID”，前面启动容器时返回的是“长ID”。短ID是长ID的前12个字符。

NAMES字段显示容器的名字，在启动容器时可以通过--name参数显式地为容器命名，如果不指定，docker会自动为容器分配名字。

对于容器的后续操作，我们需要通过“长ID”，“短ID”或者“名称”来指定要操作的容器。比如下面停止一个容器。

```dockerfile
docker stop 5801feb0f7c2
```

这里我们通过短ID指定了要停止的容器。

通过while启动的容器虽然能够保持运行，但实际上没有干什么有意义的事情。容器常见的用途是运行后台服务，例如我们前面我们已经看见的http server。

```dockerfile
docker run -name "my_http_server" -d httpd
```

这一次我们用--name指定了容器的名字。我们还看到容器运行的命令是httpd-foreground,通过docker history可知这个命令是通过CMD指定的。

![指定docker容器名称创建容器](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\指定docker容器名称创建容器.png)

#### 两种进入容器的方法

我们经常需要进到容器里去做一些工作，比如查看日志、调试、启动其它进程等。有两种方式进入容器：attach合exec。

**1.docker attach**

通过docker attach可以attach到容器启动命令的终端，例如：

```dockerfile
docker run -d ubuntu /bin/bash -c "while true;do sleep 1;echo i_am_in_container;done"
#显示长ID
docker attach  长ID 
```

这次我们通过“长ID”attach到了容器的启动命令终端，之后看到的是echo每隔一秒打印的信息。

注：可通过ctrl+p,然后ctrl+q组合键退出attach终端。

> 组合键的使用，容器必须要是-itd进行启动，而不能是-d进行启动

**2.docker exec**

通过docker exec进入相同的容器。

```dockerfile
docker exec -it 短ID bash
```

说明：

1. -it以交互模式打开pseudo-TTY,执行bash，其结果就是打开了一个bash终端。
2. 进入到容器之后，其容器的hostname就是其短ID
3. 可以像在普通的linux中一样执行命令。ps -elf显示了容器启动进程while以及当前的bash进程。
4. 执行exit退出容器，回到docker host。

`docker exec -it <container> bash|sh`

这是执行exec最常用的方式。

**3.attach VS exec**

attach和exec主要区别如下：

1. attach直接进入容器启动命令的终端，不会启动新的进程。
2. exec则是在容器中打开新的终端，并且可以启动新的进程。
3. 如果想直接在终端中查看启动命令的输出，用attach；其它情况使用exec。

当然，如果只是为了查看启动命令的输出，可以使用docker logs命令。

`docker logs -f 容器短ID`

-f的作用与tail -f类似，能够持续打印输出。

#### 运行容器的最佳实践

按用途容器大致可以分为两类：服务类容器和工具类的容器。

服务类容器以daemon的形式运行，对外提供服务，比如Web Server、数据库等。通过 -d以后台方式启动这类容器是非常合适的。如果要排查问题，可以通过exec -it进入容器。

工具类容器通常能给我们提供一个临时的工作环境，通常以run -it方式运行，比如：

`docker run -it busybox`

运行busybox，run -it的作用是在容器启动后就直接进入。我们这里通过wget验证了在容器中访问internet的能力。执行exit退出终端，同时容器停止。

工具类容器多使用基础容器，例如busybox、debian、ubuntu等。

#### 容器运行小结

容器运行相关的知识点：

1. 当CMD、ENTRYPOINT和docker run命令行指定的命令运行结束时，容器停止。
2. 通过-d参数在后台启动容器
3. 通过exec -it可进入容器并执行命令。

指定容器的三种方法：

1. 短ID
2. 长ID
3. 容器名称。可通过--name为容器命名。重命名容器可执行docker rename。

容器的用途可以分为两类：

1. 服务类的容器
2. 工具类的容器

### stop/start/restart 容器

通过docker stop可以停止运行的容器。

容器在docker host中实际上是一个进程，docker stop命令本质上是向该进程发送一个sigterm新号。如果想快速停止容器，可以使用docker kill命令，其作用是向容器进程发送sigkill新号。

对于处于停止状态的容器，可以通过docker start 重新启动。

docker start会保留容器的第一次启动时的所有参数。

docker restart可以重启容器，其作用就是依次执行docker stop和docker start。

容器可能会因为某种错误而停止运行。对于服务类容器，我们通常希望这种情况下容器能够自动重启。启动容器时设置--restart就可以达到这个效果。

--restart=always意味着无论容器因为何种原因退出（包括正常退出），都立即重启；该参数的形式还可以是--restart=on-failure:3，意思是如果启动进程退出代码非0，则重启容器，最多重启三次。

### pause/unpause容器

有时候我们只是希望让容器暂停工作一段时间，比如要对容器的文件系统打个快照，或者docker host需要使用cpu，这时可以执行docker pause。

处于暂停状态的容器不会占用CPU资源，直到通过docker unpause恢复执行。

### 删除容器

使用docker一段时间后，host可能会有大量已经退出了的容器。

这些容器依然会占用host的文件系统资源，如果确认不会再重启此类容器，可以通过docker rm删除。

```dockerfile
docker rm 短id  短id  #可多删除
```

docker rm一次可以指定多个容器，如果希望批量删除所有已经退出的容器，可以执行以下命令：

```dockerfile
docker rm -v ${docker ps -aq -f status=exited}
```

> docker rm是删除容器，docker rmi是删除镜像

### State Machine

前面我们已经讨论了容器的各种操作，对容器的生命周期有了大致的了解。

1. 可以先创建容器，稍后再启动

```dockerfile
docker create httpd
docker ps -a
docker start 短id
```

2. 只有当容器的启动进程退出时，--restart才生效。

退出包括正常 退出和非正常退出。这里举了两个例子：启动进程正常退出或发生OOM，此时Docker会根据--restart的策略判断是否需要重启容器。但如果容器是因为执行docker stop或docker kill退出，则不会自动重启。

### 资源限制

一个docker host上会运行若干个容器，每个容器都需要CPU、内存和IO资源。对于KVM、VMware等虚拟化技术，用户可以控制分配多少CPU、内存资源给每个虚拟机。对于容器，Docker也提供了类似的机制避免某个容器因占用太多资源而影响其它容器乃至整个host的性能。

#### 内存限制

与操作系统类似，容器可以使用的内存包括两部分：物理内存和swap。Docker通过下面两组参数来控制容器内存的使用量。

1. -m或--memory：设置内存的使用限额，例如：200MB、2GB.
2. --memory-swap:设置内存+swap的使用限额。

当我们执行如下命令：

`docker run -m 200M --memory-swap=300M ubuntu`

其含义是允许容器最多使用200M的内存和100M的swap。默认情况下，上面两组参数为-1，即对容器内存和swap的使用没有限制。

下面我们将使用progrium/stress镜像来学习如何为容器分配内存。该镜像可用于对容器执行压力测试。执行如下命令：

```dockerfile
docker run -it -m 200M --memory-swap=300M progrium/stress --vm 1 --vm-bytes 280M
```

- --vm 1:启动1个内存工作线程。
- --vm-bytes 200M:每个线程分配280MB内存。

因为280MB在可分配的范围（300MB）内，所以工作线程能够正常工作，其过程是：

1. 分配280MB内存。
2. 释放280MB内存。
3. 再分配280MB内存。
4. 再释放280MB内存。
5. 一直循环....

如果让工作线程分配的内存超过300MB，stress线程报错，容器退出。

![超出内存限制容器启动失败](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\超出内存限制容器启动失败.png)

如果在启动容器时只指定-m而不指定--memory-swap，那么--memory-swap默认为-m的两倍，比如：

```dockerfile
docker run -it -m 200M ubuntu
```

容器最多使用200MB物理内存和200MB swap.

#### CPU限额

默认设置下，所有容器可以平等地使用hostCPU资源并且没有限制。

Docker可以通过-c或-cpu-shares设置容器使用CPU的权重。如果不指定，默认值为1024.

与内存限额不同，通过-c设置的cpu share并不是cpu资源的绝对数量，而是一个相对的权重值。某个容器最终能分配到的CPU资源取决于它的cpu share占所有容器cpu share总和的比例。

换句话说：通过cpu share可以设置容器使用CPU的优先级。

比如在host中启动了两个容器：

```dockerfile
docker run --name "container_A" -c 1024 ubuntu docker run --name "container_B" -c 512 ubuntu
```

containerA的cpu share 是1024，是containerB的两倍。当两个容器都需要CPU资源时，containerA可以得到的CPU是containerB的两倍。

需要特别注意的是，这种按权重分配CPU只会发生在CPU资源紧张的情况下。如果containerA处于空闲状态，这时，为了充分利用CPU资源，containerB也可以分配到全部可用的CPU。

下面我们继续使用progrium/stress做实验。

1. 启动container_A,cpu share 为1024.

```dockerfile
docker run --name container_A -it -c 1024 progrium/stress --cpu 1
```

--cpu用来设置工作线程的数量。因为当前host只有一颗CPU，所以一个工作线程就能将CPU压满。如果host有多颗CPU，则需要相应添加--cpu的数量。

2.启动container_B,CPU share为512。

```
docker run --name container_B -it -c 512 progrium/stress --cpu 1
```

3.在host中执行top，查看容器对CPU的使用情况，containerA消耗的CPU是containerB的两倍。

4.现在暂停container_A

5.top显示containerB在containerA空闲的情况下能够用满整颗CPU

#### Block IO带宽限额

Block IO是另一种可以限制容器使用的资源。Block IO指的是磁盘的读写，docker可通过设置权重、限制bps和iops的方式控制容器读写磁盘的带宽，下面分别讨论：

> 目前Block IO限额只对direct IO(不使用文件缓存)有效。

**1.block IO权重**

默认情况下，所有容器能平等地读写磁盘，可以通过设置--blkio-weight参数来改变容器block IO的优先级。

--blkio-weight与--cpu-shares类似，设置的是相对权重值，默认为500.在下面的例子中，containerA读写磁盘的带宽是containerB的两倍。

```dockerfile
docker run -it --name container_A --blkio-weight 600 ubuntu docker run -it --blkio-weight 300 ubuntu
```

**2.限制bps和iops**

bps是byte per second，每秒读写的数据量。

iops是io per second,每秒IO的次数。

可通过以下参数控制容器的bps和iops：

- --device-read-bps:限制读某个设备的bps.
- --device-write-bps:限制写某个设备的bps.
- --device-read-iops:限制读某个设备的iops。
- --device-write-iops:限制写某个设备的iops.

下面这个例子限制容器写/dev/sda的速率为30MB/s。

```dockerfile
docker run -it --device-write-bps /dev/sda:30MB ubuntu
time dd if=/dev/zero of=test.out bs=1M count=800 oflag=direct
```

通过dd测试在容器中写磁盘的速度。因为容器的文件系统是在host/dev/sda上的，在容器中写文件相当于对host /dev/sda进行写操作。另外，oflag=direct指定用direct IO方式写文件，这样--direct-wirte-dps才能生效。

![限制容器dps写入速度](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\限制容器dps写入速度.png)

结果表明：bps 25.6MB/S没有超过30MB/s的限速。

作为对比测试，如果不限速：

```dockerfile
docker run -it ubuntu
time dd if=/dev/zero of=test.out bs=1M count=800 oflag=direct
```

![bps不限速进行测试](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\bps不限速进行测试.png)

### 实现容器的底层技术

为了更好地理解容器的特性，本节我们将讨论容器的底层实现技术。

cgroup和namespace是最重要的两种技术。cgroup实现资源限额，namespace实现资源隔离。

#### cgroup

cgroup全称Control Group。Linux操作系统通过cgroup可以设置进程使用cpu、内存和IO资源的限额。--cpu-shares、-m、--device-write-bps实际上就是在配置cgroup。

cgroup到底长什么样子呢？我们可以在/sys/fs/cgroup种找到它。还是用例子来说明，启动一个容器，设置--cpu-shares=512：

```dockerfile
docker run -it --cpu-shares 512 progrium-stress -c 1
```

查看容器ID

```dockerfile
docker ps
```

在/sys/fs/cgroup/cpu/docker目录种中，linux会为每个容器创建一个cgroup目录，以容器长ID命名。

目录中包含所有与cpu相关的cgroup配置，文件cpu.shares保存的就是--cpu-shares的配置，值为512.

同样的，/sys/fs/cgroup/memory/docker和sys/fs/cgroup/blkio/docker 中保存的是内存以及Block IO的cgroup配置。

#### namespace

在每个容器中，我们都可以看到文件系统、问卡等资源，这些资源看上去是容器自己的。拿网卡来说，每个容器都会认为自己有一块独立的网卡，即使host上只有一块物理网卡。这种方式非常好，它使得容器更像一个独立的计算机。

linux实现这种方式的技术是namespace。namespace管理着host中全局唯一的资源，并可以让每个容器都觉得只有它自己在使用它自己。换句话说，namespace实现了容器间资源的隔离。

linux使用了6种namespace，分别对应6种资源：Mount、UTS、IPC、PID、Network和User，下面我们分别讨论：

**1，Mount namespace**

Mount namespace让容器看上去拥有整个文件系统。

容器有自己的/目录，可以执行mount和umount命令，当然我们知道这些操作只在当前容器生效，不会影响到host和其它容器。

**2.UTS namespace**

简单的说，UTS namespace让容器有自己的hostanme。默认情况下，容器的hostname是它的短ID，可以通过-h或者-hostname参数设置。

**3.IPC namespace**

IPC namespace让容器用于自己的共享内存和信号量（semaphore)来实现进程间的通信，而不会与host和其它容器的IPC混在一起。

**4.PID namespace**

前面提到过，容器在host种以进程的形式允许。例如当前host种运行了两个容器。

通过ps axf可以查看容器进程。所有容器进程都挂在dockerd进程下，同时也可以看到容器自己的子进程。如果我们进入到某个容器，

ps就只能看到自己的进程了。

而进程的PID不同于host种对应进程的PID，容器中的PID=1的进程当然也不是host进程的Init进程。也就是说 ：容器拥有自己独立的一套PID，这就是PID namespace提供的功能。

**5.Network namespace**

Network namespace让容器拥有自己独立的网卡、IP、路由等资源，我们会在后面网络章节详细讨论。

**6.User namespace**

User namespace让容器能够管理自己的用户，Host不能看到容器中创建的用户。

![容器内创建用户](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\容器内创建用户.png)

在容器中创建了用户cloudman,但是host中并不会创建对应的用户。

### 小结

本章通过大量实验学习了容器的各种操作以及容器状态之间如何转换，然后讨论了限制容器使用的CPU、内存和Block IO的方法，最后学习了实现容器的底层技术：cgroup和namespace。

下面是容器的常用操作指令：

- create:创建容器
- run:运行容器
- pause:暂停容器
- unpause:取消暂停继续运行容器
- stop:发送sigterm停止容器
- kill:发送sigkill快速停止容器
- start：启动容器
- restart:重启容器
- attach:attach到容器启动进程的终端
- exec:在容器中启动新进程，通常使用“-it”参数
- logs:显示容器启动进程的控制台输出，用“-f”持续打印
- rm:从磁盘中删除容器

## Docker 网络

本节讨论网络，首先学习docker提供的几种原生网络，以及如何创建自定义网络；然后讨论容器之间如何通信，以及容器与外界如何交互。

docker网络从覆盖范围可分为单个host上的容器网络和跨多个host的网络，本章重点讨论前一种。对于更为复杂的多host容器网络，我们会在后面进阶技术章节讨论。

docker安装时会自动在host上创建三个网络，我们可用docker network ls命令查看。

### None网络

顾名思义，none网络就是什么都没有的网络。挂在这个网络下的容器除了lo,没有其它任何网卡。容器创建时，可以通过--network=none指定使用None网络。

```dockerfile
docker run -it --network=none busybox
```

我们不禁会问，这样一个封闭的网络有什么用呢？

其实还真有这样的场景。封闭意味着隔离，一些对安全性要求高并且不需要联网 的应用可以使用none网络。

比如某个网络的唯一用途就是生成随机密码，就可以放到none网络中避免密码被窃取。

当然大部分容器是需要网络的，我们接着看host网络。

### host网络

连接到host网络的容器共享docker host的网络栈，容器的网络配置与host完全一样。可以通过--netwrork=host指定使用host网络。

在容器中可以看到host的所有网卡，并且连hostname也是host的。host网络的使用场景又是什么呢？

直接使用docker host的网络最大的好处就是性能，如果容器对网络传输效率有较高要求，则可以选择host网络。当然不便之处就是牺牲一些灵活性，比如要考虑端口冲突问题，docker host上已经使用的端口就不能再用了。

docker host的另一个用途就是让容器可以直接配置host网络，比如某些跨host的网络解决方案，其本身也是以容器方式运行的，这些方案需要对网络进行配置，比如管理iptables，大家将会在后面的进阶技术章节看到。

下面讨论应用更广泛的bridge网络。

### bridge网络

docker安装时会创建一个命名为docker()的linux bridge。如果不指定--network，创建的容器默认都会挂到docker0上。

![bridge网络](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\bridge网络.png)

当前docker0上没有任何其它网络设备，我们创建一个容器看看有什么变化

![docker0发生变化](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\docker0发生变化.png)

一个新的网络接口veth28c57df被挂到了docker0上，veth28c57df就是新创建容器的虚拟网卡。

下面看以下容器的网络配置。

![容器网络配置](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\容器网络配置.png)

容器有一个网卡eth0@if34,为什么不是veth28c57df呢？

实际上eth0@if34和veth28c57df是一对veth pair。veth pair是一种成对出现的特殊网络设备，可以把他们想象成由一根虚拟网线连接起来的一对网卡，网卡的一头（eth0@if34)在容器中，另一头（veth28c57df）挂在网桥docker0上，其效果就是将eth0@if34也挂在了docker0上。

我们还看到了eth0@if34已经配置了ip 172.17.0.2，为什么是这个网段呢？让我们通过docker network inspect bridge看一下bridge网络的配置信息。

![bridge网络ip](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\bridge网络ip.png)

原来bridge网络配置的subnet就是172.17.0.0/16，并且网关是172.17.0.1。这个网关在哪儿呢？大概你也猜出来了，就是docker0.

![容器网络拓扑结构图](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\容器网络拓扑结构图.png)

容器创建时，docker会自动从172.17.0.0/16中分配一个IP，这里16位的掩码保证有足够多的IP可以供容器使用。

### user-defined网络

除了none、host、bridge这三个自动创建的网络，用户也可以根据业务需要创建user-defined网络。

docker提供了三种user-defined网络波动：bridge、overlay和macvlan。overlay和macvlan用于创建跨主机的网络。

我们可以通过bridge驱动创建类似前面默认的bridge网络。

```dockerfile
docker network create --driver bridge my_net
```

![创建自定义网络](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\创建自定义网络.png)

查看一下当前host的网络拓扑变化

![新增自定义网络](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\新增自定义网络.png)

新增了一个网桥刚好是my_net对应的短ID。

```dockerfile
docker network inspect my_net
```

![新建网络配置信息](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\新建网络配置信息.png)

这里的172.18.0.0/16是docker自动分配的IP网段。

我们新建网卡的时候可以自己指定IP网段。

```dockerfile
docker network create --driver bridge --subnet 172.22.16.0/24 --geteway 172.22.16.1 my_net2
```

这里我们创建了新的bridge网络my_net2，网段位172.22.16.0、24，网关为172.22.16.1

容器要使用新的网络，需要在启动时通过--network指定。

```dockerfile
docker run -it --network=my_net2 busybox
```

到目前为止，容器的IP都是docker自动从subnet中分配，我们能否指定一个静态IP呢？

```dockerfile
docker run -it --network=my_net2 --ip 172.22.16.8 busybox
```

> 只有使用subnet创建的网络才能指定静态IP。

my_net创建时没有指定--subnet，如果指定静态IP则会报错。

处于同一网段的容器可以进行互通，而不同网桥的两个容器是不能够互通 的。

不同网络如果加上路由的话能不能互通呢？

如果host上对每个网络都有一条路由，同时操作系统上打开了ip forwarding,host就成了一个路由器，挂接在不同桥段上的网络就能够互相通信。

ip r查看host上的路由表。

systemctl net..ipv4.ip_forward 查看ip forwarding是否启用。

iptables-save查看iptables：iptables drop掉了网桥docker0与br-5d863e9f78b6之间的双向流量。

从规则上命名docker-isolation可知docker在设计上就是要隔离不同的network。

怎么才能够让busybox与httpd通信呢？：为httpd容器添加一块net_my2的网卡。

```dockerfile
docker network connect my_net2 短ID
```

我们可以在httpd容器内查看容器的网络配置，可以发现增加了一张网卡分配了my_net2的IP，现在busybox应该能访问httpd了。

### 容器间通信

容器之间可通过IP、docker DNS Server或joined容器三种方式进行通信。

#### IP通信

两个容器之间要能通信，必须要有属于同一个网络的网卡。满足这个条件之后，容器就可以通过IP交互了。具体做法是在容器创建时通过--network指定相应网络，或者通过 docker network connect将现有容器加入到指定网络。

#### docker DNS Server

通过IP访问容器虽然满足了通信的需求，但还是不够灵活。因为在部署应用之前可能无法确定IP，部署之后再指定要访问的IP会比较麻烦。对于这个问题，可以通过docker 自带的DNS服务解决。

从Docker 1.10版本开始，docker daemon实现了一个内嵌的DNS server，使容器可以直接通过”容器名“通信。方法很简单，只要在启动时用--name为容器命名就可以了。

下面启动两个容器bbox1和bbox2：

```dockerfile
docker run -it --network=my_net2 --name=bbox1 busybox docker run -it --network=my_net2 --name=bbox2 busybox
```

然后，bbox2就可以直接ping到bbox1了。

使用docker DNS Server有一个限制：只能在**user-defined网络**中使用。也就是说默认的bridge网络是无法使用DNS的。

#### joined容器

joined容器是另一种实现容器间通信的方式。

joined容器非常特别，它可以使两个或多个容器共享一个网络栈，共享网卡和配置信息，joined容器之间可以通过127.0.0.1直接通信。

先创建一个容器，命名为web1。

```dockerfile
docker run -d -it --name=web1 httpd
```

然后创建busybox容器并且通过--network=container:web1指定joined容器为web1。

```dockerfile
docker run -it --network=container:web1 busybox
```

请注意busybox容器中的网络配置信息

```dockerfile
docker exec -it web1 bash #进入web1即httpd容器
ip a #linux的host中查看ip信息
```

可以发现bosybox和web1的网卡mac地址与IP完全一样，它们共享了相同的网络栈。busybox可以直接用127.0.0.1访问web1的http服务。

joined容器的使用场景：

- 不同的容器中的程序希望通过loopback高效快速地通信，比如Web Server与App Server
- 希望监控其它容器的网络流量，比如运行在独立容器中的网络监控程序。

### 将容器与外部世界连接

前面我们已经解决了容器间通信的问题，接下来讨论容器如何与外部世界通信。这里涉及两个方向：

1. 容器访问外部世界
2. 外部世界访问容器

#### 容器访问外部世界

在我们当前的实验环境下，docker host是可以直接访问外网的。

那么容器是否可以访问外网呢？答案是容器默认也是可以访问外网的。

注意：这里的外网指的是容器网络以外的网络环境，并非特指Internet。

现象很简单，但更重要的是：我们应该理解现象下的本质。

在上面这个例子中，busybox位于docker0这个私有bridge网络中（127.0.0.0/16），当busybox从容器向外ping时，数据包是怎样到达bing.com的呢？

这里的关键就是NAT，我们查看一下docker host上的iptables规则，可以发现：

```ABAP
-A POSTROUTING -s 172.17.0.0/16 ！ -o docker0 -j MASQUERADE
```

其含义是：如果网桥docker0收到来自172.17.0.0/16网段的外出包，把它交给masquerade处理.而masquerade的处理方式是将包的源地址替换成host的地址发送出去，即做了一次网络地址转换NAT。

下面我们看看tcpdump查看地址是如何转换的。先查看docker host的路由表。

默认路由通过enp0s3发送出去，所以我们要同时监控enp0s3和docker0上的icmp（ping)数据包。

当busybox ping bing.com时，tcpdump输出如下：

docker0收到busybox的ping包，源地址为容器IP 172.17.0.2，这没问题，交给maquerade处理。这时,在enp0s3上我们看见了变化。

ping包的源地址变成了enp0s3的IP 10.0.2.15。

这就是iptable NAT规则处理的结果，从而保证数据包能够到达外网。

![container NetWork](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\container NetWork.png)

1. busybox发送ping包：172.17.0.2>www.bing.com
2. docker0收到包，发现是发送给外网的，交给NAT处理
3. NAT将源地址换成enp0s3的ip：10.0.2.15>www.bing.com
4. Ping包从enp0s3发送出去，到达www.bing.com

通过NAT，docker实现了容器对外网的访问。

#### 外部世界访问容器

下面我们来讨论另外一个方向：外网如何访问容器？

答案是端口映射

docker可将容器对外提供服务的端口映射到host的某个端口，外网通过该端口访问容器。容器启动时通过-p参数映射端口。

```dockerfile
docker run -d -p 80 httpd
```

容器启动后可以通过docker ps 或docker port查看到host映射的端口。在上面的例子中，http容器的80端口被映射到host 32773上，这样就可以通过<host ip>:<32773>访问容器的web服务了。

除了映射动态端口，也可以在-p中指定映射到host的某个指定端口，例如可将80端口映射到host的8080端口。

```dockerfile
docker run -d -p 8080:80 httpd
```

每一个映射的端口，host都会启动一个docker-proxy进程来处理访问容器的流量。

```ABAP
ps -ef|grep docker-proxy
```

以0.0.0.0:32773->80/tcp为例分析整个过程。

![外部访问容器](C:\Users\汤琛\Desktop\学习资料\常用中间件\docker\images\外部访问容器.png)

1. docker-proxy监听Host的32773端口
2. 当curl访问10.0.2.15：32773时，docker-proxy转发给容器172.17.0.2：80
3. httpd容器相应请求并返回结果。

### 小结

在本节中，我们学习了docker网络中的三种网络：none、host和bridge，并讨论了它们不同的使用场景；然后我们实践了创建自定义网络：最后详细讨论了如何实现容器与容器之间、容器与外部网络之间的通信。

## Docker 存储

Docker为容器提供了两种存放数据的资源：

1. 由storage driver 管理的镜像蹭和容器层
2. Data Volume

### storage driver

容器由最上面一个可写的容器层，以及若干只读的镜像层组成，容器的数据就存放在这些层中。这样分层结构最大的特性就是：Copy-in-Write：

1. 新数据会直接存放在最上面的容器层。
2. 修改现有数据会先从镜像层将数据复制到容器层，修改后的数据直接保存在容器层中，镜像层中保持不变。
3. 如果多个层中有命名相同的文件，用户只能看到最上层的文件。

分层结构使镜像和容器的创建、共享以及分发变得非常高效，而这些都要归功于docker storage driver。正是storage driver实现了多层数据的堆叠并为用户提供了一个单一的合并之后的统一视图。

docker支持多种storage driver，有AUFS、Device Mapper、Btrfs、OverlayFS、VFS和ZFS。它们都能够实现分层的结构，同时又有各自的特性。对于docker用户来说，具体选择使用哪个storage driver是一个难题，因为：

1. 没有哪个driver能够适应所有的场景。
2. driver本身在快速发展和迭代。

不过docker官方给出了一个简单的答案：优先使用linux发行版本默认的storage driver。

docker安装时会根据当前系统的配置选择默认的driver。默认的driver具有最好的稳定性，因为默认的Driver在发行版上经历了严格的测试。

运行docker info查看host的默认driver。

```dockerfile
docker info
```

Ubuntu默认的driver是aufs，底层文件系统是extfs，各层数据存放在/var/lib/docker/aufs。

Redhat/CentOS的默认driver是Driver Mapper，SUSE则是Btrfs。

对于某些容器，直接将数据放在由storage driver维护的层中是很好的选择，比如哪些无状态的应用。无状态意味着容器没有需要持久化的数据，随时可以从镜像直接创建。

比如busybox，它是一个工具箱，启动busybox是为了执行诸如wget、ping之类的命令，不需要保存数据供以后使用，使用完直接退出，容器删除时存放在容器层的工作数据也会一起被删除，这没问题，下次再启动新的容器即可。

但对于另一类应用这种方式就不合适了，它们有持久化数据的需求，容器启动时需要加载已有的数据，容器销毁时希望保留产生的新数据，也就是说这类容器是有状态的。

这就要用到docker的另一种存储机制：data volume。

### Data Volume

Data Volume本质上是Docker host文件系统中的目录或文件，能够直接被mount到容器的文件系统中。Data Volume有以下特点：

1. Data Volume是目录或文件，而非没有格式化的磁盘（块设备）。
2. 容器可以读写volume中的数据。
3. volume数据可以被永久地保存，即使使用它的容器被销毁。

好，现在我们有数据层（镜像层和容器层）和volume都可以用来存放数据，具体使用的时候要怎么选择呢？考虑下面几个场景：

- Database软件 vs Database数据
- Web应用  vs  Web应用产生的日志
- 数据分析软件  vs input/output数据。
- Apache Server vs静态HTML文件

相信大家会做出这样的选择：

- 前者放在数据层中。因为这部分内容是无状态的，应该作为镜像的一部分。
- 后者放在Data Volume中。这是需要持久化的数据，并且应该与镜像分开存放。

还有个大家关心的问题：如何设置Data Volume的容量？

因为volume实际上是docker host文件系统的一部分，所以volume的容量取决于文件系统当前未使用的空间，目前还没有方法设置volume的容量。

在具体的使用上，docker提供了两种类型的volume: bind volume和 docker managed volume。

#### bind volume

bind volume是将host上已存在的目录或文件mount到容器。

例如：dokcer host上有目录$HOME/htdocs

通过-v将其mount到httpd容器。

```dockerfile
docker run -d -p 80:80 -v ~/htdocs:/usr/local/apache2/htdocs httpd
```

-v的格式<host path>:<container path>。/usr/local/apache2/htdocs 就是Aapche Server存放静态资源的地方。由于/usr/local/apache2/htdocs已经存在，原有数据会被隐藏起来，取而代之的是host $HOME/htdocs/中的数据，这与Linux mount命令的行为是一致的。

```dockerfile
curl 127.0.0.1:80
#curl 可以发现确实是host目录中文件的对应内容
```

此时对host中的文件内容进行修改，再次查看容器内对应的文件，可以发现容器内的文件内容也被修改了，由此可见bind mount可以让host与容器共享数据。这在管理上是非常方便的。

如果现在将容器销毁了，bind mount会有什么影响呢？

```dockerfile
docker stop 短ID
dokcer rm 短ID
cat ~/htdocs.index.html
```

执行完上面命令之后可以发现，即使容器没有了，bind mount依然存在。这也合理，bind mount是host文件系统中的数据，只是借给容器使用，删除容器并不能影响host的文件数据。

另外，bind mount时还可以指定数据的读写权限，默认是可读写的，可指定为可读。

```dockerfile
docker run -d -p 80:80 -v ~/htdocs:/usr/local/apache2/htdocs:ro httpd # ro指定为可读
```

ro设置了只读权限，在容器中是无法对bind mount数据进行修改的。只有host有权修改数据，提高了安全性。

除了bind mount目录，还可以单独指定一个文件。

```dockerfile
docker run -d -p 80:80 -v ~/htdocs/index.html:/usr/local/apache2/htdocs/new_index.html httpd
```

使用bind mount 单个文件的场景是:只需要向容器添加文件，不希望覆盖整个目录。在上面例子中，我们将Html文件加到apache中，同时也保留了原有的数据。

使用单一文件有一点要注意：host中源文件必须要存在，不然会当作一个新目录bind mount给容器。

mount point有很多应用场景，比如我们可以将源代码目录mount到容器中，在host中修改代码就可以看到应用的实时效果。再比如将MySQL容器的数据放到bind mount里， 这样host可以方便的备份和迁移数据。

bind mount的使用直观高效，易于理解，但它也有不足的地方：bind mount需要指定host文件系统的特定路径，这就限制了容器的可移植性，当需要将容器迁移到其它host，而该host没有要mount的数据或者数据不在相同的路径时，会操作失败。

移植性更好的方式是docker managed volume。

#### docker managed volume

docker managed volume与bind mount在使用上的最大区别是不需要指定Mount源，指明mount point就行了。还是以httpd容器为例。

```dockerfile
docker run -d -p 80:80 -v /usr/local/apache2/htdocs httpd
```

我们通过-v告诉docker需要一个data volume，并将其mount到/usr/local/apache2/htdocs。那么这个data volume具体在哪儿呢？

这个答案可以在容器的配置信息中找到：

```dockerfile
docker inspect 对应容器的短ID
```

会发现输出信息很多，只需要关注mount这部分，这里会显示容器当前使用的所有data volume，包括bind mount 和docker managed volume。

source就是该volume在host上的目录。

原来，每当容器申请Mount docker managed volume时，docker都会在/var/lib/docker/volumes下生成一个新的目录，这个目录就是mount源。

继续研究这个volume，看看里面有些什么东西。

ls查看目录下的文件可以发现volume内容和容器原有的/usr/local/apache2/htdocs完全一样。这是怎么回事呢？

这是因为：如果mount point指向的是已有的目录，原数据会被复制到volume中。

但要明确一点：此时的/usr/local/apahce2/htdocs已经不再由storage driver管理的层数据了，它已经是一个data volume。我们可以像bind mount一样对数据进行操作，例如更新数据。

简单回顾一下docker managed volume的创建过程：

1. 容器启动时，简单的告诉docker“我需要一个volume存放数据，帮我mount到目录/abc”。
2. docker在/var/lib/docker/volumes中生成一个随机目录作为mount源。
3. 如果/abc存在，则将数据复制到mount源。
4. 将volume mount到/abc。

除了通过docker Inspect查看volume，我们也可以用docker volume命令。

```dockerfile
docker volume ls
dokcer volume inspect volume短ID或长ID
```

目前，docker volume只能查看docker managed volume，还看不到bind mount；同时也无法知道volume对应的容器，这些信息还得靠dokcer inspect。

我们已经学习了两种data volume的原理和基本使用方法，下面做一个对比：

1. 相同点：两者都是host文件系统中的某个路径。

2. 不同点：

   |         不同点          |         bind mount         |   docker managed volume    |
   | :---------------------: | :------------------------: | :------------------------: |
   |       volume位置        |         可任意指定         | /var/lib/docker/volumes... |
   | 对已有的mount point影响 |     隐藏并替换为volume     |    原有数据复制到volume    |
   |    是否支持单个文件     |            支持            |     不支持，只能是目录     |
   |        权限控制         | 可设置为可读，默认为可读写 |    无控制，均为读写权限    |
   |         移植性          | 移植性弱，与host path绑定  | 移植性强，无须指定host目录 |

### 数据共享

数据共享是volume的关键特性，本节我们将讨论通过volume如何在容器与host之间、容器与容器之间共享数据。

#### 容器与host共享数据

有两种类型的data volume，它们均可实现在容器与host之间共享数据，但方式有所区别。

对于bind mount是非常明确的：直接将要共享的目录mount到容器。

docker managed volume就要麻烦点。由于volume位于host中的目录，是在容器启动时才生成，所以需要将共享数据复制到volume。

docker cp可以在容器和host之间复制数据，当然我们也可以直接通过linux的cp命令复制到/var/lib/docker/volumes/xxx

#### 容器之间共享数据

第一种方法是将共享数据放在bind mount中，然后将其mount到多个容器。还是以httpd为例，不过这次的场景复杂些，我们要创建由三个httpd容器组成的Web server集群，它们使用相同的html文件。操作如下：

1. 将$HOME/htdocs mount到三个httpd容器

```dockerfile
docker run --name web1 -d -p 80 -v ~/htdocs:/usr/local/apache2/htdocs httpd
docker run --name web2 -d -p 80 -v ~/htdocs:/usr/local/apache2/htdocs httpd
docker run --name web3 -d -p 80 -v ~/htdocs:/usr/local/apache2/htdocs httpd
```

2. 查看当前主页内容

```dockerfile
docker ps
curl 127.0.0.1:端口号1
curl 127.0.0.1:端口号2
curl 127.0.0.1:端口号3
```

3. 修改volume中的主页内容，再次查看并确认所有的容器都使用的新的主页。

```dockerfile
echo "this is a new index page for web cluster!" -> ~/htdocs/index.html
curl 127.0.0.1:端口号1
curl 127.0.0.1:端口号2
curl 127.0.0.1:端口号3
```

另一种在容器之间共享数据的方式是使用volume container。

### volume container

volume container是专门为其它容器提供volume的容器。它提供的卷可以是bind mount,也可以是docker managed volume。下面我们创建一个volume container。

```dockerfile
docker create --name vc_data \
		-v ~/htdocs:/usr/local/apahce2/htdocs \
		-v /other/userful/tools \
		busybox
```

我们将容器命名为vc_data（vc是volume container的缩写）。注意这里执行的是docker create命令，这是因为volume container 的作用只是提供数据，它本身不需要处于运行状态。容器mount 了两个 volume：

1. bind mount,存放web server的静态文件。
2. docker managed volume ，存放一些实用工具（当然现在是空的，这里只是做个实例）

通过docker inspect 可以查看到这两个volume。

```dockerfile
docker inspect vc_data
```

其它容器可以通过--volumes-from使用vc_data这个volume container。

```dockerfile
docker run --name web1 -d -p 80 --volume-from vc_data httpd
docker run --name web2 -d -p 80 --volume-from vc_data httpd
docker run --name web3 -d -p 80 --volume-from vc_data httpd
```

三个httpd容器都使用了vc_data，看看它们有哪些volume，以web1为例：

```dockerfile
docker inspect web1
```

web1容器使用的就是vc_data的volume，而且连mount point都是一样的。验证一下共享结果。

```dockerfile
#修改host的volume
echo "this is a new index page for web cluster!" -> ~/htdocs/index.html
#查看三个容器中的volume
curl 127.0.0.1:端口号1
curl 127.0.0.1:端口号2
curl 127.0.0.1:端口号3
```

可见，三个容器已经成功共享了volume container 中的volume。

下面我们讨论一下volume container的特点：

1. 与bind mount相比，不必为每一个容器指定host path，所有的path都在volume container中定义好了，容器只需要与volume conatiner关联，实现了容器与host的解耦。
2. 使用volume container的容器，其mount point是一致的，有利于配置的规范和标准化，但也带来了一定的局限性，使用时需要综合考虑。

### data-packed volume container

上面的例子中volume container的数据归根到底还是在host里，有没有办法将数据完全放到volume container中，同时又能与其他容器共享呢？

当然可以，通常我们称这种容器为：data-packed volume container。其原理是将数据打包到镜像中，然后通过docker managed volume共享。

我们用下面的dockerfile构建镜像：

```dockerfile
from busybox:latest
add htdocs /usr/local/apache2/htdocs
volume /usr/local/apahce2/htdocs
```

add将静态资源添加到容器目录/usr/local/apache2/htdocs。

volume 的作用与-v等效，用来创建docker managed volume，mount point为/usr/local/apahce2/htdocs，因为这个目录就是add添加的目录，所以会将已有数据复制到volume中。

build新镜像datapacked。

```dockerfile
docker build -t datapacked .
```

用新镜像创建data-packed volume container。

```dockerfile
docker create --name vc_data datapacked
```

因为在dockerfile中已经使用了volume指令，这里就不需要指定volume的mount point了。启动httpd容器并使用data-packed volume container。

```dockerfile
docker run -d -p 80:80 --volume-from vc_data httpd
```

容器能够正确读取volume的数据。data-packed volume container是自包含的，不依赖host提供的数据，具有很强的移植性，非常适合只使用静态数据的场景，比如应用的配置信息、web server的静态文件等。

### Data Volume 生命周期管理

Data Volume中存放的是重要的应用数据，如何管理volume对应用至关重要。前面我们主要关注的是volume的创建、共享和使用，本节将讨论如何备份、恢复、迁移和销毁volume。

#### 备份

因为volume实际上是host文件系统中的目录和文件，所以volume的备份实际上是对文件系统的备份。

还记得前面我们是如何搭建本地的registry吗？

```dockerfile
docker run -d -p 5000:5000 -v /myregistry:/var/lib/registry registry:2
```

所有的本地镜像都保存在host的/myregistry目录中，我们要做的就是定期备份这个目录。

#### 恢复

volume的恢复也非常简单，如果数据损坏了，直接用之前备份的数据复制到/myregistry就可以了。

#### 迁移

如果我们想使用更新版本的registry，这就涉及数据迁移，方法是：

1. docker stop 当前Registry容器。
2. 启动新版本容器并mount原有的volume。

```dockerfile
docker run -d -p 5000:5000 -v /myregistry:/var/lib/registry
```

当然，在启用新容器前要确保新版本的默认数据路径是否发生了改变。

#### 销毁

可以删除不需要的volume，但一定要确保知道自己在做什么，volume删除数据后是找不回来的。

docker不会销毁bind mount，删除数据的工作只能由host负责。对于docker managed volume，在执行docker rm删除容器时可以带上-v参数，docker会将容器使用到的volume一并删除，但前提是没有其它容器mount该volume，目的是保护数据，非常合理。

如果删除容器时没有带-v呢？这样会产生孤儿volume，好在docker提供了volume 子命令可以对docker managed volume进行维护。

```dockerfile
docker volume ls
docker run --name bbox -v /test/data busybox
docker volume ls
```

容器bbox使用的docker managed volume可以通过docker volume ls看到。

删除bbox如下：

```dockerfile
docker rm bbox
docker volume ls
```

因为没有使用-v,volume遗留了下来。对于这样的孤儿volume，可以使用docker volume rm删除。

```dockerfile
docker volume rm volume长ID
```

如果想批量删除孤儿volume，可以执行

```dockerfile
docker volume rm $(docker volume ls -q)
```

### 小结

- docker为容器提供了两种存储资源：数据层和data volume。
- 数据层包括容器层和镜像层，由storage driver管理
- data volume有两种类型：bind mount和docker managed volume。
- bind mount可以实现容器与host之间、容器与容器之间共享数据。
- volume container是一种具有更好移植性的容器间数据共享方案，特别是data-packed volume container。
- 最后我们学习了如何备份、恢复、迁移和销毁data volume。

