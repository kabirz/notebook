
[PCIe扫盲系列博文连载目录_pcie扫阿蒙_山音水月的博客-CSDN博客](https://blog.csdn.net/linbian1168/article/details/87304193)
[arm-linux 系统调用流程 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/363290974)
[深入理解Linux进程调度O(1)调度算法 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/464176766)
[一文搞定 Linux 设备树 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/425420889)
[ISP——ISP PIPELINE_海思pipeline_wtzhu_13的博客-CSDN博客](https://blog.csdn.net/wtzhu_13/article/details/118255864)
[Linux启动过程初始化（十二）----watchdog初始化_嵌入式攻城狮小白的博客-CSDN博客](https://blog.csdn.net/qq_40788950/article/details/84845749)
[Linux Watchdog 机制_ericstarmars的博客-CSDN博客](https://blog.csdn.net/ericstarmars/article/details/81750919)
[强力科普一下PCIe/CXL（Compute Express Link）_大话存储的博客-CSDN博客](https://blog.csdn.net/TV8MbnO2Y2RfU/article/details/106184058)
[《大话计算机》序言⑥ by 张勇 (qq.com)](https://mp.weixin.qq.com/s?__biz=MzAwNzU3NzQ0MA%3D%3D&chksm=809b04b3b7ec8da573db1580f069e132d5e21d09248be63042b27710343eabc5e015bb31a765&idx=1&mid=2652090044&scene=21&sn=4f4dcfedceb7cb76df174a5d7826fd9f#wechat_redirect)



- 虚拟地址 <==>物理地址

```C
#define __pa(x)                 __virt_to_phys((unsigned long)(x))
static inline unsigned long virt_to_phys(volatile void *address)
{
      return __pa((unsigned long)address);
}
#define __va(x)                 ((void *)__phys_to_virt((phys_addr_t)(x)))
static inline void *phys_to_virt(unsigned long address)
{
      return __va(address);
}
```

-   硬中断和软中断的区别
    
    ①硬中断是由外部事件引起的因此具有随机性和突发性；软中断是执行中断指令产生的，无面外部施加中断请求信号，因此中断的发生不是随机的而是由程序安排好的。 ②硬中断的中断响应周期，CPU需要发中断回合信号（NMI不需要），软中断的中断响应周期，CPU不需发中断回合信号。 ③硬中断的中断号是由中断控制器提供的（NMI硬中断中断号系统指定为02H）；软中断的中断号由指令直接给出，无需使用中断控制器。 ④硬中断是可屏蔽的（NMI硬中断不可屏蔽），软中断不可屏蔽。
    
    软中断是一种推后执行的机制,定时器,网卡的数据的处理是很典型的软中断,这个和中断向 量表里的中断是完全不一样的,以网络数据的处理为例,当网卡接到一个数据包后,其中断处理程序（硬中断）只是把数据复制到缓冲区,然后就告诉网卡,你可以再传数据给 我了,也就是中断返回,但在此之前,网卡的中断处理程序要置一个标志位,告诉操作系统有事要做,这个事就是软中断,但软中断只是很多中断返回时要做的事情之一,操作系统每次中断返回时会检查着个标志位,看是否有事要做,如果有,就会去处理,象前面提到的网卡,这时候操作系统就回调用软中断的处理函数,网卡的软中断程序就是做分析数据包啊,这个数据应该传给谁啊等这些工作.没有,就返回了,除了必须的部分。
    
    编写两个中断服务函数的区别 1.软中断发生的时间是由程序控制的,而硬中断发生的时间是随机的 2.软中断是由程序调用发生的,而硬中断是由外设引发的 3.硬件中断处理程序要确保它能快速地完成它的任务,这样程序执行时才不会等侍较长时间 ———————————————— 版权声明：本文为CSDN博主「木木总裁」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。 原文链接：[https://blog.csdn.net/ll148305879/article/details/90902282](https://blog.csdn.net/ll148305879/article/details/90902282)


-   linux sleep
```C
#include <linux/delay.h>
void msleep(unsigned int msecs)
{
  unsigned long timeout = msecs_to_jiffies(msecs) + 1;
  while (timeout)
    timeout = schedule_timeout_uninterruptible(timeout);
}
mdelay(1000);
void ndelay(unsigned long nsecs); //纳秒延时
void udelay(unsgined long usecs); //微秒延时
void mdelay(unsigned long msecs); //毫秒延时
static inline void usleep_range(unsigned long min, unsigned long max)
{
  // TASK_UNINTERRUPTIBLE状态是一种不可中断的睡眠状态，不可以被信号打断，
  // 必须等到等待的条件满足时才被唤醒
  usleep_range_state(min, max, TASK_UNINTERRUPTIBLE);
}
usleep_range(11, 22);
ssleep(seconds);
// 短延时
delay(int ms)
{
    unsigned long end_time = jiffies + HZ * 5; //表示在当前时间点5秒后为终止时间点。
    while(time_before(jiffies, end_time)); //jiffies未到end_time的话会一直在这个循环中打转。time_before()函数位于jiffies.h中。
}
do_anything_you_wanna_do(); //时间到！
```

linux如何enable clock linux clk 框架

```C
#if 0
cam1: camera@3e {
    compatible = "toshiba,et8ek8";
    reg = <0x3e>;
    vana-supply = <&vaux4>;
    clocks = <&isp 0>;
    clock-names = "extclk";
    clock-frequency = <9600000>;
    reset-gpio = <&gpio4 6 GPIO_ACTIVE_HIGH>; /* 102 */
    lens-focus = <&ad5820>;
    port {
        csi_cam1: endpoint {
            bus-type = <3>; /* CCP2 */
            strobe = <1>;
            clock-inv = <0>;
            crc = <1>;
            remote-endpoint = <&csi_isp>;
        };
    };
};
#endif
// drivers/media/i2c/et8ek8/et8ek8_driver.c
#include <linux/clk.h>
struct et8ek8_sensor {
  struct clk *ext_clk;
  u32 xclk_freq;
};

sensor->ext_clk = devm_clk_get(dev, NULL); 
ret = of_property_read_u32(dev->of_node, "clock-frequency",&sensor->xclk_freq);
clk_disable_unprepare(sensor->ext_clk);  // power off
// power on
rval = clk_set_rate(sensor->ext_clk, xclk_freq);
rval = clk_prepare_enable(sensor->ext_clk);
```