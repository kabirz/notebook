#socket

## socket.socket(family=-1, type=-1, proto=-1, fileno=None))
在fileno没有设置的情况下：
 如果family = -1，设置family为`AF_INET`,
 如果type = -1， 设置tpye为`SOCK_STREAM`
 [socket函数的domain、type、protocol解析_socket domain-CSDN博客](https://blog.csdn.net/liuxingen/article/details/44995467)

![[net.png]]
### family
| 名称 | 目的 |
| ---- | ---- |
| AF_UNIX, AF_LOCAL | 本地通信 |
| AF_INET | IPv4网络通信 |
| AF_INET6 | IPv6网络通信 |
| AF_PACKET | 链路层通信 |
### type
| 名称 | 目的 | 编号 |
| ---- | ---- | ---- |
| SOCK_STREAM | TCP | 1 |
| SOCK_DGRAM | UDP | 2 |
| SOCK_RAW | RAW | 3 |
| SOCK_RDM |  | 4 |
| SOCK_SEQPACKET |  | 5 |
| SOCK_DCCP |  | 6 |
| SOCK_PACKET |  | 7 |
[原始套接字SOCK_RAW - aspirant - 博客园 (cnblogs.com)](https://www.cnblogs.com/aspirant/p/4084127.html)
[socket 中 SOCK_STREAM 和 SOCK_DGRAM的区别？-CSDN博客](https://blog.csdn.net/Dontla/article/details/123622895)

###  在UDP/TCP通信中，server端本地信息初始化时ip设置为0.0.0.0了
> 1. INADDR_ANY就是指定地址为0.0.0.0的地址，这个地址事实上表示不确定地址，或“所有地址”、“任意地址”。 一般来说，在各个系统中均定义成为0值。
> 2. 在建立网络服务器应用程序时，server会通过bind()函数通知操作系统：请在某地址 xxx.xxx.xxx.xxx上的某端口 yyyy上进行侦听，并且把侦听到的数据包发送给我，即server程序会申请绑定服务器的某地址及端口。而服务器操作系统可以给这个指定的地址，也可以不给。
 >3. 如果server端操作系统内有多个网卡（每个网卡上有不同的IP地址），而你的服务（不管是在udp端口上侦听，还是在tcp端口上侦听）出于某种原因：可能是服务器操作系统可能随时增减IP地址，也有可能是为了省去确定服务器上有什么网络端口（网卡）的麻烦 —— 可以要在调用bind()的时候，告诉操作系统：“我需要在 yyyy 端口上侦听，所有发送到服务器的这个端口，不管是哪个网卡/哪个IP地址接收到的数据，都是我处理的。”这时候，服务器程序则在0.0.0.0这个地址上进行侦听。
 
 ## UDP 单播
 ```python
import socket


def sender(host, port, message):
    """发送 UDP 数据包"""

    # 创建 UDP 套接字
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
    # 发送数据包
    sock.sendto(message.encode("utf-8"), (host, port))


def receiver(host, port):
    """接收 UDP 数据包"""

    # 创建 UDP 套接字
    sock = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)

    # 绑定到本地地址和端口
    sock.bind((host, port))

    # 接收数据包
    while True:
        data, addr = sock.recvfrom(1024)
        print("收到来自 {} 的数据：{}".format(addr, data.decode("utf-8")))
```

[asyncio实现异步socket server - 掘金 (juejin.cn)](https://juejin.cn/post/7120545649891737614)
