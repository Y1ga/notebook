# IP

## 基础认识

### IP地址分类

![IP 地址分类](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/7.jpg)

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/8.jpg)

最大主机数计算= 2^位 - 2，有2个IP是特殊的分布是主机号全1和全0地址

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/10.jpg)

- 全1：指定某个网络下的所有主机，用于广播
- 全0：指定某个网络本身

#### 广播

- **局域网内的广播叫本地广播**，例如广播地址为192.168.48.128/24的情况下**广播地址是192.168.0.255**,因为这个广播地址包会被路由器屏蔽，因此不会到达192.168.48.0/24以外的其他链路上
- **在不同网络之间的广播叫直接广播**，例如网络地址为192.168.0.0/24的主机**向192.168.1.255/24（广播地址）的目标地址**发送IP包，收到这个包的路由器，将数据**转发给192.168.1.0/24网络本身**，从而使得**所有192.168.1.1~192.168.1.254**的主机都接收到这个包（**直接广播有安全问题，多数情况路由器上设置为不转发**）

![本地广播与直接广播](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/11.jpg)

#### D、E类地址

D类地址和E类地址都**没有主机号**，不可以用于主机IP，**D类常用于多播**，E类是预留分类暂时无用

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/12.jpg)

#### 多播

多播用于**将包发送给特定组内的所有主机**

![单播、广播、多播通信](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/13.jpg)

D类地址：前四位1110是多播地址，剩下28位是多播的组编号，224.0.0.0~239.255.255.255都是多播可用范围

- 224.0.0.0~224.0.0.255：预留的组播地址，只用于**局域网，路由器不转发**
- 224.0.1.0~238.255.255.255：用户可用的组播地址，可以用于**Internet**
- 239.0.0.0~239.255.255.255：**本地管理组播地址，供内部网内部使用**，特定的本地范围生效

#### IP分类优点

简单明了、选路简单

![IP 分类判断](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/14.jpg)****

#### IP分类缺点

1. 同一网络下没有地址层次，众生平等的主机号
2. A、B、C类主机号分配不平衡，254太少了、65534又太多了

### 无分类地址CIDR

- 舍弃掉IP分类，使用`a.b.c.d/x`，x范围是0~32表示网络号
- 也可以用子网掩码，顾名思义掩盖掉主机号，IP号与子网掩码作位与就得到网络号，与`/x`是一个意思



![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/15.jpg)

#### 为什么要分离网络号和主机号

两台计算机通讯，首先要判断**是否处于同一个广播域**，若网络地址相同直接由路由器转发到目标主机就行了

![IP地址的网络号](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/17.jpg)

#### 子网划分

子网掩码除了划分网络号和主机号，还可以划分**子网网络和子网主机号**

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/18.jpg)



![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/19.jpg)

- IP:192.168.1.0 

- 子网掩码：255.255.255.192，已知192.168.1是C类地址，可用主机为8位，**192说明子网掩码偷了2位划给了子网网号，这样就能起到很好的分层效果**，把256个再分成4个64

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/20.jpg)

划分后的4个子网：

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/21.jpg)

### 共有IP和私有IP

A、B、C类中会划分共有和私有IP，不同共有IP下的私有IP可以重复

![img](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/22.jpg)

#### 公有IP由谁管理

![image-20240829105412268](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240829105412268.png)

### IP地址与路由控制

IP地址的**网络地址由路由进行控制**，路由控制表记录着网络地址与下一步应该发送路由器的地址，主机和路由器上都有各自的路由表

![IP 地址与路由控制](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/25.jpg)

1. 源地址：`10.1.1.30/24`，目的地址：`10.1.2.10/24`，主机A路由表没找到与`10.1.2.10`相同的网络地址便转发到默认路由（路由1）
2. 路由1根据路由表进行匹配，发现匹配到了就发给了`10.1.0.2`路由2
3. 路由2匹配路由表发现`10.1.2.0/24`就在自家，便从接口发送出去经过交换机把IP数据包转发到了目标主机

