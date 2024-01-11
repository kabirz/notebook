报错信息：OMP: Error #15: Initializing libiomp5md.dll, but found libiomp5md.dll already initialized 

```python
import os
 os.environ["KMP_DUPLICATE_LIB_OK"]="TRUE"
```
