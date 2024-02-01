#docker
当docker把当前用户的文件夹mount到容器的时候，如果容器内用户的uid和gid都和host的不同时会出现文件权限问题
解决方法：
在docker容器中添加一个用户组
```shell
sudo groupadd -g <GID> <group_name>
```
其中`<GID>`的值和host的`id -g $USER`相同
在容器中把用户添加到`<group_name>`组中
```shell
sudo usermod -a-G <group_name> <user_name>
```
然后推出容器并把容器commit到新的image中
```shell
# 找到容器id
docker ps -l
# 提交到image
docker commit <container_id>  <image>
```
`<image>`可以覆盖旧的image，也可以用一个新的名字创建一个新的image
