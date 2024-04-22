查看网络连接情况

#  1. `ifconfig` /`ip`
 
 安装方法 `sudo apt install net-tools`
 使用`ifconfig -a` 查看所有以太网网络接口信息, 如下图，对于带网口的主机通常至少会有两个(物理网卡和lo)
 ```text
 eth0: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.18.0.2  netmask 255.255.0.0  broadcast 172.18.255.255
        ether 02:42:ac:12:00:02  txqueuelen 0  (Ethernet)
        RX packets 141  bytes 216714 (216.7 KB)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 110  bytes 7400 (7.4 KB)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0

lo: flags=73<UP,LOOPBACK,RUNNING>  mtu 65536
        inet 127.0.0.1  netmask 255.0.0.0
        loop  txqueuelen 1000  (Local Loopback)
        RX packets 0  bytes 0 (0.0 B)
        RX errors 0  dropped 0  overruns 0  frame 0
        TX packets 0  bytes 0 (0.0 B)
        TX errors 0  dropped 0 overruns 0  carrier 0  collisions 0
```
里面的flags是标志位，UP代表接口已启动，启用/禁用网卡用命令`ifconfig eth0 up/down`, 开启DHCP之后ip会自动设置，如果没有DHCP，可以手动设置：`ifconfig eth0 172.16.129.108 netmask 255.255.255.0` 当然也更建议使用`ip`命令来操作，事实上`ifconfig`已经被丢弃，使用`ip`命令需要安装`sudo apt install iproute2`
对应的命令分别是

| ifconfig | ip |
| ---- | ---- |
| ifconfig -a | ip a[ddress] |
| ifconfig eth0 up/down | ip link set eth0 up/down |
| ifconfig eth0 172.16.129.108 netmask 255.255.255.0 | ip addr add 172.16.129.108/24 dev eth0 |
# 2. `ping`

`ping` 命令发生ICMP包，它网络诊断中最常用的工具之一，它主要用于测试网络连接和延迟。  
命令`ping -c 4 10.239.71.xx`，其中`-c`参数用来指定ICMP请求的次数, 输出结果大致如下
```text
❯  ping 10.239.71.xx -c 4
PING 10.239.71.47 (10.239.71.xx) 56(84) bytes of data.
64 bytes from 10.239.71.xx: icmp_seq=1 ttl=64 time=0.377 ms
64 bytes from 10.239.71.xx: icmp_seq=2 ttl=64 time=0.328 ms
64 bytes from 10.239.71.xx: icmp_seq=3 ttl=64 time=0.256 ms
64 bytes from 10.239.71.xx: icmp_seq=4 ttl=64 time=0.260 ms

--- 10.239.71.xx ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3069ms
rtt min/avg/max/mdev = 0.256/0.305/0.377/0.050 ms
```
最后两行显示的是发送接收到丢失的包情况和时间
`rtt` 表示的是往返最小/平均/最大和偏差时间

# 3. `ethtool`
用于查看和修改网络设备（尤其是有线以太网设备）的驱动参数和硬件设置。你可以根据需要更改以太网卡的参数，包括自动协商、速度、双工和局域网唤醒等参数。通过对以太网卡的配置，你的计算机可以通过网络有效地进行通信。该工具提供了许多关于接驳到你的 Linux 系统的以太网设备的信息。
安装方法`sudo apt install ethtool` 

### 查看网卡信息`sudo ethtool eth0`
```text
Settings for eth0:
        Supported ports: [ TP    MII ]
        Supported link modes:   10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
                                2500baseT/Full
        Supported pause frame use: Symmetric Receive-only
        Supports auto-negotiation: Yes
        Supported FEC modes: Not reported
        Advertised link modes:  10baseT/Half 10baseT/Full
                                100baseT/Half 100baseT/Full
                                1000baseT/Full
                                2500baseT/Full
        Advertised pause frame use: Symmetric Receive-only
        Advertised auto-negotiation: Yes
        Advertised FEC modes: Not reported
        Link partner advertised link modes:  10baseT/Half 10baseT/Full
                                             100baseT/Half 100baseT/Full
                                             1000baseT/Full
        Link partner advertised pause frame use: Symmetric Receive-only
        Link partner advertised auto-negotiation: Yes
        Link partner advertised FEC modes: Not reported
        Speed: 1000Mb/s
        Duplex: Full
        Auto-negotiation: on
        master-slave cfg: preferred slave
        master-slave status: slave
        Port: Twisted Pair
        PHYAD: 0
        Transceiver: external
        MDI-X: Unknown
        Supports Wake-on: pumbg
        Wake-on: d
        Link detected: yes
```
以下是该命令输出中每项的含义：
- **Supported ports**: [ TP MII ]
    - TP: Twisted Pair（双绞线）
    - MII: Media Independent Interface（媒体独立接口），这是一种用于以太网MAC和PHY之间的标准接口。
- **Supported link modes**: 支持的链路模式
    - 10baseT/Half: 10Mbps半双工
    - 10baseT/Full: 10Mbps全双工
    - 100baseT/Half: 100Mbps半双工
    - 100baseT/Full: 100Mbps全双工
    - 1000baseT/Full: 1000Mbps全双工
    - 2500baseT/Full: 2500Mbps全双工 这些是接口支持的不同的速率和双工模式。
- **Supported pause frame use**: 支持暂停帧使用
    - Symmetric Receive-only: 支持对称接收暂停帧，但不支持发送暂停帧。
- **Supports auto-negotiation**: 支持自动协商
    - Yes: 接口支持自动协商链路参数。
- **Supported FEC modes**: 支持的前向纠错模式
    - Not reported: 没有报告支持的前向纠错模式。
