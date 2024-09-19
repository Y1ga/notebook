Build，Ship and Run anywhere

# 知识讲解

## docker与虚拟机

### 区别

![image-20240806170838718](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240806170838718.png)

- 虚拟机：**应用层抽象，容器虚拟化的是操作系统**，通过**Hypervisor**(虚拟机管理系统，常见的有VMWare, VirturalBox)，能虚拟出网卡、CPU、内存等虚拟硬件，再在其上建立虚拟机，每个虚拟机都是独立的操作系统，有自己的系统内核（GuestOS）
- 容器：**物理硬件层抽象，虚拟出一套硬件**，利用**namespace**将文件系统、进程、网络、设备等资源进行隔离，利用**cgroup**对权限、**cpu资源进行限制**，最终让容器之间互不影响，容器无法影响宿主机

![image-20240807182237460](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807182237460.png)

优点：

1. **秒级** VS 分钟级
2. **性能** 原生 > 弱于原生
3. **数量**：容器上千个 > 虚拟机几十个
4. 硬盘使用：**MB** 优于 GB

## **Docker 思想**

- **集装箱**：就像海运中的集装箱一样，Docker 容器包含了**应用程序及其所有依赖项，确保在任何环境中都能以相同的方式运行**。
- **标准化：**运输方式、存储方式、API 接口。
- **隔离**：每个 Docker 容器都在自己的隔离环境中运行，**与宿主机和其他容器隔离**。

## Docker 容器的特点

- **轻量** : 在一台机器上运行的多个 Docker 容器可以共享这台机器的操作系统内核；它们能够迅速启动，只需占用很少的计算和内存资源。镜像是通过文件系统层进行构造的，并共享一些公共文件。这样就能尽量降低磁盘用量，并能更快地下载镜像。
- **标准** : Docker 容器基于开放式标准，能够在**所有主流 Linux 版本、Microsoft Windows 以及包括 VM**、裸机服务器和云在内的任何基础设施上运行。
- **安全** : Docker 赋予应用的隔离性不仅限于彼此隔离，还独立于底层的基础设施。Docker 默认提供最强的隔离，因此应用出现问题，也只是单个容器的问题，而不会波及到整台机器。





# 核心

**namespace控制资源访问；cgroups限制进程对资源消耗**

## namespace

namespace用于**隔离资源**，将一组系统资源的视图从一个进程中隔离出来，使得该进程以及子进程有自己的独立资源集

- PID(Process ID)：隔离进程ID，使一个namespace中的进程无法看到其他namespace的进程
- UTS(Unix Time-sharing System)：隔离hostname(主机名)和NIS domain name(域名)
- MNT(Mount)：隔离挂载点（mount point）
- NET：隔离网络空间，包括：网络设备、路由表、iptables、sockets等
- IPC：隔离进程间通信资源，如消息队列、信号量等
- USER(鸡肋)：隔离用户和用户组
- CGROUP(鸡肋)：隔离cgroup

### PID

- 终端1

```bash
[root@localhost ~]# unshare --pid --uts --mount --fork bash
### 设置hostname以便于区分 ###
[root@namespace-01 ~]# hostname namespace-01 && exec bash
### 查看进程树，依然可以看到完整的进程树，因为pstree读取/proc目录的信息，而/proc继承自unshare所在的mount namespace ###
[root@namespace-01 ~]# pstree -pl
systemd(1)─┬─NetworkManager(664)─┬─{NetworkManager}(671)
           │                     └─{NetworkManager}(678)
           ...
           
root@namespace01:/home/yiga# readlink /proc/$$/ns/uts
uts:[4026531838]
root@namespace01:/home/yiga# hostname
namespace01
# 重新挂载后只能看到当前namesapce的信息！
root@namespace-01:/home/yiga# mount --types proc proc /proc/
# 只能看到自身namespace下的进程信息，无法看到上级或者平行级
root@namespace-01:/home/yiga# pstree -pl
bash(1)───pstree(23)
```