#### 环回地址

**环回地址是指同一台计算机上的程序之间网络通信所使用的默认地址**，一般默认为**`127.0.0.1`作为环回地址，localhost作为主机名，使用这个地址数据包不会流向网络**

### IP分片与重组

每种数据链路层的最大传输单元`MTU`都不一要，Internet是1500Bytes，FDDI是4352

IP数据包>MTU时就会分片，重组发生在应用层

![分片与重组](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/26.jpg)

TCP传输中引入`MSS`由TCP分片，且MSS<MTU；**对UDP尽量不要传>1个MTU的数据报文**

### IPv6基本认识

- IPv4：32位，2^32 ≈ 42亿，2011年已经分完
- IPv6：128位，16个字节 = 32个16进制，每16位为一组每组用`:`分隔开；出现连续的0用`：：`隔开，一个IP地址只允许出现两个连续的冒号

![IPv6 地址表示方法](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/27.jpg)

![Pv6 地址缺省表示方](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/28.jpg)

#### IPv6优点

- **可自动配置，即使没有DHCP服务器也可以实现自动分配IP地址**
- 包头包首使用**固定的40字节，去掉包头校验和简化首部结构**，**减轻路由器负荷大大提高传输性能**
- 有应用伪造IP地址的网络安全功能以及防线路窃听功能，大大提升安全性

#### IPv6地址结构

- 单播地址：**一对一**通信
- 组播地址：**一对多**通信
- 任播地址：通信最近的节点，由路由协议决定
- 没有广播地址

![IPv6地址结构](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/29.jpg)

#### IPv6单播地址类型

分为三类单播地址

1. **IPv6独有：同一链路单播通信，不经过路由器只经过交换机可以使用链路本地单播地址**
2. 内网内单播通信：可以使用**唯一本地地址**，相当于IPv4的私有IP
3. 互联网通信，可以使用**全局单播地址**，相当于IPv4的公有IP

![ IPv6 中的单播通信](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/30.jpg)

#### IPv6首部对比IPv4首部

![IPv4 首部与 IPv6 首部的差异](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/31.jpg)

- 取消首部校验和字段：因为**数据链路层和传输层都会校验**，因此直接取消IP校验
- 取消分片/重新组装相关字段：**不允许在中间路由器进行分片与重组浪费时间**，只能在应用层即主机大大提高路由器转发速度
- 取消选项字段：**使IPv6首部为固定长度的40字节**，**选项字段可能出现在IPv6中的下一个首部指出的位置**

## IP协议相关技术

### ARP

1. 发送方直到接收方的IP地址但不知道MAC地址，构建一个ARP请求报文，包括了发送方的IP地址和MAC地址和**接收方的IP地址**
2. 发送方将**ARP请求广播到本地网络**，目标IP匹配的会响应并构建**ARP应答报文**包括自己的IP地址和**MAC地址**，接收方将ARP应答报文发送给发送方
3. 发送方接收到后会接收方的**IP地址和MAC地址存储在ARP缓存中**

ARP:通过IP地址找MAC地址

![ARP 广播](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/34.jpg)

Linux中可以使用`arp-a`查看ARP缓存内容

#### RARP

**已知MAC地址找IP地址**，需要架一台RARP服务器，在服务器上注册设备的MAC地址和IP地址再将设备接入网络中

![RARP](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/35.jpg)

### DHCP

DHCP(Dynamic Host Configuration Protocol，动态主机配置协议)

DISCOVER -> OFFER -> REQUEST -> ACK

1. 客户端发起DHCP`发现报文（DHCP DISCOVER）`的IP数据报，由于客户端没有IP地址也不知道DHCP服务器的地址，因此用**UDP广播通信，使用广播目的地址是255.255.255.255（端口67），使用0.0.0.0（端口68）作为源IP地址**，DHCP客户端将此IP数据报传递给链路层，链路层将帧广播到所有的网络设备
2. DHCP服务器收到DHCP发现报文时，用DHCP`提供报文（DHCP OFFER）`的客户端做出响应，报文仍然**使用IP广播地址255.255.255.255，报文信息携带服务器可供租约的IP地址、子网掩码、默认网关、DNS服务器以及地址租用期**
3. 客户端**收到一个或多个服务器的DHCP提供报文后，从中选择一个服务器**，并向选择的服务器发送`DHCP请求报文（DHCP REQUEST）`进行响应，回显配置的参数
4. 服务端用`DHCP ACK`报文对`DHCP REQUEST`进行响应，应答所要求的参数

