# socket

## 数据结构

### sockaddr

`sockaddr` 是 `socket` 的通用地址結構

```c
struct sockaddr
  {
    __SOCKADDR_COMMON (sa_);	/* Common data: address family and length. */
    char sa_data[14];		/* Address data. */
  };
// 上面的結構把巨集展開後，等價於下方的資料結構
struct sockaddr {
    unsigned short int sa_family; // 2 bytes
    char sa_data[14];             // 14 bytes
};
```

### sockaddr_in

後來的更新中，為了讓龐大的程式碼可讀性上升，新增了 `sockaddr_in` 的結構用來存取網路相關的應用， `in` 指的是 `internet`，`sockaddr_in` 專門用來存 `IPv4` 的相關地址。

```c
struct sockaddr_in
  {
    __SOCKADDR_COMMON (sin_);
    in_port_t sin_port;			/* Port number.  */
    struct in_addr sin_addr;		/* Internet address.  */

    /* Pad to size of `struct sockaddr'.  */
    unsigned char sin_zero[sizeof (struct sockaddr)
			   - __SOCKADDR_COMMON_SIZE
			   - sizeof (in_port_t)
			   - sizeof (struct in_addr)];
  };
struct sockaddr_in {
    // sa_family_t sin_family
    unsigned short int sin_family; // 2 bytes
    unsigned short int sin_port;   // 2 bytes
    struct in_addr sin_addr;       // 4 bytes
    unsigned char sin_zero[8];     // 填充，讓 sockaddr_in 的 size 跟 sockaddr 相同
};
```

這邊觀看原始碼會覺得奇怪，為什麼還需要使用 `sin_zero` 來做填充的動作。

原因是很多 `socket` 的 `api`，參數都需要填入 **`sockaddr`**，`sockaddr_in` 則是後來加入的 `struct`。 今天如果我們 `address` 的資料是用 **`sockaddr_in`** 來儲存，並且想調用相關的函式時，我們就需要強制轉型。

### in_addr

```c
typedef uint32_t in_addr_t;
// in_addr_t本质就是uint32_t
struct in_addr
  {
    in_addr_t s_addr;
  };
```

### in_port

```c
typedef uint16_t in_port_t;
```

in_port_t本质是uint16_t

### 位制转换相关函数

一般我們在表示 `ip` 位置時都會寫成人類比較容易讀的形式，像是`125.102.25.62`

以 `ipv4` 來說，`address` 是由4個 `byte`，32個 `bit`所組成，在實務上我們常常需要做**字串與實際數值**(`uint32_t`)的轉換，`linux` 函式庫提供了一系列輔助位置轉換的 `function`。

一般來說，`address` 的實際數值都會用 `in_addr` 或者 `in_addr_t` 來表示 其本質就是 **`uint32_t`**，用總共 32 個 `bits` 來表示一個 `IPv4` 的地址

```c
typedef uint32_t in_addr_t;
struct in_addr
  {
    in_addr_t s_addr;
  };
```

常用的有以下這五種

- 只能用在`IPv4`的處理
  - inet_addr
  - inet_aton
  - inet_ntoa
- 兼容Ipv4I與Pv6
  - inet_pton
  - inet_ntop

使用前必須先

```c
#include <arpa/inet.h>
```

#### inet_addr

```c
in_addr_t inet_addr(const char *cp)
```

**功能**: 將**字串轉換成數值**表示的 `ip address`

**回傳**: 假如輸入的地址合法，會回傳 `uint32_t` 的數值，若不合法則回傳 `INADDR_NONE`

#### inet_aton

```c
int inet_aton(const char *string, struct in_addr *addr)
```

**功能**: 將**字串轉換成數值**表示的 `ip address`

**回傳**: 轉換成功，會回傳一個非零的值，失敗則會回傳 `0`

#### inet_ntoa

```c
char *inet_ntoa(struct in_addr)
```

**功能**: 將 `in_addr` **轉換成字串形式**的 `ip address`

**回傳**: 如果沒有錯誤，會傳回成功轉換的字串，失敗時則會回傳 `NULL`

```c
#include <arpa/inet.h>
  const char *addr1 = "8.8.8.8";
  struct in_addr *addr;
  if (inet_aton(addr1, addr) == 0) {
    printf("inet_aton failed! \n");
    exit(0);
  }
  in_addr_t addr3 = inet_addr(addr1);
  printf("address %u, after inet_aton function, ip address = %u\n", addr3,
         addr->s_addr);
  char *addr2 = inet_ntoa(*addr);
  if (addr2 == NULL) {
    printf("inet_ntoa failed!\n");
    exit(0);
  }
  printf("after inet_ntoa function, address is %s\n", addr2);
