### 目录
1. 参考
2. FFmpeg参数说明
3. 实用视频处理命令
4. 实用音频处理命令
5. 实用字幕处理命令
6. 使用图片处理命令
7. ffplay命令

### 1. 参考
### 2. FFmepg参数说明
#### 1.1 通用选项
FFmpeg的命令语法比较简单，如下面的格式：
```
ffmpeg [global options] [input file options] -i input_file [output file
options] output_file
```
golbal options对输入和输出都有影响，中括号中的参数是可选项。

- -formats 显示可用的格式，编解码的，协议的
- -i filename输入文件
- -ss position 搜索到指定的时间[-] hh:mm:ss[.xxx]的时间格式也支持
- -an, -vn, -sn 分别代表排除音频、视频、字幕流
- -vf, -af 视频过滤器选项和音频过滤器选项

#### 1.2 视频选项
- -b bitrate设置比特率，缺省200kb/s
- -r fps 设置帧频率，缺省25
- -vcodec codec 强制使用codec编解码方式，如果用copy表示原始编码数据必须被拷贝

#### 1.3 音频选项
- -acodec codec 使用 codec 编解码
- -ab bitrate 设置音频码率
- -ar freq 设置音频采样率

#### 1.4 流选择选项
一些媒体格式如AVI，MP4等可以包含多种类型的流。
FFmpeg可以识别5种流类型： 音频audio (a), 附件attachment (t), 数据data (d), 字幕subtitle (s) 和 视频video (v)。
可以通过-map选项来选择需要的流，语法格式如下：
```
file_number:stream_type[:stream_number]
```
file_number和stream_number分别表示文件索引和流索引。
有一些特别的流选择指令：
-map 0 ：选择所有的流类型和其中所有的流。
-map i:v ：选择文件i中的所有视频流； -map i:a 选择所有的音频流；-map i:s 
 选择所有的字幕流。

### 2. 实用视频处理命令
#### 视频旋转
顺时针翻转90度
```
ffmpeg -i input -vf transpose=1 output
```
翻转180度
```
ffmpeg -i in.mp4 -vf "transpose=1,transpose=1" out.mp4
```

#### 分离视频音频流
```
ffmpeg -i input-video -vn -acodec copy output-audio //分离音频流
//-vn is no video.
//-acodec copy says use the same audio stream that's already in there.

ffmpeg -i input-video -vcodec copy -an output-video //分离视频流
//-an is no audio.
//-vcodec copy says use the same video stream that's already in there.
```

####  查看支持的格式
```
ffmpeg -codecs 
//查看支持的编码格式

ffmpeg -formats 
//查看支持的封装格式

ffmpeg -encoders
//查看支持的编码器

ffmpeg -decoders
//查看支持的解码器

ffmpeg -filters
//查看支持的过滤器

ffmpeg -pix_fmts
//查看支持的像素格式

ffmpeg -protocols
//查看支持的流媒体协议格式

ffmpeg -sample_fmts
//查看支持的音频采样格式

ffmpeg -bsfs
//查看支持的流过滤器

ffmpeg -layouts
//查看支持的音频通道布局
```

#### 在视频中加上水印
```
ffmpeg -i xizong.mp4 -i fleight.jpg -filter_complex "overlay=main_w-overlay_w-5:5" -codec:a copy xizong_fleight.mp4
//在右上角加水印，边距为5像素。
```

#### 生成视频上一个时间点的图片快照
```
ffmpeg -ss 01:23:45 -i input -vframes 1 -q:v 2 output.jpg

-i input file           the path to the input file
-ss 01:23:45            seek the position to the specified timestamp
-vframes 1              only handle one video frame
-q:v 2                  to control output quality. Full range is a linear scale of 1-31 where a lower value results in a higher quality. 2-5 is a good range to try.
output.jpg              output filename, should have a well-known extension
```