![DHCP 工作流程](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/36.jpg)

租约的DHCP IP地址快到期后，客户端向服务端发送DHCP请求报文：

- 服务器如果同意继续租用，则用`DHCP ACK`报文进行应答，客户端延长租期
- 服务器若不同意继续租用，则用`DHCP NACK`报文，客户端停止使用租约的地址

**DHCP交互中，全部都是用UDP广播通信**

- 如果使用的是广播，DHCP服务器与客户端不在同一个局域网时，路由器不会转发广播包那岂不是每个网络都要配一个DHCP服务器？

为了解决这个问题，**出现了DHCP中继代理**，可以对不同网段的IP地址分配也可以由一个DHCP服务器统一进行管理。

- DHCP中继代理通常部署在路由器或三层交换机上，它可以接收来自客户端的 DHCP 广播请求，并将其转换为**单播**数据包转发给其他网络中的 DHCP 服务器。
- 当 DHCP 服务器收到中继代理转发的请求后，会为客户端分配 IP 地址等网络参数，并将响应数据包发送回中继代理。中继代理再将响应转换为**广播或单播**数据包发送给客户端。

因此DHCP服务器即使不在同一个链路上也可以实现统一分配和管理地址

![ DHCP 中继代理](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/37.jpg)

### NAT

NAT（Network Address Translation，网络地址转换），**将私网IP转换为公网IP**，这样就能访问Internet了



![NAT](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/38.jpg)

但是并不能解决IPv4不够用的问题，本质是一比一转换

#### NAPT

NAPT（Network Address Port Translation，网络地址端口转换）将**IP地址+端口一起转换，这样就能白嫖端口号多出来的额外位数**，解决了IPv4不够用的问题

![NAPT](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/39.jpg)

两个私有IP端口都为1025的都转为相同的公网IP`120.229.175.121`，但是端口号不同

#### NAPT缺点

1. **外部无法主动与NAT内部服务器建立连接**，因为NAPT转换表没有转换记录
2. 转换表的生成与转换操作都产生**性能开销**
3. 通信过程中，过于依赖NAPT转换表，**NAT路由器重启了所有的TCP连接都将被重置**

#### 解决NAPT问题

1. 使用IPv6
2. NAT穿透技术：能让客户端主动从NAT设备获取公有IP地址，然后自己建立端口映射条目，用这个条目与外界通信，就不需要NAT设备进行转换

### ICMP

ICMP(Internet Control Message Protocol，互联网控制报文协议)，用于**确认IP包是否成功送达目标地址、报告发送过程中IP包被废弃的原因和改善网络设置**

![ICMP 目标不可达消息](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/40.jpg)

主机A->主机B发送的数据包达不到，路由器2就会向主机A发送一个ICMP目标不可达数据包说明发包失败。**ICMP使用IP进行发包**，因此会经过路由1转发给主机A，收到ICMP包的主机分解ICMP的首部和数据域

#### ICMP包头

ICMP报文封装在IP包中，工作在网络层，是IP的助手

![ICMP 报文](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/5.jpg)



#### ICMP类型

- **查询报文**类型：用于诊断的查询消息
- **差错报文**类型：通知出错原因的错误信息

![常见的 ICMP 类型](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/41.jpg)

#### 查询报文类型

会送消息用于通信的主机或路由器之间判断所发送的数据包是否已成功到达对端

 ![ICMP 回送消息](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/7.jpg)

