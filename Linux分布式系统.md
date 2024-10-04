# 1. Docker

Build，Ship and Run anywhere

## 知识讲解

### docker与虚拟机

#### 区别

![image-20240806170838718](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240806170838718.png)

- 虚拟机：**应用层抽象，容器虚拟化的是操作系统**，通过**Hypervisor**(虚拟机管理系统，常见的有VMWare, VirturalBox)，能虚拟出网卡、CPU、内存等虚拟硬件，再在其上建立虚拟机，每个虚拟机都是独立的操作系统，有自己的系统内核（GuestOS）
- 容器：**物理硬件层抽象，虚拟出一套硬件**，利用**namespace**将文件系统、进程、网络、设备等资源进行隔离，利用**cgroup**对权限、**cpu资源进行限制**，最终让容器之间互不影响，容器无法影响宿主机

![image-20240807182237460](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807182237460.png)

优点：

1. **秒级** VS 分钟级
2. **性能** 原生 > 弱于原生
3. **数量**：容器上千个 > 虚拟机几十个
4. 硬盘使用：**MB** 优于 GB

### **Docker 思想**

- **集装箱**：就像海运中的集装箱一样，Docker 容器包含了**应用程序及其所有依赖项，确保在任何环境中都能以相同的方式运行**。
- **标准化：**运输方式、存储方式、API 接口。
- **隔离**：每个 Docker 容器都在自己的隔离环境中运行，**与宿主机和其他容器隔离**。

### Docker 容器的特点

- **轻量** : 在一台机器上运行的多个 Docker 容器可以共享这台机器的操作系统内核；它们能够迅速启动，只需占用很少的计算和内存资源。镜像是通过文件系统层进行构造的，并共享一些公共文件。这样就能尽量降低磁盘用量，并能更快地下载镜像。
- **标准** : Docker 容器基于开放式标准，能够在**所有主流 Linux 版本、Microsoft Windows 以及包括 VM**、裸机服务器和云在内的任何基础设施上运行。
- **安全** : Docker 赋予应用的隔离性不仅限于彼此隔离，还独立于底层的基础设施。Docker 默认提供最强的隔离，因此应用出现问题，也只是单个容器的问题，而不会波及到整台机器。





## 核心

**namespace控制资源访问；cgroups限制进程对资源消耗**

### namespace

namespace用于**隔离资源**，将一组系统资源的视图从一个进程中隔离出来，使得该进程以及子进程有自己的独立资源集

- PID(Process ID)：隔离进程ID，使一个namespace中的进程无法看到其他namespace的进程
- UTS(Unix Time-sharing System)：隔离hostname(主机名)和NIS domain name(域名)
- MNT(Mount)：隔离挂载点（mount point）
- NET：隔离网络空间，包括：网络设备、路由表、iptables、sockets等
- IPC：隔离进程间通信资源，如消息队列、信号量等
- USER(鸡肋)：隔离用户和用户组
- CGROUP(鸡肋)：隔离cgroup

#### PID

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



#### UTS

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

#### MNT

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



#### NET

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

### Cgroups

Control groups，允许将**一组process**和**一组subsystem**放在一个cgroup中，并为这个cgroup分配**特定的资源限制和优先级**（例如CPU占用率、内存占用量、硬盘访问速率）

确保容器在**共享主机上合理利用系统资源，避免资源竞争和过度使用**。 

#### task

系统的一个进程

#### subsystem

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

#### hierarchy

1. 把cgroups串成一个**树形结构**，使cgroups可以继承

2. hierarchy中的cgroup节点可以包含零或多个子节点，子节点继承父节点的属性。整个系统可以有多个hierarchy。





#### docker的优势

- 在dockerfile中更改和重新编译image后，只有重现编译的docker object会重现编译，因此很轻量级

## 基本组成

![image-20240806171120071](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240806171120071.png)

![image-20240807182729320](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240807182729320.png)

#### 镜像(image)

1. 镜像是一个特殊的**文件系统**，类比：操作系统分为**内核和用户空间**，对Linux而言，内核启动后会**挂载root文件系统**为其提供用户空间支持，docker就是一个root文件系统
2. **除了提供容器运行时需要的程序、库、资源、配置等文件外，还包含运行时准备的配置参数(如匿名卷、环境变量、用户等)**，镜像**不包含任何动态数据**，构建后也不会被改变。
3. 镜像构建时，会一层层构建，前一层是后一层的基础，**每一层构建完不再发生改变，后一层的任何改变只发生在自己这一层**。例如：**删除前一层文件操作，实际不会真正删除前一层文件，而只是在该层标记文件已删除，容器运行时也不会看到这个文件，但这个标记被删除的文件已知跟随镜像**，因此构建image时要尽量使额外的东西在构建该层结束前删除。
4. 镜像使用的是分层存储

