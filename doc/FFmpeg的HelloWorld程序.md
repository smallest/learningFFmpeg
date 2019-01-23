### 目录
1. 参考
2. 程序的流程简介
3. 编译FFmepg类库
4. 编写FFmpeg调用代码
5. 编写Android.mk
6. 运行输出

### 1. 参考
- [1] [雷霄骅/最简单的基于FFmpeg的移动端例子：Android HelloWorld] (https://blog.csdn.net/leixiaohua1020/article/details/47008825)

### 2. 程序的流程简介
Android应用使用FFmpeg类库的流程图如下所示:
![FFmpegInAndroid.png](https://upload-images.jianshu.io/upload_images/8744338-6e9b184c59161b78.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


上图的流程可以分为“编译FFmpeg类库”、“编写Java代码”、“编写native代码”、“编写Android.mk”。

### 3. 编译FFmepg类库
参照：[FFmpeg编译-Android平台](https://www.jianshu.com/p/366a4927a532)

### 4. 编写FFmepg调用代码
编写获取FFmpeg支持信息的方法：
```
//FFmpeger.h
#ifndef HELLOJNI_FFMPEGER_H
#define HELLOJNI_FFMPEGER_H
class FFmpeger {
public:
    /**
    * FFmpeg类库支持的封装格式
    */
    static void avformat_info(char *);

    /**
    * FFmpeg类库支持的编解码器
    */
    static void avcodec_info(char *);

    /**
    * FFmpeg类库支持的滤镜
    */
    static void avfilter_info(char *);

    /**
    * FFmpeg类库的配置信息
    */
    static void avcodec_config(char *);
};
#endif //HELLOJNI_FFMPEGER_H

```

```
//FFmpeger.cpp
#include "FFmpeger.h"
extern "C"
{
#include "libavcodec/avcodec.h"
#include "libavformat/avformat.h"
#include "libavfilter/avfilter.h"
}


void FFmpeger::avformat_info(char *info) {
    av_register_all();
    AVInputFormat *avInputFormat = av_iformat_next(NULL);
    AVOutputFormat *avOutputFormat = av_oformat_next(NULL);
    while(avInputFormat != NULL) {
    	sprintf(info, "%s[In ][%10s]\n", info, avInputFormat->name);
    	avInputFormat = avInputFormat->next;
    }
    while(avOutputFormat != NULL) {
    	sprintf(info, "%s[Out][%10s]\n", info, avOutputFormat->name);
    	avOutputFormat = avOutputFormat->next;
    }
}

void FFmpeger::avcodec_info(char *info) {
    av_register_all();
    AVCodec *c_temp = av_codec_next(NULL);
    while(c_temp != NULL){
       if(c_temp->decode!=NULL){
          sprintf(info,"%s[Dec]",info);
       }else{
          sprintf(info,"%s[Enc]",info);
       }
       switch(c_temp->type){
        case AVMEDIA_TYPE_VIDEO:
          sprintf(info,"%s[Video]",info);
          break;
        case AVMEDIA_TYPE_AUDIO:
          sprintf(info,"%s[Audio]",info);
          break;
        default:
          sprintf(info,"%s[Other]",info);
          break;
       }
       sprintf(info,"%s[%10s]\n",info,c_temp->name);
       c_temp=c_temp->next;
    }
}

void FFmpeger::avfilter_info(char *info) {
	avfilter_register_all();
	AVFilter *avFilter = (AVFilter *) avfilter_next(NULL);
	while (avFilter != NULL) {
		sprintf(info, "%s[%10s]\n", info, avFilter->name);
		avFilter = avFilter->next;
	}
}

void FFmpeger::avcodec_config(char *info) {
	av_register_all();
	sprintf(info, "%s[%10s]\n", avcodec_configuration());
}

```
#### 使用的方法和数据结构说明
- AVInputFormat,AVOutputFormat：描述支持的输入/输出格式的结构体。
- AVCodec：存储编解码器信息的结构体。
- AVFilter：过滤器信息结构体。
- av_register_all()：初始化所有组件，只有调用了该函数，才能使用复用器和编解码器。
- av_iformat_next，av_oformat_next()：用来递归获取所有已注册的输入/输出格式。
- av_codec_next：获取指向链表下一个解码器的指针，如果想要获得指向第一个解码器的指针，则将该函数的参数设置为NULL。
- avfilter_register_all：注册所有内置AVFilter。
- avcodec_configuration：获取FFmpeg类库的配置信息。

```
//MainActivity.java
package com.smallest.test.hellojni;

import android.support.v7.app.AppCompatActivity;
import android.os.Bundle;
import android.widget.TextView;
import android.text.method.ScrollingMovementMethod;

public class MainActivity extends AppCompatActivity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

    // Example of a call to a native method
    TextView tv = (TextView) findViewById(R.id.sample_text);
    tv.setText(stringFromJNI());
    tv.setMovementMethod(ScrollingMovementMethod.getInstance());
    }

    /**
     * A native method that is implemented by the 'native-lib' native library,
     * which is packaged with this application.
     */
    public native String stringFromJNI();

    // Used to load the 'native-lib' library on application startup.
    static {
        System.loadLibrary("avutil");
        System.loadLibrary("swresample");
        System.loadLibrary("avcodec");
        System.loadLibrary("avformat");
        System.loadLibrary("swscale");
        System.loadLibrary("avfilter");
        System.loadLibrary("avdevice");
        System.loadLibrary("hello-jni");
    }
}
```

JNI层的调用代码：
```
//
#include <string.h>
#include <jni.h>
#include <android/log.h>
#include "com_smallest_test_hellojni_MainActivity.h"
#include "FFmpeger.h"
#include <stdio.h>

JNIEXPORT jstring JNICALL Java_com_smallest_test_hellojni_MainActivity_stringFromJNI(JNIEnv *env, jobject thiz)
{
  char info[60000] = {0};
  FFmpeger::avformat_info(info);
  __android_log_print(ANDROID_LOG_INFO,"myTag","avformatInfo:\n%s", info);
  char info2[60000] = {0};
  FFmpeger::avcodec_info(info2);
  __android_log_print(ANDROID_LOG_INFO,"myTag","avcodecInfo:\n%s", info2);
  char info3[60000] = {0};
  FFmpeger::avfilter_info(info3);
  __android_log_print(ANDROID_LOG_INFO,"myTag","avfilterInfo:\n%s", info3);
  char info4[60000] = {0};
  FFmpeger::avcodec_config(info4);
  __android_log_print(ANDROID_LOG_INFO,"myTag","avcodecConfiguration:\n%s", info4);
  char infoTotal[300000] = {0};
  sprintf(infoTotal, "avformatInfo:\n%s avcodecInfo:\n%s avfilterInfo:\n%s avcodecConfiguration:\n%s", info, info2, info3, info4);
  return (*env).NewStringUTF(infoTotal);
}
```

### 5. 编写Android.mk文件

```
LOCAL_PATH := $(call my-dir)

include $(CLEAR_VARS)
#FFmpeg 库的路径
FFMPEG_LIB_PATH:=$(LOCAL_PATH)/../libs

include $(CLEAR_VARS)
LOCAL_MODULE:= libavcodec
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavcodec.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE:= libavformat
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavformat.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE:= libswscale
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libswscale.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE:= libavutil
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavutil.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE:= libavfilter
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavfilter.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE:= libswresample
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libswresample.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE:= libavdevice
LOCAL_SRC_FILES:= $(FFMPEG_LIB_PATH)/libavdevice.so
include $(PREBUILT_SHARED_LIBRARY)

include $(CLEAR_VARS)
LOCAL_MODULE    := hello-jni
LOCAL_C_INCLUDES := $(LOCAL_PATH)/../include
LOCAL_SRC_FILES := $(wildcard $(LOCAL_PATH)/../src/*.cpp)
LOCAL_LDLIBS := -llog
LOCAL_SHARED_LIBRARIES := libavcodec libavfilter libavformat libavutil libswresample libswscale libavdevice
include $(BUILD_SHARED_LIBRARY)
```

Application.mk如下：
```
APP_BUILD_SCRIPT := Android.mk
APP_ABI := armeabi
```

### 6. 运行输出
![ffmpeg_hh.jpg](https://upload-images.jianshu.io/upload_images/8744338-785bd974172641a6.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