#### ffmpeg 把文件当做直播推送至服务器
```
ffmpeg -re -i jack.mp4 -c copy -f flv rtmp://host/live/test
```

#### 截取视频的一部分
```
ffmpeg -ss 5 -i input.mp4 -t 10 -c:v copy -c:a copy output.mp4
//-ss 5指定从输入视频第5秒开始截取，-t 10指明最多截取10秒。
//而-c:v copy -c:a copy标示视频与音频的编码不发生改变，而是直接复制，这样会大大提升速度，因为这样就不需要完全解码视频。
```

#### 视频格式转换
```
ffmpeg -i input.avi output.mp4 
ffmpeg -i input.mp4 output.ts
```

#### 视频解复用
```
ffmpeg -i test.mp4 -vcodec copy -an -f m4v test.264
```

#### 视频封装
```
ffmpeg -i video_file -i audio_file -vcodec copy -acodec copy output_file
```

#### 图片序列与视频的互相转换
```
ffmpeg -i %04d.jpg output.mp4
ffmpeg -i input.mp4 %04d.jpg
\\第一行命令是把0001.jpg、0002.jpg、0003.jpg等编码成output.mp4，
\\第二行则是相反把input.mp4变成0001.jpg……。
\\%04d.jpg表示从1开始用0补全的4位整数为文件名的jpg文件序列。 
```

#### ffprobe查看视频文件的信息
```
ffprobe target.mp4 -show_format -show_streams -print_format json -loglevel fatal
```

示例输出：
```
root@smallest:/home/video# ffprobe jack.mp4 -show_format -show_streams -print_format json -loglevel fatal
{
    "streams": [
        {
            "index": 0,
            "codec_name": "h264",
            "codec_long_name": "H.264 / AVC / MPEG-4 AVC / MPEG-4 part 10",
            "profile": "High",
            "codec_type": "video",
            "codec_time_base": "1/50",
            "codec_tag_string": "avc1",
            "codec_tag": "0x31637661",
            "width": 848,
            "height": 480,
            "coded_width": 848,
            "coded_height": 480,
            "has_b_frames": 1,
            "sample_aspect_ratio": "1:1",
            "display_aspect_ratio": "53:30",
            "pix_fmt": "yuv420p",
            "level": 30,
            "chroma_location": "left",
            "refs": 1,
            "is_avc": "true",
            "nal_length_size": "4",
            "r_frame_rate": "25/1",
            "avg_frame_rate": "25/1",
            "time_base": "1/25000",
            "start_pts": 0,
            "start_time": "0.000000",
            "duration_ts": 39754000,
            "duration": "1590.160000",
            "bit_rate": "449785",
            "bits_per_raw_sample": "8",
            "nb_frames": "7499",
            "disposition": {
                "default": 1,
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
            },
            "tags": {
                "creation_time": "2016-09-14T08:14:35.000000Z",
                "language": "und",
                "handler_name": "TrackHandler"
            }
        },
        {
            "index": 1,
            "codec_name": "aac",
            "codec_long_name": "AAC (Advanced Audio Coding)",
            "profile": "HE-AAC",
            "codec_type": "audio",
            "codec_time_base": "1/48000",
            "codec_tag_string": "mp4a",
            "codec_tag": "0x6134706d",
            "sample_fmt": "fltp",
            "sample_rate": "48000",
            "channels": 2,
            "channel_layout": "stereo",
            "bits_per_sample": 0,
            "r_frame_rate": "0/0",
            "avg_frame_rate": "0/0",
            "time_base": "1/24000",
            "start_pts": 0,
            "start_time": "0.000000",
            "duration_ts": 7199744,
            "duration": "299.989333",
            "bit_rate": "48030",
            "max_bit_rate": "56888",
            "nb_frames": "7031",
            "disposition": {
                "default": 1,
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
            },
            "tags": {
                "creation_time": "2016-09-14T08:11:52.000000Z",
                "language": "und",
                "handler_name": "Sound Media Handler"
            }
        }
    ],
    "format": {
        "filename": "jack.mp4",
        "nb_streams": 2,
        "nb_programs": 0,
        "format_name": "mov,mp4,m4a,3gp,3g2,mj2",
        "format_long_name": "QuickTime / MOV",
        "start_time": "0.000000",
        "duration": "299.988333",
        "size": "18813764",
        "bit_rate": "501719",
        "probe_score": 100,
        "tags": {
            "major_brand": "mp42",
            "minor_version": "0",
            "compatible_brands": "isomavc1mp42",
            "creation_time": "2016-09-14T08:14:35.000000Z"
        }
    }
}

```

