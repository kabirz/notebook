[尝试使用其他构建系统构建 - xmake](https://xmake.io/#/zh-cn/features/trybuild?id=%e6%97%a0%e7%bc%9d%e5%af%b9%e6%8e%a5xmake%e5%91%bd%e4%bb%a4)

## 配置
```shell
## 配置
xmake f -p cross  -a arm64  --trybuild=cmake
## 交叉编译配置
xmake f -p cross  -a arm64  --trybuild=cmake --cc=aarch64-linux-gnu-gcc--ld=aarch64-linux-gnu-gcc --cxx=aarch64-linux-gnu-g++
## 也可以使用tryconfig设置其他参数
--tryconfigs="-DCMAKE_EXPORT_COMPILE_COMMANDS=ON“


## 编译
### 默认使用makefile编译
xmake
### 配置使用Ninja编译
CMAKE_GENERATOR=Ninja xmake
```