```



#### inet_pton & inet_ntop

```c
const char *inet_pton(int domain, const void *restrict addr, char *restrict str, socklen_t size)
int inet_pton(int domain, const char *restrict str, void *restrict addr)
```

最後這兩個函式是為了因應 `IPv6` 而新增的，除了轉換 `IPv6` 之外，也可以兼容之前 `IPv4` 相關的轉換

要做 `IPv6` 相關的轉換，要把 `domain` 填入 `AF_INET6` 即可，後面需要搭配 `IPv6` 相關的 `struct`

```cpp
#include <stdio.h>
#include <arpa/inet.h>

int main()
{
    struct in_addr addr;
    if (inet_pton(AF_INET, "8.8.8.8", &addr.s_addr) == 1) {
        printf("Ip address: %u\n", addr.s_addr);
    }

    char ip_addr[20];
    if (inet_ntop(AF_INET, &addr.s_addr, ip_addr, sizeof(ip_addr))) {
        printf("After inet_ntop function, ip address: %s\n", ip_addr);
    }
}
```

### socket

```c
extern int socket (int __domain, int __type, int __protocol) __THROW;

int socket(int domain, int type, int protocol)
```

example

```c
// AF_INET:IPv4
// SOCK_DRAM:UDP
// 0:IP
//   socket返回文件描述符：成功；-1：失败
int server_fd;
server_fd = socket(AF_INET, SOCK_DGRAM, 0);
if (server_fd < 0){
    printf("craete socket failed\n");
    return -1;
}
```



### bind

```c
extern int bind (int __fd, __CONST_SOCKADDR_ARG __addr, socklen_t __len)
     __THROW;
// 因為 bind 可以用在不同種類的 socket，所以是用 sockaddr 宣告而不是sockaddr_in
int bind(int sockfd, struct sockaddr *addr, unsigned int addrlen)
```

#### *sockfd*

一開始呼叫 `socket()` 的回傳值

#### *addr*

`sockaddr` 來描述 `bind` 要綁定的 `address` 還有 `port`。

在先前的介紹有簡單提到，實際存放 `ip address` 的是 `sockaddr_in.sin_addr.s_addr`，如果今天不想綁定 `ip address`，而是單單想綁定某個 `port` 的時候，`s_addr` 就要設成 `INADDR_ANY`，通常會出現在你的主機有多個 `ip` 或者 `ip` 不是固定的情況。

[INADDR_ANY 參考](https://blog.csdn.net/qq_26399665/article/details/52932755)

#### *addrlen*

`sizeof(addr)`

#### *return*

如果綁定成功就會回傳 `0`，失敗回傳 `-1`

## UDP

![img](https://camo.githubusercontent.com/78276e347df03379a9c9120ba3d6819d3f685c6bdab996758a79bb91701da1f2/68747470733a2f2f692e696d6775722e636f6d2f737850757569632e706e67)

### sendto

```cpp
ssize_t sendto(int sockfd, const void *buf, size_t len, int flags,
               const struct sockaddr *dest_addr, socklen_t addrlen);
```

#### *sockfd*

`socket` 的文件描述符

#### *buf*

資料本體

#### *len*

資料長度

#### *flags*

一般填入 `0`，想知道詳細參數意義可以參考 [man page](https://linux.die.net/man/2/sendto)

#### *dest_addr*

目標位置相關資訊

#### *addrlen*

`sizeof(addr)`

#### *return value*

傳送成功時回傳具體傳送成功的 `byte` 數，傳送失敗時會回傳 `-1` 並且把錯誤訊息存進 [errno](https://man7.org/linux/man-pages/man3/errno.3.html)

### recvfrom

```c
ssize_t recvfrom(int sockfd, void *buf, size_t len, int flags,
                 struct sockaddr *src_addr, socklen_t *addrlen);