![img](https://oss.javaguide.cn/github/javaguide/tools/docker/docker-build-ship-run.jpg)

- **Build（构建镜像）**：镜像就像是集装箱包括文件以及运行环境等等资源。
- **Ship（运输镜像）**：主机和仓库间运输，这里的仓库就像是超级码头一样。
- **Run （运行镜像）**：**运行的镜像就是一个容器，容器就是运行程序的地方。**



#### 容器(container)

1. 容器是image的一个**运行实例**（就是Java中类与实例的关系），Docker利用容器技术，独立运行一个或者一个组应用，通过镜像来**创建，启动，停止，删除，暂停**
2. 容器的**实质是进程，运行于属于自己的独立的命名空间，相互独立**】
3. 容器**存储层的生命周期和容器一样**，保存于容器存储层的信息都随容器删除而丢失
4. 容器**不应该向存储层写入任何数据，保持无状态化**，所有的文件写入操作都应该使用**数据卷(Volume)、或者绑定宿主目录**，这些位置的读写都能**跳过存储层，直接对宿主发生读写，性能和稳定性更高**。数据卷生命周期独立于容器，**容器可以随意删除、重新run，数据卷并不会消失。**

container与container之间是相互**独立**的

#### 仓库(Repository)

存放镜像的地方，仓库分为公有仓库和私有仓库

Docker Hub(国外)

阿里云

#### 运行流程

1. 优先本地
2. 本地不存在Docker Hub下载
3. 都不存在报错

![image-20240806171910301](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240806171910301.png)

#### 基础指令

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

#### 常用指令

```sh
docker version # 查看docker版本
docker images # 查看所有已下载镜像，等价于：docker image ls 命令
docker container ls # 查看所有容器
docker ps #查看正在运行的容器
docker image prune # 清理临时的、没有被使用的镜像文件。-a, --all: 删除所有没有用的镜像，而不仅仅是临时文件；

```

#### docker数据卷

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



## 依赖库安装

### /usr/local目录

专门用来安装第三方软件的目录，区别于安装系统时自带的软件

- /usr：由系统安装的所有系统范围的**只读文件**
- /usr/local：**本地管理员**安装的系统范围的只读文件

基本都是用cmake编译

### abseil

C++14的开源集合，protbuf需要

### cmake

跨平台、开源构建系统生成器

### protobuf

### grpc

### qt

### cuteci

安装QT的辅助便捷工具

### Idconfig

动态链接库管理命令，目的为了让动态链接库为系统所共享



## dockerfile

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

## 安装

### 1 写好dockerfile

```dockerfile
FROM ubuntu::20.04
ENV DEBIAN_FRONTEND=noninteractive
RUN apt-get clean && apt-get autoclean

RUN apt-get upgrade && apt-get upgrade -y && \
    apt-get install -y \
    vim \
    htop \
    apt-utils \
    curl \
    cmake \
    net-tools \
    gdb  gcc g++ \
    libgoogle-glog-dev

WORKDIR /work
```

### 2 基于dockerfile创建镜像

```
docker build --network host -t my:udp -f test.dockerfile .
```

- `--network host`：
  - 这个参数指定 Docker 在构建镜像的过程中使用宿主机的网络模式。这意味着容器将**直接使用宿主机**的网络栈，能够访问宿主机所能访问的网络资源，并且容器的网络性能与宿主机基本一致。
  - 例如，如果宿主机可以直接访问某个特定的网络服务，那么在这种网络模式下构建的容器也可以直接访问该服务，**无需进行额外的网络配置**。
- `-t safe:udp`：
  - 这里的 “-t” 参数用于为构建的镜像指定**一个标签（tag）**。在这个例子中，镜像的标签为 “safe:udp”，可以方便地通过这个标签来引用和管理该镜像。
  - 例如，在后续运行容器时，可以使用 “docker run -it safe:udp” 来启动基于这个镜像的容器。
- `-f safe-udp.dockerfile`：
  - “-f” 参数用于指定构建镜像**所使用的 Dockerfile 文件**。在这个例子中，指定的 Dockerfile 文件名为 “safe-udp.dockerfile”。
- `.`表示 Docker 构建上下文的路径。Docker 在构建镜像时，会将这个路径下的文件和目录作为构建上下文发送给 Docker 守护进程。构建过程中可以在 Dockerfile 中引用构建上下文中的文件。

### 3 编写docker_run.sh

```sh
# $()是命令
# ${}是环境变量

# 获取Linux环境变量用于初始化docker
MONITOR_HOME_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}")/../.." && pwd )"
local_host="$(hostname)"
user="${USER}"
uid="$(id -u)"
group="$(id -g -n)"
gid="$(id -g)"

# 真正的docker操作
# 关闭旧容器
echo "stop and rm docker"
docker stop my_udp > /dev/null
docker rm -v -f my_udp > /dev/null
# 启动新容器
# -i：可交互 t：伪终端
# -v挂载
# ${XDG_RUNTIME_DIR}表示1000这个目录
# 注意事项： '\'前必有空格，后必不能有空格
# --network host直接使用本机的网络配置
# my:udp my：image REPOSITORY udp：image TAG
echo "start docker"
docker run -it -d \
--privileged=true \
--name my_udp \
-e DOCKER_USER="${user}" \
-e USER="${user}" \
-e DOCKER_USER_ID="${uid}" \
-e DOCKER_GRP="${group}" \
-e DOCKER_GRP_ID="${gid}" \
-v ${MONITOR_HOME_DIR}:/work \
-v ${XDG_RUNTIME_DIR}:${XDG_RUNTIME_DIR} \
-v /tmp:/tmp \
--network host \
my:tdp

```



### 4 编写docker_into.sh

```sh
docker exec \
    -u root \
    -it safe_udp \
    env LANG=C.UTF-8 \
    /bin/bash
```



- `docker exec`：
  - 这是 Docker 的一个命令，用于在正在运行的容器中执行命令。它允许你在不进入容器的情况下，直接在容器内部执行特定的命令或脚本。
- `-u root`：
  - 这个参数指定在容器中以 “root” 用户身份执行命令。这在需要执行需要管理员权限的操作时非常有用。例如，安装软件包或修改系统配置可能需要 root 权限。
- `-it`：
  - “-i” 表示允许交互输入，“-t” 表示分配一个伪终端（pseudo-tty）。这使得你可以在容器中像在终端中一样进行交互操作。
- `safe_udp`：
  - 这是要执行命令的容器的名称。Docker 根据这个名称找到对应的正在运行的容器，并在其中执行后续的命令。
- `env LANG=C.UTF-8`：
  - 这是在容器中设置环境变量的部分。在这里，设置了 “LANG” 环境变量为 “C.UTF-8”，这通常用于确保容器中的字符编码和语言设置符合特定的需求。例如，设置正确的语言环境可以确保命令的输出和错误消息以正确的字符编码显示。
- `/bin/bash`：
  - 这是要在容器中执行的命令，即启动一个 Bash shell。这使得你可以在容器的终端中进行各种操作，如运行命令、编辑文件、查看系统状态等。

# 待补坑：Docker联合文件系统







# 2. protobuf

## 定义

通信协议一般用分层模型，**不同模型功能定义及颗粒度不同**，例如：TCP/IP协议是一个**四层**协议，OSI模型是**7层**协议。OSI7层模型中展现层(Presentation Layer)功能是把**应用层对象(class)转换成一段连续的二进制串即序列化**，或者反序列化。**TCP/IP协议应用层对应OSI七层协议模型的应用层+展示层+会话层，所以序列化协议属于TCP/IP协议应用层的一部分。** 









## 序列化



### 为什么需要序列化和反序列化

序列化的目的是**将数据转换为一种通用的结构**，这样**其他系统或程序可以轻松地解析和使用这些数据**。这种通用的结构**通常是文本格式，如JSON、XML或YAML，但也可以是二进制格式，如Protocol Buffers**或MessagePack。

1. **数据可持久化**：

   - 序列化：通过将内存中的对象或数据结构转换为可存储的结构，如：二进制、json、xml等，这样数据就可以被保存在文件系统、数据库或其他持久存储介质中。

   - 反序列化：从存储介质中读取序列化的数据，并还原为内存中的对象或数据结构，允许数据在应用程序关闭后得以保留和恢复

2. **网络通信**：
   - 发送方将数据**序列化为可传输的格式**，数据能够在不同的计算机之间通过网络传递。接收方通过反序列化将接收到的数据还原成原始对象和数据结构，以便在本地使用。

3. **跨语言**： 



### 序列化协议特性

#### 1. 通用性

第一、技术层面，序列化协议是否支持**跨平台、跨语言**。如果不支持，在技术层面上的通用性就大大降低了。

第二、**流行程度**，序列化和反序列化需要多方参与，很少人使用的协议往往意味着昂贵的学习成本；另一方面，流行度低的协议，往往缺乏稳定而成熟的跨语言、跨平台的公共包。

#### 2. 强健性、鲁棒性

**成熟度**不够，一个协议从制定到实施，到最后成熟往往是一个漫长的阶段。协议的强健性依赖于大量而全面的测试，对于致力于提供高质量服务的系统，采用处于测试阶段的序列化协议会带来很高的风险。

#### 3. 可调试性、可读性

序列化和反序列化的数据正确性和业务正确性的**调试往往需要很长的时间**，良好的调试机制会大大提高开发效率。序列化后的**二进制串往往不具备人眼可读性**，为了验证序列化结果的正确性，写入方不得同时撰写反序列化程序，或提供一个查询平台–这比较费时；另一方面，如果读取方未能成功实现反序列化，这将给问题查找带来了很大的挑战–难以定位是由于自身的反序列化程序的bug所导致还是由于写入方序列化后的错误数据所导致。

#### 4.  性能

第一、**空间开销（Verbosity）**， 序列化需要在原有的数据上加上描述字段，以为反序列化解析之用。如果序列化过程引入的额外开销过高，可能会导致过大的网络，磁盘等各方面的压力。对于海量分布式存储系统，数据量往往以TB为单位，巨大的的额外空间开销意味着高昂的成本。

第二、**时间开销（Complexity）**，复杂的序列化协议会导致较长的解析时间，这可能会使得序列化和反序列化阶段成为整个系统的瓶颈。

#### 5. 可拓展性/兼容性

移动互联时代，业务系统需求的更新周期变得更快，新的需求不断涌现，而老的系统还是需要继续维护。如果序列化协议具有良好的可扩展性，支持自动增加新的业务字段，而不影响老的服务，这将大大提供系统的灵活度。

#### 6. 安全性/访问限制

在序列化选型的过程中，安全性的考虑往往发生在跨局域网访问的场景。当通讯发生在公司之间或者跨机房的时候，出于安全的考虑，对于**跨局域网的访问往往被限制为基于HTTP/HTTPS的80和443端口**。如果使用的序列化协议没有兼容而成熟的HTTP传输层框架支持，可能会导致以下三种结果之一：

第一、因为访问限制而降低服务可用性。 第二、被迫重新实现安全协议而导致实施成本大大提高。 第三、开放更多的防火墙端口和协议访问，而牺牲安全性。

### 数据结构、对象与二进制串

- 序列化：将**数据结构(struct)或对象(class)转换成二进制串(byte[])**的过程
- 反序列化：将在序列化过程中生成的**二进制串转换成数据结构或者对象**的过程

### 序列化底层原理

1. **IDL(Interface Description Language)**文件：为了建立一个与语言和平台无关的约定，需要用**与具体开发语言、开放平台无关的约定**进行描述，这种语言称为**接口描述语言**，采用IDL撰写的协议约定称为IDL文件，例如：protobuf对应的idl文件叫.proto

   ![image-20240808110541375](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808110541375.png)

2. IDL Complier：对应IDL语言的的编译器，将IDL文件转换成各语言对应的**动态库**，例如：protobuf对应的编译器叫**protobuf_complier，简称protoc**

3. Stub/Skeleton Lib：**负责序列化和反序列化的具体工作代码**

   记住**传输过程使用的是序列化的数据**即可

   - stub：是一段部署在分布式系统客户端的代码，一方面**接收应用层的参数，并对其序列化**后通过底层协议栈发送到服务端，另一方面**接收服务端序列化后的结果数据，反序列化**后交给客户端应用层
   - skeleton：部署在服务端，功能与stub相反，从**传输层接收序列化参数，反序列化**后交给服务端应用层，并**将应用层的执行结果序列化后传送给客户端stub**

4. Client/Server：指的是**应用层程序代码**，应用层面对的是IDL生成的**特定语言的class或struct**，例如c++对应编译后的.pb.h

   ```Plaintext
   protoc --proto_path=IMPORT_PATH --cpp_out=DST_DIR --java_out=DST_DIR  path/to/file.proto
   # 指定特定语言
   --cpp_out生成 C++ 代码存储在DST_DIR
   --java_out生成 Java 代码存储在DST_DIR
   ```

5. 底层协议栈和互联网：序列化之后的数据**通过底层的传输层、网络层、链路层以及物理层协议转换成数字信号**在互联网中传递。

![image-20240808121114132](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808121114132.png)



![image-20240808122245604](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808122245604.png)

### 常见序列化

#### XML&SOAP

- XML(eXtensible Markup Language，可拓展标记语言)
- SOAP(Simple Object Access Protocol，简单对象访问协议)

##### XML

1. XML是一种常用的序列化和反序列化协议，跨机器、跨语言。设计之初用于对互联网文档进行标记，因此具有人和机器兼具可读性。
2. 但作为描述语言用于序列化对象显得冗余复杂，因此一般用于配置文件中，例如maven里的pom.xml

##### SOAP

1. 基于XML为序列化和反序列化的结构化消息传递协议，在互联网影响太大以至于我们给基于SOAP的解决方案一个特定的名称–Web service，常见的使用方式是**XML+HTTP**。SOAP协议的IDL是**WSDL(Web Service Description Language, web描述语言)**

#### 区别

- xml简单好调试，适合于**传输量小和实时性低（秒级）**例如公司之间的通信
- xml**空间和时间开销大**，soap虽然是simple但绝对不简单，wsdl文件不直观。

#### JSON(Javascript Object Notation)

起源于JavaScript，采用"attribute - value"的方式描述对象，web典型应用是json+http，适合跨防火墙访问

优点：

1. 保持了xml的人眼可读(Human-readable)优点
2. 序列化数据更简洁
3. 与XML相比，协议比较简单，解析速度比较快
4. JSON太像语言里面的类，因此进行序列化时不需要IDL，是一种天然的序列化协议

应用场景：

1. 公司之间传输数据量小，实时性要求低
2. json有很强的前后兼容性，对于**接口经常发生变化**，并对可调性要求高的场景
3. JSON对序列化的**内存和磁盘开销大**，由于在一些语言的序列化和反序列化需要使用**反射机制**因此实时性较低**（秒级）**

#### protobuf

**空间小，高性能**

1. 有标准的IDL和标准的IDLC
2. 序列化数据非常简洁，是**XML的1/3到1/10**
3. 解析速度快，比XML**快20-100倍**
4. 动态库好用，**反序列化只需要一行代码** 
5. 是**纯粹的展示层协议**，可以和各种传输协议一起使用
6. 由于产生于Google，因此只支持Java、C++、python三种语言

## protbuf基础知识

### 数据类型

![image-20240808113106590](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808113106590.png)

### proto文件最终生成了什么

当运行 [protocol buffer compiler](https://protobuf.dev/programming-guides/proto3/#generating) 编译test.proto时，编译器会以您选择的语言生成代码，您需要使用文件中描述的消息类型，包括获取和设置字段值、将消息序列化为输出流，并从输入流解析消息。

- 对于**C++**，编译器会根据每个 `.proto` 生成一个`.h`和`.cc`文件，其中包含文件中描述的每种消息类型的类。
- 对于**Java**，编译器会生成一个`.java`文件，其中包含每个消息类型的类，以及`Builder`用于创建消息类实例的特殊类。

![image-20240808163024668](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808163024668.png)

### protobuf_generate

**cmake 指令**，把proto文件生成`pb.h`、`pb.cc`

- `APPEND_PATH`— 一个标志，导致所有原型模式文件的基本路径被添加到`IMPORT_DIRS`.
- `LANGUAGE`— 单个值：cpp 或 python。确定正在生成什么类型的源文件。
- `OUT_VAR`— CMake 变量的名称，该变量将填充生成的源文件的路径。
- `EXPORT_MACRO`— 应用于所有生成的 Protobuf 消息类和`extern`变量的宏的名称。
- `PROTOC_OUT_DIR`—生成的源文件的输出目录，默认为`CMAKE_CURRENT_BINARY_DIR`.
- `PLUGIN`— 可选的插件可执行文件。例如，这可能是通往 的路径`grpc_cpp_plugin`。
- `PLUGIN_OPTIONS`— 为插件提供的附加选项，例如`generate_mock_code=true`gRPC cpp 插件。
- `TARGET`— 生成的文件将作为源添加到提供的目标。
- `PROTOS`— 原型模式文件列表。如果省略，则将使用每个以`proto`of结尾的源文件。`TARGET`
- `IMPORT_DIRS`— 模式文件的公共父目录。例如，如果架构文件是`proto/helloworld/helloworld.proto`且导入目录是`proto/`，则生成的文件是`${PROTOC_OUT_DIR}/helloworld/helloworld.pb.cc`.
- `GENERATE_EXTENSIONS`— 如果`LANGUAGE`省略，则必须将其设置为`protoc`生成的扩展。
- `PROTOC_OPTIONS`— 转发到`protoc`调用的其他参数。

### 使用步骤

1. 定义IDL文件即.proto文件：test.proto

   ```protobuf
   syntax = "proto3";
   # package：生成java类的层级
   option java_package = "com.test.protobuf";
   # 若是c++则是 直接package
   package monitor.proto
   
   message CpuLoad {
       float load_avg_1 = 1;
       float load_avg_3 = 2;
       float load_avg_15 = 3;
     }
     
   message NetInfo {
       string name = 1;
       float send_rate = 2;
     }
     
    message MonitorInfo{
     CpuLoad cpu_load = 1; #std::string
     repeated NetInfo net_info = 2; # 此处的repeated相当于C++中的std::vector
   }
   ```

   ![image-20240808163024668](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808163024668.png)

2. 使用protoc编译生成动态库

   cd到/proto所在的目录

   ```sh
   protoc -I =./ --java_out=./ ./test.proto
   # -I：待compile的proto文件所在的目录
   # --java_out：编译生成的java文件存放的目录
   # ./test.proto：待编译的proto文件
   # 这里的三个路径要么全绝对要么全相对，不要混用
   
   # 编译生成.cc和.h的c++文件
   protoc -I =./ --cpp_out=./ ./test.proto
   ```

3. 使用protobuf的API来读写消息，常用API:

   ```cpp
   // 序列化
   bool SerializeToString(string* output) const; // 此处的string是二进制即序列化后的数据，选用string是因为它是个好用的容器
   
   // 反序列化
   bool ParseFromString(const string& data);
   
   // 将消息序列化为数组
   bool SerializeToArray(void * data, int size) const;
   
   // 将数组反序列化
   bool ParseFromArray(const void * data, int size);
   
   // 将消息序列化到C++的ostream中
   bool SerializeToOstream(ostream * output) const;
   
   // 将C++ isteam的数据反序列化
   bool ParseFromStream(istream * input);
   ```

   ```protobuf
   syntax = "proto3";
   package monitor.proto;
   
   # message相当于class的意思
   message CpuLoad {
       float load_avg_1 = 1;
       float load_avg_3 = 2;
       float load_avg_15 = 3;
     }
     
   message NetInfo {
       string name = 1;
       float send_rate = 2;
     }
     
   message MonitorInfo {
     std::string happly = 1;
     CpuLoad cpu_load = 2; #std::string
     repeated NetInfo net_info = 3; # std::vector
   }
   
   
   monitor::proto::MonitorInfo monitor_info;
   monitor_info.set_happly("1111");
   
   ::monitor::proto::CpuLoad* cpu_load_msg
   cpu_load_msg->set_load_avg_3(1.4);
   cpu_load_msg->set_load_avg_15(1.8); 
   
   ::monitor::proto::NetInfo*  net_info_msg1  = monitor_info.add_net_info();#适用于多个类型
   net_info_msg1->set_name("super-1");
   net_info_msg1->set_send_rate(12.5);
   
   auto net_info_msg2  = monitor_info.add_net_info();
   net_info_msg2->set_name("super-2");
   net_info_msg2->set_send_rate(8.5);
   
    // 对消息对象MonitorInfo序列化到string容器
   std::string serializedStr;
   monitor_info.SerializeToString(&serializedStr);
   std::cout<<"serialization result:"<<serializedStr<<std::endl; //序列化后的字符串内容是二进制内容，非可打印字符，预计输出乱码
   
   
   //反序列化
   monitor::proto::MonitorInfo monitor_info;
   monitor_info.ParseFromString(serializedStr)；
       
   std::cout << monitor_info.happly()<<std::endl;
   
   
   message CpuLoad {
       float load_avg_1 = 1;
       float load_avg_3 = 2;
       float load_avg_15 = 3;
     }
     
   auto cpu_load_parse =  monitor_info.cpu_load();
   std::cout << cpu_load_parse.load_avg_1()<< cpu_load_parse.load_avg_3()<<std::endl;
   
   
   for (int i = 0; i < monitor_info.net_info_size(); i++) {
           std::cout <<monitor_info.net_info(i).name();
           std::cout << monitor_info.net_info(i).send_rate();
   }
   ```

   

​	

# 3. monitor模块

## C++基础知识

### struct和class区别

1. struct：public；class：private
2. struct 默认是公有继承，即子类仍然是public；而 class 是私有继承，子类是private
3. struct：描述一个数据结构；class：对一个对象数据的封装

### C++虚拟内存

![image-20240811140857702](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240811140857702.png)

一个程序可由**代码段、数据段和BSS段**组成；**可执行程序**在运行时会出现：堆区和栈区

1. 代码段(Text Segment)

​	又称为文本段，由编译器编译代码后生成的**机器代码**组成

2. 数据段(Data Segment)

​	存储**已初始化的全局变量和静态变量**

```cpp
int globalVar = 10;  // 全局已初始化变量，存储在数据段
static int staticVar = 20;  // 静态已初始化变量，存储在数据段
```

3. BSS(Block Started by Symbol Segment)

​	存储**未初始化的全局变量和静态变量**

```cpp
int globalUninitVar;  // 全局未初始化变量，存储在 BSS 段
static int staticUninitVar;  // 静态未初始化变量，存储在 BSS 段
```

4. 栈

   - 由**编译器自动管理**，当函数被调用时为其分配栈空间，**执行完毕后自动释放栈空间，也是存放局部变量**的地方
   - 主要存储**局部变量、函数参数、返回值**，是一块**连续的空间**
   - **占据空间较小**，一般几兆字节；分配和释放速度快，因为是编译器自动处理
   - 从**高地址向低地址**生长

   ```cpp
   void someFunction() {
       int arr[1000];  // 在栈上分配
   }
   ```

5. 堆

   - 由**程序员手动管理**，使用new分配，delete释放
   - 主要存储**动态分配的对象、大型数据结构**
   - **占据空间大，达到计算机可用的虚拟内存大小**
   - 分配和释放**速度慢**，涉及系统调用和复杂的内存管理操作
   - 从低地址**向高地址方向**生长

   ```cpp
   int* arr = new int[1000];  // 在堆上分配
   delete[] arr;  // 手动释放
   ```

6. 最后还有一个**文件映射区**，位于堆和栈之间。

### C++抽象类（接口）

- C++没有官方定义的接口，用抽象类做接口
- 类中至少有一个函数声明为纯虚函数时，这个类就是抽象类

### 工厂模式

工厂模式包括了**面向接口编程**的思想

- **解耦对象的创建和使用**：使用者不需要关心对象具体是如何创建的，只关注如何使用创建好的对象。好比你去商店购买一台电视，你不需要了解电视的生产过程，只需要知道如何操作和使用它。
- **对象创建的封装**：将对象的创建过程封装在一个工厂类中，客户端无需直接创建对象，降低了对象创建的复杂性。例如，在一个汽车生产的场景中，客户端不需要知道汽车各个零部件的组装细节，只需要向汽车工厂请求一辆汽车即可。

#### 例子

1. 定义一个形状接口 `Shape`

```java
interface Shape {
    void draw();
}
```

2. 创建两个具体的形状类 `Circle` 和 `Rectangle` 实现 `Shape` 接口

```java
class Circle implements Shape {
    @Override
    public void draw() {
        System.out.println("绘制圆形");
    }
}

class Rectangle implements Shape {
    @Override
    public void draw() {
        System.out.println("绘制矩形");
    }
}
```

3. 创建工厂类 `ShapeFactory`

```java
class ShapeFactory {
    public Shape createShape(String type) {
        if (type.equalsIgnoreCase("circle")) {
            return new Circle();
        } else if (type.equalsIgnoreCase("rectangle")) {
            return new Rectangle();
        }
        return null;
    }
}
```

4. 在主函数中使用工厂模式创建对象并调用方法

```java
public class Main {
    public static void main(String[] args) {
        ShapeFactory factory = new ShapeFactory();
        Shape shape1 = factory.createShape("circle");
        Shape shape2 = factory.createShape("rectangle");

        shape1.draw();
        shape2.draw();
    }
}
```

通过工厂类 `ShapeFactory` 根据输入的类型创建不同的具体形状对象，使用者只需要与 `Shape` 接口交互，无需关心具体的实现类，提高了代码的灵活性和可扩展性



### 表

#### map

- **有序**
- 基于**红黑树**，value按照key的升序排列，与**key的大小顺序强相关**
- 时间复杂度**O(log n)**
- 数据量不大，对顺序有要求，例如学生的成绩

#### unordered_map

- 无序
- 基于**哈希表**实现，通过哈希函数将映射到特定的存储位置，数据存储位置与**key的哈希值直接相关**不依赖key值比较
- 时间复杂度**o(1) ，最坏情况o(n)**
- 适用于对顺序没有要求，更**注重CRUD操作**，例如网站的用户名-密码登陆校验



### lambda表达式

cpp在代码中的**任何位置定义的小型、无名函数**。

```cpp
[capture list] (param list) {function body};
[capture list] () -> param type {function body};
// -> int 明确指定了这个 lambda 表达式的返回类型是 int 。
auto lambda = []() -> int { return 42; };
```

传参方式：

1. 值捕获：获得的是变量的副本

2. 引用捕获：获得的是引用，修改会影响外部作用域原始变量

3. 混合捕获，[x ,&y]

4. 默认捕获：`[=]` 表示值捕获外部作用域中的所有自动变量。

   `[&]` 表示引用捕获外部作用域中的所有自动变量。

```cpp
// 值捕获
int factor = 2;
auto multiplyByFactor = [factor](int num) { 
    return num * factor; 
    // 尝试修改 factor 会导致编译错误，因为是值捕获，无法修改
    // factor = 3; 
};

// 引用捕获
auto lambda2 = [&b]() {
    std::cout << "Value of b inside lambda2 before modification: " << b << std::endl;
    b = 25;
    std::cout << "Value of b inside lambda2 after modification: " << b << std::endl;
};

multiplyByFactor();
lambda2();
```



### 智能指针

用于管理heap的动态内存管理

### C++多态

C++多态分为编译时多态（重载）和运行时多态（重写）；Java多态则是指对象多态（运行类型和编译类型可以不一致）和方法多态（重载和重写）

#### 编译时多态

- 在**编译阶段**编译器就能根据调用时提供的参数类型和数量决定调用哪个函数
- 地址早绑定early binding，根据**指针或引用的声明类型**在编译阶段就确定**函数的调用地址**

```cpp
void print(int num) {
    std::cout << "Integer: " << num << std::endl;
}

void print(double num) {
    std::cout << "Double: " << num << std::endl;
}
```

#### 运行时多态

- 通过**重写虚函数**实现，直接重写成员非虚函数不算体现C++多态性
- 函数地址晚绑定(late binding)，只有在运行时才确定，编译时无需检查对象类型只需检查对象是否支持特征和方法

```cpp
class Base {
public:
    // 定义虚函数
    virtual void virtualMethod() {
        std::cout << "Base::virtualMethod()" << std::endl;
    }
};

class Derived : public Base {
public:
    // 重写虚函数
    void virtualMethod() override {
        std::cout << "Derived::virtualMethod()" << std::endl;
    }
};

int main() {
    // 编译阶段只知道ptr是指向Base类型的指针
    Base* ptr = new Derived();
    ptr->virtualMethod();  // 调用派生类的虚函数实现
    delete ptr;
    return 0;
}
```

### 虚函数

1. 虚函数机制：当一个函数被声明为虚函数时，C++ 的运行时系统会建立一个**虚函数表（Virtual Function Table，简称 VTable）**来管理这些虚函数的地址。当通过基类的指针或引用调用虚函数时，运行时系统会**根据实际对象的类型在虚函数表中查找并调用相应的重写版本**
2. 在子类中重写虚函数的函数**没有显式使用virtual关键字，但仍然也是虚函数**
3. 行为的**灵活性**：重写虚函数允许子类根据自身的特点和需求来**定制**特定的行为。不同的子类可以对同一个虚函数有不同的实现，**使得相同的函数调用在不同的对象上产生不同的效果**，这正是多态性的核心概念
4. 如果子类没有重写父类的非析构虚函数，并且通过父类指针或引用调用该虚函数，那么会调用父类中定义的版本。

```cpp
#include <iostream>

class Base {
public:
    virtual void virtualMethod() {
        std::cout << "Base::virtualMethod()" << std::endl;
    }
};

class Derived : public Base {
public:
    // 但注意！override不能出现在类以外的地方
    void virtualMethod() override {
        std::cout << "Derived::virtualMethod()" << std::endl;
    }
};

int main() {
    Base* ptr = new Derived();
    ptr->virtualMethod();  // 输出 "Derived::virtualMethod()"，体现多态性
    delete ptr;
    return 0;
}
```

#### 纯虚函数

```cc
class Shape{
	public:
		// virtural  + " = 0" 表面其是一个纯虚函数，实现只能在子类实现
		virtual void hi() = 0;
}
```

#### override

h文件写了override，那么在源文件**一定要重写/实现**，否则直接报错

```sh
In file included from /work/test_monitor/src/monitor/cpu_softirq_monitor.cpp:2:0:
/work/test_monitor/include/monitor/cpu_softirq_monitor.h:14:47: error: expected class-name before '{' token
 class CpuSoftIrqMonitor : public MonitorInter {
                                               ^
/work/test_monitor/include/monitor/cpu_softirq_monitor.h:33:8: error: 'void monitor::CpuSoftIrqMonitor::Stop()' marked 'override', but does not override
   void Stop() override {}
        ^~~~
/work/test_monitor/src/monitor/cpu_softirq_monitor.cpp: In member function 'void monitor::CpuSoftIrqMonitor::UpdateOnce(monitor::proto::MonitorInfo*)':

```



### 析构函数

1. 资源清理：
   - 释放**动态分配的内存**，例如通过new操作符分配的内存
   - **关闭文件、网络连接、数据库连接等资源**
2. 对象销毁的收尾工作：
   - 执行必要的**清理工作**，如对象的状态重置、释放相关的锁
3. 避免内存泄漏
   - 确保在对象不再使用时，其所占用的资源得到正确释放，防止资源的持续占用导致内存泄漏

```cpp
class MyClass {
private:
    int* data;

public:
    MyClass(int size) {
        data = new int[size];
    }

    ~MyClass() {
        delete[] data;  // 在析构函数中释放动态分配的内存
    }
};

class FileHandler {
private:
    std::fstream file;

public:
    FileHandler(const std::string& filename) {
        file.open(filename);
    }
	// 如果子类没有显式实现析构函数则会隐式调用父类析构函数
    ~FileHandler() {
        if (file.is_open()) {
            file.close();  // 在析构函数中关闭文件
        }
    }
};
```

- 为什么Java没有析构函数：
  1. 自动垃圾回收机制：Java 有一个**自动的垃圾回收器**，它负责回收不再被使用的对象所占用的内存。程序员**不需要手动管理内存的释放**，因此也就不太需要像 C++ 那样的显式析构函数来执行特定的清理操作。
  2. 确定性的资源释放问题：在 Java 中，对于像文件、网络连接等资源的管理，通常是通过在**使用完后显式地调用相应的关闭方法来完成**，而不是依赖于析构函数。

### chrono 库

https://subingwen.cn/cpp/chrono/

#### 时间间隔 duration

duration表示一段时间间隔，用来记录时间长度，可以表示几秒、几分钟、几个小时的时间间隔

#### 时间点 time point

chrono 库中提供了一个表示时间点的类 time_point

#### 时钟clock

chrono 库中提供了获取当前的系统时间的时钟类，包含的时钟一共有三种：

- system_clock：系统的时钟，系统的时钟可以修改，甚至可以网络对时，因此使用系统时间计算时间差可能不准。
- steady_clock：是固定的时钟，相当于秒表。开始计时后，时间只会增长并且不能修改，适合用于记录程序耗时
- high_resolution_clock：和时钟类 steady_clock 是等价的（是它的别名）。

如果我们通过时钟不是为了获取当前的系统时间，而是进行程序耗时的时长，此时使用 syetem_clock 就不合适了，因为这个时间可以跟随系统的设置发生变化。在 C++11 中提供的时钟类 steady_clock 相当于秒表，只要启动就会进行时间的累加，并且不能被修改，非常适合于进行耗时的统计。



### Tips

#### explicit关键字

当一个构造函数被声明为 `explicit` 时，它只能用于显式的对象初始化，不能用于隐式的类型转换

```cpp
class MyClass {
public:
    int value;

    explicit MyClass(int v) : value(v) {}
};

MyClass obj(5);  // 显式初始化，合法
MyClass obj = 5;  // 非法，因为构造函数是 explicit
```

#### 头文件

为什么Java不用头文件

1. 编译模型差异：c++编译是按文件进行，编译每个源文件时需要提前知道其他文件中定义的类型、函数等的声明；Java**基于jvm**，**运行时动态加载和链接库**，因此不需要在编译时就知道所有的声明
2. 分离声明和实现：头文件负责存放类、函数、变量等的**声明**，源文件存放**实现**。提高可读性
3. 历史和设计理念：Java强调“**一次编写，到处运行**”的特性和更简洁的代码结构
4. 大型项目中：修改函数的实现，cpp只需**重写编译对应的源文件**，而包含该函数的其他源文件无需重新编译；Java则需要对**整个项目重写编译和打包**

##### 引用头文件

- `<>`:编译器会首先在预定义的目录（通常是系统目录或编译器指定的标准库目录）中查找要包含的头文件。例如，`#include <boost/chrono.hpp>` 会让编译器在标准库或特定的系统目录中查找 `boost/chrono.hpp` 头文件。
- `""`：编译器会首先在当前文件所在的目录查找头文件，如果在当前目录未找到，再按照与 `<>` 相同的方式在预定义的目录中查找。例如，`#include "monitor/monitor_inter.h"` 会先在当前文件所在的目录下查找 `monitor/monitor_inter.h` ，如果未找到，再去系统目录查找。

#### 命名规范

##### 变量

- 采用小写字母，多个单词之间用下划线 `_` 连接，例如 `my_variable` 。同python
- 对于类的成员变量，可以添加前缀 `m_` ，如 `m_member_variable` 

##### 函数

- 采用小写字母，多个单词之间用下划线 `_` 连接，例如 `my_function` 。同python
- 对于类的成员函数，与普通函数命名规则相同，或者使用驼峰命名法，如 `MyMemberFunction` 

##### 类

- 采用大驼峰命名法，即每个单词的首字母大写，例如 `MyClass`，同Java

##### 常量

- 全部使用大写字母，多个单词之间用下划线 `_` 连接，例如 `MAX_VALUE` 。

##### 命名空间

- 采用小写字母，多个单词之间用下划线 `_` 连接，例如 `my_namespace`

##### 指针和引用

- 如果变量是指针，在变量名前添加 `p_` ，如 `p_pointer` 。
- 如果变量是引用，在变量名前添加 `r_` ，如 `r_reference`

## Linux monitor

在GNU/Linux系统中，/proc是一个位于内存中的伪文件系统(in-memory)



### cpu load

/proc/loadavg中保存了CPU负载的平均值，三列分别表示1分钟、5分钟、15分钟的平均负载，反映了当前系统的繁忙情况

![image-20240808184620159](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808184620159.png)

```
lavg_1 (4.61) 1-分钟平均负载
lavg_5 (4.36) 5-分钟平均负载
lavg_15(4.15) 15-分钟平均负载
```

#### 平均负载

- 单位时间内，系统**处于可运行状态和不可中断状态**的平均进程数，也就是**平均活跃进程数**，或者**单位时间内等待运行的进程数量平均值**，与CPU使用率没有直接关系。它不仅考虑了正在使用 CPU 的进程，还包括**等待 CPU 资源（处于就绪状态）和等待 I/O 等其他资源**的进程。
- CPU使用率指的是**单位时间内CPU处在非空闲态的时间比**，反映了CPU的繁忙程度
- 高平均负载不一定意味着高 CPU 使用率，如果等待的进程主要是在等待 I/O 操作完成，CPU 使用率可能不高。

##### 举例

平均负载=2

- 对4个CPU系统：50%CPU空闲
- 2个CPU系统：100%CPU占用
- 1个CPU系统：**一半的进程竞争不到CPU**

平均负载**最理想的情况=CPU数量**，当平均负载>CPU个数时，说明出现了**过载红温警告警告**



- 如果1分钟平均负载>15分钟平均负载，说明最近1分钟负载在增加，可能是临时性的
- 如果1分钟平均负载<15分钟平均负载，说明最近1分钟负载在减小，过去15负载有很大的负载
- 如果1分钟平均负载接近或超过CPU个数，说明正在发生过载，要想办法优化
- 如果单CPU平均负载为1.73 0.60 7.98，说明过去1分钟有73%超载，过去15分钟有698%超载，整体趋势是系统负载在降低
- 实际生产中，平均负载>70%CPU时，就该分析负载高的问题

### cpustat

/proc/stat

- `user(通常缩写为us)`，代表用户态CPU时间，**不包括nice时间，但包括guest时间**
- `nice(ni)`：低优先级用户态CPU时间，即进程的nice值被调整为1-19之间时的CPU时间，nice可以是-20到19，数值越大，优先级越低
- `system(sys)`：内核态CPU时间
- `idle(id)`：空闲时间，不包括等待I/O的时间(iowait)
- `iowait(wa)`：等待I/O的CPU时间
- `irq(hi)`：处理硬中断的CPU时间
- `softirq(si)`：处理软中断的CPU时间
- `steal(st)`：系统运作在虚拟机时，被其他虚拟机占用的CPU时间
- `guest(guest)`：虚拟化运行其他操作系统的时间，即运行虚拟机的时间
- `guest_nice(gnice)`：低优先级运行虚拟机的时间

#### cpu使用率

![image-20240808200344411](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808200344411.png)

=忙碌时间/CPU时间

间隔一段时间**(比如3秒)的2次值，做差后**，再计算出这段时间内的平均CPU使用率

![image-20240808200404137](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808200404137.png)

### cpu softirqs

- `TIMER`：定时中断
- `NET_TX`：网络发送
- `NET_RX`：网络接收
- `SCHED`：内核调度
- `RCU`：RCU锁中，网络接收变化最快

```shell
root@yiga-virtual-machine:/proc# cat /proc/softirqs
                    CPU0       CPU1       CPU2       CPU3       CPU4       CPU5       CPU6       CPU7       CPU8       CPU9       CPU10      CPU11      CPU12      CPU13      CPU14      CPU15      CPU16      CPU17      CPU18      CPU19      CPU20      CPU21      CPU22      CPU23      CPU24      CPU25      CPU26      CPU27      CPU28      CPU29      CPU30      CPU31      CPU32      CPU33      CPU34      CPU35      CPU36      CPU37      CPU38      CPU39      CPU40      CPU41      CPU42      CPU43      CPU44      CPU45      CPU46      CPU47      CPU48      CPU49      CPU50      CPU51      CPU52      CPU53      CPU54      CPU55      CPU56      CPU57      CPU58      CPU59      CPU60      CPU61      CPU62      CPU63      CPU64      CPU65      CPU66      CPU67      CPU68      CPU69      CPU70      CPU71      CPU72      CPU73      CPU74      CPU75      CPU76      CPU77      CPU78      CPU79      CPU80      CPU81      CPU82      CPU83      CPU84      CPU85      CPU86      CPU87      CPU88      CPU89      CPU90      CPU91      CPU92      CPU93      CPU94      CPU95      CPU96      CPU97      CPU98      CPU99      CPU100     CPU101     CPU102     CPU103     CPU104     CPU105     CPU106     CPU107     CPU108     CPU109     CPU110     CPU111     CPU112     CPU113     CPU114     CPU115     CPU116     CPU117     CPU118     CPU119     CPU120     CPU121     CPU122     CPU123     CPU124     CPU125     CPU126     CPU127     
          HI:          0          0          1     199512          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
       TIMER:     575593     590851     962137    1908052          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
      NET_TX:          5          1          2          7          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
      NET_RX:      37873      37159      32595     323769          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
       BLOCK:      76799      21475      55407      29366          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
    IRQ_POLL:          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
     TASKLET:        241          2         28     799154          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
       SCHED:    1494798    1360639    1713100    3154266          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
     HRTIMER:          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
         RCU:     880363     867429     995143    1138718          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0          0
```

查看中断次数

- /proc/softirqs：记录**开机以来软中断**累计次数
- /proc/interrupts：记录**开机以硬中断**累计次数



- 中断是一种**异步的事件处理机制**，与单片机、java的中断机制相同，提高系统并发处理能力
- 为了解决中断处理程序执行过长和中断丢失的问题，Linux将中断处理过程分成了两个阶段
  - 上半部(top half)：用于**快速**处理中断，在中断禁止模式下运行，处理跟**硬件紧密相关或时间敏感的工作**
  - 下半部(bottom half)：用于**延迟**处理上半部未完成的工作，由于中断处理程序需要尽快完成，以减少对系统性能的影响，一些**耗时**的操作会被**推迟到中断下半部（Bottom Half）**处理。常见的中断下半部机制包括**软中断（SoftIRQ）、任务队列（Task Queue）和工作队列（Work Queue）**等。
  - 下半部以**内核线程**的方式执行，并且每个 CPU 都对应

#### 硬中断

 由**外部硬件设备（如键盘**、鼠标、网卡等）产生，通知 CPU 设备需要服务或发生了重要事件。

#### 软中断

由软件指令触发，例如系统调用或异常。

#### 举例

- `Top Half`（上半部）：

​	这部分的中断处理程序需要尽快执行并且要尽可能地简短，主要完成一些**关键和紧急**的任务，例如**保存中断现场、识别中断源**等。

​	例如，当网络数据包到达网卡产生中断时，上半部可能只是简单地将数据包**接收并放入一个缓冲区，然后标记数据包已到达**。

- `Bottom Half`（下半部）：

​	下半部在稍后执行，用于处理那些相对不太紧急或者耗时较长的任务。

​	比如，还是对于网络数据包的处理，下半部可能会对数据包进行**解析、处理协议、将数据传递给上层应用程序等更复杂和耗时的操作**。

​	另一个例子是磁盘 I/O 中断。上半部可能只是确认 I/O 操作完成，将相关信息**记录**下来。而下半部则负责**将数据从磁盘缓冲区传输到用户空间的缓冲区**，或者更新文件系统的相关数据结构。

​	通过将中断处理分为上半部和下半部，可以在保证及时响应中断的同时，又能有效地处理那些复杂和耗时的任务，提高系统的整体性能和响应能力。



##### RCU锁

内核使用的同步机制，解决在多处理器环境下，读操作>>写操作情况下的并发数据访问问题

1. 读端无锁：读操作无任何锁，可以自由并发进行
2. 写操作：先复制修改的数据然后修改副本，然后修改副本，完成后等待时机（grace period，大大的timing）来删除旧数据
3. 内存回收：在grace period期间，确保所有正在进行的读操作完成后才安全的释放旧数据

### meminfo

/proc/meminfo

1. `MemTotal`：系统总的物理内存量。
2. `MemFree`：未被使用的物理内存量。
3. `MemAvailable`：估算的可供新进程使用的物理内存量。
4. `Buffers`：用于块设备（如磁盘）I/O 缓冲区的内存量。
5. `Cached`：用于文件系统缓存的内存量。
6. `SwapCached`：被交换出去但仍在交换缓存中的内存量。
7. `Active`：活跃使用的内存量，通常是最近被使用过的内存。
8. `Inactive`：不活跃使用的内存量，近期未被使用。
9. `Active(anon)`：活跃的匿名内存量（不与文件关联的内存）。
10. `Inactive(anon)`：不活跃的匿名内存量。
11. `Active(file)`：活跃的与文件关联的内存量。
12. `Inactive(file)`：不活跃的与文件关联的内存量。
13. `Unevictable`：不能被回收的内存量。
14. `Mlocked`：被锁定在内存中的内存量。
15. `SwapTotal`：交换分区的总容量。
16. `SwapFree`：交换分区的空闲容量。
17. `Dirty`：等待写回磁盘的脏页内存量。
18. `Writeback`：正在写回磁盘的内存量。
19. `AnonPages`：匿名页的内存量（不与文件关联）。
20. `Mapped`：映射到进程地址空间的文件内存量。
21. `Shmem`：共享内存的使用量。
22. `Slab`：内核 slab 分配器使用的内存量。
23. `SReclaimable`：可回收的 slab 内存量。
24. `SUnreclaim`：不可回收的 slab 内存量。
25. `KernelStack`：内核栈使用的内存量。
26. `PageTables`：页表使用的内存量。
27. `NFS_Unstable`：NFS 不稳定页的内存量。
28. `Bounce`：用于 DMA 映射的内存量。
29. `WritebackTmp`：临时的写回内存量。
30. `CommitLimit`：内存分配的提交限制。
31. `Committed_AS`：已提交的内存量。
32. `VmallocTotal`：虚拟内存的总分配量。
33. `VmallocUsed`：已使用的虚拟内存量。
34. `VmallocChunk`：最大的连续可用虚拟内存块大小。

```shell
yiga@yiga-virtual-machine:~$ cat /proc/meminfo
MemTotal:        3963556 kB
MemFree:          202660 kB
MemAvailable:    2108796 kB
Buffers:          292316 kB
Cached:          1508820 kB
SwapCached:          520 kB
Active:          1377516 kB
Inactive:        1476136 kB
Active(anon):     855068 kB
Inactive(anon):   237160 kB
Active(file):     522448 kB
Inactive(file):  1238976 kB
Unevictable:           0 kB
Mlocked:               0 kB
SwapTotal:       2191356 kB
SwapFree:        2190064 kB
Zswap:                 0 kB
Zswapped:              0 kB
Dirty:               852 kB
Writeback:             0 kB
AnonPages:       1052028 kB
Mapped:           481260 kB
Shmem:             39756 kB
KReclaimable:     433624 kB
Slab:             596084 kB
SReclaimable:     433624 kB
SUnreclaim:       162460 kB
KernelStack:       13216 kB
PageTables:        24524 kB
SecPageTables:         0 kB
NFS_Unstable:          0 kB
Bounce:                0 kB
WritebackTmp:          0 kB
CommitLimit:     4173132 kB
Committed_AS:    5052008 kB
VmallocTotal:   34359738367 kB
VmallocUsed:       34760 kB
VmallocChunk:          0 kB
Percpu:           104448 kB
HardwareCorrupted:     0 kB
AnonHugePages:         0 kB
ShmemHugePages:        0 kB
ShmemPmdMapped:        0 kB
FileHugePages:         0 kB
FilePmdMapped:         0 kB
Unaccepted:            0 kB
HugePages_Total:       0
HugePages_Free:        0
HugePages_Rsvd:        0
HugePages_Surp:        0
Hugepagesize:       2048 kB
Hugetlb:               0 kB
DirectMap4k:      173888 kB
DirectMap2M:     2971648 kB
DirectMap1G:     3145728 kB
```

#### free

1.  `total` 表示系统总的内存大小。
2.  `used` 表示已使用的内存大小。
3.  `free` 表示**未使用**的内存大小。
4.  `shared` 表示**共享**内存大小。
5.  `buff/cache` 表示**缓冲区和缓存**使用的内存大小。
6.  `available` 表示可用于**新进程分配**的可用内存大小。

##### 共享内存

共享内存是一种进程间通信（IPC）机制，允许多个进程访问同一块物理内存区域。比如，在一个多媒体处理系统中，多个进程可能需要同时访问和处理同一段视频数据，共享内存可以提供快速的数据访问，提高处理效率。

### net

/proc/net/dev 网络流入流出的统计信息，包括接收包的数量、发送包的数量，发送数据包时的错误和冲突情况等

![image-20240808203424147](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240808203424147.png)

- `Receive` 部分的字段：
  - `bytes`：接收到的字节数。
  - `packets`：接收到的数据包数量。
  - `errs`：接收错误的数量。
  - `drop`：接收时丢弃的数据包数量。
  - `fifo`：由于 FIFO 缓冲区溢出而丢弃的数据包数量。
  - `frame`：帧对齐错误的数量。
  - `compressed`：接收到的压缩数据包数量。
  - `multicast`：接收到的多播数据包数量。
- `Transmit` 部分的字段：
  - 含义与 `Receive` 部分类似，但统计的是发送的相关数据。



### stress压测指令

1. `-c` ：指定要产生的 CPU 工作进程的数量。    - 例如：`stress -c 4` 表示启动 4 个 CPU 工作进程。 
2. `-i` ：指定要产生的 I/O 操作进程的数量。
3. `-m` ：指定要产生的内存分配进程的数量。 
4. `-d` ：指定要产生的磁盘写进程的数量。 ` 
5. `-n` ：指定要产生的网络操作进程的数量。
6. `-t` ：指定运行的时间，后面接时间值和时间单位，如 `stress -t 60s` 表示运行 60 秒。
7. `-v` ：显示详细的信息，包括进程的创建和资源使用情况。 通过组合这些选项，可以模拟不同类型和程度的系统负载，以测试系统在压力下的性能和稳定性。 





# 4. CMake

CMakeList.txt文件内，指令类推荐用小写



## example

### 1. hello world

```cmake
# 设置最低cmake版本
cmake_minimum_required(VERSION 3.15)
# 设置项目名
project(hello_cmake)
# 设置build后生成的exe文件
add_executable(hello_cmake_bin main.cpp)
```

build

```shell
mkdir build
cd build
cmake ..
make 
./hello_cmake
```

### 

### 2. 添加依赖库

```cmake
cmake_minimum_required(VERSION 3.15)
project(hello_cmake)
# 添加依赖
add_library(hello_library STATIC
	src/Hello.cpp)
# 更改别名
add_library(hello::library ALIAS hello_library)
# 引入头文件
target_include_directories(hello::library
	PUBLIC
		${PROJECT_SOURCE_DIR}/include # PROJECT_SOURCE_DIR：cmakefiles所在的目录；
									  # PROJECT_BINARY_DIR：cmake编译后所在的目录，例如/build
)

add_executable(hello_cmake_bin 
	src/main.cpp)

# 链接依赖
target_link_libraries(hello_cmake_bin
	PRIVATE
		hello::library
)
```



### 3. install

```cmake
cmake_minimum_required(VERSION 3.15)
project(hello_install)
add_library(hello_library SHARED
	src/Hello.cpp)	
target_include_directories(hello_library
	PUBLIC
		${PROJECT_SOURCE_DIR}/include
)
add_executable(hello_install_bin
	src/main.cpp
)
target_link_libraries(hello_install_bin
	PRIVATE
		hello_library
)

# 将可执行文件安装到/usr/local/bin目录下，文件名：hello_install_bin
install(TARGETS hello_install
	DESTINATION bin)
# 将library库文件安装到/usr/local/lib目录下，文件名：hello_library.so
install(TARGETS hello_library
	LIBRARY DESTINATION lib)
# 头文件,注意用的是directory
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
	DESTINATION include)
# 注意用的是file，配置(config)文件，将cmake-examples.conf安装到/usr/local/etc目录下
install(FILES cmake-examples.conf
	DESTINATION etc)
```

```shell
mkdir build
cd build
cmake ..
make 

sudo make install
# 卸载
sudo xargs rm < install_manifest.txt
```



### 4. CPack

```cmake
cmake_minimum_required(VERSION 3.15)
project(hello_cpack)
add_library(library SHARED
	src/hello.cpp
)
target_include_directories(hello_cpack
	PUBLIC
		${PROJECT_SOURCE_DIR}/include
)
add_executable(hello_cpack_bin
	src/main.cpp
)
target_link_libraries(hello_cpack_bin
	PUBLIC
		library
)

install(TARGETS hello_cpack_bin
	DESTINATION bin
)
install(TARGETS library
	LIBRARY DESTINATION lib
)
install(DIRECTORY ${PROJECT_SOURCE_DIR}/include/
	DESTINATION include
)
install(FILE cmake-examples.conf
	DESTINATION etc
)

# 设置打包的文件类型
set(CPACK_GENERATOR "DEB")
# 设置打包人
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "yiga")
# 设置包版本号
set(CPACK_PACKAGE_VERSION "0.2.1")
# 打包启动！
include(Cpack)
```

```shell
mkdir build
cd build
cmake ..
make 

