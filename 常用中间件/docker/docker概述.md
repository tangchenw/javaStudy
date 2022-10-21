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

Docker daemon是服务器组件，以linux后台服务的方式运行。如下图所示：

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

（1）添加文件。在容器中创建文件时，新文件
