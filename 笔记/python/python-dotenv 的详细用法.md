# 安装
```shell
pip install -U python-dotenv
```
确保你的项目目录下 有 .env 文件
```css
.
├── .env
└── settings.py
```
# 用法
```python
# settings.py
from dotenv import load_dotenv, find_dotenv
from pathlib import Path

# 一、自动搜索 .env 文件

load_dotenv(verbose=True)


# 二、与上面方式等价

load_dotenv(find_dotenv(), verbose=True)

# 三、与上面方式等价 
指定 .env 文件位置

env_path = Path('.') / '.env'
load_dotenv(dotenv_path=env_path, verbose=True)
```