make help
make package
# 会将Debian软件包放在${PROJECT_BINARY_DIR}目录下
```



### 5. 利用头文件模板生成头文件

```cmake
cmake_minimum_required(VERSION 3.5)
project(hello_cg)

set(hello_cg_VERSION_MAJOR 0)
set(hello_cg_VERSION_MINOR 2)
set(hello_cg_VERSION_PATCH 1)
set(hello_cg_VERSION"${hello_cg_VERSION_MAJOR}.${hello_cg_VERSION_MINOR}.${hello_cg_VERSION_PATCH}")

# 根据ver.h.in头文件模板在PROJECT_BINARY_DIR目录下生成头文件
# 在cmake构建过程（即在make之前）已生成
configure_file(ver.h.in ${PROJECT_BINARY_DIR}/ver.h)
# @ONLY：只有使用@VAR@的方式才能引用被改变的量
configure_file(path.h.in ${PROJECT_BINARY_DIR}/path.h @ONLY)

add_executable(hello_cg_bin
	src/main.cpp
)
# 引用新生成的头文件目录
target_include_directories(hello_cg
	PUBLIC
		${PROJECT_BINARY_DIR}
)
```

`path.h.in`

```h
#ifndef __PATH_H__
#define __PATH_H__

// version variable that will be substituted by cmake
// This shows an example using the @ variable type
// 此处的@CMAKE_SOURCE_DIR@来自于camke文件
const char *path = "@CMAKE_SOURCE_DIR@";