- 终端2

```bash
root@namespace-02:/home/yiga# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026531836]
uts:[4026531838]
mnt:[4026531841]
# 重新挂载并创建新的namespace，不再能看到父级的pstree
root@namespace-02:/home/yiga# unshare --pid --uts --mount --fork --mount-proc bash
root@namespace-02:/home/yiga# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026532733]
uts:[4026532732]
mnt:[4026532731]
root@namespace-02:/home/yiga# pstree -pl
bash(1)───pstree(9)
```

- 终端1下继续创建子命名空间

```bash
root@namespace-01:/home/yiga# pstree -pl
bash(1)───pstree(24)
# --fork：使bash作为unshare命令的子进程运行
# 意味着unshare命令不会直接执行bash而是fork出一个子进程来执行他
# --mount-proc 挂载在自己隔离的/proc目录中
#一个bash就是一个进程
root@namespace-01:/home/yiga# unshare --pid --uts --mount --fork --mount-proc bash
root@namespace-01:/home/yiga# hostname namespace-03 && exec bash
root@namespace-03:/home/yiga# readlink /proc/$$/ns/{pid,uts,mnt}
pid:[4026532736]
uts:[4026532735]
mnt:[4026532734]
root@namespace-03:/home/yiga# pstree -pl
bash(1)───pstree(16)
```

三者关系

                              +-------------------------------+
                              |hostname: localhost.localdomain|
                          +---+pid namespace:4026531836       +---+
                          |   +-------------------------------+   |
                          |                                       |
          +---------------v---------------+       +---------------v---------------+
          |hostname: namespace-01         |       |hostname: namespace-02         |
          |pid namespace:4026532501       |       |pid namespace:4026532504       |
          +---------------+---------------+       +-------------------------------+
                          |
          +---------------v---------------+
          |hostname: namespace-03         |
          |pid namespace:4026532507       |
          +-------------------------------+

继续端口1

```bash
mnt:[4026532734]
root@namespace-03:/home/yiga# pstree -pl
bash(1)───pstree(16)
# 为什么数字会16->17，是上一个pstree被kill掉，这个进程号17的是新的pstree
root@namespace-03:/home/yiga# pstree -pl
bash(1)───pstree(17)
# -p能输出进程号
root@namespace-03:/home/yiga# pstree
bash───pstree
# 增加一个bash子进程
root@namespace-03:/home/yiga# bash
# 进程号是22
root@namespace-03:/home/yiga# echo $$
22
# 增加一个bash孙进程
root@namespace-03:/home/yiga# bash
root@namespace-03:/home/yiga# echo $$
29
root@namespace-03:/home/yiga# bash
root@namespace-03:/home/yiga# echo $$
36
# 增加一个sleep进程
root@namespace-03:/home/yiga# sleep 3600 &
[1] 43
root@namespace-03:/home/yiga# pstree -pl
bash(1)───bash(22)───bash(29)───bash(36)─┬─pstree(44)
                                         └─sleep(43)
# 退出bash(36)
root@namespace-03:/home/yiga# exit
exit
root@namespace-03:/home/yiga# pstree -pl
bash(1)─┬─bash(22)───bash(29)───pstree(45)
        └─sleep(43)
root@namespace-03:/home/yiga# exit
exit
root@namespace-03:/home/yiga# pstree -pl
bash(1)─┬─bash(22)───pstree(46)
        └─sleep(43)
root@namespace-03:/home/yiga# exit
exit
root@namespace-03:/home/yiga# pstree -pl
bash(1)─┬─pstree(47)
        └─sleep(43)
# 退出sleep进程
root@namespace-03:/home/yiga# exit
exit
root@namespace-01:/home/yiga# pstree -pl
bash(1)───pstree(73)
# 当bash(1)退出时，整个namespace-01所有进程都被kill掉
root@namespace-01:/home/yiga# exit
exit
# 因此输出上一级的进程树
root@namespace-01:/home/yiga# pstree -pl
systemd(1)─┬─ModemManager(875)─┬─{ModemManager}(927)
           │                   └─{ModemManager}(950)
           ....
```



