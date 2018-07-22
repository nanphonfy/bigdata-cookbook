### 确保Docker已经就绪
①查看Docker是否能正常工作；  
`docker info`  
（返回所有容器和镜像的数量、docker使用的执行驱动和存储存储，以及Docker基本配置）

②学习基本Docker工作流：创建并管理容器。  
容器典型生命周期：创建、管理、停止、删除。

### 运行我们的第一个容器
>docker run：Docker容器的创建到启动

`docker run -i -t centos /bin/bash`
>列出完整的docker命令列表  
docker help 或 man docker-run

>-i标志保证容器中STDIN是开启的，-t标志告诉Docker为要创建的容器分配一个伪tty终端（这样，新创建的容器才能提供一个交互式shell）。  
接下来，我们告诉Docker基于什么镜像来创建容器。  
>>常备镜像（“基础”镜像）由Docker公司提供，保存在DockerHub 的 Registry上。可以以ubuntu、fedora、debian、centos等镜像为基础，在选择的操作系统上构建自己的镜像。

>首先Docker检查本地是否存在centos镜像，若没有，Docker会连接Docker Hub Registry，查看是否有该镜像。找到会下载并将其保存到本地宿主机。  
随后，Docker在文件系统内部用这个镜像创建了一个新容器。该容器拥有自己的网络、IP地址，以及一个用来和宿主机进行通信的桥接网络接口。最后，我们告诉Docker在新容器中要运行什么命令，在本例我们在容器中运行/bin/bash命令启动了一个Bash shell。  
当容器创建完毕之后，Docker就会执行容器中的/bin/bash命令，这时就可以看到容器内的shell了：  
>>root@f7cbdac22a02:/#

### 使用第一个容器
- 启动和进入容器
[root@localhost ~]# docker run -i -t centos /bin/bash

- 检查容器的主机名
[root@64400cb50071 /]# hostname
64400cb50071

- 检查容器的/etc/hosts文件
[root@64400cb50071 /]# cat /etc/hosts
```Linux
127.0.0.1	localhost
::1	localhost ip6-localhost ip6-loopback
fe00::0	ip6-localnet
ff00::0	ip6-mcastprefix
ff02::1	ip6-allnodes
ff02::2	ip6-allrouters
172.17.0.2	64400cb50071
```

- 检查容器的进程
[root@64400cb50071 /]# ps -aux
```Linux
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
root         1  0.0  0.1  11820  1916 ?        Ss   18:53   0:00 /bin/bash
root        15  0.0  0.0  51708  1712 ?        R+   18:55   0:00 ps -aux
```

只有指定的/bin/bash命令处于运行状态时，容器才处于运行状态。一旦退出容器，/bin/bash命令也结束了，这时容器随之停止。
但容器仍存在，可用docker ps -a命令查看当前系统中容器的列表。

- 列出Docker容器
[root@localhost ~]# docker ps -a

```Linux
CONTAINER ID        IMAGE               COMMAND                  CREATED             STATUS                           PORTS               NAMES
64400cb50071        centos              "/bin/bash"              4 minutes ago       Exited (130) 1 second ago                            unruffled_joliot
b7c645a7a8c5        centos              "/bin/sh -c 'while..."   About an hour ago   Exited (137) About an hour ago                       daemon_dave
eac7e8c4f9fd        centos              "/bin/bash"              About an hour ago   Exited (137) About an hour ago                       bob_the_container
cb88bdbe9df5        centos              "/bin/bash"              About an hour ago   Exited (137) About an hour ago                       zsr_the_container
949a715e4d0f        centos              "/bin/bash"              About an hour ago   Exited (0) About an hour ago                         zealous_pasteur
```
>默认，当执行docker ps命令时，只能看到正在运行的容器。指定-a标志：docker ps命令会列出所有容器，包括正在运行和已停止的。

- 列出最后一个运行的容器
docker ps -l

- 3种方式可以唯一指代容器
短UUID（如f7cbdac22a02）、长UUID（如f7cbdac
22a02e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778）或者名称（如gray_cat）

```
--centos7
yum install subscription-manager
 
Dependency Installed:
  audit-libs-python.x86_64 0:2.8.1-3.el7                           checkpolicy.x86_64 0:2.5-6.el7                             
  container-selinux.noarch 2:2.66-1.el7                            container-storage-setup.noarch 0:0.10.0-1.gitdf0dcd5.el7   
  docker-client.x86_64 2:1.13.1-68.gitdded712.el7.centos           docker-common.x86_64 2:1.13.1-68.gitdded712.el7.centos     
  libcgroup.x86_64 0:0.41-15.el7                                   libsemanage-python.x86_64 0:2.5-11.el7                     
  oci-register-machine.x86_64 1:0-6.git2b44233.el7                 oci-systemd-hook.x86_64 1:0.1.16-1.git05bd9a0.el7          
  oci-umount.x86_64 2:2.3.3-3.gite3c9055.el7                       policycoreutils-python.x86_64 0:2.5-22.el7                 
  python-IPy.noarch 0:0.75-6.el7                                   setools-libs.x86_64 0:3.3.8-2.el7                          
  skopeo-containers.x86_64 1:0.1.31-1.dev.gitae64ff7.el7.centos    yajl.x86_64 0:2.0.4-4.el7 
  
ps aux|grep docker

docker run -i -t centos /bin/bash
Unable to find image 'centos:latest' locally
Trying to pull repository docker.io/library/centos ... 
latest: Pulling from docker.io/library/centos
7dc0dca2b151: Pull complete 
Digest: sha256:b67d21dfe609ddacf404589e04631d90a342921e81c40aeaf3391f6717fa5322
Status: Downloaded newer image for docker.io/centos:latest

docker run --name bob_the_container -i -t centos /bin/bash

sudo docker run --name daemon_dave -d centos /bin/sh -c "while true; do echo hello world; sleep 1; done"
```