#endif
```

`ver.h.in`

```h
#ifndef __VER_H__
#define __VER_H__

// version variable that will be substituted by cmake
// This shows an example using the $ variable type
// 此处的$hello_cg_VERSION来自于camke文件定义
const char* ver = "${hello_cg_VERSION}";

#endif
```

`path.h`

```h
#ifndef __PATH_H__
#define __PATH_H__

// version variable that will be substituted by cmake
// This shows an example using the @ variable type
const char *path = "/home/yiga/work/cmake-examples/03-code-generation/configure-files";

#endif
```

`ver.h`

```h
#ifndef __VER_H__
#define __VER_H__

// version variable that will be substituted by cmake
// This shows an example using the $ variable type
const char* ver = "0.2.1";

#endif
```

### 6. 子项目

```cmake
cmake_minimum_required(VERSION 3.15)
project(subprojects)
# 加入子项目
add_subdirectory(sublibrary)
add_subdirectory(sublibrary1)
add_subdirectory(sublibrary2)
```

`subbinary/CMakeLists.txt`

```cmake
project(subbinary)
add_executable(${PROJECT_NAME} main.cpp)

# Link the static library from subproject1 using its alias sub::lib1
# Link the header only library from subproject2 using its alias sub::lib2
# This will cause the include directories for that target to be added to this project
# 这里居然能直接识别sub::lib1？
target_link_libraries(${PROJECT_NAME}
    sub::lib1
    sub::lib2
)
```

`subbinary1/CMakeLists.txt`

```cmake
project (sublibrary1)
add_library(${PROJECT_NAME} src/sublib1.cpp)
add_library(sub::lib1 ALIAS ${PROJECT_NAME})
target_include_directories( ${PROJECT_NAME}
    PUBLIC ${PROJECT_SOURCE_DIR}/include
)

