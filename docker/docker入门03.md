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
>默认，当执行docker ps命令时，只能看到正在运行的容器。指定-a标志：`docker ps`命令会列出所有容器，包括正在运行和已停止的。

- 列出最后一个运行的容器
`docker ps -l`

- 3种方式可以唯一指代容器
短UUID（如f7cbdac22a02）、长UUID（如f7cbdac
22a02e03c9438c729345e54db9d20cfa2ac1fc3494b6eb60872e74778）或者名称（如gray_cat）

### 容器命名
>Docker会为我们创建的每一个容器自动生成一个随机的名称。

- 给容器命名
```
[root@localhost ~]# docker run --name bob_the_container -i -t centos /bin/bash
root@aa3f365f0f4e:/# exit  
```
上述命令会创建一个名为bob_the_container的容器。一个合法的容器名称只能包含[a-zA-Z0-9_.-]。  
容器名称有助于分辨容器，当构建容器和应用程序之间的逻辑连接时，容器的名称也有助于从逻辑上理解连接关系。  
具体的名称（如web、db）比容器ID和随机容器名好记多了。  
容器的命名必须是唯一的。
- 删除容器  
`docker rm`

### 重新启动已经停止的容器
- 启动已经停止运行的容器  
`sudo docker start bob_the_container`  
- 通过ID启动已经停止运行的容器  
`sudo docker start aa3f365f0f4e`  
>也可用`docker restart`重启容器。
这时运行不带-a标志的docker ps命令，就应该看到我们的容器已经开始运行了。  

### 附着到容器上
Docker容器重启时，会沿用docker run命令时指定的参数来运行，因此我们的容器重启后会运行一个交互式会话shell。此外，也可以用docker attach命令，重新附着到该容器的会话上。   
- 附着到正在运行的容器   
`docker attach bob_the_container`  
- 重新附着到容器的会话
root@aa3f365f0f4e:/_#_
>可能需要按下回车键才能进入该会话。  
如果退出容器的shell，容器会再次停止运行。  

### 创建守护式容器
>除交互式运行的容器，也可创建长期运行的容器。守护式容器没有交互式会话，非常适合运行应用程序和服务。大多数时候都需要以守护式来运行容器。
- 创建长期运行的容器
>$ sudo docker run --name daemon_dave -d centos /bin/sh -c "while
　true; do echo hello world; sleep 1; done"

>`docker run`命令使用-d参数，会将容器放到后台运行。  
还使用了一个while循环，会一直打印hello world，直到容器或其进程停止。  
组合以上参数，没将主机的控制台附着到新的shell会话上，而是仅返回容器ID，还在主机命令行中。

- 查看正在运行的daemon_dave容器
`docker ps`    
CONTAINER ID IMAGE　　　　　COMMAND　　　　　　　　CREATED
　STATUS PORTS NAMES
1333bb1a66af ubuntu:14.04 /bin/sh -c 'while tr 32 secs ago Up 27
　　　　　　daemon_dave

### 容器内部都在干些什么
>探究守护型容器内部在干什么  
`docker logs`  
```
-- 获取守护式容器的日志,Docker会输出最后几条日志项并返回
$ sudo docker logs daemon_dave
hello world
hello world
hello world
hello world
hello world
hello world
hello world

-- 跟踪守护式容器的日志，与tail -f相似
$ sudo docker logs -f daemon_dave
hello world
hello world
hello world
hello world
hello world
hello world
hello world
```
`Ctr+C`退出日志跟踪。
`docker logs --tail 10 daemon_dave`获取日志的最后10行内容。  
`docker logs --tail 0 -f daemon_dave`跟踪某个容器的最新日志而不必读取整个日志文件。  

