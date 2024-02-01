#mtqq
mtqq 服务端环境搭建
[Docker 部署 MQTT服务器(EMQX)_搭建mqtt服务器](https://blog.csdn.net/m0_46601820/article/details/134969551)
使用docker部署mqtt
```shell
docker pull emqx/emqx:latest
docker run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx:latest 
```

端口：

- 1883：MQTT协议端口
- 8083：WebSocket协议端口
- 8084：HTTP协议端口
- 8883：MQTT over SSL协议端口
- 18083：EMQX Dashboard Web管理界面端口
相关学习文档
[MQTT系列 | MQTT 5.0协议新特性 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/80204183)
[MQTT协议，终于有人讲清楚了 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/421109780)
[MQTT搭建 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/628122165)
![[beginners-guide-to-the-mqtt-protocol.pdf]]

客户端
[MQTTX Download](https://mqttx.app/downloads)