```

#### *sockfd*

`socket` 的文件描述符

#### *buf*

接收資料的 `buffer`

#### *len*

資料長度

#### *flags*

一般填入 `0`，想知道詳細參數意義可以參考 [man page](https://linux.die.net/man/2/recvfrom)

#### *src_addr*

資料來源地址，收到訊息之後我們可以一併收到來源地址，透過 `src_addr`，我們才能順利的把處理完的資料發回。

#### *addrlen*

`sizeof(addr)`

#### *return value*

接收成功時回傳具體接收成功的 `byte` 數，傳送失敗時會回傳 `-1` 並且把錯誤訊息存進 [errno](https://man7.org/linux/man-pages/man3/errno.3.html)



### demo

#### udp_server

```c
#include <arpa/inet.h>
#include <netinet/in.h>
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <sys/socket.h>
#include <unistd.h>

#include <cstdio>
#include <cstdlib>

#define SERVER_PORT 48763
#define SERVER_IP "127.0.0.1"

int main(int argc, char *argv[]) {
  // 建立 socket
  int server_fd = socket(AF_INET, SOCK_DGRAM, 0);
  if (server_fd < 0) {
    printf("socket create failed!\n");
  }
  // server 地址
  struct sockaddr_in server_addr = {AF_INET, htons(SERVER_PORT), INADDR_ANY};

  // 將建立的 socket 綁定到 serverAddr 指定的 port
  if (bind(server_fd, (const struct sockaddr *)&server_addr,
           sizeof(sockaddr_in)) < 0) {
    printf("socket bind failed!");
    exit(0);
  }
  // message buffer
  char buf[1024] = {0};
  struct sockaddr_in client_addr;
  // 此处必须使用unsigned int因为函数定义如此#define __U32_TYPE		unsigned
  // int
  unsigned int len = sizeof(client_addr);
  while (1) {
    // 當有人使用 UDP 協定送資料到 48763 port
    // 會觸發 recvfrom()，並且把來源資料寫入 clientAddr 當中
    if (recvfrom(server_fd, buf, sizeof(buf), 0,
                 (struct sockaddr *)&client_addr, &len) < 0) {
      break;
    }

    // 收到 exit 指令就關閉 server
    if (strcmp(buf, "exit") == 0) {
      printf("get exit order, closing the server...\n");
      break;
    }
    // 顯示資料來源，原本資料，以及修改後的資料
    printf("get message from[%s:%d]: \n", inet_ntoa(client_addr.sin_addr),
           ntohs(client_addr.sin_port));
    printf("%s \n", buf);
    // 根據 clientAddr 的資訊，回傳至 client 端
    // sendto不需要检查是否小于0，因为UDP只管生不管养
    sendto(server_fd, buf, sizeof(buf), 0,
           (const struct sockaddr *)&client_addr, sizeof(client_addr));
    // memset常常用于结构体清0
    // 清空 buffer
	//  例如：struct sockaddr_in client_addr{};或者struct sockaddr_in client_addr = {};也可以将结构体初始化为全零。
    memset(buf, 0, sizeof(buf));
  }
  if (close(server_fd) < 0) {
    perror("close socket failed!");
  }
}
```

#### udp_client

```c
#include <arpa/inet.h>
#include <netinet/in.h>
#include <sys/socket.h>
#include <unistd.h>

#include <cstdio>
#include <cstring>

#define SERVER_PORT 48763
#define SERVER_IP "127.0.0.1"

int main(int argc, char *argv[]) {
  // 建立 socket
  int client_fd = socket(AF_INET, SOCK_DGRAM, 0);
  // server 地址
  sockaddr_in server_addr = {AF_INET, htons(SERVER_PORT), inet_addr(SERVER_IP)};
  // message buffer
  char buf[1024] = {0};
  char recvbuf[1024] = {0};
  unsigned int len = sizeof(server_addr);

  while (1) {
    // 輸入資料到 buffer
    printf("please input your message: ");
    scanf("%s", buf);

    // 傳送到 server 端
    sendto(client_fd, buf, sizeof(buf), 0,
           (const struct sockaddr *)&server_addr, sizeof(server_addr));
    // 接收到 exit 指令就退出迴圈
    if (strcmp(buf, "exit") == 0) break;
    // 清空 message buffer
    memset(buf, 0, sizeof(buf));
    // 等待 server 回傳轉成大寫的資料
    if (recvfrom(client_fd, recvbuf, sizeof(recvbuf), 0,
                 (struct sockaddr *)&server_addr, &len) < 0) {
      printf("recvfrom data from %s:%d, failed!",
             inet_ntoa(server_addr.sin_addr), ntohs(server_addr.sin_port));
    }
    // 顯示 server 地址，以及收到的資料
    printf("get receive message from [%s:%d]: %s\n",
           inet_ntoa(server_addr.sin_addr), ntohs(server_addr.sin_port),
           recvbuf);
    // 清空 recv buffer
    memset(recvbuf, 0, sizeof(recvbuf));
  }
  // 關閉 socket，並檢查是否關閉成功
  if (close(client_fd) < 0) {
    perror("close socket failed!");
  }
}