- **Advertised link modes**: 宣布支持的链路模式
    - 这些是与“Supported link modes”相同的信息，但这里表示接口向对端宣布它支持的链路模式。
- **Advertised pause frame use**: 宣布的暂停帧使用
    - 与“Supported pause frame use”相同，表示接口宣布支持的暂停帧使用。
- **Advertised auto-negotiation**: 宣布的自动协商
    - Yes: 接口宣布它支持自动协商。
- **Link partner advertised link modes**: 链路伙伴宣布的链路模式
    - 这些是链路伙伴（对端设备）宣布支持的链路模式。
- **Link partner advertised pause frame use**: 链路伙伴宣布的暂停帧使用
    - 与“Advertised pause frame use”相似，但来自链路伙伴的信息。
- **Link partner advertised auto-negotiation**: 链路伙伴宣布的自动协商
    - Yes: 链路伙伴宣布它支持自动协商。

以下是接口的当前状态：
- **Speed**: 1000Mb/s
    - 接口的当前速率是1000Mbps。
- **Duplex**: Full
    - 接口的当前双工模式是全双工。
- **Auto-negotiation**: on
    - 自动协商当前是开启的。
- **master-slave cfg**: preferred slave
    - 接口配置为首选从设备。
- **master-slave status**: slave
    - 接口的当前主从状态是从设备。
- **Port**: Twisted Pair
    - 接口类型是双绞线。
- **PHYAD**: 0
    - PHY地址是0。
- **Transceiver**: external
    - 传输器类型是外部的。
- **MDI-X**: Unknown
    - 是否支持自动交叉模式未知。
- **Supports Wake-on**: pumbg
    - 支持的网络唤醒功能包括：pumbg代表电源管理、USB唤醒、魔术包（Magic Packet）、断电唤醒（Wake-on LAN）和全局唤醒（Wake-on Wireless）。
- **Wake-on**: d
    - 当前启用的网络唤醒功能是断电唤醒（Wake-on LAN）。
- **Link detected**: yes
    - 检测到链路连接。

### 查看网卡详细信息使用命令`sudo ethtool -i eth0`

```text
# sudo ethtool -i eth0
driver: r8169
version: 6.5.0-14-generic
firmware-version: rtl8125b-2_0.0.2 07/13/20
expansion-rom-version:
bus-info: 0000:08:00.0
supports-statistics: yes
supports-test: no
supports-eeprom-access: no
supports-register-dump: yes
supports-priv-flags: no
```
其中`r8169` 对应是驱动，通过modinfo工具可以查看驱动模块对应的信息  

### 检查网络使用情况统计`ethtool`
```text
ethtool -S eth0
NIC statistics:
     tx_packets: 4018269
     rx_packets: 7711581
     tx_errors: 0
     rx_errors: 0
     rx_missed: 0
     align_errors: 0
     tx_single_collisions: 0
     tx_multi_collisions: 0
     unicast: 3530891
     broadcast: 2906700
     multicast: 1273990
     tx_aborted: 0
     tx_underrun: 0
```
其中:
- `tx_packets`: 发送的数据包总数。
- `rx_packets`: 接收的数据包总数
- `tx_errors`: 发送过程中遇到的错误总数。
- `rx_errors`: 接收过程中遇到的错误总数。
- `rx_missed`: 由于硬件队列满而未被接收的数据包数量。
- `align_errors`: 接收到的未对齐的数据包数量。
- `tx_single_collisions`: 发送数据包时发生的单次冲突数。
- `tx_multi_collisions`: 发送数据包时发生的多次冲突数。
- `unicast`: 发送或接收的单播数据包数量。
- `broadcast`: 发送的广播数据包数量。
- `multicast`: 发送或接收的多播数据包数量。
- `tx_aborted`: 由于错误而中途放弃的发送尝试次数。
- `tx_underrun`: 发送队列因没有数据而导致的空转次数。

### 改变以太网设备的速度 
`-s` 参数是设置  
`ethtool -s eth0 speed 100`改变之后会导致网卡掉线，需要重新up `ip link set eth0 up`  
另外也可以同时改变多个参数  
`ethtool –s [device_name] speed [10/100/1000] duplex [half/full] autoneg [on/off]`  

### 多个设备中识别出特定的网卡
执行`ethtool -p eth0`后看网口的等闪烁情况  


# 4. `nc`

`nc`是一种功能强大的网络工具，用于在计算机之间进行网络通信和数据传输。  
服务器端开启监听， 使用`-l` 参数， 如果是UDP另外加 `-u`  
TCP  
`nc -l 0.0.0.0 1234`  
UDP  
`nc -lu 0.0.0.0 2345`  

客户端给服务器端发送数据， 如果是UDP另外加 `-u`, 假设服务器端ip `172.10.2.3`  
TCP  
`nc 172.10.2.3 1234 < file_name`  
UDP  
`nc -u 172.10.2.3 2345 < file_name`  
服务器端终端会有文件内容输出  


## ethernet loopback
需要rework或者需要专用的设备

![[Pasted image 20240205132211.png]]
如上图所示
- Pin 1 TX 和 Pin 3 RX 短接
- Pin 2 TX 和 Pin 4 RX 短接

这个测试我在网上查到的是交换机的配置
1. [juniper EX系列交换机VLAN配置操作]
2. [Performing Loopback Testing](https://www.juniper.net/documentation/us/en/software/junos/interfaces-ethernet/topics/topic-map/ethernet-fast-and-gigabit-loopback-testing.html#id-configure-a-local-loopback)
里面涉及到的网络配置命令在linux pc上没有。