#### 视频分辨率大小转换
转为720P：
```
ffmpeg -i input.wmv -s hd720 -c:v libx264 -crf 23 -c:a aac -strict -2 output.mp4
```
转为480p
```
ffmpeg -i input.mp4 -s hd480 -c:v libx264 -crf 23 -c:a aac -strict -2 output.mp4
```
####14. 裁剪视频
使用-crop选项，语法如下：
```
crop=ow[:oh[:x[:y[:keep_aspect]]]]
```
ow、oh表示裁减之后输出视频的宽和高，
x、y表示在输入视频上开始裁减的横坐标和纵坐标，
keep_aspect: 1表示保持裁剪后输出的纵横比与输入一致，0表示不保持。

裁剪输入视频的左三分之一，中间三分之一，右三分之一:
ffmpeg -i input -vf crop=iw/3:ih :0:0 output 
ffmpeg -i input -vf crop=iw/3:ih :iw/3:0 output 
ffmpeg -i input -vf crop=iw/3:ih :iw/3*2:0 output

裁剪中间一半区域：
ffmpeg -i input -vf crop=iw/2:ih/2 output

### 3. 实用音频处理命令
#### 倍速
速度减半
```
ffmpeg -i input.mp3 -af atempo=0.5 output.mp3
```
2倍速率
```
ffmpeg -i input.mp3 -af atempo=2 output.mp3
```
#### PCM原始数据和WAV格式转换
(1) wav格式转换为PCM裸流
```
ffmpeg -i xizong.wav -f f32le -ar 44100 -acodec pcm_f32le output_f32le.raw   
```
参数说明：
-f f32le … 浮点数32为小字端的采样格式
-ar 44100 … 采样频率
-ac 2 … 声道数量

(2) PCM裸流转wav格式
```
ffmpeg -f f32le -ar 44100 -ac 2 -acodec pcm_f32le -i xizong_f32le.raw xizong_f32le.wav  
```
说明需要先使用ffprobe查看wav中音频的采样格式，以上例子是使用的f32le采样格式的数据。

### 4. 实用字幕处理命令
#### 字幕文件转换
字幕文件有很多种，常见的有 .srt , .ass 文件等,下面使用FFmpeg进行相互转换。

```
//将.srt文件转换成.ass文件
ffmpeg -i subtitle.srt subtitle.ass

将.ass文件转换成.srt文件
ffmpeg -i subtitle.ass subtitle.srt
```
#### 集成字幕
```
ffmpeg -i input.mp4 -i subtitles.srt -c:s mov_text -c:v copy -c:a copy output.mp4
```

### 5. 使用图片处理命令
#### 图片分辨率修改
```
ffmpeg -i sample.jpg -s w*h out.jpg 
```
### 6. ffplay命令
#### 播放yuv视频数据
```
ffplay -f <文件格式> -pix_fmt <像素格式> -video_size <视频尺寸> <文件名>
示例：
ffplay -f rawvideo -pix_fmt yuv420p -video_size 848x480 yuv_video
```

#### 播放PCM音频数据
```
ffplay -f <格式名> -ac <声道数> -ar <采样率> <文件名>
示例：
ffplay -f f32le -ac 1 -ar 48000 pcm_audio
```
