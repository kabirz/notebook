
```shell
ffmpeg -i in.avi -c:v libx264 -crf 20 -pix_fmt yuv420p -c:a copy -movflags +faststart out.mp4
# 或者
ffmpeg -i input.avi -c:v libx264 -crf 20 -pix_fmt yuv422p output.mp4
ffmpeg -i output.mp4 -pix_fmt yuv420p -c:a copy -movflags +faststart output1.mp4
```

