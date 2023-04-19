#hw #linux 
1. lshw

lshw命令显示详细硬件信息。

如果要用概要方式显示，可以加上short参数：lshw -short

要显示指定硬件信息，加上class(或C)参数：lshw -class memory

2. sysstat

监测系统性能及效率的一组工具，这些工具对于我们收集系统性能数据，

比如CPU使用率、硬盘和网络吞吐数据。

3. lspci -v (相比cat /proc/pci更直观）

查看PCI信息，lspci 是读取 hwdata 数据库。

4. uname -a

查看系统体系结构。

5. dmidecode

查看硬件信息，包括bios、cpu、内存等信息

**6. dmesg**

**显示内核缓冲区系统控制信息，如系统启动时的信息会写到/var/log/。**

注：dmesg 工具并不是专门用来查看硬件芯片组标识的工具，

但通过这个工具能让我们知道机器中的硬件的一些参数；因为系统在启动的时候，

会写一些硬件相关的日志到 /var/log/message* 或 /var/log/boot* 文件中。

7. lshal 和 hal-device-manager  

8. 查看 /proc

对于“/proc”中文件可使用文件查看命令浏览其内容，文件中包含系统特定信息：

Cpuinfo       主机CPU信息

Dma          主机DMA通道信息

Filesystems    文件系统信息

Interrupts       主机中断信息

Ioprots           主机I/O端口号信息

Meninfo       主机内存信息

Version           Linux内存版本信息

查看CPU信息：cat /proc/cpuinfo 查看板卡信息：cat /proc/pci 查看内存信息：cat /proc/meminfo 查看USB设备：cat /proc/bus/usb/devices 查看键盘和鼠标：cat /proc/bus/input/devices 查看各设备的中断请求(IRQ)：cat /proc/interrupts