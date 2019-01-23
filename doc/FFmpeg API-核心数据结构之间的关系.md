### 目录
1. 参考
2. 简介
3. 解协议
4. 解封装
5. 解码
6. 存数据

### 1. 参考
- [1] [雷霄骅/
FFMPEG中最关键的结构体之间的关系](https://blog.csdn.net/leixiaohua1020/article/details/11693997)
- [2] [FFmpeg/tree/release/4.1](https://github.com/FFmpeg/FFmpeg/tree/release/4.1/)

### 2. 简介
最关键的结构体可以分成以下几类：
- 协议: AVIOContext 
- 封装: AVFormatContext, AVInputFormat, AVOutputFormat 
- 编解码: AVCodecContext, AVCodec, AVStream
- 存数据: AVPacket, AVFrame

![core_structure.jpg](https://upload-images.jianshu.io/upload_images/8744338-e653386a65e96108.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
### 3. 协议
协议包括http,rtsp,rtmp,mms等。

AVIOContext，URLProtocol，URLContext主要存储视音频使用的协议的类型以及状态。
- URLProtocol存储输入视音频使用的封装格式。每种协议都对应一个URLProtocol结构。

### 4. 封装
封装格式包括flv,avi,rmvb,mp4等。

AVFormatContext主要存储视音频封装格式中包含的信息；
- AVInputFormat存储输入视音频使用的封装格式。每种视音频封装格式都对应一个AVInputFormat 结构。

### 5. 编解码
编码格式包括h264,mpeg2,aac,mp3等。

每个AVStream存储一个视频/音频流的相关数据；
- 每个AVStream对应一个AVCodecContext，存储该视频/音频流使用解码方式的相关数据。
- 每个AVCodecContext中对应一个AVCodec，包含该视频/音频对应的编解码器。每种编解码器都对应一个AVCodec结构。

### 6. 存数据
视频的话，每个结构一般是存一帧；音频可能有好几帧
- AVPacket: 解码前的压缩数据。
- AVFrame: 解码后非压缩数据。


