### 目录
1. 参考
2. 转封装流程介绍
3. FFmpeg流程
4. 示例代码

### 1. 参考 
- [1] [FFmpeg\doc\examples\remuxing.c](https://github.com/FFmpeg/FFmpeg/blob/master/doc/examples/remuxing.c)
- [2] [最简单的基于FFMPEG的封装格式转换器（无编解码）](https://blog.csdn.net/leixiaohua1020/article/details/25422685)

### 2. 转封装流程介绍
转封装是指mp4、flv、avi等文件格式之间的转换。

本文不涉及音视频的编解码，是直接将视音频压缩码流从一种封装格式文件中获取出来然后打包成另外一种封装格式的文件。

工作原理如下图所示：
![remux流程.png](https://upload-images.jianshu.io/upload_images/8744338-980756380e719b83.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

### 3. FFmpeg流程
图中使用浅红色标出了关键的数据结构，浅蓝色标出了输出视频数据的函数。
程序包含了对两个文件的处理：读取输入文件（位于左边）和写入输出文件（位于右边）。中间使用了一个avcodec_parameters_copy()拷贝输入的AVCodecParameters 到输出的AVCodecParameters 。
![FFmpeg remuxer.png](https://upload-images.jianshu.io/upload_images/8744338-4af507abff2ede9c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

关键函数说明：
- avformat_open_input()：打开输入文件，初始化输入的AVFormatContext。
- avformat_find_stream_info() : 读取视音频数据来获取一些相关的信息。
- av_read_frame()：从输入中读取一个AVPacket。
- avformat_alloc_output_context2()：初始化输出的AVFormatContext。
- avformat_new_stream()：创建输出的AVStream。
- avcodec_parameters_copy()：拷贝输入的编码参数到输出的编码参数。
- avio_open()：打开输出文件。
- avformat_write_header()：写文件头（对于某些没有文件头的封装格式，不需要此函数。比如说MPEG2TS）。
- av_interleaved_write_frame()：将AVPacket写入文件。
- av_write_trailer()：写文件尾（对于某些没有文件头的封装格式，不需要此函数。比如说MPEG2TS）。

### 4. 示例代码
```
/*
  * reference:FFmpeg\doc\examples\remuxing.c
  * */
 #include <stdio.h>
 #include "libavutil/timestamp.h"
 #include "libavformat/avformat.h"

 static void log_packet(const AVFormatContext *fmt_ctx, const AVPacket *pkt, const char *tag) 
 {
     AVRational *time_base = &fmt_ctx->streams[pkt->stream_index]->time_base;

     printf("%s: pts:%s pts_time:%s dts:%s dts_time:%s duration:%s duration_time:%s stream_index:%d\n", 
             tag, 
             av_ts2str(pkt->pts), av_ts2timestr(pkt->pts, time_base),
             av_ts2str(pkt->dts), av_ts2timestr(pkt->dts, time_base),
             av_ts2str(pkt->duration), av_ts2timestr(pkt->duration, time_base),
             pkt->stream_index);
     
 }

 int main(int argc, char** argv) 
 {
     AVOutputFormat *ofmt = NULL;
     AVFormatContext *ifmt_ctx = NULL, *ofmt_ctx = NULL;
     AVPacket pkt;
     const char *in_filename, *out_filename;
     int ret, i;
     int stream_index = 0;
     int *stream_mapping = NULL;
     int stream_mapping_size = 0;
     if (argc < 3) {
         printf("Usage: %s <input> <output>\n"
                "API example program to remux a media file with libavformat and libavcodec.\n"
                "The output format is guessed according to the file extension.\n"
                "\n", argv[0]);
         return 1;
     }

     in_filename = argv[1];
     out_filename = argv[2];

     if ((ret = avformat_open_input(&ifmt_ctx, in_filename, 0, 0)) < 0) {
         fprintf(stderr, "Could not open input file '%s'", in_filename);
         goto end;
     }
     if ((ret = avformat_find_stream_info(ifmt_ctx, 0)) < 0) {
         fprintf(stderr, "Failed to retrieve input stream information");
         goto end;
     }

     av_dump_format(ifmt_ctx, 0, in_filename, 0);

     avformat_alloc_output_context2(&ofmt_ctx, NULL, NULL, out_filename);
     if (!ofmt_ctx) {
         fprintf(stderr, "Could not create output context\n");
         ret = AVERROR_UNKNOWN;
         goto end;
     }

     stream_mapping_size = ifmt_ctx->nb_streams;
     stream_mapping = av_mallocz_array(stream_mapping_size, sizeof(*stream_mapping));
     if (!stream_mapping) {
         ret = AVERROR(ENOMEM);
         goto end;
     }

     ofmt = ofmt_ctx->oformat;

     for (i = 0; i < ifmt_ctx->nb_streams; i++) {
         AVStream *in_stream = ifmt_ctx->streams[i];
         AVStream *out_stream = avformat_new_stream(ofmt_ctx, NULL);
         AVCodecParameters *in_codecpar = in_stream->codecpar;

         if (in_codecpar->codec_type != AVMEDIA_TYPE_AUDIO &&
             in_codecpar->codec_type != AVMEDIA_TYPE_VIDEO &&
             in_codecpar->codec_type != AVMEDIA_TYPE_SUBTITLE) {
             stream_mapping[i] = -1;
             continue;
         }

         stream_mapping[i] = stream_index++;

         if (!out_stream) {
             fprintf(stderr, "Failed allocating output stream\n");
             ret = AVERROR_UNKNOWN;
             goto end;
         }

         ret = avcodec_parameters_copy(out_stream->codecpar, in_codecpar);
         if (ret < 0) {
             fprintf(stderr, "Failed to copy codec parameters\n");
             goto end;
         }
         out_stream->codecpar->codec_tag = 0;
     }
     av_dump_format(ofmt_ctx, 0, out_filename, 1);

     if (!(ofmt->flags & AVFMT_NOFILE)) {
         ret = avio_open(&ofmt_ctx->pb, out_filename, AVIO_FLAG_WRITE);
         if (ret < 0) {
             fprintf(stderr, "Could not open output file '%s'", out_filename);
             goto end;
         }
     }

     ret = avformat_write_header(ofmt_ctx, NULL);
     if (ret < 0) {
         fprintf(stderr, "Error occurred when opening output file\n");
         goto end;
     }

     while (1) {
         AVStream *in_stream, *out_stream;

         //Get an AVPacket
         ret = av_read_frame(ifmt_ctx, &pkt);
         if (ret < 0) {
             break;
         }
         in_stream = ifmt_ctx->streams[pkt.stream_index];
         if (pkt.stream_index >= stream_mapping_size || 
             stream_mapping[pkt.stream_index] < 0) {
             av_packet_unref(&pkt);
             continue;
         }

         pkt.stream_index = stream_mapping[pkt.stream_index];
         out_stream = ofmt_ctx->streams[pkt.stream_index];

         log_packet(ifmt_ctx, &pkt, "in");

         /* copy packet */
         pkt.pts = av_rescale_q_rnd(pkt.pts, in_stream->time_base, out_stream->time_base, AV_ROUND_NEAR_INF|AV_ROUND_PASS_MINMAX);
         pkt.dts = av_rescale_q_rnd(pkt.dts, in_stream->time_base, out_stream->time_base, AV_ROUND_NEAR_INF|AV_ROUND_PASS_MINMAX);
         pkt.duration = av_rescale_q(pkt.duration, in_stream->time_base, out_stream->time_base);
         pkt.pos = -1;
         log_packet(ofmt_ctx, &pkt, "out");

         //Write
         ret = av_interleaved_write_frame(ofmt_ctx, &pkt);
         if (ret < 0) {
             fprintf(stderr, "Error muxing packet\n");
             break;
         }         
         av_packet_unref(&pkt);   
     }
     av_write_trailer(ofmt_ctx);
 end:
     avformat_close_input(&ifmt_ctx);

     /* close output */
     if (ofmt_ctx && !(ofmt->flags & AVFMT_NOFILE))
         avio_closep(&ofmt_ctx->pb);
     avformat_free_context(ofmt_ctx);
     av_freep(&stream_mapping);

     if (ret < 0 && ret != AVERROR_EOF) {
         fprintf(stderr, "Error occurred: %s\n", av_err2str(ret));
         return 1;
     }
     return 0;
 }
```




