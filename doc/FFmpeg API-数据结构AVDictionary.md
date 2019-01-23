### 目录
1. 参考
2. AVDictionary数据结构介绍
3. FFmpeg中AVDictionary的使用

### 1. 参考
- [1] [FMPEG Tips (5) 如何利用 AVDictionary 配置参数](https://zhuanlan.zhihu.com/p/24876689)
- [2] [FFmpeg/tree/release/4.1](https://github.com/FFmpeg/FFmpeg/tree/release/4.1/)

### 2. AVDictionary数据结构介绍
AVDictionary是一个健值对存储工具，类似于c++中的map，ffmpeg中有很多 API 通过它来传递参数。

AVDictionary 所在的头文件在 libavutil/dict.h，其定义如下：
```
struct AVDictionary {  
    int count;  
    AVDictionaryEntry *elems;  
};
```
其中，AVDictionaryEntry 的定义如下：
```
typedef struct AVDictionaryEntry {  
    char *key;  
    char *value;  
} AVDictionaryEntry;  
```
下面简单介绍一下用法

（1）创建一个字典
```
AVDictionary *d = NULL;
```
（2） 销毁一个字典
```
av_dict_free(&d);
```
（3）添加一对 key-value
```
av_dict_set(&d, "version", "1.0", 0);
av_dict_set_int(&d, "count", 3, 0);
```
（4） 获取 key 的值
```
AVDictionaryEntry *t = NULL;

t = av_dict_get(d, "version", NULL, AV_DICT_IGNORE_SUFFIX);
av_log(NULL, AV_LOG_DEBUG, "version: %s", t->value);

t = av_dict_get(d, "count", NULL, AV_DICT_IGNORE_SUFFIX);
av_log(NULL, AV_LOG_DEBUG, "count: %d", (int) (*t->value));
```
（5） 遍历字典
```
AVDictionaryEntry *t = NULL;
while ((t = av_dict_get(d, "", t, AV_DICT_IGNORE_SUFFIX))) {
    av_log(NULL, AV_LOG_DEBUG, "%s: %s", t->key, t->value);
}
```
### 3. FFmpeg中AVDictionary的使用
ffmpeg 中很多 API 都是靠 AVDictionary 来传递参数的，比如常用的：
```
int avformat_open_input(AVFormatContext **ps, const char *url, AVInputFormat *fmt, AVDictionary **options);
```
最后一个参数就是AVDictionary，我们可以在打开码流前指定各种参数，比如：探测时间、超时时间、最大延时、支持的协议的白名单等等，例如：
```
AVDictionary *options = NULL;
av_dict_set(&options, “probesize”, “4096", 0);
av_dict_set(&options, “max_delay”, “5000000”, 0);

AVFormatContext *ic = avformat_alloc_context();
if (avformat_open_input(&ic, url, NULL, &options) < 0) {
    LOGE("could not open source %s", url);
    return -1;
} 
```
那么，我们怎么知道 ffmpeg 的这个 API 支持哪些可配置的参数呢 ？

我们可以查看 ffmpeg 源码，比如 avformat_open_input 是结构体 AVFormatContext 提供的 API，在 libavformat/options_table.h 中定义了 AVFormatContext 所有支持的 options 选项，如下所示：[doxygen文档](https://www.ffmpeg.org/doxygen/trunk/libavformat_2options__table_8h-source.html)

同理，AVCodec 相关 API 支持的 options 选项则可以在 libavcodec/options_table.h 文件中找到，如下所示：[doxygen文档](https://www.ffmpeg.org/doxygen/3.1/libavcodec_2options__table_8h_source.html)




