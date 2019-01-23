### 目录
1. 参考
2. lswr功能介绍
3. lswr使用说明
4. 示例代码

### 1. 参考
- [1] [FFmpeg/Libswresample Documentation](http://ffmpeg.org/libswresample.html)
- [2] [FFmpeg/Libswresample Detailed Description](http://ffmpeg.org/doxygen/trunk/group__lswr.html#details)
- [3] [FFmpeg/doc/examples/resampling_audio.c](https://github.com/FFmpeg/FFmpeg/blob/master/doc/examples/resampling_audio.c)

### 2. lswr功能介绍
FFmpeg中重采样的功能由libswresample(后面简写为lswr)提供。
lswr提供了高度优化的转换音频的采样频率、声道格式或样本格式的功能。

功能说明：
- 采样频率转换：对音频的采样频率进行转换的处理，例如把音频从一个高的44100Hz的采样频率转换到8000Hz。从高采样频率到低采样频率的音频转换是一个有损的过程。API提供了多种的重采样选项和算法。
- 声道格式转换：对音频的声道格式进行转换的处理，例如立体声转换为单声道。当输入通道不能映射到输出流时，这个过程是有损的，因为它涉及不同的增益因素和混合。
- 样本格式转换：对音频的样本格式进行转换的处理，例如把s16的PCM数据转换为s8格式或者f32的PCM数据。此外提供了Packed和Planar包装格式之间相互转换的功能，Packed和Planar的区别见[FFmpeg中Packed和Planar的PCM数据区别](https://www.jianshu.com/p/fd43c1c82945)。


此外，还提供了一些其他音频转换的功能如拉伸和填充，通过专门的设置来启用。

### 3. lswr使用说明
重采样的处理流程：
1. 创建上下文环境：重采样过程上下文环境为SwrContext数据结构。
2. 参数设置：转换的参数设置到SwrContext中。
3. SwrContext初始化：swr_init()。
4. 分配样本数据内存空间：使用av_samples_alloc_array_and_samples、av_samples_alloc等工具函数。
5. 开启重采样转换：通过重复地调用swr_convert来完成。
6. 重采样转换完成， 释放相关资源：通过swr_free()释放SwrContext。

下面是流程的一个示意图：
![resampling_audio.png](https://upload-images.jianshu.io/upload_images/8744338-4cea483ba092483f.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

函数说明：
- swr_alloc() ：创建SwrContext对象。
- av_opt_set_*()：设置输入和输出音频的信息。
- swr_init()： 初始化SwrContext。
- av_samples_alloc_array_and_samples：根据音频格式分配相应大小的内存空间。
- av_samples_alloc：根据音频格式分配相应大小的内存空间。用于转换过程中对输出内存大小进行调整。
- swr_convert：进行重采样转换。

#### 3.1 创建上下文环境
重采样过程上下文环境为SwrContext数据结构（SwrContext的定义没有对外暴露）。

创建SwrContext的方式有两种：
1. swr_alloc() ： 创建SwrContext之后再通过AVOptions的API来设置参数。
2. swr_alloc_set_opts()：在创建SwrContext的同时设置必要的参数。

两个函数的定义如下：
```
struct SwrContext* swr_alloc()

struct SwrContext* swr_alloc_set_opts(struct SwrContext * 	s, //如果为NULL则创建一个新的SwrContext，否则对已有的SwrContext进行参数设置
                                      int64_t 	            out_ch_layout, //输出的声道格式，AV_CH_LAYOUT_*
                                      enum AVSampleFormat 	out_sample_fmt,
                                      int 	                out_sample_rate,
                                      int64_t 	            in_ch_layout,
                                      enum AVSampleFormat 	in_sample_fmt,
                                      int 	                in_sample_rate,
                                      int 	                log_offset,
                                      void * 	            log_ctx 
)		
```
#### 3.2 参数设置
参数设置的方式有两种：
1. [AVOptions的API](http://ffmpeg.org/doxygen/trunk/group__opt__set__funcs.html)
2. swr_alloc_set_opts()：如果第一个参数为NULL则创建一个新的SwrContext，否则对已有的SwrContext进行参数设置。

假定要进行如下的重采样转换：
```
“f32le格式、采样频率48kHz、5.1声道格式”的PCM数据
转换为
“s16le格式、采样频率44.1kHz、立体声格式”的PCM数据
```
swr_alloc()的使用方式如下所示：
```
SwrContext *swr = swr_alloc();
av_opt_set_channel_layout(swr, "in_channel_layout", AV_CH_LAYOUT_5POINT1, 0);
av_opt_set_channel_layou(swr, "out_channel_layout", AV_CH_LAYOUT_STEREO, 0);
av_opt_set_int(swr, "in_sample_rate", 48000, 0);
av_opt_set_int(swr, "out_sample_rate", 44100, 0);
av_opt_set_sample_fmt(swr, "in_sample_fmt", AV_SAMPLE_FMT_FLPT, 0);
av_opt_set_sample_fmt(swr, "out_sample_fmt", AV_SAMPLE_FMT_S16, 0);
```
swr_alloc_set_opts()的使用方式如下所示：
```
SwrContext *swr = swr_alloc_set_opts(NULL,  // we're allocating a new context
                      AV_CH_LAYOUT_STEREO,  // out_ch_layout
                      AV_SAMPLE_FMT_S16,    // out_sample_fmt
                      44100,                // out_sample_rate
                      AV_CH_LAYOUT_5POINT1, // in_ch_layout
                      AV_SAMPLE_FMT_FLTP,   // in_sample_fmt
                      48000,                // in_sample_rate
                      0,                    // log_offset
                      NULL);                // log_ctx
```
#### 3.3 SwrContext初始化：swr_init()
参数设置好之后必须调用swr_init()对SwrContext进行初始化。

**如果需要修改转换的参数：**
1. 重新进行参数设置。
2. 再次调用swr_init()。

#### 3.4 分配样本数据内存空间
转换之前需要分配内存空间用于保存重采样的输出数据，内存空间的大小跟通道个数、样本格式需要、容纳的样本个数都有关系。libavutil中的[samples处理API](http://ffmpeg.org/doxygen/trunk/group__lavu__sampmanip.html)提供了一些函数方便管理样本数据，例如av_samples_alloc()函数用于分配存储sample的buffer。

av_sample_alloc()的定义如下：
```
/**
 * @param[out] audio_data  输出数组，每个元素是指向一个通道的数据的指针。
 * @param[out] linesize    aligned size for audio buffer(s), may be NULL
 * @param nb_channels      通道的个数。
 * @param nb_samples       每个通道的样本个数。
 * @param align            buffer size alignment (0 = default, 1 = no alignment)
 * @return                 成功返回大于0的数，错误返回负数。
 */
int av_samples_alloc(uint8_t **audio_data, int *linesize, int nb_channels,
                     int nb_samples, enum AVSampleFormat sample_fmt, int align);
```
#### 3.5 开启重采样转换
重采样转换是通过重复地调用swr_convert()来完成的。

swr_convert()函数的定义如下：
```
 * @param out            输出缓冲区，当PCM数据为Packed包装格式时，只有out[0]会填充有数据。
 * @param out_count      每个通道可存储输出PCM数据的sample数量。
 * @param in             输入缓冲区，当PCM数据为Packed包装格式时，只有in[0]需要填充有数据。
 * @param in_count       输入PCM数据中每个通道可用的sample数量。
 *
 * @return               返回每个通道输出的sample数量，发生错误的时候返回负数。
 */
int swr_convert(struct SwrContext *s, uint8_t **out, int out_count,
                                const uint8_t **in , int in_count);
```
说明：
1. 如果没有提供足够的空间用于保存输出数据，采样数据会缓存在swr中。可以通过 swr_get_out_samples()来获取下一次调用swr_convert在给定输入样本数量下输出样本数量的上限，来提供足够的空间。
2. 如果是采样频率转换，转换完成后采样数据可能会缓存在swr中，它期待你提供更多的输入数据。
3. 如果实际上并不需要更多输入数据，通过调用swr_convert()，其中参数in_count设置为0来获取缓存在swr中的数据。
4. 转换结束之后需要冲刷swr_context的缓冲区，通过调用swr_convert()，其中参数in设置为NULL，参数in_count设置为0。


下面的代码演示了重采样转换处理的流程，其中假定依照上面的参数设置、get_input()和handle_output()已经定义好。
```
uint8_t **input;
int in_samples;
while (get_input(&input, &in_samples)) {
    uint8_t *output;
    int out_samples = av_rescale_rnd(swr_get_delay(swr, 48000) +
                                     in_samples, 44100, 48000, AV_ROUND_UP);
    av_samples_alloc(&output, NULL, 2, out_samples,
                     AV_SAMPLE_FMT_S16, 0);
    out_samples = swr_convert(swr, &output, out_samples,
                                     input, in_samples);
    handle_output(output, out_samples);
    av_freep(&output);
}
```

#### 3.6 重采样转换完成， 释放相关资源
转换结束之后，需要调用av_freep(&audio_data[0])来释放内存。

### 4. 示例代码
[3] 示例的代码。
```
/**
 * @example resampling_audio.c
 * libswresample API use example.
 */

#include <libavutil/opt.h>
#include <libavutil/channel_layout.h>
#include <libavutil/samplefmt.h>
#include <libswresample/swresample.h>

static int get_format_from_sample_fmt(const char **fmt,
                                      enum AVSampleFormat sample_fmt)
{
    int i;
    struct sample_fmt_entry {
        enum AVSampleFormat sample_fmt; const char *fmt_be, *fmt_le;
    } sample_fmt_entries[] = {
        { AV_SAMPLE_FMT_U8,  "u8",    "u8"    },
        { AV_SAMPLE_FMT_S16, "s16be", "s16le" },
        { AV_SAMPLE_FMT_S32, "s32be", "s32le" },
        { AV_SAMPLE_FMT_FLT, "f32be", "f32le" },
        { AV_SAMPLE_FMT_DBL, "f64be", "f64le" },
    };
    *fmt = NULL;

    for (i = 0; i < FF_ARRAY_ELEMS(sample_fmt_entries); i++) {
        struct sample_fmt_entry *entry = &sample_fmt_entries[i];
        if (sample_fmt == entry->sample_fmt) {
            *fmt = AV_NE(entry->fmt_be, entry->fmt_le);
            return 0;
        }
    }

    fprintf(stderr,
            "Sample format %s not supported as output format\n",
            av_get_sample_fmt_name(sample_fmt));
    return AVERROR(EINVAL);
}

/**
 * Fill dst buffer with nb_samples, generated starting from t.
 */
static void fill_samples(double *dst, int nb_samples, int nb_channels, int sample_rate, double *t)
{
    int i, j;
    double tincr = 1.0 / sample_rate, *dstp = dst;
    const double c = 2 * M_PI * 440.0;

    /* generate sin tone with 440Hz frequency and duplicated channels */
    for (i = 0; i < nb_samples; i++) {
        *dstp = sin(c * *t);
        for (j = 1; j < nb_channels; j++)
            dstp[j] = dstp[0];
        dstp += nb_channels;
        *t += tincr;
    }
}

int main(int argc, char **argv)
{
    int64_t src_ch_layout = AV_CH_LAYOUT_STEREO, dst_ch_layout = AV_CH_LAYOUT_SURROUND;
    int src_rate = 48000, dst_rate = 44100;
    uint8_t **src_data = NULL, **dst_data = NULL;
    int src_nb_channels = 0, dst_nb_channels = 0;
    int src_linesize, dst_linesize;
    int src_nb_samples = 1024, dst_nb_samples, max_dst_nb_samples;
    enum AVSampleFormat src_sample_fmt = AV_SAMPLE_FMT_DBL, dst_sample_fmt = AV_SAMPLE_FMT_S16;
    const char *dst_filename = NULL;
    FILE *dst_file;
    int dst_bufsize;
    const char *fmt;
    struct SwrContext *swr_ctx;
    double t;
    int ret;

    if (argc != 2) {
        fprintf(stderr, "Usage: %s output_file\n"
                "API example program to show how to resample an audio stream with libswresample.\n"
                "This program generates a series of audio frames, resamples them to a specified "
                "output format and rate and saves them to an output file named output_file.\n",
            argv[0]);
        exit(1);
    }
    dst_filename = argv[1];

    dst_file = fopen(dst_filename, "wb");
    if (!dst_file) {
        fprintf(stderr, "Could not open destination file %s\n", dst_filename);
        exit(1);
    }

    /* create resampler context */
    swr_ctx = swr_alloc();
    if (!swr_ctx) {
        fprintf(stderr, "Could not allocate resampler context\n");
        ret = AVERROR(ENOMEM);
        goto end;
    }

    /* set options */
    av_opt_set_int(swr_ctx, "in_channel_layout",    src_ch_layout, 0);
    av_opt_set_int(swr_ctx, "in_sample_rate",       src_rate, 0);
    av_opt_set_sample_fmt(swr_ctx, "in_sample_fmt", src_sample_fmt, 0);

    av_opt_set_int(swr_ctx, "out_channel_layout",    dst_ch_layout, 0);
    av_opt_set_int(swr_ctx, "out_sample_rate",       dst_rate, 0);
    av_opt_set_sample_fmt(swr_ctx, "out_sample_fmt", dst_sample_fmt, 0);

    /* initialize the resampling context */
    if ((ret = swr_init(swr_ctx)) < 0) {
        fprintf(stderr, "Failed to initialize the resampling context\n");
        goto end;
    }

    /* allocate source and destination samples buffers */

    src_nb_channels = av_get_channel_layout_nb_channels(src_ch_layout);
    ret = av_samples_alloc_array_and_samples(&src_data, &src_linesize, src_nb_channels,
                                             src_nb_samples, src_sample_fmt, 0);
    if (ret < 0) {
        fprintf(stderr, "Could not allocate source samples\n");
        goto end;
    }

    /* compute the number of converted samples: buffering is avoided
     * ensuring that the output buffer will contain at least all the
     * converted input samples */
    max_dst_nb_samples = dst_nb_samples =
        av_rescale_rnd(src_nb_samples, dst_rate, src_rate, AV_ROUND_UP);

    /* buffer is going to be directly written to a rawaudio file, no alignment */
    dst_nb_channels = av_get_channel_layout_nb_channels(dst_ch_layout);
    ret = av_samples_alloc_array_and_samples(&dst_data, &dst_linesize, dst_nb_channels,
                                             dst_nb_samples, dst_sample_fmt, 0);
    if (ret < 0) {
        fprintf(stderr, "Could not allocate destination samples\n");
        goto end;
    }

    t = 0;
    do {
        /* generate synthetic audio */
        fill_samples((double *)src_data[0], src_nb_samples, src_nb_channels, src_rate, &t);

        /* compute destination number of samples */
        dst_nb_samples = av_rescale_rnd(swr_get_delay(swr_ctx, src_rate) +
                                        src_nb_samples, dst_rate, src_rate, AV_ROUND_UP);
        if (dst_nb_samples > max_dst_nb_samples) {
            av_freep(&dst_data[0]);
            ret = av_samples_alloc(dst_data, &dst_linesize, dst_nb_channels,
                                   dst_nb_samples, dst_sample_fmt, 1);
            if (ret < 0)
                break;
            max_dst_nb_samples = dst_nb_samples;
        }

        /* convert to destination format */
        ret = swr_convert(swr_ctx, dst_data, dst_nb_samples, (const uint8_t **)src_data, src_nb_samples);
        if (ret < 0) {
            fprintf(stderr, "Error while converting\n");
            goto end;
        }
        dst_bufsize = av_samples_get_buffer_size(&dst_linesize, dst_nb_channels,
                                                 ret, dst_sample_fmt, 1);
        if (dst_bufsize < 0) {
            fprintf(stderr, "Could not get sample buffer size\n");
            goto end;
        }
        printf("t:%f in:%d out:%d\n", t, src_nb_samples, ret);
        fwrite(dst_data[0], 1, dst_bufsize, dst_file);
    } while (t < 10);

    if ((ret = get_format_from_sample_fmt(&fmt, dst_sample_fmt)) < 0)
        goto end;
    fprintf(stderr, "Resampling succeeded. Play the output file with the command:\n"
            "ffplay -f %s -channel_layout %"PRId64" -channels %d -ar %d %s\n",
            fmt, dst_ch_layout, dst_nb_channels, dst_rate, dst_filename);

end:
    fclose(dst_file);

    if (src_data)
        av_freep(&src_data[0]);
    av_freep(&src_data);

    if (dst_data)
        av_freep(&dst_data[0]);
    av_freep(&dst_data);

    swr_free(&swr_ctx);
    return ret < 0;
}
```


