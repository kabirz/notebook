[制作Ubuntu arm rootfs](https://wiki.ubuntu.com/ARM/RootfsFromScratch/QemuDebootstrap)

按照以下步骤使用 fallocate 和 mtools 制作 vfat 的根文件系统：
1. 使用 fallocate 命令创建一个指定大小的空文件，作为 vfat 文件系统的映像文件。例如，创建一个大小为 100MB 的文件：
```shell
fallocate -l 100M vfat.img
```

2. 使用 losetup 命令将映像文件与一个未使用的循环设备连接起来：
```shell
sudo losetup -fP vfat.img
```

3. 使用 mkfs.vfat 命令对连接的设备进行格式化：
```shell
sudo mkfs.vfat /dev/loopX
```
其中“X”是连接设备的编号，您可以使用 losetup -a 命令查看。

4. 使用 mkdir 命令创建用于挂载文件系统的目录：
```shell
sudo mkdir /mnt/vfat
```
5. 使用 mount 命令将文件系统挂载到目录中：
```shell
sudo mount /dev/loopX /mnt/vfat
```

6. 使用 mtools 命令在挂载的文件系统中创建文件和文件夹：
```shell
sudo mmd -i vfat.img@@ /foldername
sudo mcopy -i vfat.img@@ file.txt ::/
```
其中，-i 参数指定映像文件的路径和设备号，@ 字符用于区分文件系统和文件名。::/ 表示将文件复制到根目录，/foldername 是要创建的目录的名称。
7. 使用 umount 和 losetup 命令卸载挂载的文件系统并解除与映像文件的连接：
```shell
sudo umount /mnt/vfat
sudo losetup -d /dev/loopX
```
现在，您已经成功使用 fallocate 和 mtools 制作了 vfat 的根文件系统。

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