- 回送请求（ICMP Echo **Request** Message，类型`8`）：**向对端主机发送**回送请求消息
- 回送应答（ICMP Echo **Reply** Message，类型`0`）：**接收对端主机**发送的回送应答消息

![ICMP 回送请求和回送应答报文](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/8.jpg)



比原生的ICMP多了两个字段

- 标识符：区分哪个应用程序发送的ICMP包，如用**进程`PID`作标识符**
- 序号：从`0`开始，每发送一次新的回送请求就加`1`，用来确认网络包是否有丢失
- 选项数据：`ping`存放发送请求的时间值，来计算往返时间说明路程长短

#### 差错报文类型

1. 目标不可达消息：类型`3`
2. 原点抑制消息：类型`4`
3. 重定向消息：类型`5`
4. 超时消息：类型`11`

##### 1 目标不可达消息

- 端口不可达：找到了网络+主机但对端主机没有进程监听8080端口，则ICMP协议以端口不可达告知主机
- 需分片但设置不了分片：发送端主机发送IP数据报时将IP首部分片禁止标志位置1，**途中路由器遇到超过MTU大小的数据包不会进行分片直接抛弃**

![目标不可达类型的常见代码号](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/9.jpg)

##### 2 原点抑制消息（ICMP Source Quench Message）

缓和低速广域线路的网络拥堵情况，**路由器向低速线路发送数据时其发送队列缓存变为0而无法发出去**时，向IP包源地址发送ICMP原点抑制消息，收到这个消息的主机借此了解某个线路的某一处发生拥堵的情况，**增大IP包传输间隔减少网络拥堵**。

**但会引起不公平网络通信一般不使用**

##### 3 重定向消息（ICMP Redirect Message）

若路由器发送端主机使用了**不是最优的路径**发送数据，就发送一个ICMP重定向消息，这个消息包含了**最合适的路由信息和源数据**，下次就不用绕远路了

##### 4 超时消息（ICMP Time Exceeded Message）

IP包有一个`TTL`(Time To Live，生命周期)，值每经过一个路由器就减1，**减到0包就丢弃**，此时路由器发送一个ICMP超时消息给发送端主机，通知包已丢弃。

设置TTL是防止路由器遇到问题**发生循环状况时，避免IP包被无休止地被转发**

![ICMP 时间超过消息](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/11.jpg)

### IGMP

IGMP(Internet Group Management Protocol，网络组管理协议)，工作在**主机（组播成员）和最后一条路由之间，用于确认主机是在组播的同一组里**

**组播没有主机号和端口只有28位IP地址，属于D类地址，可以大大减少数据流量因为是一对多只需发一次**

- 例如，在一个视频会议应用中，多个主机可能希望接收来自同一个视频源的多播流量。这些主机可以通过 IGMP 向本地路由器表明它们希望加入特定的多播组，以便接收视频会议的数据流。

![组播模型](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/42.jpg)

- IGMP 有三个主要版本：IGMPv1、IGMPv2 和 IGMPv3。每个版本都在功能和效率上有所改进。

- IGMPv1 是最基本的版本，支持**主机加入和离开多播组**的功能。IGMPv2 增加了一些功能，如**离开组的明确通知和查询机制的改进**。IGMPv3 则进一步**增强了对多播源的选择和过滤功能，允许主机指定它希望接收来自哪些特定源的多播流量**。

#### 常规查询与响应工作机制

![ IGMP 常规查询与响应工作机制](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/43.jpg)

1. 路由器周期发送目的地址224.0.0.1（表示同一网段内的所有主机和路由器）IGMP常规查询报文
2. 主机1和主机3收到查询，启动报告延迟计时器，计时器时间随机0-10s，超时后主机发送IGMP成员关系报告报文（源IP地址位自己主机IP地址，目的地址为轮播地址）。定时器超时之前，收到同一个组内的其他主机发送的成员关系报告报文，则自己就不发送了
3. 路由器收到主机的成员关系报文后，**在IGMP路由表加入该组播组**，后续网络一旦该组播地址的数据到达路由器，就把数据包转发出去

#### 离开组播组工作机制

