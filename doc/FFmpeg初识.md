### 目录
1. 参考
2. FFmpeg简介
3. FFmpeg命令行工具介绍
4. FFmepg类库介绍

### 1. 参考
- [1] [FFmpeg官网介绍](http://ffmpeg.org/about.html])

### 2. FFmpeg简介
FFmpeg 是一个开源的、免费的、跨平台的音视频方案。属于自由软件， 采用 LGPL 或GPL 许可证（依据你选择的组件）。 它提供了录制、转换以及流化音视频的完整解决方案。

FFmpeg已被广泛应用。如Facebook使用FFmpeg工具来处理用户上传的视频；Google Chrome使用FFmpeg的库来支持HTML5中的音频和视频；Youtube使用FFmpeg对上传的视频进行转换。

FFmpeg名称中的"FF"是“Fast Forward”的缩写，是快进的意思。"mpeg" 则是标准化组织“Moving Pictures Experts Group”的缩写。

FFmpeg项目由Fabrice Bellard创建于2000年，使用C语言编写，遵循GNU协议。

### 3. FFmpeg命令行工具介绍
FFmpeg命令行工具主要有以下几个：
1. ffmpeg：对视频音频和图片进行编解码、格式转换、分割和合并等。
2. ffplay：简单的媒体播放器，使用了ffmpeg 和 sdl 库。
3. ffprobe：多媒体流分析工具。

以下是命令行工具的几个简单的使用示例，更多的介绍可以参考[FFmpeg命令行工具-实用命令](https://www.jianshu.com/p/124aee284a61)：
1. 修改图片\视频分辨率
```
ffmpeg -i input -vf scale=iw/2:-1 output
//iw:输入帧宽，此处把帧宽缩小为原来的1/2。
//-1告诉scale filter保持纵横比。
```
2. ffplay播放文件
```
ffplay test.mp4
```
3. 播放网络文件
```
ffplay rtsp://184.72.239.149/vod/mp4://BigBuckBunny_175k.mov
//大白熊
```
4. 以json格式显示每个流的信息
```
ffprobe -print_format json -show_streams xizong.mp4
```
输出结果示例：
```
{
Input #0, mpegts, from 'xizong.mp4':
  Duration: 00:00:06.13, start: 10.000000, bitrate: 349 kb/s
  Program 1 
    Stream #0:0[0x101]: Video: h264 (High) ([27][0][0][0] / 0x001B), yuv420p(progressive), 640x360 [SAR 1:1 DAR 16:9], 15 fps, 15 tbr, 90k tbn, 30 tbc
    Stream #0:1[0x102]: Audio: aac (LC) ([15][0][0][0] / 0x000F), 44100 Hz, stereo, fltp, 70 kb/s
    "streams": [
        {
            "index": 0,
            "codec_name": "h264",
            "codec_long_name": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10",
            "profile": "High",
            "codec_type": "video",
            "codec_time_base": "1/30",
            "codec_tag_string": "[27][0][0][0]",
            "codec_tag": "0x001b",
            "width": 640,
            "height": 360,
            "coded_width": 640,
            "coded_height": 360,
            "has_b_frames": 2,
            "sample_aspect_ratio": "1:1",
            "display_aspect_ratio": "16:9",
            "pix_fmt": "yuv420p",
            "level": 22,
            "chroma_location": "left",
            "field_order": "progressive",
            "refs": 1,
            "is_avc": "false",
            "nal_length_size": "0",
            "id": "0x101",
            "r_frame_rate": "15/1",
            "avg_frame_rate": "15/1",
            "time_base": "1/90000",
            "start_pts": 912000,
            "start_time": "10.133333",
            "duration_ts": 540000,
            "duration": "6.000000",
            "bits_per_raw_sample": "8",
            "disposition": {
                "default": 0,
                "dub": 0,
                "original": 0,
                "comment": 0,
                "lyrics": 0,
                "karaoke": 0,
                "forced": 0,
                "hearing_impaired": 0,
                "visual_impaired": 0,
                "clean_effects": 0,
                "attached_pic": 0,
                "timed_thumbnails": 0
            }
        },
        {
            "index": 1,
            "codec_name": "aac",
            "codec_long_name": "AAC (Advanced Audio Coding)",
            "profile": "LC",
            "codec_type": "audio",
            "codec_time_base": "1/44100",
            "codec_tag_string": "[15][0][0][0]",
            "codec_tag": "0x000f",
            "sample_fmt": "fltp",
            "sample_rate": "44100",
            "channels": 2,
            "channel_layout": "stereo",
            "bits_per_sample": 0,
            "id": "0x102",
            "r_frame_rate": "0/0",
            "avg_frame_rate": "0/0",
            "time_base": "1/90000",
            "start_pts": 900000,
            "start_time": "10.000000",
            "duration_ts": 532897,
            "duration": "5.921078",
            "bit_rate": "70628",
            "disposition": {
                "default": 0,
                "dub": 0,
                "original": 0,
                "comment": 0,
                "lyrics": 0,
                "karaoke": 0,
                "forced": 0,
                "hearing_impaired": 0,
                "visual_impaired": 0,
                "clean_effects": 0,
                "attached_pic": 0,
                "timed_thumbnails": 0
            }
        }
    ]
}

```

### 4. FFmepg类库介绍
FFmepg从功能上划分为几个模块，每个模块独立为一个库，以下是各模块的简要介绍：
1. libavutil：工具函数的库，包括随机数生成器、数据结构、数学工具、核心多媒体工具等功能。
2. libavformat：包含多媒体容器格式的复用器和解复用器。
3. libavcodec：包含音视频的编解码器。
4. libavdevice：用于从许多常见的多媒体输入/输出设备获取和呈现，并支持多种输入和输出设备，包括Video4Linux2，VfW，DShow和ALSA。
5. libavfilter：包含各种媒体过滤器。
6. libswscale：执行高度优化的图像比例缩放、图像颜色空间/像素格式转换，如rgb与yuv之间转换。
7. libswresample：提供了转换音频的采样频率、声道格式或样本格式的功能。