```

## TCP

![img](https://camo.githubusercontent.com/0f307ac404cb70b04a4fbd3389e9ac2a0ab43f9cccff075677709c9e8a116062/68747470733a2f2f692e696d6775722e636f6d2f46444f494d6a392e706e67)

當 `client` 呼叫 `connect` 時才會開始發起 **three-way handshake**，當 `connect` 結束時，`client` 與 `server` 基本已經完成了整個流程。

那 `server` 端的 `accept` 具體只是從 `server socket` 維護的 `completed connection queue` 中取出一個已完成交握過程的 `socket`。

在 `kernel` 中每個 `socket` 都會維護兩個不同的 `queue`:

- 未完成連線佇列 (***incomplete connection queue***): FIFO with syn_rcvd state
- 已完成連線佇列 (***complete connection queue***): FIFO with established state

![img](https://camo.githubusercontent.com/2711e5c53d0b0f74d0e2b356405d84967cc48c724aba7f2dc802c606a1a4c293/68747470733a2f2f692e696d6775722e636f6d2f494b386c6178712e706e67)

- server端
  - `listen`: 初始化佇列，準備接受 `connect`
  - `accept`: 從 `complete connection queue` 中取出一個已連線的 `socket`
- client端
  - `connect`: 發起 `three-way handshake`，必須要等 `server` 端開始 `listen` 後才可以使用

### connect

```c
int connect(int sockfd, const struct sockaddr *addr,
            socklen_t addrlen);
```

#### *sockfd*

一開始呼叫 `socket()` 的回傳值

#### *addr*

想要建立連線的 `server` 資料

#### *addrlen*

`sizeof(addr)`

#### *return*

錯誤時回傳 `-1`，並且設定 `errno`

### accept

```c
int accept(int sockfd, struct sockaddr *restrict addr,
           socklen_t *restrict addrlen);
```

#### *sockfd*

`server` 端 `socket` 的檔案描述符

#### *addr*

建立 `TCP` 連線的 `Client` 端資料

#### *addrlen*

`sizeof(addr)`

#### *return*

返回一個新的 `sock_fd`，專門跟請求連結的 `client` 互動

### demo

![img](https://camo.githubusercontent.com/6cbe9b1995368792657106521e2dc618cac99118babc86255bf5b1b28ca21c6c/68747470733a2f2f692e696d6775722e636f6d2f543335433776732e706e67)

### tcp_server

```c
#define serverPort 48763

// message buffer
char buf[1024] = {0};

// 建立 socket
int socket_fd = socket(PF_INET , SOCK_STREAM , 0);
if (socket_fd < 0){
    printf("Fail to create a socket.");
}

// server 地址
struct sockaddr_in serverAddr = {
    .sin_family = AF_INET,
    .sin_addr.s_addr = INADDR_ANY,
    .sin_port = htons(serverPort)
};