![ IGMPv2 离开组播组工作机制 情况1](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/44.jpg)

1. 主机1要离开组播组224.1.1.1，发送IGMPv2离组报文，报文目的地址是224.0.0.2（表示发向网段内所有路由器）
2. 路由器收到报文后，以1s为间隔连续发送IGMP从特定组查询报（共计发送2个），**确认该网络是否还有224.1.1.1组的其他成员**
3. 主机3仍是组224.1.1.1的成员，因此立即响应这个特定组查询，路由器知道该网络还有该组播组成员，继续向网络转发224.1.1.1的组播数据包

- 没有该组播组的情况：

![ IGMPv2 离开组播组工作机制 情况2](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/IP/45.jpg)

1. 主机1离开并发送IGMP离组报文
2. 路由器收到后，以1s为间隔连续发送IGMP从特定组查询报（共计发送2个），发向没有其他成员了
3. 一定时间后，路由器直到没有组播成员了，停止转发组播地址的数据包

### ping--查询报文类型的使用

ping是**应用层**的命令，底层利用了**ICMP**的`echo request（类型8`）和`echo response（类型0）`

**虽然ICMP和IP都是网络层协议，但ICMP利用了IP协议进行消息的传输**

![主机 A ping 主机 B](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/12.jpg)

源主机构建一个(ICMP Echo Request Message)数据包，包内包含多个字段最重要的有2个

1. 类型：对于request消息字段为`8`
2. 序号：**区分**连续ping时发出的多个数据包

![主机 A 的 ICMP 回送请求报文](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/13.jpg)

然后与IP包头一起封装交给IP层

![主机 A 的 IP 层数据包](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/14.jpg)

接下来需要加入`MAC`头，在本地ARP映射表找出IP为`192.168.1.2`对应的MAC地址；若无就发送ARP协议查询MAC地址，获得MAC地址后由数据链路层**构建数据帧并附加一些控制信息**，依据Internet介质访问规则，将他们传送出去

![主机 A 的 MAC 层数据包](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/15.jpg)

主机B收到数据帧后，先检查目的MAC地址看看与本机是否相符，然后检查数据帧将IP数据包从帧提取出来，交给本机IP层，IP层检查后将有用信息提取给ICMP协议。

主机B会构建一个ICMP回送响应消息（ICMP Echo Response Message）数据包，回送响应数据包类型字段为`0`，序号为接收的请求数据包的序号，然后发给A

规定时间内，源主机没有收到ICMP应答包，说明目标主机不可达；若收到，说明目标主机可达。此时源主机检查，**用当前时刻减去该数据包最初从源主机发出的时刻，就是ICMP数据包的时间延迟**

![主机 A ping 主机 B 期间发送的事情](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/17.png)

这只是局域网内的情况，若跨网段则会涉及网关的转发、路由器的转发等等

但对于ICMP的头讲，没影响，**影响的只是根据目标IP地址，选择路由的下一跳，还有每经过一个路由器到达一个新的局域网需要换MAC头里面的MAC地址**

### traceroute-差错报文类型的使用

#### 作用1：追踪去往目的地时沿途经过的路由器

**故意设置特殊的TTL，追踪去往目的地时沿途经过的路由器**

例如，**将TTL设置为1**，遇到第一个路由就牺牲了返回ICMP差错报文网络包，类型是时间超时，TTL设置为2，第一个路由器过了，遇到第二个路由器也牺牲了同时返回ICMP差错报文数据包如此反复直到到达目的主机，**这样就能拿到所有的路由器IP**

但也有路由器不会返回这个ICMP，对有的公网地址是看不到中间经过的路由的

- 发送方如何知道发出的UDP包到达目的主机？

traceroute发送UDP包时，目的端口会填一个不可能的端口号：`33434`，对于下个探针都会增加一个，目的主机收到UDP包后返回ICMP差错报文消息，类型是**端口不可达**（类型3中的3）

#### 作用2：故意设置不分片，确认路径的MTU

在非Internet的MTU值不知道，因此需要试探得到

