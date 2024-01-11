# 类似构造析构
```python
from fastapi import FastAPI
import datetime

app = FastAPI()

@app.on_event("startup")
async def startup_event():
   print('Server started :', datetime.datetime.now())


@app.on_event("shutdown")
async def shutdown_event():
   print('server Shutdown :', datetime.datetime.now())
```

<div>
<h1> Hello, Java </h1>
<img src="https://docs.anyscale.com/site-assets/logo.png" alt="Image 3">
</div>