```

`subbinary2/CMakeLists.txt`

```cmake
project (sublibrary2)
add_library(${PROJECT_NAME} INTERFACE)
add_library(sub::lib2 ALIAS ${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}
    INTERFACE
        ${PROJECT_SOURCE_DIR}/include
)
```

### 7. 引入第三方包

```cmake
cmake_minimum_required(VERSION 3.5)
project (imported_targets)

# 在本地找名为Boost的包，要求版本号1.46.1 且一定要有filesystem 和 system部件
find_package(Boost 1.46.1 REQUIRED COMPONENTS filesystem system)
add_executable(imported_targets main.cpp)

# 将第三方包链接bin文件
target_link_libraries( imported_targets
    PRIVATE
        Boost::filesystem
)

```



### 8. C++规范

设置C++11规范

```cmake
set(CMAKE_CXX_STANDARD 11)
```

使用`clang`编译

```cmake
mkdir build.clang
cd build.clang/
cmake .. -DCMAKE_C_COMPILER=clang-3.6 -DCMAKE_CXX_COMPILER=clang++-3.6
make VERBOSE=1
```

使用`ninja`编译

```bash
mkdir build.ninja
cd build.ninja/
cmake .. -G Ninja
cd build.ninja/
ninja -v
```

## bug examples

### 1. undefined reference

#### 1

![image-20240819201317788](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240819201317788.png)

- CMake文件中没有**链接**正确的库boost::system

  ```CMAKE
  # 这里一定要REQUIRED COMPONENTS system，不然找到的boost是残缺的！
  find_package(Boost REQUIRED COMPONENTS system)
  target_link_libraries(hi
      libraries
      Boost::system
      protobuf::libprotobuf
  )
  ```

  ```sh
  MakeFiles/hi.dir/main.cpp.o: In function `__static_initialization_and_destruction_0(int, int)':
  main.cpp:(.text+0xd7): undefined reference to `boost::system::generic_category()'
  main.cpp:(.text+0xe3): undefined reference to `boost::system::generic_category()'
  main.cpp:(.text+0xef): undefined reference to `boost::system::system_category()'
  CMakeFiles/hi.dir/main.cpp.o: In function `boost::system::error_category::std_category::equivalent(int, std::error_condition const&) const':
  main.cpp:(.text._ZNK5boost6system14error_category12std_category10equivalentEiRKSt15error_condition[_ZNK5boost6system14error_category12std_category10equivalentEiRKSt15error_condition]+0xb8): undefined reference to `boost::system::generic_category()'
  main.cpp:(.text._ZNK5boost6system14error_category12std_category10equivalentEiRKSt15error_condition[_ZNK5boost6system14error_category12std_category10equivalentEiRKSt15error_condition]+0xf3): undefined reference to `boost::system::generic_category()'
  CMakeFiles/hi.dir/main.cpp.o: In function `boost::system::error_category::std_category::equivalent(std::error_code const&, int) const':
  main.cpp:(.text._ZNK5boost6system14error_category12std_category10equivalentERKSt10error_codei[_ZNK5boost6system14error_category12std_category10equivalentERKSt10error_codei]+0xb8): undefined reference to `boost::system::generic_category()'
  main.cpp:(.text._ZNK5boost6system14error_category12std_category10equivalentERKSt10error_codei[_ZNK5boost6system14error_category12std_category10equivalentERKSt10error_codei]+0xf3): undefined reference to `boost::system::generic_category()'
  main.cpp:(.text._ZNK5boost6system14error_category12std_category10equivalentERKSt10error_codei[_ZNK5boost6system14error_category12std_category10equivalentERKSt10error_codei]+0x1d2): undefined reference to `boost::system::generic_category()'
  collect2: error: ld returned 1 exit status
  CMakeFiles/hi.dir/build.make:97: recipe for target 'hi' failed
  make[2]: *** [hi] Error 1
  CMakeFiles/Makefile2:104: recipe for target 'CMakeFiles/hi.dir/all' failed
  make[1]: *** [CMakeFiles/hi.dir/all] Error 2
  Makefile:83: recipe for target 'all' failed
  make: *** [all] Error 2
  
  ```

