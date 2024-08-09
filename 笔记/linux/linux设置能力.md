#setcap
[linux setcap命令详解(包括各个cap的使用举例)【转】 - Sky&Zhang - 博客园 (cnblogs.com)](https://www.cnblogs.com/sky-heaven/p/12096758.html)
[linux setcap命令详解(包括各个cap的使用举例)-CSDN博客](https://blog.csdn.net/megan_free/article/details/100357702)

例子：
python开启http服务无法使用80端口，可以这样打开
```shell
sudo setcap 'cap_net_bind_service=+ep' /usr/bin/python3.12
```
注意这里的`/usr/bin/python3.12`不可以是链接文件