### UTS

隔离hostanme和NIS domain name

- 端口1

```bash
# 隔离uts
root@yiga-virtual-machine:/home/yiga# unshare --uts bash
root@yiga-virtual-machine:/home/yiga# readlink /proc/$$/ns/uts
uts:[4026532728]
root@yiga-virtual-machine:/home/yiga# unshare --uts bash
root@yiga-virtual-machine:/home/yiga# readlink /proc/$$/ns/uts
uts:[4026532729]
root@yiga-virtual-machine:/home/yiga# 

#隔离hostname
root@yiga-virtual-machine:/home/yiga# hostname namespace01 && exec bash
root@namespace01:/home/yiga# hostname
namespace01
```

- 端口2

```bash
root@namespace01:/home/yiga# hostname namespace02 && exec bash
root@namespace02:/home/yiga# hostname
namespace02

```

关系

                            +-----------------------------------+
                            |  hostname: localhost.localdomain  |
                        +---+  uts: 4026531838                  +---+
                        |   |  pid: 1425, 1744                  |   |
                        |   +-----------------------------------+   |
                        v                                           v
     +-----------------------------------+        +-----------------------------------+
     |  hostname: namespace-01           |        |  hostname: namespace-02           |
     |  uts: 4026532418                  |        |  uts: 4026532499                  |
     |  pid: 1717                        |        |  pid: 1763                        |
     +-----------------------------------+        +-----------------------------------+

### MNT

```bash
yiga@yiga-virtual-machine:~$ ll /proc/$$/mount*
-r--r--r-- 1 yiga yiga 0  8-р сар  7 14:10 /proc/6917/mountinfo
-r--r--r-- 1 yiga yiga 0  8-р сар  7 14:10 /proc/6917/mounts
-r-------- 1 yiga yiga 0  8-р сар  7 14:10 /proc/6917/mountstats
# 在namespace01和02文件夹中分别创建01和02.txt
yiga@yiga-virtual-machine:~$ mkdir namespace01 && touch namespace01/namespace01.txt
yiga@yiga-virtual-machine:~$ mkdir namespace02 && touch namespace02/namespace02.txt
```



- 终端1

```bash
# 隔离mount并启动新的bash
root@yiga-virtual-machine:/home/yiga# unshare --mount --uts bash
# 隔离hostname并启动新的bash
root@yiga-virtual-machine:/home/yiga# hostname namespace-01 && exec bash
# 查看INODE
root@namespace-01:/home/yiga# readlink /proc/$$/ns/{uts,mnt}
uts:[4026532730]
mnt:[4026532729]
# 将namespace01下的文件01.txt挂载到/mnt/
root@namespace-01:/home/yiga# mount --bind namespace01/ /mnt/
root@namespace-01:/home/yiga# ll /mnt/
total 8
drwxrwxr-x  2 yiga yiga 4096  8-р сар  7 14:10 ./
drwxr-xr-x 20 root root 4096  8-р сар  6 21:20 ../
-rw-rw-r--  1 yiga yiga    0  8-р сар  7 14:10 namespace01.txt
```

- 终端2

```bash
root@yiga-virtual-machine:/home/yiga# unshare --mount --uts bash
root@yiga-virtual-machine:/home/yiga# hostname namespace-02 && exec bash
root@namespace-02:/home/yiga# readlink /proc/$$/{uts,mnt}
root@namespace-02:/home/yiga# readlink /proc/$$/ns/{uts,mnt}
# 此处的mnt显然不同
uts:[4026532731]
mnt:[4026532728]
root@namespace-02:/home/yiga# ll /mnt/
total 8
drwxr-xr-x  2 root root 4096  2-р сар 21 03:22 ./
drwxr-xr-x 20 root root 4096  8-р сар  6 21:20 ../
# 挂载02.txt
root@namespace-02:/home/yiga# mount --bind namespace02/ /mnt/
root@namespace-02:/home/yiga# ll /mnt/
# mnt目录下的文件也被隔离了，此处只看得到02.txt看不到01.txt
total 8
drwxrwxr-x  2 yiga yiga 4096  8-р сар  7 14:10 ./
drwxr-xr-x 20 root root 4096  8-р сар  6 21:20 ../
-rw-rw-r--  1 yiga yiga    0  8-р сар  7 14:10 namespace02.txt

```