#### 2 忘记link package

![image-20240819202858890](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240819202858890.png)

```sh
liblibraries.a(net_monitor.cpp.o): In function `monitor::NetMonitor::UpdateOnce(monitor::proto::MonitorInfo*)':
net_monitor.cpp:(.text+0x2e2): undefined reference to `boost::chrono::steady_clock::now()'
collect2: error: ld returned 1 exit status
CMakeFiles/hi.dir/build.make:98: recipe for target 'hi' failed
make[2]: *** [hi] Error 1
CMakeFiles/Makefile2:104: recipe for target 'CMakeFiles/hi.dir/all' failed
make[1]: *** [CMakeFiles/hi.dir/all] Error 2
Makefile:83: recipe for target 'all' failed
make: *** [all] Error 2

```

- CMake文件中没有**链接**正确的库boost::chrono

```
find_package(Boost REQUIRED COMPONENTS system chrono)
target_link_libraries(hi
    libraries
    Boost::system
    Boost::chrono
    protobuf::libprotobuf
)


```

#### 3 忘记link源文件

![image-20240819211859786](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240819211859786.png)

```sh
CMakeFiles/hi.dir/main.cpp.o: In function `monitor::MemMonitor::MemMonitor()':
main.cpp:(.text._ZN7monitor10MemMonitorC2Ev[_ZN7monitor10MemMonitorC5Ev]+0x1b): undefined reference to `vtable for monitor::MemMonitor'
CMakeFiles/hi.dir/main.cpp.o: In function `monitor::MemMonitor::~MemMonitor()':
main.cpp:(.text._ZN7monitor10MemMonitorD2Ev[_ZN7monitor10MemMonitorD5Ev]+0xf): undefined reference to `vtable for monitor::MemMonitor'
collect2: error: ld returned 1 exit status
CMakeFiles/hi.dir/build.make:100: recipe for target 'hi' failed
```