![MTU 路径发现（UDP的情况下）](https://cdn.xiaolincoding.com/gh/xiaolincoder/ImageHost/%E8%AE%A1%E7%AE%97%E6%9C%BA%E7%BD%91%E7%BB%9C/ping/18.jpg)

1. 发送IP数据报时，将IP包首部分片禁止标志位置1
2. 通过ICMP的不可达消息将数据链路上MTU值一起给发送主机，不可达消息类型为**需要进行分片但设置了不分片位**（类型3中的4）
3. 发送端每次收到ICMP差错报文就减少包的大小，定位一个合适的MTU值

### 断网了，还能ping通127.0.0.1吗

yes!

![image-20240830114652424](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240830114652424.png)

#### 什么是127.0.0.1

**127开头的都属于回环地址**，`127.0.0.1`是众多回环地址的一个

![图片](https://cdn.xiaolincoding.com//mysql/other/fa904fbcf66cc7abf510a8dc16f867fa.png)

**ipv6的回环地址是`::1`**，为什么只有2个`:`因为**ipv6只允许出现一次两个连续的冒号**

#### TCP发数据和PING的区别

![image-20240830115801785](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240830115801785.png)

Linux里万物皆文件，要**发消息的目的地也是个文件**

##### TCP

1. TCP传输中创建`socket`的方式是`socket(AF_INET, SOCK_STREAM, 0)`，**`AF_INET`表示使用IPv4的`host:port`的方式解析待会输入的网络地址**，**`SOCK_STREAM`是指使用面向字节流的TCP协议，工作在传输层**
2. 创建好socket后便可以将传输的数据写到这个文件里，调**用socket的`sendto`接口过程中进程会从用户态进入到内核态**，最后调用到`sock_sendmsg`方法
3. 进入传输层后，带上TCP头，网络层带上IP头，链路层带上MAC头等系列操作后，进入网卡的发送队列ring buffer，顺着网卡发出去

##### ping

1. 与TCP大致相似，差异的地方在于创建`socket`的时候使用的是`socket(AF_INET, SOCK_RAW, IPPROTO_ICMP)`，**`SOCK_RAW`是原始套接字，工作在网络层**，因此**很适合构建ICMP(网络层协议)**。
2. ping在内核态最后也是调用到`sock_sendmsg`方法，进入到网络层后加上ICMP和IP头再加上MAC头顺着网卡发出，本质与TCP没有太大差别

#### 为什么断网了还是能ping 127.0.0.1

因为发向目标IP是回环地址时，在数据链路层网卡就会选择**本地网卡（”假“网卡）**，不像**真网卡有个`ring buffer`，假网卡会把数据推到一个叫`input_pkt_queue`的链表中，这个链表所有网卡共享，上面挂着发给本机的各种消息。消息发送到这个链表后，会再触发一个软中断**

![图片](https://cdn.xiaolincoding.com//mysql/other/c1019a8be584b27c4fc8b8abda9d3cf1.png)

专门处理软中断的工具人`ksoftirqd（内核线程）`，在收到软中断后立马去链表里把消息取出，然后顺着数据链路层、网络层等层层往上传递最后给到应用程序

**ping回环地址和通过TCP等各种协议发送数据到回环地址都是走这条路径**，整条路经从发到收，都没有经过真网卡。”回环“：理解为消息发出到这个地址上不会出网络，在本机打个转又回来了

#### ping 回环地址和本机地址区别

没有区别，都是走的lo0本地回环地址，不会进入物理网络接口

![image-20240830121903441](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240830121903441.png)

#### 127.0.0.1 和 localhost 和 0.0.0.0区别

1. localhost不是地址，而是**域名**跟`google.com`是一种类型的，只不过会解析为`127.0.0.1`，可以在`/etc/hosts`下修改
2. `0.0.0.0`在ipv4中是无效的目标地址，启动服务器时会listen一个IP和端口，等待客户端连接。**若listen的是本机的`0.0.0.0`，表示本机的所有ipv4地址**