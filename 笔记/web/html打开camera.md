
#camera #webcam

```html
<!DOCTYPE html>
<html lang="en">

<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Camera Preview</title>
</head>

<body>
    <video></video>
    <div>
        <button onclick="openCamera()">打开摄像头</button>
        <button onclick="closeCamera()">关闭摄像头</button>
    </div>
    <script>
        var cameraStream;
        var video = document.querySelector("video");
    
        function openCamera() {
            if ((cameraStream !== undefined) && (cameraStream.readyState === 'live')) {
                console.log('摄像头已经打开')
                return
            }
            navigator.mediaDevices.getUserMedia(
                { video: true }
            ).then(stream => {
                cameraStream = stream.getTracks()[0]

                video.srcObject = stream;
                video.onloadedmetadata = e => video.play();
            }).catch(err => {
                console.log(err.name + ": " + err.message);
            });
        }

        function closeCamera() {
            if (cameraStream.readyState === 'live') {
                cameraStream.stop()
                console.log('摄像头已经关闭')
            }
        }
    </script>
</body>

</html>
```