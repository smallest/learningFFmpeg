### 目录
1. 参考

### 1. 参考
[1] [FFmpeg/tree/release/4.1](https://github.com/FFmpeg/FFmpeg/tree/release/4.1/)

待整理...
### av_log_set_callback	
函数原型：
```
void av_log_set_callback(void(*)(void *, int, const char *, va_list) callback)	

```
设置日志打印的回调。

### av_log
函数原型：
```
void av_log(void* avcl, int level, const char *fmt, ...)
```
输出日志。

### av_malloc
av_malloc()就是简单的封装了系统函数malloc()，并做了一些错误检查工作。

### avformat_alloc_output_context2()
函数原型
```
int avformat_alloc_output_context2(AVFormatContext **ctx, AVOutputFormat * oformat, const char * format_name, const char * filename)
```
初始化一个用于输出的AVFormatContext结构体

### avformat_network_init
全局地初始化网络组件，需要用到网络功能的时候需要调用。

### av_register_all
初始化libavformat和注册所有的复用器和解复用器和协议。
如果不调用这个函数，可以使用av_register_input_format()和av_register_out_format()来选择支持的格式。

### AVPacket
AVPacket是存储压缩编码数据的数据结构。
通常是解复用器的输出，然后被传递给解码器。或者是编码器的输出，然后被传递给复用器。
AVPacket.size:data的大小。
	AVPacket.dts:解码时间戳。
AVPacket.stream_index:标识该AVPacket所属的视频/音频流。

### av_copy_packet
函数原型：
```
int av_copy_packet(AVPacket * dst, const AVPacket * src)	
```
复制packet，包含内容。

### av_packet_unref
函数原型：
```
void av_packet_unref(AVPacket * pkt)	
```
解除packet引用的buffer，并且将其余的字段重置为默认值。

### AVCodecContext
这是一个描述解码器上下文的数据结构，包含了众多编解码器需要的参数信息。

### AVCodec
存储编码器信息的结构体。
主要包含以下信息：
```
const char *name：编解码器的名字的简称

const char *long_name：编解码器名字的全称

enum AVMediaType type：指明了类型，是视频，音频，还是字幕

enum AVCodecID id：ID，不重复

const AVRational *supported_framerates：支持的帧率（仅视频）

const enum AVPixelFormat *pix_fmts：支持的像素格式（仅视频）,如RGB24、YUV420P等。

const int *supported_samplerates：支持的采样率（仅音频）

const enum AVSampleFormat *sample_fmts：支持的采样格式（仅音频）

const uint64_t *channel_layouts：支持的声道数（仅音频）

int priv_data_size：私有数据的大小
```

### avcodec_send_packet
将原始分组数据作为解码器的输入。
在函数内部，会拷贝相关的AVCodecContext结构变量，将这些结构变量应用到解码的每一个包。例如
AVCodecContext.skip_frame参数通知解码器扔掉包含该帧的包。

### avcodec_alloc_context3()
创建AVCodecContext结构体。

### avcodec_parameters_to_context
将音频流信息拷贝到新的AVCodecContext结构体中。

### avcodec_free_context
释放AVCodecContext和与之相关联的所有内容，并且把指针置空。

### avcodec_open2
原型：
```
int avcodec_open2(AVCodecContext *avctx, const AVCodec *codec, AVDictionary **options)	
```
使用给定的AVCodec初始化AVCodecContext。
在使用这个函数之前需要使用avcodec_alloc_context3()分配的context。

### av_frame_alloc
原型
```
AVFrame* av_frame_alloc(void)	
```
分配一个avframe和设置字段的默认值。分配出来的AVFrame必须使用av_frame_free()释放。


### avcodec_receive_frame
原型
```
int avcodec_receive_frame(AVCodecContext *avctx, AVFrame *frame)	
```
返回解码器输出的解码数据

### av_read_pause
原型
```
int av_read_pause(AVFormatContext *s)	
```
暂停网络流（例如RSTP流），使用av_read_play()重新开始。

### av_read_play
原型
```
int av_read_play(AVFrameContext *s)
```
从当前的位置开始播放网络流（例如RSTP流）。

### av_seek_frame
原型
```
int av_seek_frame(AVFormatContext *s, int stream_index, int64_t timestamp, int flags)	
```
seek到某个时间点的关键帧。

### av_read_frame
原型
```
int av_read_frame(AVFormatContext *s, AVPacket *pkt)	
```
读取码流中的音频若干帧或者视频一帧。例如，解码视频的时候，每解码一个视频帧，需要先调用av_read_frame()获得一帧视频的压缩数据，然后才能对该数据进行解码。

### AVFormatContext
这个结构体描述了一个媒体文件或媒体流的构成和基本信息。
这是FFMpeg中最为基本的一个结构，是其他所有结构的根，是一个多媒体文件或流的根本抽象。

### avformat_alloc_context
分配一个AVFormatContext，使用avformat_free_context来释放分配出来的AVFormatContext。

### avformat_open_input
原型
```
int avformat_open_input(AVFormatContext **ps, const char *url, AVInputFormat *fmt, AVDictionary **options) 
```
打开输出的流和读取头信息。需要使用avformat_close_input关闭打开的流。

### avformat_close_input
原型
```
void avformat_close_input(AVFormatContext **s)	
```
关闭一个打开的输入AVFormatContext，释放它得很所有内容和把指针(*s)置空。

### av_dump_format
原型
```
void av_dump_format(AVFormatContext *ic, int index, const char * url, int is_output)
```
打印输入或者输出格式的详细信息，比如duration, bitrate, streams, container, programs, metadata, side data, codec and time base。

### avformat_new_stream
原型
```
AVStream* avformat_new_stream(AVFormatContext *s, const AVCodec* c)
```
添加一个stream到媒体文件中。

### avformat_write_header
原型
```
int avformat_write_header(AVFormatContext *s, AVDictionary ** options)
```
分配一个stream的私有数据而且写stream的header到一个输出的媒体文件。

### AVPixelFormat
像素格式的枚举类型，例如AV_PIX_FMT_YUV420P、AV_PIX_FMT_RGB24 

### AVMediaType
媒体类型的枚举类型，如AVMEDIA_TYPE_VIDEO(视频)、AVMEDIA_TYPE_AUDIO(音频)、AVMEDIA_TYPE_SUBTITLE(字幕)

### avformat_find_stream_info
原型
```
int avformat_find_stream_info(AVFormatContext *ic, AVDictionary **options)	
```
读取视音频数据来获取一些相关的信息。

### av_file_map
原型：
```
av_file_map(const char *filename, uint8_t **bufptr, size_t * size, int log_offset, void * log_ctx)
```
读取文件名为filename的文件，并将其内容放入新分配的缓冲区中。
如果成功，则将*bufptr设置为读缓冲区或映射缓冲区，并将*size设置为*bufptr中缓冲区的字节大小。返回的缓冲区必须使用av_file_unmap()释放。

### AVInputFormat
AVInputFormat为FFMPEG的解复用器对象。

### AVStream
该结构体描述一个媒体流。

### AVCodecParameters
该结构体描述了编码的流的属性。

### avcodec_parameters_copy
原型
```
int avcodec_parameters_copy(AVCodecParameter *dst, const AVCodecParameter* src)
```
把src中的内容拷贝到dst中。

### AVRational
这个结构标识一个分数，num为分数，den为分母。

### AVCodecID
解码器标识ID的枚举类型。

### swr_alloc
原型
```
struct SwrContext* swr_alloc(void)	
```
分配一个SwrContext，如果你使用这个函数，需要在调用swr_init()之前设置SwrContext的参数（手工的或者调用swr_alloc_set_opts()）

### SwrContext
libswresample 的上下文信息。
不像libavcodec和libavformat，这个结构是不透明的，如果你需要设置选项，你必须使用AVOptions而不能直接给这个结构的成员赋值。

### swr_alloc_set_opts
原型
```
struct SwrContext* swr_alloc_set_opts(struct SwrContext *s,
int64_t out_ch_layout,
enum AVSampleFormat out_sample_fmt,
int out_sample_rate,
int64_t in_ch_layout,
enum AVSampleFormat in_sample_fmt,
int in_sample_rate,
int log_offset,
void *log_ctx 
)	
```
设置通用的参数，如果SwrContext为空则分配一个SwrContext。

### swr_init
原型
```
init swr_init(struct SwrContext *s)
```
在参数设置好以后初始化context。

### swr_free
原型
```
void swr_free(struct SwrContext ** 	s)	
```
释放给定的SwrContext，并且把指针置为空。

### sws_getContext
原型
```

struct SwsContext* sws_getContext	(	int 	srcW,
int 	srcH,
enum AVPixelFormat 	srcFormat,
int 	dstW,
int 	dstH,
enum AVPixelFormat 	dstFormat,
int 	flags,
SwsFilter * 	srcFilter,
SwsFilter * 	dstFilter,
const double * 	param 
)	
```
分配和返回一个SwsContext。

### avio_open
原型
```
int avio_open(AVIOContext **s, const char* filename, int flags)
```
创建和初始化一个AVIOContext用于访问filename指示的资源。

### avio_closep
原型
```
int avio_closep(AVIOContext **s)
```
关闭AVIOContext** s打开的资源，释放它并且把指针置为空。
### AV_ROUND
```
AV_ROUND_ZERO     = 0, // Round toward zero.      趋近于0  
AV_ROUND_INF      = 1, // Round away from zero.   趋远于0  
AV_ROUND_DOWN     = 2, // Round toward -infinity. 趋于更小的整数  
AV_ROUND_UP       = 3, // Round toward +infinity. 趋于更大的整数  
AV_ROUND_NEAR_INF = 5, // Round to nearest and halfway cases away from zero.  
                       //                         四舍五入,小于0.5取值趋向0,大于0.5取值趋远于0  
```

### avio_alloc_context
原型
```
AVIOContext* avio_alloc_context(unsigned char* buffer,
                                                      int                     buffer_size,
                                                      int                     write_flag,
                                                      void*                 opaque,
                int(*)(void *opaque, uint8_t *buf, int buf_size) read_packet,
                int(*)(void *opaque, uint8_t *buf, int buf_size) write_packet,
                int64_t(*)(void *opaque, int64_t offset, int whence) seek)
```
分配和初始化一个AVIOContext用于缓冲的I/O。之后需要使用avio_context_free()释放。

### av_file_unmap
原型
```
void av_file_unmap(uint8_t *bufptr, size_t size)
```
取消映射或释放av_file_map()创建的缓冲区bufptr。





