#docker #podman

## 环境安装(ubuntu)
```shell
sudo apt install podman slirp4netns uidmap
```

## 使用
### 安装和使用ubuntu
```
podman run -it ubuntu 
```

安装和使用mqtt服务
```shell
podman pull docker.io/emqx/emqx
podman run -d --name emqx -p 1883:1883 -p 8083:8083 -p 8084:8084 -p 8883:8883 -p 18083:18083 emqx/emqx
```

# 开机启动
[Podman设置容器开机自启 - 哈喽哈喽111111 - 博客园 (cnblogs.com)](https://www.cnblogs.com/hahaha111122222/p/17373380.html)