### NET

- 终端1

```bash
# 隔离net和uts并启动新的bash
root@namespace-01:/home/yiga# unshare --net --uts bash
# 显示ip
root@namespace-01:/home/yiga# ip addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
# 启动loop back
root@namespace-01:/home/yiga# ip link set dev lo up
# 测试loop back
root@namespace-01:/home/yiga# ping -c 5 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.023 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.039 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.029 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.032 ms
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.030 ms

--- 127.0.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4074ms
rtt min/avg/max/mdev = 0.023/0.030/0.039/0.005 ms
# 查看PID
root@namespace-01:/home/yiga# echo $$
7148
```

- 终端2

```bash
root@namespace-02:/home/yiga# unshare --net --uts bash
root@namespace-02:/home/yiga# ip addr
1: lo: <LOOPBACK> mtu 65536 qdisc noop state DOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
root@namespace-02:/home/yiga# ping 127.0.0.1
ping: connect: Network is unreachable
root@namespace-02:/home/yiga# ip link set dev lo up
root@namespace-02:/home/yiga# ip addr
1: lo: <LOOPBACK,UP,LOWER_UP> mtu 65536 qdisc noqueue state UNKNOWN group default qlen 1000
    link/loopback 00:00:00:00:00:00 brd 00:00:00:00:00:00
    inet 127.0.0.1/8 scope host lo
       valid_lft forever preferred_lft forever
    inet6 ::1/128 scope host 
       valid_lft forever preferred_lft forever
root@namespace-02:/home/yiga# ping -c 5 127.0.0.1
PING 127.0.0.1 (127.0.0.1) 56(84) bytes of data.
64 bytes from 127.0.0.1: icmp_seq=1 ttl=64 time=0.020 ms
64 bytes from 127.0.0.1: icmp_seq=2 ttl=64 time=0.031 ms
64 bytes from 127.0.0.1: icmp_seq=3 ttl=64 time=0.030 ms
64 bytes from 127.0.0.1: icmp_seq=4 ttl=64 time=0.030 ms
64 bytes from 127.0.0.1: icmp_seq=5 ttl=64 time=0.029 ms

--- 127.0.0.1 ping statistics ---
5 packets transmitted, 5 received, 0% packet loss, time 4090ms
rtt min/avg/max/mdev = 0.020/0.028/0.031/0.004 ms
root@namespace-02:/home/yiga# echo $$
7133
```

此时，namespace-01和namespace-02是**两个完全封闭的网络**，彼此之间不能访问

## Cgroups

Control groups，允许将**一组process**和**一组subsystem**放在一个cgroup中，并为这个cgroup分配**特定的资源限制和优先级**（例如CPU占用率、内存占用量、硬盘访问速率）

确保容器在**共享主机上合理利用系统资源，避免资源竞争和过度使用**。 

### task

系统的一个进程

### subsystem

一个**资源调度控制器（Resource Controller）**

- cpu 子系统，主要限制 cpu 使用率。

- cpuacct 子系统，可以统计 cgroups 中的进程的 cpu 使用报告。
- cpuset 子系统，可以为 cgroups 中的进程分配单独的 cpu 节点或者内存节点。
- memory 子系统，可以限制进程的 memory 使用量。
- blkio 子系统，可以限制进程的块设备 io。
- devices 子系统，可以控制进程能够访问某些设备。
- net_cls 子系统，可以标记 cgroups 中进程的网络数据包，然后可以使用 tc 模块（traffic control）对数据包进行控制。
- freezer 子系统，可以挂起或者恢复 cgroups 中的进程。
- ns 子系统，可以使不同 cgroups 下面的进程使用不同的 namespace。

