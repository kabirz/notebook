[制作Ubuntu arm rootfs](https://wiki.ubuntu.com/ARM/RootfsFromScratch/QemuDebootstrap)

1.  生成指定大小的镜像文件：
```shell
fallocate -l 100M rootfs.img
```
2.  格式化文件系统
```shell
mkfs.vfat rootfs.img
```
3.  拷贝文件到文件系统镜像
```shell
mcopy -i rootfs.img ~/data ::/
```