- main函数中include了头文件并且调用了其中的成员

  ```cpp
  #include "monitor/net_monitor.h"
   monitor::MemMonitor mem_monitor;
  ```

  **但是没有在cmake中add并link其源文件！**

- ```cmake
  add_library(libraries
      ${CMAKE_SOURCE_DIR}/src/monitor/mem_monitor.cpp
      ...
  }
  target_link_libraries(hi
      libraries
      ...
  }
  ```

#### 4 target_link_library写错

```cmake
CMakeFiles/rpc_client.dir/rpc_client.cpp.o: In function `grpc::InsecureChannelCredentials()':
rpc_client.cpp:(.text+0x144): undefined reference to `grpc_impl::InsecureChannelCredentials()'
CMakeFiles/rpc_client.dir/rpc_client.cpp.o: In function `grpc::CreateChannel(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::shared_ptr<grpc_impl::ChannelCredentials> const&)':
rpc_client.cpp:(.text+0x198): undefined reference to `grpc_impl::CreateChannelImpl(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&, std::shared_ptr<grpc_impl::ChannelCredentials> const&)'
CMakeFiles/rpc_client.dir/rpc_client.cpp.o: In function `monitor::RpcClient::RpcClient(std::__cxx11::basic_string<char, std::char_traits<char>, std::allocator<char> > const&)':
rpc_client.cpp:(.text+0x237): undefined reference to `monitor::proto::GrpcManager::NewStub(std::shared_ptr<grpc::ChannelInterface> const&, grpc::StubOptions const&)'
CMakeFiles/rpc_client.dir/rpc_client.cpp.o: In function `monitor::RpcClient::SetMonitorInfo(monitor::proto::MonitorInfo const&)':
rpc_client.cpp:(.text+0x30c): undefined reference to `grpc_impl::ClientContext::ClientContext()'
rpc_client.cpp:(.text+0x357): undefined
```





```cmake
link_libraries(rpc_client
lib
)
# 更正
target_link_libraries(rpc_client
lib
)
```

#### 5 add源文件少了

```cmake
liblib.a(monitor_info.pb.cc.o): In function `monitor::proto::MonitorInfo::clear_cpu_load()':
monitor_info.pb.cc:(.text+0x113): undefined reference to `monitor::proto::CpuLoad::~CpuLoad()'
liblib.a(monitor_info.pb.cc.o): In function `monitor::proto::MonitorInfo::clear_mem_info()':
monitor_info.pb.cc:(.text+0x199): undefined reference to `monitor::proto::MemInfo::~MemInfo()'
liblib.a(monitor_info.pb.cc.o): In function `monitor::proto::MonitorInfo::MonitorInfo(monitor::proto::MonitorInfo const&)':
monitor_info.pb.cc:(.text+0x493): undefined reference to `monitor::proto::CpuLoad::CpuLoad(monitor::proto::CpuLoad const&)'
monitor_info.pb.cc:(.text+0x4d9): undefined reference to `monitor::proto::MemInfo::MemInfo(monitor::proto::MemInfo const&)'
liblib.a(monitor_info.pb.cc.o): In function `monitor::proto::MonitorInfo::Clear()':
monitor_info.pb.cc:(.text+0x71a): undefined reference to `monitor::proto::CpuLoad::~CpuLoad()'
monitor_info.pb.cc:(.text+0x771): undefined reference to `monitor::proto::MemInfo::~MemInfo()'

```



```cmake
add_library(lib
${CMAKE_SOURCE_DIR}/../../proto/build/monitor_info.grpc.pb.cc

    ${CMAKE_SOURCE_DIR}/../../proto/build/monitor_info.pb.cc

)
# 修正
add_library(lib
${CMAKE_SOURCE_DIR}/../../proto/build/monitor_info.grpc.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/cpu_load.grpc.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/cpu_stat.grpc.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/cpu_softirq.grpc.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/mem_info.grpc.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/net_info.grpc.pb.cc

    ${CMAKE_SOURCE_DIR}/../../proto/build/monitor_info.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/cpu_load.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/cpu_stat.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/cpu_softirq.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/mem_info.pb.cc
    ${CMAKE_SOURCE_DIR}/../../proto/build/net_info.pb.cc
)
```

#### 6 构造函数定义了但没实现

```cmake
CMakeFiles/display.dir/monitor_widget.cpp.o: In function `monitor::MonitorWidget::InitCpuStatMonitorWidget()':
monitor_widget.cpp:(.text+0x8bc): undefined reference to `monitor::CpuStatModel::CpuStatModel(QVariant*)'
collect2: error: ld returned 1 exit status
CMakeFiles/display.dir/build.make:615: recipe for target 'display' failed

```



```cpp
  CpuStatModel(QVariant *parent = nullptr)
  # 修正
	CpuStatModel(QVariant *parent = nullptr){}
```



### 2.incomplete type

#### 1

```sh
/work/test_monitor/src/monitor/mem_monitor.cpp: In member function 'virtual void monitor::MemMonitor::UpdateOnce(monitor::proto::MonitorInfo*)':
/work/test_monitor/src/monitor/mem_monitor.cpp:10:18: error: aggregate 'monitor::MemMonitor::UpdateOnce(monitor::proto::MonitorInfo*)::MemInfo mem_info' has incomplete type and cannot be defined
```

- 在头文件或者源文件都不存在`MemInfo`这个类型，可能是写错了写出来`MenInfo`

#### 2

![image-20240819220829086](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240819220829086.png)

![image-20240819220842926](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240819220842926.png)

只看`error`就行：经典`imcplete type`未定义，找到蓝色划线报错处`SoftIrq`发现是改了变量名后忘了全局更改，应为`CpuSoftIrq`

```
In file included from /usr/include/c++/7/bits/stl_algobase.h:64:0,
                 from /usr/include/c++/7/bits/char_traits.h:39,
                 from /usr/include/c++/7/ios:40,
                 from /usr/include/c++/7/ostream:38,
                 from /usr/include/c++/7/iostream:39,
                 from /work/test_monitor/main.cpp:1:
/usr/include/c++/7/bits/stl_pair.h: In instantiation of 'struct std::pair<const std::__cxx11::basic_string<char>, monitor::SoftIrq>':
/usr/include/c++/7/ext/aligned_buffer.h:85:34:   required from 'struct __gnu_cxx::__aligned_buffer<std::pair<const std::__cxx11::basic_string<char>, monitor::SoftIrq> >'
/usr/include/c++/7/bits/hashtable_policy.h:248:43:   required from 'struct std::__detail::_Hash_node_value_base<std::pair<const std::__cxx11::basic_string<char>, monitor::SoftIrq> >'
/usr/include/c++/7/bits/hashtable_policy.h:279:12:   required from 'struct std::__detail::_Hash_node<std::pair<const std::__cxx11::basic_string<char>, monitor::SoftIrq>, true>'
/usr/include/c++/7/bits/hashtable_policy.h:2007:60:   required from 'struct std::__detail::_Hashtable_alloc<std::allocator<std::__detail::_Hash_node<std::pair<const std::__cxx11::basic_string<char>, monitor::SoftIrq>, true> > >'
/usr/include/c++/7/bits/hashtable.h:173:11:   required from 'class std::_Hashtable<std::__cxx11::basic_string<char>, std::pair<const std::__cxx11::basic_string<char>, monitor::SoftIrq>, std::allocator<std::pair<const std::__cxx11::basic_string<char>, monitor::SoftIrq> >, std::__detail::_Select1st, std::equal_to<std::__cxx11::basic_string<char> >, std::hash<std::__cxx11::basic_string<char> >, std::__detail::_Mod_range_hashing, std::__detail::_Default_ranged_hash, std::__detail::_Prime_rehash_policy, std::__detail::_Hashtable_traits<true, false, true> >'
/usr/include/c++/7/bits/unordered_map.h:104:18:   required from 'class std::unordered_map<std::__cxx11::basic_string<char>, monitor::SoftIrq>'
/work/test_monitor/include/monitor/cpu_softirq_monitor.h:36:51:   required from here
/usr/include/c++/7/bits/stl_pair.h:215:11: error: 'std::pair<_T1, _T2>::second' has incomplete type
       _T2 second;                /// @c second is a copy of the second object
           ^~~~~~
In file included from /work/test_monitor/main.cpp:4:0:
/work/test_monitor/include/monitor/cpu_softirq_monitor.h:36:42: note: forward declaration of 'struct monitor::SoftIrq'
   std::unordered_map<std::string, struct SoftIrq> cpu_soft_irq_map_;
                                          ^~~~~~~
CMakeFiles/hi.dir/build.make:62: recipe for target 'CMakeFiles/hi.dir/main.cpp.o' failed
make[2]: *** [CMakeFiles/hi.dir/main.cpp.o] Error 1
CMakeFiles/Makefile2:104: recipe for target 'CMakeFiles/hi.dir/all' failed
make[1]: *** [CMakeFiles/hi.dir/all] Error 2
Makefile:83: recipe for target 'all' failed
make: *** [all] Error 2

```



