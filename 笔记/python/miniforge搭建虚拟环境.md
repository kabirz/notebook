使用[miniforge](https://github.com/conda-forge/miniforge)搭建虚拟环境

鉴于某些企业没有购买anaconda使用版权，这里介绍一个完全开源的anaconda替代方法

更简单是使用方法
[Micromamba Installation — documentation](https://mamba.readthedocs.io/en/latest/installation/micromamba-installation.html)

Unix
```shell
"${SHELL}" <(curl -L micro.mamba.pm/install.sh)
```
Windows
```powershell
Invoke-Expression ((Invoke-WebRequest -Uri https://micro.mamba.pm/install.ps1).Content)
# config
micromamba config append channels conda-forge
micromamba config set channel_priority strict
```
