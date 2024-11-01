# 服务器和客户端进行简单的 TCP 通信

### 实验介绍

在 TCP/IP 四层参考模型中，从上往下有四种层次：应用层、传输层、网络层、网络接口层。TCP 是传输层中重要的一项协议，也是我们常用的进程间通信方式，因此掌握 C++ 中的 TCP 套接字网络编程十分重要。

#### 知识点

- 计算机网络传输层基础知识
- TCP 套接字网络编程

### 传输层的基本概念

在 TCP/IP 四层参考模型中，从上往下有四种层次：应用层、传输层、网络层、网络接口层，应用层包括 HTTP、FTP、DNS 等协议，而传输层包括 TCP、UDP 两种协议，网络层则包含 IP、ARP 等协议，网络接口层较为底层，一般不是我们研究的对象。其中，传输层是我们在编程开发中较为重要的一层，需要对其中的两种协议尤其是 TCP 理解透彻。

![图片描述](https://doc.shiyanlou.com/courses/3573/1116908/9195a5673454d7fd8f457fb1e07c7701-0)

#### 传输层的作用

传输层的根本目的是在网络层提供的数据通信服务基础上，实现主机的进程间通信的可靠服务。

有以下两个要点：为位于两个主机内部的两个应用进程之间提供通信服务、提供可靠的通信服务。

#### 套接字

“套接字”表示一个 IP 地址与对应的一个端口号。例如，一个 IP 地址为 172.31.75.8 的客户端使用 8050 端口号，那么标识客户端的套接字为“172.31.75.8:8050”。

#### 端口号

端口号为 0-65535 之间的整数，有 3 种类型：熟知端口号、注册端口号、临时端口号。

熟知端口号：给每种服务器分配确定的全局端口号。每个用户进程都知道相应的服务器进程的熟知端口号，范围为 0-1023，它是统一分配和控制的。

注册端口号：在 IANA 注册的端口号，数值范围为 1024-49151。

临时端口号：客户端程序使用临时端口号，它是运行在客户端上的 TCP/IP 软件随机选取的，范围为 49152-65535。

我们平时进行网络编程时服务器最好使用注册端口号，而客户端的端口号则是系统随机分配的，即临时端口号。

#### UDP 用户数据报协议

UDP 协议主要有以下一些特点：

- 无连接的：发送数据之前不需要建立连接，因此减少了开销和发送数据之前的时延。
- 尽最大努力交付：即不保证可靠交付，因此主机不需要维持复杂的连接状态表。
- 面向报文的：UDP 对应用层传递下来的报文，既不合并，也不拆分，而是保留这些报文的边界。UDP 对于应用程序提交的报文，添加头部后就向下提交给网络层。
- 没有拥塞控制：网络出现的拥塞时，UDP 不会使源主机的发送速率降低。这对某些实时应用是很重要的，很适合多媒体通信的要求。
- 支持多对多的交互通信

UDP 的适用场景：

- 适用于少量（几百个字节）的数据。
- 对性能的要求高于对数据完整性的要求，如视频播放、P2P、DNS 等。
- 需要“简短快捷”的数据交换 简单的请求与应答报文交互，如在线游戏。
- 需要多播和广播的应用，源主机以恒定速率发送报文，拥塞发生时允许丢弃部分报文，如本地广播、隧道 VPN。

#### TCP 传输控制协议

TCP 协议有以下特点：

- 面向连接的传输服务。打电话式、会话式通信。
- 面向字节流传输服务（而 UDP 是面向报文）。字节管道、字节按序传输和到达。
- 全双工通信。一个应用进程可以同时接收和发送数据、捎带确认；通信双方都设置有发送和接收缓冲区，应用程序将要发送的数据字节提交给发送缓冲区，实际发送由 TCP 协议控制，接收方收到数据字节后将它存放在接收缓冲区，等待高层应用程序读取。
- 可建立多个并发的 TCP 连接。如 Web 服务器可同时与多个客户端建立的连接会话。
- 可靠传输服务。不丢失数据、保持数据有序、向上层不重复提交数据（通过确认机制、拥塞控制等方式实现）， 想象一下 ATM 机转帐应用就需要上述可靠性。

TCP 的报头如下图所示。

![图片描述](https://doc.shiyanlou.com/courses/3573/1116908/544e589943a8426e0f4f4c16981663e7-0)

重点关注几个标志位的意义，如下：

![图片描述](https://doc.shiyanlou.com/courses/3573/1116908/18b10372343df793e922a617fc11c27f-0)

TCP 连接包括连接建立、报文传输、连接释放三个阶段，其中连接建立的三次握手过程较为重要。

建立连接的三次握手过程：

（1）当客户端准备发起一次 TCP 连接，首先向服务器发送第一个“SYN”报文（控制位 SYN=1）。

（2）服务器收到 SYN 报文后，如果同意建立连接，则向客户端发送第二个“SYN+ACK”报文（控制位 SYN=1，ACK=1），该报文表示对第一个 SYN 报文请求的确认。

（3）接收到 SYN+ACK 报文后，客户端发送第三个 ACK 报文，表示对 SYN+ACK 报文的确认。

![图片描述](https://doc.shiyanlou.com/courses/3573/1116908/3c83420afc02fdd2a2e7639275d1d863-0)

### 用 C++ 进行 TCP 套接字网络编程

网络编程中我们一般会使用 C/S 架构，即包含服务器端和客户端。

#### TCP 网络编程的服务器端

服务器端一般先用 socket 创建一个套接字，然后用 bind 给这个套接字绑定地址（即 ip+端口号），然后调用 listen 把这个套接字置为监听状态，随后调用 accept 函数从已完成连接队列中取出成功建立连接的套接字，以后就在这个新的套接字上调用 send、recv 来发送数据、接收数据，最后调用 close 来断开连接释放资源即可。

过程如下：

![图片描述](https://doc.shiyanlou.com/courses/3573/1116908/e1413de95f7af4c44f7d10a2f09d49d4-0)

#### TCP 网络编程的客户端

与服务器不同，客户端并不需要 bind 绑定地址，因为端口号是系统自动分配的，而且客户端也不需要设置监听的套接字，因此也不需要 listen。客户端在用 socket 创建套接字后直接调用 connect 向服务器发起连接即可，connect 函数通知 Linux 内核完成 TCP 三次握手连接，最后把连接的结果作为返回值。成功建立连接后我们就可以调用 send 和 recv 来发送数据、接收数据，最后调用 close 来断开连接释放资源。

过程如下：

![图片描述](https://doc.shiyanlou.com/courses/3573/1116908/e6a08660699bd4f0e26119e7de2c20bd-0)

#### 完整流程

TCP 网络编程的整体流程如下图所示：

![图片描述](https://doc.shiyanlou.com/courses/3573/1116908/87c29545a64eed3b2b2338ef5f901fc4-0)

#### TCP 网络编程的相关数据结构

地址结构：

```cpp
struct sockaddr_in {
    short int sin_family; /* 地址族 */
    unsigned short int sin_port; /* 端口号 */
    struct in_addr sin_addr; /* ip地址 */
    unsigned char sin_zero[8];
};
```

上述结构体涉及到的另一个结构体 in_addr 如下：

```cpp
struct in_addr {
    unsigned long s_addr;
};
```

#### TCP 网络编程各函数的定义

socket 函数：

```cpp
int socket( int domain, int type,int protocol)
/*
功能：创建一个新的套接字，返回套接字描述符
参数说明：
domain：域类型，指明使用的协议栈，如TCP/IP使用的是PF_INET，其他还有AF_INET6、AF_UNIX
type:指明需要的服务类型, 如
SOCK_DGRAM:数据报服务，UDP协议
SOCK_STREAM:流服务，TCP协议
protocol:一般都取0(由系统根据服务类型选择默认的协议)
*/
```

bind 函数：

```cpp
int bind(int sockfd,struct sockaddr* my_addr,int addrlen)
/*
功能：为套接字绑定地址
TCP/IP协议使用sockaddr_in结构，包含IP地址和端口号，服务器使用它来指明熟知的端口号，然后等待连接
参数说明：
sockfd:套接字描述符，指明创建连接的套接字
my_addr:本地地址，IP地址和端口号
addrlen:地址长度
*/
```

listen 函数：

```cpp
int listen(int sockfd,int backlog)
/*
功能：
将一个套接字置为监听模式，准备接收传入连接。用于服务器，指明某个套接字连接是被动的监听状态。
参数说明：
Sockfd:套接字描述符，指明创建连接的套接字
backlog: linux内核2.2之前，backlog参数=半连接队列长度+已连接队列长度；linux内核2.2之后，backlog参数=已连接队列（Accept队列）长度
*/
```

accept 函数：

```cpp
int accept(int sockfd, structsockaddr *addr, int *addrlen)
/*
功能：从已完成连接队列中取出成功建立连接的套接字，返回成功连接的套接字描述符。
参数说明：
Sockfd:套接字描述符，指明正在监听的套接字
addr:提出连接请求的主机地址
addrlen:地址长度
*/
```

send 函数：

```cpp
int send(int sockfd, const void * data, int data_len, unsigned int flags)
/*
功能：在TCP连接上发送数据,返回成功传送数据的长度，出错时返回－1。send会将数据移到发送缓冲区中。
参数说明：
sockfd:套接字描述符
data:指向要发送数据的指针
data_len:数据长度
flags:通常为0
*/
```

recv 函数：

```cpp
int recv(int sockfd, void *buf, intbuf_len,unsigned int flags)
/*
功能：接收数据,返回实际接收的数据长度，出错时返回－1。
参数说明：
Sockfd:套接字描述符
Buf:指向内存块的指针
Buf_len:内存块大小，以字节为单位
flags:一般为0
*/
```

close 函数：

```cpp
close(int sockfd)
/*
功能：撤销套接字。如果只有一个进程使用，立即终止连接并撤销该套接字，如果多个进程共享该套接字，将引用数减一，如果引用数降到零，则关闭连接并撤销套接字。
参数说明：
sockfd:套接字描述符
*/
```

connect 函数：

```cpp
int connect(int sockfd,structsockaddr *server_addr,int sockaddr_len)
/*
功能： 同远程服务器建立主动连接，成功时返回0，若连接失败返回－1。
参数说明：
Sockfd:套接字描述符，指明创建连接的套接字
Server_addr:指明远程端点：IP地址和端口号
sockaddr_len :地址长度
*/
```

### 实战练习

编写两个程序：一个服务器、一个客户端，让用户能在客户端不断输入信息并发送到服务器终端上显示。

#### 服务器程序编写

新建 `server.cpp` 文件，在其中写服务器的代码：

```cpp
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <iostream>
using namespace std;

int main()
{
    //定义sockfd
    int server_sockfd = socket(AF_INET,SOCK_STREAM, 0);

    //定义sockaddr_in
    struct sockaddr_in server_sockaddr;
    server_sockaddr.sin_family = AF_INET;//TCP/IP协议族
    server_sockaddr.sin_port = htons(8023);//端口号
    server_sockaddr.sin_addr.s_addr = inet_addr("127.0.0.1");//ip地址，127.0.0.1是环回地址，相当于本机ip

    //bind，成功返回0，出错返回-1
    if(bind(server_sockfd,(struct sockaddr *)&server_sockaddr,sizeof(server_sockaddr))==-1)
    {
        perror("bind");//输出错误原因
        exit(1);//结束程序
    }

    //listen，成功返回0，出错返回-1
    if(listen(server_sockfd,20) == -1)
    {
        perror("listen");//输出错误原因
        exit(1);//结束程序
    }

    //客户端套接字
    struct sockaddr_in client_addr;
    socklen_t length = sizeof(client_addr);

    //成功返回非负描述字，出错返回-1
    int conn = accept(server_sockfd, (struct sockaddr*)&client_addr, &length);
    if(conn<0)
    {
        perror("connect");//输出错误原因
        exit(1);//结束程序
    }
    cout<<"客户端成功连接\n";

    //接收缓冲区
    char buffer[1000];

    //不断接收数据
    while(1)
    {
        memset(buffer,0,sizeof(buffer));
        int len = recv(conn, buffer, sizeof(buffer),0);
        //客户端发送exit或者异常结束时，退出
        if(strcmp(buffer,"exit")==0 || len<=0)
            break;
        cout<<"收到客户端信息："<<buffer<<endl;
    }
    close(conn);
    close(server_sockfd);
    return 0;
}
```

#### 客户端程序编写

新建创建 `client.cpp` 文件，在其中写客户端的代码：

client.cpp:

```cpp
#include <sys/types.h>
#include <sys/socket.h>
#include <stdio.h>
#include <netinet/in.h>
#include <arpa/inet.h>
#include <unistd.h>
#include <string.h>
#include <stdlib.h>
#include <fcntl.h>
#include <sys/shm.h>
#include <iostream>
using namespace std;

int main()
{
    //定义sockfd
    int sock_cli = socket(AF_INET,SOCK_STREAM, 0);

    //定义sockaddr_in
    struct sockaddr_in servaddr;
    memset(&servaddr, 0, sizeof(servaddr));
    servaddr.sin_family = AF_INET;//TCP/IP协议族
    servaddr.sin_port = htons(8023);  //服务器端口
    servaddr.sin_addr.s_addr = inet_addr("127.0.0.1");  //服务器ip

    //连接服务器，成功返回0，错误返回-1
    if (connect(sock_cli, (struct sockaddr *)&servaddr, sizeof(servaddr)) < 0)
    {
        perror("connect");
        exit(1);
    }
    cout<<"连接服务器成功！\n";

    char sendbuf[100];
    char recvbuf[100];
    while (1)
    {
        memset(sendbuf, 0, sizeof(sendbuf));
        cin>>sendbuf;
        send(sock_cli, sendbuf, strlen(sendbuf),0); //发送
        if(strcmp(sendbuf,"exit")==0)
            break;
    }
    close(sock_cli);
    return 0;
}
```

#### 测试结果

接下来编译这两个 cpp 文件，如下：

```bash
g++ -o server server.cpp
g++ -o client client.cpp
```

![图片描述](https://doc.shiyanlou.com/courses/3573/600404/74090bc531b16287cb0485a090b0c412-0)

然后运行服务器程序：

```bash
./server
```

再开另一个新的终端运行客户端程序:

```bash
./client
```

![图片描述](https://doc.shiyanlou.com/courses/3573/600404/bf4b9a3110f661a4d684a7eb8a3caaaa-0)

![图片描述](https://doc.shiyanlou.com/courses/3573/600404/0d486d3d58c7014a7fdbff22a2564f3e-0)

可以看到连接成功，接下来我们就可以在客户端不断输入信息并且在服务器端显示：

![图片描述](https://doc.shiyanlou.com/courses/3573/600404/7517e331c81c167e29b6da66f019d63b-0)

![图片描述](https://doc.shiyanlou.com/courses/3573/600404/a674b140e5ff34aaad86b8bc7bdb24c6-0)

### 实验总结

通过这次实验，我们了解了传输层的基本概念，对 TCP 协议有了一定的理解，同时尝试了使用 C++ 套接字进行网络编程，最终写出了一个简单的服务器程序和客户端程序。

这次实验的知识将会在我们后面的实验中不断出现，因此掌握好这些内容十分重要。

可以通过如下命令下载本次实验的代码：

```bash
wget https://labfile.oss.aliyuncs.com/courses/3573/code2.zip
```