// 將建立的 socket 綁定到 serverAddr 指定的 port
if (bind(socket_fd, (const struct sockaddr *)&serverAddr, sizeof(serverAddr)) < 0) {
    perror("Bind socket failed!");
    close(socket_fd);
    exit(0);

// 初始化，準備接受 connect
// backlog = 5，在 server accept 動作之前，最多允許五筆連線申請
// 回傳 -1 代表 listen 發生錯誤
if (listen(socket_fd, 5) == -1) {
    printf("socket %d listen failed!\n", socket_fd);
    close(socket_fd);
    exit(0);
}

printf("server [%s:%d] --- ready\n", 
        inet_ntoa(serverAddr.sin_addr), ntohs(serverAddr.sin_port));

while(1) {
    int reply_sockfd;
    struct sockaddr_in clientAddr;
    int client_len = sizeof(clientAddr);

    // 從 complete connection queue 中取出已連線的 socket
    // 之後用 reply_sockfd 與 client 溝通
    reply_sockfd = accept(socket_fd, (struct sockaddr *)&clientAddr, &client_len);
    printf("Accept connect request from [%s:%d]\n", 
            inet_ntoa(clientAddr.sin_addr), ntohs(clientAddr.sin_port));
    
    // 不斷接收 client 資料
    while (recv(reply_sockfd, buf, sizeof(buf), 0)) {
        // 收到 exit 指令就離開迴圈
        if (strcmp(buf, "exit") == 0) {
            memset(buf, 0, sizeof(buf));
            goto exit;
        }

        // 將收到的英文字母換成大寫
        char *conv = convert(buf);

        // 顯示資料來源，原本資料，以及修改後的資料
        printf("get message from [%s:%d]: ",
                inet_ntoa(clientAddr.sin_addr), ntohs(clientAddr.sin_port));
        printf("%s -> %s\n", buf, conv);

        // 傳回 client 端
        // 不需要填入 client 端的位置資訊，因為已經建立 TCP 連線
        if (send(reply_sockfd, conv, sizeof(conv), 0) < 0) {
            printf("send data to %s:%d, failed!\n", 
                    inet_ntoa(clientAddr.sin_addr), ntohs(clientAddr.sin_port));
            memset(buf, 0, sizeof(buf));
            free(conv);
            goto exit;
        }

        // 清空 message buffer
        memset(buf, 0, sizeof(buf));
        free(conv);
    }

    // 關閉 reply socket，並檢查是否關閉成功
    if (close(reply_sockfd) < 0) {
        perror("close socket failed!");
    }
}
```

### tcp_client

```c
#define serverPort 48763

 // message buffer
char buf[1024] = {0};
char recvbuf[1024] = {0};

// 建立 socket
int socket_fd = socket(PF_INET, SOCK_STREAM, 0);
if (socket_fd < 0) {
    printf("Create socket fail!\n");
    return -1;
}

// server 地址
struct sockaddr_in serverAddr = {
    .sin_family = AF_INET,
    .sin_addr.s_addr = inet_addr(serverIP),
    .sin_port = htons(serverPort)
};
int len = sizeof(serverAddr);

// 試圖連結 server，發起 tcp 連線
// 回傳 -1 代表 server 可能還沒有開始 listen
if (connect(socket_fd, (struct sockaddr *)&serverAddr, len) == -1) {
    printf("Connect server failed!\n");
    close(socket_fd);
    exit(0);
}

printf("Connect server [%s:%d] success\n",
            inet_ntoa(serverAddr.sin_addr), ntohs(serverAddr.sin_port));

while (1) {
    // 輸入資料到 buffer
    printf("Please input your message: ");
    scanf("%s", buf);

    // 傳送到 server 端
    if (send(socket_fd, buf, sizeof(buf), 0) < 0) {
        printf("send data to %s:%d, failed!\n", 
                inet_ntoa(serverAddr.sin_addr), ntohs(serverAddr.sin_port));
        memset(buf, 0, sizeof(buf));
        break;
    }

    // 接收到 exit 指令就退出迴圈
    if (strcmp(buf, "exit") == 0)
        break;

    // 清空 message buffer
    memset(buf, 0, sizeof(buf));

    // 等待 server 回傳轉成大寫的資料
    if (recv(socket_fd, recvbuf, sizeof(recvbuf), 0) < 0) {
        printf("recv data from %s:%d, failed!\n", 
                inet_ntoa(serverAddr.sin_addr), ntohs(serverAddr.sin_port));
        break;
    }

    // 顯示 server 地址，以及收到的資料
    printf("get receive message from [%s:%d]: %s\n", 
            inet_ntoa(serverAddr.sin_addr), ntohs(serverAddr.sin_port), recvbuf);
    memset(recvbuf, 0, sizeof(recvbuf));
}

// 關閉 socket，並檢查是否關閉成功
if (close(socket_fd) < 0) {
    perror("close socket failed!");
}
```

![image-20240903102908392](C:\Users\z1002\AppData\Roaming\Typora\typora-user-images\image-20240903102908392.png)

## 抓包

```sh
# lo表示本地127.0.0.1
tcpdump -i lo tcp and port 2233 -s0 -w /home/tcp_close.pcap
```