- 跟踪守护式容器的最新日志,使用-t标志为每条日志项加上时间戳  
```
$ sudo docker logs -ft daemon_dave
[May 10 13:06:17.934] hello world
[May 10 13:06:18.935] hello world
[May 10 13:06:19.937] hello world
[May 10 13:06:20.939] hello world
[May 10 13:06:21.942] hello world
```
#### Docker日志驱动
>1.6后可控制Docker守护进程和容器所用的日志驱动，`--log-driver`，启动Docker守护进程或执行docker run命令时使用这个选项。
`json-file`，`json-file`提供了基础，`syslog`将禁用docker logs命令，且将所有容器的日志输出都重定向到Syslog。
- 在容器级别启动Syslog
```
$ sudo docker run --log-driver="syslog" --name daemon_dwayne -d
　ubuntu /bin/sh -c "while true; do echo hello world; sleep 1;
　done"
```

#### 查看容器内的进程
>除容器日志，也可查看容器内部运行的进程。使用`docker top`
- 查看守护式容器的进程  
`$ sudo docker top daemon_dave`
>执行后，可看到容器内的所有进程（主要还是我们的while循环）、运行进程的用户及进程ID

#### Docker统计信息
`docker stats`显示一个或多个容器的统计信
息。
- docker stats命令
```
docker run --name daemon_dave -d centos /bin/sh

CONTAINER           CPU %               MEM USAGE / LIMIT   MEM %               NET I/O             BLOCK I/O           PIDS
zsr_the_container   --                  -- / --             --                  --                  --                  --
daemon_dave         --                  -- / --             --                  --                  --                  --
```
>守护式容器列表，以及CPU、内存、网络I/O、存储I/O的性能和指标。

### 在容器内部运行进程
`docker exec`  
>在容器内部额外启动新进程。可在容器内运行：后台任务（在容器内运行且没交互需求）和交互式任务（保持在前台运行）。
```
- 在容器内运行交互命令    
docker exec -t -i daemon_dave /bin/bash
```
>和运行交互容器时一样，-t和-i标志创建了TTY并捕捉STDIN。接着指定了容器的名字以及要执行的命令。

#### 停止守护式容器
- 停止正在运行的Docker容器  
`docker stop daemon_dave`
- 通过容器ID停止正在运行的容器  
`docker stop c2c4e57c12c4`

### 自动重启容器
>由于某种错误导致容器停止运行，还可通过--restart标志，让Docker自动重新启动。--restart标志会检查容器的退出代码，并据此来决定是否要重启容器。默认Docker不重启容器。
```
- 自动重启容器  
docker run --restart=always --name daemon_dave -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```
>--restart标志被设置为always。无论容器的退出代码是什么，Docker都会自动重启。
>>除always，还可将这个标志设为on-failure（只有当容器的退出代码非0，才会自动重启）。on-failure还接受一个可选的重启次数参数。

```
- 为on-failure指定count参数
--restart=on-failure:5
```
>这样，当容器退出代码为非0时，Docker会尝试自动重启该容器，最多重启5次。

### 深入容器
- 查看容器
```
[root@localhost ~]# docker inspect daemon_dave
[
    {
        "Id": "b7c645a7a8c5ea53861685b98e2ee74298cb4e9e94d8291a0f08121d06b8550f",
        "Created": "2018-07-21T17:35:21.063296159Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true; do echo hello world; sleep 1; done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 3058,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2018-07-21T21:03:38.028921078Z",
            "FinishedAt": "2018-07-21T17:42:03.866625535Z"
        },
        ...
```
>docker inspect命令会对容器详细检查，返回配置信息（名称、命令、网络配置及很多有用数据）

- 有选择地获取容器信息
```
docker inspect --format='{{ .State.Running }}' daemon_dave
false
```

#### 删除容器
`docker rm`

```
- 删除容器
docker rm 80430f8d0921

- 删除所有容器
docker rm `sudo docker ps -a -q`
```
>docker ps命令会列出现有全部容器，-a标志代表列出所有容器，而-q标志只需返回容器的ID而不返回容器的其他信息。
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