### hierarchy

1. 把cgroups串成一个**树形结构**，使cgroups可以继承

2. hierarchy中的cgroup节点可以包含零或多个子节点，子节点继承父节点的属性。整个系统可以有多个hierarchy。





### docker的优势

- 在dockerfile中更改和重新编译image后，只有重现编译的docker object会重现编译，因此很轻量级

## 基本组成

![image-20240806171120071](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240806171120071.png)

![image-20240807182729320](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807182729320.png)

### 镜像(image)

1. 镜像是一个特殊的**文件系统**，类比：操作系统分为**内核和用户空间**，对Linux而言，内核启动后会**挂载root文件系统**为其提供用户空间支持，docker就是一个root文件系统
2. **除了提供容器运行时需要的程序、库、资源、配置等文件外，还包含运行时准备的配置参数(如匿名卷、环境变量、用户等)**，镜像**不包含任何动态数据**，构建后也不会被改变。
3. 镜像构建时，会一层层构建，前一层是后一层的基础，**每一层构建完不再发生改变，后一层的任何改变只发生在自己这一层**。例如：**删除前一层文件操作，实际不会真正删除前一层文件，而只是在该层标记文件已删除，容器运行时也不会看到这个文件，但这个标记被删除的文件已知跟随镜像**，因此构建image时要尽量使额外的东西在构建该层结束前删除。
4. 镜像使用的是分层存储

![img](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-build-ship-run.jpg)

- **Build（构建镜像）**：镜像就像是集装箱包括文件以及运行环境等等资源。
- **Ship（运输镜像）**：主机和仓库间运输，这里的仓库就像是超级码头一样。
- **Run （运行镜像）**：**运行的镜像就是一个容器，容器就是运行程序的地方。**



### 容器(container)

1. 容器是image的一个**运行实例**（就是Java中类与实例的关系），Docker利用容器技术，独立运行一个或者一个组应用，通过镜像来**创建，启动，停止，删除，暂停**
2. 容器的**实质是进程，运行于属于自己的独立的命名空间，相互独立**】
3. 容器**存储层的生命周期和容器一样**，保存于容器存储层的信息都随容器删除而丢失
4. 容器**不应该向存储层写入任何数据，保持无状态化**，所有的文件写入操作都应该使用**数据卷(Volume)、或者绑定宿主目录**，这些位置的读写都能**跳过存储层，直接对宿主发生读写，性能和稳定性更高**。数据卷生命周期独立于容器，**容器可以随意删除、重新run，数据卷并不会消失。**

container与container之间是相互**独立**的

### 仓库(Repository)

存放镜像的地方，仓库分为公有仓库和私有仓库

Docker Hub(国外)

阿里云

## 运行流程

1. 优先本地
2. 本地不存在Docker Hub下载
3. 都不存在报错

![image-20240806171910301](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240806171910301.png)

### 基础指令

- FROM-基础镜像源，从这里开始构建
- MAINTAINER-镜像源谁写的，姓名+邮箱
- **RUN**-镜像构建时需要运行的命令
- ADD-tomcat镜像，添加内容
- WORKDIR 镜像的工作目录
- VOLUME-挂载的目录
- EXPOSE-保留端口配置
- CMD-指定这个容器的时候要运行的命令，只有最后一个会生效，可被替代
- ENTRYPOINT-指定容器的时候要运行的命令，可以追加命令
- ONBUILD-构建一个被继承DockerFile这个时候就会运行ONBUILD指令，触发指令
- COPY-类似ADD，将文件拷贝到镜像中
- ENV-构建环境时设置环境变量

## 常用指令