### 3.

```sh
/work/test_monitor/main.cpp:19:47: error: 'begin' was not declared in this scope
   for (const auto& st : monitor_info.mem_info())
```

- 调用成员错误，成员里没有这个方法

  ```cpp
  // 这里mem_info不是repeated也就是容器类型，因此无法用foreach遍历
  for (const auto& st : monitor_info.mem_info()) {
      std::cout << st.used_percent() << std::endl;
    }
  ```

- 修正：

  ```cpp
    auto st = monitor_info.mem_info();
    std::cout << st.used_percent() << std::endl;
  ```

### 4. CMake Error 

```cmake
root@yiga-virtual-machine:/work/test_monitor/build# make -j4
-- Boost version: 1.65.1
-- Found the following Boost libraries:
--   system
--   chrono
-- Configuring done
CMake Error at CMakeLists.txt:10 (add_library):
  Cannot find source file:

    /work/test_monitor/src/monitor/cpu_soft_irq_monitor.cpp

  Tried extensions .c .C .c++ .cc .cpp .cxx .m .M .mm .h .hh .h++ .hm .hpp
  .hxx .in .txx


CMake Error: Cannot determine link language for target "libraries".
CMake Error: CMake can not determine linker language for target: libraries
-- Generating done
-- Build files have been written to: /work/test_monitor/build
Makefile:520: recipe for target 'cmake_check_build_system' failed
make: *** [cmake_check_build_system] Error 1
```

- 按照提示找到相关bug代码即可，CMake Error: Cannot determine link language for target "libraries"，这里是说library添加的路径有误，可能有错字

  ```protobuf
  add_library(libraries
  ${CMAKE_SOURCE_DIR}/src/monitor/cpu_soft_irq_monitor.cpp
   // 改为！
   ${CMAKE_SOURCE_DIR}/src/monitor/cpu_softirq_monitor.cpp
   }
  ```

### 5

```cmake
In file included from /work/test_monitor/src/monitor/cpu_softirq_monitor.cpp:2:0:
/work/test_monitor/include/monitor/cpu_softirq_monitor.h:14:47: error: expected class-name before '{' token
 class CpuSoftIrqMonitor : public MonitorInter {
```

`public MonitorInter`父类没有问题，可能是忘了引入父类头文件

```
#include "monitor/monitor_inter.h"
```

### 6

```
isplay_monitor  docker  proto  test_monitor
root@yiga-virtual-machine:/work# cd display_monitor
root@yiga-virtual-machine:/work/display_monitor# mkdir build
root@yiga-virtual-machine:/work/display_monitor# cd build
root@yiga-virtual-machine:/work/display_monitor/build# cmake ..
-- The C compiler identification is GNU 7.5.0
-- The CXX compiler identification is GNU 7.5.0
-- Check for working C compiler: /usr/bin/cc
-- Check for working C compiler: /usr/bin/cc -- works
-- Detecting C compiler ABI info
-- Detecting C compiler ABI info - done
-- Detecting C compile features
-- Detecting C compile features - done
-- Check for working CXX compiler: /usr/bin/c++
-- Check for working CXX compiler: /usr/bin/c++ -- works
-- Detecting CXX compiler ABI info
-- Detecting CXX compiler ABI info - done
-- Detecting CXX compile features
-- Detecting CXX compile features - done
CMake Error at CMakeLists.txt:9 (target_link_libraries):
  Cannot specify link libraries for target "Qt5::Core" which is not built by
  this project.


-- Configuring incomplete, errors occurred!

```

忘了在target_xx_xx处指定target

```cmake
target_link_libraries(
    Qt5::Core
    Qt5::Widgets
)
#更正
add_executable(display main.cpp)
target_link_libraries(display
    Qt5::Core
    Qt5::Widgets
)
```



# 5. Qt

# 6. grpc





# 7. Bug

1. display变量多写一个:，导致xhost找不到

   ```sh
   if [ -z ${DISPLAY} ];then
       display=":1"
   else
       display=":${DISPLAY}"
   fi
   # 修正
   if [ -z ${DISPLAY} ];then
       display=":1"
   else
       display="${DISPLAY}"
   fi
   ```

2. 使用qt显示界面要启动**`xhost`**

   ```sh
   xhost +local:root 1>/dev/null 2>&1
   docker exec \
       -u root \
       -it my_monitor \
       /bin/bash
   xhost -local:root 1>/dev/null 2>&1
   ```

3. 把源文件、头文件和main文件分开文件夹导致链接出错

   ```cmake
   set(SOURCES
       main.cpp
       ${CMAKE_SOURCE_DIR}/src/monitor_inter.cpp
       ${CMAKE_SOURCE_DIR}/src/mem_model.cpp
   
       ${CMAKE_SOURCE_DIR}/../rpc_manager/client/rpc_client.cpp
       ${CMAKE_SOURCE_DIR}/../proto/build/monitor_info.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/cpu_load.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/cpu_stat.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/cpu_softirq.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/mem_info.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/net_info.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/monitor_info.grpc.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/cpu_load.grpc.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/cpu_stat.grpc.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/cpu_softirq.grpc.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/mem_info.grpc.pb.cc
       ${CMAKE_SOURCE_DIR}/../proto/build/net_info.grpc.pb.cc
   )
   # 修正
   # 不要分开放
   set(SOURCES
       main.cpp
       ${CMAKE_SOURCE_DIR}/monitor_inter.cpp
       ${CMAKE_SOURCE_DIR}/mem_model.cpp
   ...
   )
   ```

4. docker_into.sh文件写错容器名称误启动成其他服务器

   尴尬的是linux_monitor容器刚好也启动了，让人误解是**挂载错目录**了...

   ```sh
   xhost +local:root 1>/dev/null 2>&1
   docker exec \
       -u root \
       -it linux_monitor \
       /bin/bash
   xhost -local:root 1>/dev/null 2>&1
   # 更正
   xhost +local:root 1>/dev/null 2>&1
   docker exec \
       -u root \
       -it my_monitor \
       /bin/bash
   xhost -local:root 1>/dev/null 2>&1
   ```

5. 继承了qt的`QAbstractTableModel`的`model`文件，如果不初始化headerData则会直接闪退

   ```cpp
   CpuStatModel(QVariant *parent = nullptr){}
   QStringList header_;
   ```

   - 构造函数实现后，没有初始数据header导致调用headerData闪退
   - 但是data可以为空

   ```cpp
   // 修正
   CpuStatModel::CpuStatModel(QVariant *parent) {
     header_ << tr("cpu name");
     header_ << tr("cpu percent");
     header_ << tr("user percent");
     header_ << tr("system percent");
     ;
   }
   QVariant CpuStatModel::headerData(int section, Qt::Orientation orientation,
                                     int role) const {
     if (role == Qt::DisplayRole && orientation == Qt::Horizontal) {
       return header_[section];
     }
   
     return MonitorInterModel::headerData(section, orientation, role);
   }
   ```

6. `was not decleared in this scopt`

   源文件实现函数忘了写作用域

   ```sh
   work/display_monitor/monitor_widget.cpp: In function 'QWidget* monitor::InitMemMonitorWidget()':
   /work/display_monitor/monitor_widget.cpp:67:3: error: 'mem_model_' was not declared in this scope
      mem_model_ = new MemModel;
      ^~~~~~~~~~
   /work/display_monitor/monitor_widget.cpp:67:3: note: suggested alternative: 'MemModel'
      mem_model_ = new MemModel;
      ^~~~~~~~~~
      MemModel
   
   ```

   ```cpp
   QWidget* InitMemMonitorWidget() {...}
   // 修正
   QWidget* MonitorWidget::InitMemMonitorWidget() {...}
   ```

7. 头文件函数定义了默认形参值，源文件实现时不能显示出现

   ```sh
   error: default argument given for parameter 2 of 'QVariant monitor::NetModel::data(const QModelIndex&, int) const' [-fpermissive]
                            int role = Qt::DisplayRole) const {
                                                        ^~~~~
   In file included from /work/display_monitor/net_model.cpp:1:0:
   /work/display_monitor/net_model.h:20:12: note: previous specification in 'virtual QVariant monitor::NetModel::data(const QModelIndex&, int) const' here
      QVariant data(const QModelIndex &index,
               ^~~~
   ```

   ```cpp
   // .头文件
   QVariant data(const QModelIndex &index,
                   int role = Qt::DisplayRole) const override;
   // .cpp文件
   QVariant NetModel::data(const QModelIndex &index, 
                           int role = Qt::DisplayRole) const {
     return monitor_data_[index.row()][index.column()];
   };
   // 修正
   // .cpp文件
   QVariant NetModel::data(const QModelIndex &index, 
                           int role) const {
     return monitor_data_[index.row()][index.column()];
   };
   ```

8. QT定义了slots等响应事件必须在每个头文件都加Q_OBJECT否则会直接闪退

   ```cpp
   class CpuSoftIrqModel : public MonitorInterModel {
     Q_OBJECT
      signals:
     void dataChanged(const QModelIndex &topLeft, const QModelIndex &bottomRight,
                      const QVector<int> &roles);
     }
   ```

9. 对QTableView设置Model后闪退

   ```cpp
     cpu_soft_irq_model = new CpuSoftIrqModel;
     cpu_soft_irq_monitor_view_ = new QTableView();
     cpu_soft_irq_monitor_view_->setModel(cpu_soft_irq_model);
   ```

   需检查model初始化情况是否有误，特别是headerData，多一个少一个都会出错

   ```cpp
   CpuSoftIrqModel::CpuSoftIrqModel(QObject *parent) : MonitorInterModel(parent) {
     header_ << tr("cpu_name");
     header_ << tr("hi");
     header_ << tr("net_tx");
     header_ << tr("net_rx");
     header_ << tr("block");
     header_ << tr("irq_poll");
     header_ << tr("tasklet");
     header_ << tr("sched");
     header_ << tr("hrtimer");
     header_ << tr("rcu");
   }
   ```

   ![image-20240823142723487](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240823142723487.png)

发现少了一个`timer`，补上使得header数目正确

```
CpuSoftIrqModel::CpuSoftIrqModel(QObject *parent) : MonitorInterModel(parent) {
  header_ << tr("cpu_name");
  header_ << tr("hi");
  header_ << tr("timer);
  ...
}
```

10. Qt data没有显示/更新

    ```cpp
    void CpuSoftIrqModel::UpdateMonitorInfo(
        const monitor::proto::MonitorInfo &monitor_info) {
      // QT更新model数据一定要beginResetModel()和结尾endResetModel()！！
      beginResetModel();
        。。。
      endResetModel();
    }
    ```

    

