### 简介
>- Docker是一个开源的应用容器引擎，开发者可利用Docker打包自己的应用及依赖包到一个可移植的容器中，然后发布到任何流行的Linux机器，也可实现虚拟化。
>- 容器与管理程序虚拟化有所不同，容器是直接运行在操作系统内核上的用户空间，后者是通过中间层将一台或多台独立的机器虚拟运行在物理硬件上。
>- 容器“客居”于操作系统，只能运行与底层宿主机相同或相似的操作系统。
>- 容器需要的开销有限，和传统虚拟化及半虚拟化相比，容器运行不需模拟层和管理层，而使用操作系统的系统调用接口，降低了运行单个容器所需开销，使得宿主机可运行更多的容器。
### docker简介
>- Docker在虚拟化的容器执行环境中增加了一个应用程序部署引擎。
>>该引擎的目标就是提供一个轻量、快速的环境，能够运行开发者的程序，并方便高效地将程序从开发者的笔记本部署到测试环境，然后再部署到生产环境。Docker极其简洁，它所需的全部环境只是一台仅仅安装了兼容版本的Linux内核和二进制文件最小限的宿主机。

- 提供一个简单、轻量的建模方式
>Docker上手很快，只需几分钟，即可把自己的程序“Docker化”。大多数Docker容器只需不到1秒钟即可启动。由于去除了管理程序的开销，Docker容器拥有很高的性能，同一台宿主机中也可运行更多容器，尽可能充分利用系统资源。

- 职责的逻辑分离
>开发人员只需关心容器中运行的应用程序，而运维人员只需关心如何管理容器。

- 快速、高效的开发生命周期
>Docker目标：缩短代码从开发、测试到部署、上线运行的周期，让程序具备可移植性，易于构建，并易于协作。

- 鼓励使用面向服务的架构
>Docker推荐单个容器只运行一个应用程序或进程，形成一个分布式应用程序模型，使分布式部署应用程序，扩展或调试都变得非常简单，同时也提高了程序的内省性。(也可不拘泥此模式，在一个容器内运行多个进程的应用程序。)

### Docker组件
核心组件：
>Docker客户端和服务器（Docker引擎）；
Docker镜像；  
Registry；  
Docker容器。

- Docker客户端和服务器
>Docker客户端只需向Docker服务器或守护进程（有时称Docker引擎）发出请求，服务器或守护进程将完成所有工作并返回结果。  
Docker提供了一个命令行工具docker以及一整套RESTful API来与守护进程交互。  
用户可在同一台宿主机上运行Docker守护进程和客户端，也可本地Docker客户端连接到远程服务器的Docker守护进程。

![image](http://s6.51cto.com/wyfs02/M02/6B/79/wKiom1UuO5zQCnskAABiPf3ycV4025.jpg)

- docker镜像
>镜像是构建Docker世界的基石。用户基于镜像来运行自己的容器。镜像也是Docker生命周期中的“构建”部分。镜像是基于联合（Union）文件系统的一种层式的结构，由一系列指令一步一步构建出来。  
也可把镜像当作容器的“源代码”。镜像体积很小，非常“便携”，易于分享、存储和更新。

- Registry
>Docker用Registry来保存用户构建的镜像。Registry分为公共和私有两种。Docker公司运营的公共Registry叫作Docker Hub。用户可以在Docker Hub注册账号 ，分享并保存自己的镜像。
用户也可以架设自己的私有Registry（可受到防火墙的保护，将镜像保存在防火墙后面，以满足一些组织的特殊需求）。

- 容器
>Docker可帮用户构建和部署容器，用户只需把自己的应用程序或服务打包放进容器即可。  
容器基于镜像启动的，容器可运行一或多个进程。镜像是Docker生命周期中的【构建或打包阶段】，而容器则是【启动或执行阶段】。  
总结，容器就是：一个镜像格式；一系列标准的操作；一个执行环境。  
Docker借鉴了标准集装箱的概念，Docker运输软件。  
每个容器都包含一个软件镜像（“货物”），镜像可被创建、启动、关闭、重启以及销毁。   
和集装箱一样，Docker执行上述操作时，并不关心容器中到底塞进了什么，所有容器都按照相同的方式将内容“装载”进去。  
它也不关心用户要把容器运到何方。它方便替换，可叠加，易分发，且尽量通用。   
使用Docker，可快速构建一个应用程序服务器、一个消息总线、一套实用工具、一个持续集成测试环境或任意一种应用程序、服务或工具。

### 能用Docker做什么
>①加速本地开发和构建流程，使其更加高效、更加轻量化。本地开发人员可以构建、运行并分享Docker容器。容器可在开发环境中构建，然后轻松地提交到测试环境，并最终进入生产环境。  
②能够让独立服务或应用程序在不同的环境中，得到相同的运行结果。  
③用Docker创建隔离的环境进行测试。eg.Jenkins持续集成工具启动一个用于测试的容器。  
④Docker可让开发者先在本机上构建一个复杂的程序或架构来进行测试，而不是一开始就在生产环境部署、测试。  
⑤构建一个多用户的平台即服务（PaaS）基础设施。  
⑥为开发、测试提供一个轻量级的独立沙盒环境，或者将独立的沙盒环境用于技术教学。  
⑦提供软件即服务（SaaS）应用程序。  
⑧高性能、超大规模的宿主机部署。

### docker与配置管理
与传统的镜像模型相比，Docker就显得轻量多了：镜像是分层的，可以对其进行迅速的迭代。  
Docker显著特点：对不同的宿主机、应用程序和服务，可能会表现出不同的特性与架构：Docker可短生命周期的，也可用于恒定的环境，可用一次即销毁，也可提供持久服务。

### Docker的技术组件
Docker可运行于任何安装了现代Linux内核的x64主机上。它包括以下几个部分：
>①一个原生的Linux容器格式，Docker中称为libcontainer。    
②Linxu内核的命名空间 ，用于隔离文件系统、进程和网络。    
③文件系统隔离：每个容器都有自己的root文件系统。    
④进程隔离：每个容器都运行在自己的进程环境中。    
⑤网络隔离：容器间的虚拟网络接口和IP地址都是分开的。    
⑥资源隔离和分组：使用cgroups（即control group，Linux的内核特性之一）将CPU和内存之类的资源独立分配给每个Docker容器。  
⑦写时复制 ：文件系统都是通过写时复制创建的，这就意味着文件系统是分层的、快速的，而且占用的磁盘空间更小。  
⑧日志：容器产生的STDOUT、STDERR和STDIN这些IO流都会被收集并记入日志，用来进行日志分析和故障排错。  
⑨交互式shell：用户可以创建一个伪tty终端，将其连接到STDIN，为容器提供一个交互式的shell。