```sh
docker version # 查看docker版本
docker images # 查看所有已下载镜像，等价于：docker image ls 命令
docker container ls # 查看所有容器
docker ps #查看正在运行的容器
docker image prune # 清理临时的、没有被使用的镜像文件。-a, --all: 删除所有没有用的镜像，而不仅仅是临时文件；

```

## docker数据卷

在容器中管理数据主要有两种方式：

1. 数据卷（Volumes）
2. **挂载主机目录 (Bind mounts)**

数据卷是一个**虚拟目录**，将**宿主机目录映射到容器内目录**，实现**宿主与容器**之间的文件共享。这样可以直接对宿主机的文件进行修改直接影响容器，而无需将宿主机的文件再复制到容器中。

- 即使容器被删除，数据卷的数据也不会自动删除，确保数据持久性
- 对数据卷的修改会立刻生效，且不影响镜像
- ![image-20240807205450160](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807205450160.png)

![image-20240807205016067](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807205016067.png)

这里的挂载，就是**monitor_docker_run.sh脚本里的-v**操作

把我们**本机的代码挂载到容器**中，这样我们本地修改代码，容器中代码也会修改在容器中修改代码，本地代码也会修改

![image-20240807205106598](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807205106598.png)

![image-20240807205118950](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807205118950.png)



# 依赖库安装

## /usr/local目录

专门用来安装第三方软件的目录，区别于安装系统时自带的软件

- /usr：由系统安装的所有系统范围的**只读文件**
- /usr/local：**本地管理员**安装的系统范围的只读文件

基本都是用cmake编译

## abseil

C++14的开源集合，protbuf需要

## cmake

跨平台、开源构建系统生成器

## protobuf

## grpc

## qt

### cuteci

安装QT的辅助便捷工具

## Idconfig

动态链接库管理命令，目的为了让动态链接库为系统所共享



# dockerfile

![image-20240816162900838](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240816162900838.png)

![image-20240807205143395](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807205143395.png)

```dockerfile
# 母镜像源
FROM  ubuntu:18.04

ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=Asia/Shanghai

SHELL ["/bin/bash", "-c"]

RUN apt-get clean && \
    apt-get autoclean
COPY apt/sources.list /etc/apt/

# apt-get upgrade -y 安装包时询问Y/N时默认为Y
# 下面为Ubuntu基础安装包
RUN apt-get update  && apt-get upgrade -y  && \
    apt-get install -y \
    htop \
    apt-utils \
    curl \
    cmake \
    git \
    openssh-server \
    build-essential \
    qtbase5-dev \
    qtchooser \
    qt5-qmake \
    qtbase5-dev-tools \
    libboost-all-dev \
    net-tools \
    vim \
    stress 

# 下面为gprc依赖的安装包
RUN apt-get install -y libc-ares-dev  libssl-dev gcc g++ make 
# 下面为qt依赖的安装包
# 以下皆为xhost指令
RUN apt-get install -y  \
    libx11-xcb1 \
    libfreetype6 \
    libdbus-1-3 \
    libfontconfig1 \
    libxkbcommon0   \
    libxkbcommon-x11-0

# 以下为cuteil依赖的安装包
RUN apt-get install -y python-dev \
    python3-dev \
    python-pip \
    python-all-dev 

# /tmp目录表示从ubuntu18放到本地目录下
COPY install/protobuf /tmp/install/protobuf
# 执行安装的脚本指令
RUN /tmp/install/protobuf/install_protobuf.sh

COPY install/abseil /tmp/install/abseil
RUN /tmp/install/abseil/install_abseil.sh

COPY install/grpc /tmp/install/grpc
RUN /tmp/install/grpc/install_grpc.sh

# COPY install/cmake /tmp/install/cmake
# RUN /tmp/install/cmake/install_cmake.sh

# RUN apt-get install -y python3-pip
# RUN pip3 install cuteci -i https://mirrors.aliyun.com/pypi/simple

# COPY install/qt /tmp/install/qt
# RUN /tmp/install/qt/install_qt.sh

```





# 待补坑：联合文件系统
