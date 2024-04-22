
必须在pytorch 加载之前设置这个环境变量才有效
```python
import os
os.environ['CUDA_VISIBLE_DEVICES'] = '1,2'
import torch
```

或者直接在终端中设置
```shell
export CUDA_VISIBLE_DEVICES="1,2"
```