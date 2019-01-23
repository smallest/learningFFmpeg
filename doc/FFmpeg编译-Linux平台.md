### 目录
1. 参考
2. 编译FFmpeg
3. 编写测试代码

### 1. 参考

### 2. 编译FFmpeg
(1) 下载FFmpeg的源码 [官方地址](https://ffmpeg.org/download.html)
(2) 配置编译参数

关于FFMPEG的配置参数，我们可以通过下面命令来查看：
```
./configure --help
```
简单考虑，这里只指定以下几个参数：
```
./configure --prefix=buildout --enable-shared --disable-static --disable-doc 
```
这一步执行成功，将会生成Makefile文件。
(3) 编译
```
make
make install
```
编译生成的库和头文件将会复制到前面指定的目录中（--prefix=../buildout）

### 3. 编写测试代码
这里我们使用与[FFmpeg学习(三) Android平台FFmpeg的HelloWrold程序](https://www.jianshu.com/p/6656f9872bfa)类似的例子，打印FFmpeg支持的文件格式、编码器、过滤器的信息。

测试代码如下
```
#include <stdio.h>
#include "libavformat/avformat.h"
#include "libavcodec/avcodec.h"
#include "libavfilter/avfilter.h"

int main() {
    av_register_all();
    AVInputFormat *inputFormat = av_iformat_next(NULL);
    AVOutputFormat *outputFormat = av_oformat_next(NULL);
    while (inputFormat) {
        printf("[In][%10s]\n", inputFormat->name);
        inputFormat = inputFormat->next;
    }
    while (outputFormat) {
        printf("[Out][%10s]\n", outputFormat->name);
        outputFormat = outputFormat->next;
    }

    AVCodec *codec = av_codec_next(NULL);
    while (codec) {
        if (codec->decode) {
            printf("[Dec]");
        } else {
            printf("[Eec]");
        }
        switch (codec->type) {
            case AVMEDIA_TYPE_VIDEO:
                printf("[Video]");
                break;
            case AVMEDIA_TYPE_AUDIO:
                printf("[Audio]");
                break;
            default:
                printf("[Other]");
                break;
        }
        printf("[%10s]\n", codec->name);
        codec = codec->next;
    }

    avfilter_register_all();
    AVFilter *filter  = avfilter_next(NULL);
    while (filter) {
        printf("[F][%s]\n", filter->name);
        filter = filter->next;
    }   
  return 0;
}
```

**编译**
```
gcc -o hello_ffmpeg hello_ffmpeg.c -Iffmpeg/include -Lffmpeg/lib/ -lavformat -lavfilter -lavcodec -lswscale -lavutil -lswresample 
```
说明：这里把编译生成的FFmpeg动态库和include文件放在了测试代码同级目录的ffmpeg文件夹下。

**运行**
执行测试程序之前需要先配置一下链接库的路径，否则运行的时候会报找不到FFmpeg的动态库的错误。
```
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:ffmpeg/lib
```
配置好后再执行测试程序：
```
./hello_ffmpeg
```
输出示例如下：
```
[In][        aa]  
[In][       aac] 
[In][       ac3] 
[In][       acm] 
[In][       act] 
[In][       adf] 
[In][       adp] 
[In][       ads] 
```
