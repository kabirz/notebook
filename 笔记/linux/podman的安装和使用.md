## 安装

```shell
sudo apt install podman
```

## 配置(ubuntu)
添加
```shell
unqualified-search-registries=["docker.io"]
```
到`/etc/containers/registries.conf`
接下来就可以和docker无缝对接
比如
```shell
podman run -d -p 3210:3210 -e ACCESS_CODE=lobe66 --name lobe-chat lobehub/lobe-chat
```
就可以直接使用，当然也可以不修改配置文件，直接执行
```shell
podman run -d -p 3210:3210 -e ACCESS_CODE=lobe66 --name lobe-chat docker.io/lobehub/lobe-chat
```

[如何配置 Podman 使用国内镜像源？_podman国内镜像_podman 配置 国内 加速 镜像-CSDN博客](https://blog.csdn.net/m0_55025322/article/details/137677307)
[podman修改国内镜像源 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/616622801)
[用 Podman Compose 管理容器 | Linux 中国 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/350998895)
