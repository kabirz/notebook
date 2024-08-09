[hf-mirror.com - Huggingface 镜像站](https://hf-mirror.com/)
[如何快速下载huggingface模型——全方法总结 - 知乎 (zhihu.com)](https://zhuanlan.zhihu.com/p/663712983)
最近huggingface在使用mirror下载模型时我出现了无法下载的情况，解决方法
安装`hf_transfer`
```shell
pip install hf_transfer
```
并设置环境变量
```shell
export HF_HUB_ENABLE_HF_TRANSFER=1
```

