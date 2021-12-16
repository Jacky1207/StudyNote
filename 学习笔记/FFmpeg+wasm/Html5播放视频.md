## 解决思路

### 1、插件播放
插件播放最简单的就是vlc插件。

### 2、websocket推流
    服务端获取视频流后通过websocket服务器，推送到html websocket客户端。
web端收到视频流后，解码方式有多种， 以H264裸流为例：

#### 1、js将收到的H264裸流进行封装，变成mp4格式。在通过Html5的video控件进行解码显示。
#### 2、用ffmpeg Wasm进行解码播放。