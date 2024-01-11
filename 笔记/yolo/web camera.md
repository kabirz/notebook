## 使用fastapi打开摄像头

```python
import cv2
from fastapi import FastAPI
from fastapi.responses import StreamingResponse
 
app = FastAPI()
 
 
def generate_frames():
    camera = cv2.VideoCapture(0)
    while True:
        success, frame = camera.read()
        frame = cv2.flip(frame, 1)
        if not success:
            break
        ret, buffer = cv2.imencode('.jpg', frame)
        frame = buffer.tobytes()
        yield (b'--frame\r\n'
               b'Content-Type: image/jpeg\r\n\r\n' + frame + b'\r\n')
 
 
@app.get('/video_feed')
def video_feed():
    return StreamingResponse(generate_frames(), media_type='multipart/x-mixed-replace; boundary=frame')
```
保存为`main.py`, 使用`uvicorn`执行
```shell
uvicorn --host 0.0.0.0 main:app --reload
```

## 使用flask打开摄像头
参考：[利用Flask+Python+OpenCV实现摄像头读取图像帧网页视频流_](https://blog.csdn.net/qq_33952811/article/details/119140307)

