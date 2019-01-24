### 目录
1. 参考
2. 简介
3. 初始化和释放相关的函数
4. 引用计数的内存管理-AVBuffer

### 1. 参考
- [1] [ffmpeg.org/doxygen/4.1/AVPacket](http://ffmpeg.org/doxygen/4.1/structAVPacket.html)
- [2] [FFmpeg/tree/release/4.1](https://github.com/FFmpeg/FFmpeg/tree/release/4.1/)
- [3] [雷霄骅/FFMPEG结构体分析：AVPacket](https://blog.csdn.net/leixiaohua1020/article/details/14215755)
### 2. 简介
AVPacket数据结构保存了压缩的编码数据。

用途：
- 解封装过程，它是解封装器的输出。
- 封装过程，它是封装器的输入。
- 解码过程，它是解码器的输入。
- 编码过程，它是编码器的输出。

内部存储的数据：
- 对于音频，它通常是包含一帧压缩数据。
- 对于音频，它可能包含多帧压缩数据。

是否使用引用计数来管理数据：
- AVPacket数据所有权的语义（是否使用引用计数的方式存储数据）取决于其buf字段。
- 如果设置了buf字段，则packet数据的存储空间是动态分配的并且一直有效，直到调用av_packet_unref()将引用计数减少到0。
- 如果没有设置buf字段，av_packet_ref()将进行数据的复制，而不是增加引用计数。

side data
- 编码器允许输出空的（即没有压缩数据的）packet，它只包含side data（比如需要在编码的结束更新一些流的参数的时候）。
- side data是使用av_malloc()来分配内存，使用av_packet_ref()进行复制，使用av_packet_unref()进行释放。

AVPacket的定义在libavcodec/avcodec.h，如下所示。
```
/**
 * A reference counted buffer type. It is opaque and is meant to be used through
 * references (AVBufferRef).
 */
typedef struct AVBuffer AVBuffer;

/**
 * A reference to a data buffer.
 *
 * The size of this struct is not a part of the public ABI and it is not meant
 * to be allocated directly.
 */
typedef struct AVBufferRef {
    AVBuffer *buffer;

    /**
     * The data buffer. It is considered writable if and only if
     * this is the only reference to the buffer, in which case
     * av_buffer_is_writable() returns 1.
     */
    uint8_t *data;
    /**
     * Size of data in bytes.
     */
    int      size;
} AVBufferRef;

typedef struct AVPacketSideData {
    uint8_t *data;
    int      size;
    enum AVPacketSideDataType type;
} AVPacketSideData;

/**
 * This structure stores compressed data. It is typically exported by demuxers
 * and then passed as input to decoders, or received as output from encoders and
 * then passed to muxers.
 *
 * For video, it should typically contain one compressed frame. For audio it may
 * contain several compressed frames. Encoders are allowed to output empty
 * packets, with no compressed data, containing only side data
 * (e.g. to update some stream parameters at the end of encoding).
 *
 * AVPacket is one of the few structs in FFmpeg, whose size is a part of public
 * ABI. Thus it may be allocated on stack and no new fields can be added to it
 * without libavcodec and libavformat major bump.
 *
 * The semantics of data ownership depends on the buf field.
 * If it is set, the packet data is dynamically allocated and is
 * valid indefinitely until a call to av_packet_unref() reduces the
 * reference count to 0.
 *
 * If the buf field is not set av_packet_ref() would make a copy instead
 * of increasing the reference count.
 *
 * The side data is always allocated with av_malloc(), copied by
 * av_packet_ref() and freed by av_packet_unref().
 *
 * @see av_packet_ref
 * @see av_packet_unref
 */
typedef struct AVPacket {
    /**
     * A reference to the reference-counted buffer where the packet data is
     * stored.
     * May be NULL, then the packet data is not reference-counted.
     */
    AVBufferRef *buf;
    /**
     * Presentation timestamp in AVStream->time_base units; the time at which
     * the decompressed packet will be presented to the user.
     * Can be AV_NOPTS_VALUE if it is not stored in the file.
     * pts MUST be larger or equal to dts as presentation cannot happen before
     * decompression, unless one wants to view hex dumps. Some formats misuse
     * the terms dts and pts/cts to mean something different. Such timestamps
     * must be converted to true pts/dts before they are stored in AVPacket.
     */
    int64_t pts;
    /**
     * Decompression timestamp in AVStream->time_base units; the time at which
     * the packet is decompressed.
     * Can be AV_NOPTS_VALUE if it is not stored in the file.
     */
    int64_t dts;
    uint8_t *data;
    int   size;
    int   stream_index;
    /**
     * A combination of AV_PKT_FLAG values
     */
    int   flags;
    /**
     * Additional packet data that can be provided by the container.
     * Packet can contain several types of side information.
     */
    AVPacketSideData *side_data;
    int side_data_elems;

    /**
     * Duration of this packet in AVStream->time_base units, 0 if unknown.
     * Equals next_pts - this_pts in presentation order.
     */
    int64_t duration;

    int64_t pos;                            ///< byte position in stream, -1 if unknown

#if FF_API_CONVERGENCE_DURATION
    /**
     * @deprecated Same as the duration field, but as int64_t. This was required
     * for Matroska subtitles, whose duration values could overflow when the
     * duration field was still an int.
     */
    attribute_deprecated
    int64_t convergence_duration;
#endif
} AVPacket;
```

**重要字段说明：**
- buf: 指向引用计数的buffer。如果是NULL则压缩编码数据不是以引用计数的方式存储的。buf实际上是指向AVBufferRef指针，AVBufferRef的buf字段指向实际的引用计数的buffer(AVBuffer)，AVBuffer中存储了压缩编码数据。
- data：指向存储压缩编码数据的buffer。当buf不为NULL时，data与buf->data的值是一样的。在进行视音频处理的时候，常常可以将得到的AVPacket的data数据直接写到文件，从而得到视音频的码流文件。
- size：data的大小。当buf不为NULL时，size + AV_INPUT_BUFFER_PADDING_SIZE 等于buf->size。AV_INPUT_BUFFER_PADDING_SIZE是用于解码的输入流的末尾必要的额外字节个数，需要它主要是因为一些优化的流读取器一次读取32或者64比特，可能会读取超过size大小内存的末尾。看代码知道AV_INPUT_BUFFER_PADDING_SIZE的宏定义为了64。
- pts：显示时间戳，时间的单位为AVStream->time_base，pts必须大于或者等于dts，因为显示肯定是在解码之后。
- dts：解码时间戳，时间的单位为AVStream->time_base。
- stream_index：AVPacket所属的流的索引。

### 3. 初始化和释放相关的函数
- av_packet_alloc(): 为AVPacket分配内存，不涉及存储压缩编码数据的buffer，可以使用其他方法来为buffer分配空间比如av_new_packet()。
- av_init_packet(): 使用默认值初始化一些字段，并不会触碰AVPacket的成员data和size。
- av_new_packet(): 根据指定的数据大小为AVPacket的buf分配的内存，然后调用了av_init_packet()进行初始化。
- av_packet_from_data(): 使用已经分配好的buffer初始化一个AVPacket，会设置AVPacket的data和size成员。传入的size参数是减去了AV_INPUT_BUFFER_PADDING_SIZE的，也就是size + AV_INPUT_BUFFER_PADDING_SIZE等于buffer总的大小。
- av_packet_free(): 释放AVPacket内存，如果AVPacket的数据是以引用计数方式存储的，则先解引用。

函数间的调用关系示意图如下：
![AVPacket_core_func.png](https://upload-images.jianshu.io/upload_images/8744338-1ea70245aa2e40e7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


它们的定义在libavcodec/avcodec.h中，如下所示：
```
/**
 * Allocate an AVPacket and set its fields to default values.  The resulting
 * struct must be freed using av_packet_free().
 *
 * @return An AVPacket filled with default values or NULL on failure.
 *
 * @note this only allocates the AVPacket itself, not the data buffers. Those
 * must be allocated through other means such as av_new_packet.
 *
 * @see av_new_packet
 */
AVPacket *av_packet_alloc(void);

/**
 * Initialize optional fields of a packet with default values.
 *
 * Note, this does not touch the data and size members, which have to be
 * initialized separately.
 *
 * @param pkt packet
 */
void av_init_packet(AVPacket *pkt);

/**
 * Allocate the payload of a packet and initialize its fields with
 * default values.
 *
 * @param pkt packet
 * @param size wanted payload size
 * @return 0 if OK, AVERROR_xxx otherwise
 */
int av_new_packet(AVPacket *pkt, int size);

/**
 * Initialize a reference-counted packet from av_malloc()ed data.
 *
 * @param pkt packet to be initialized. This function will set the data, size,
 *        buf and destruct fields, all others are left untouched.
 * @param data Data allocated by av_malloc() to be used as packet data. If this
 *        function returns successfully, the data is owned by the underlying AVBuffer.
 *        The caller may not access the data through other means.
 * @param size size of data in bytes, without the padding. I.e. the full buffer
 *        size is assumed to be size + AV_INPUT_BUFFER_PADDING_SIZE.
 *
 * @return 0 on success, a negative AVERROR on error
 */
int av_packet_from_data(AVPacket *pkt, uint8_t *data, int size);

/**
 * Free the packet, if the packet is reference counted, it will be
 * unreferenced first.
 *
 * @param pkt packet to be freed. The pointer will be set to NULL.
 * @note passing NULL is a no-op.
 */
void av_packet_free(AVPacket **pkt);
```

av_packet_alloc定义位于libavcodec\avpacket.c。如下所示。
```
AVPacket *av_packet_alloc(void)
{
    AVPacket *pkt = av_mallocz(sizeof(AVPacket));
    if (!pkt)
        return pkt;

    av_packet_unref(pkt);

    return pkt;
}
```

av_init_packet()的定义位于libavcodec\avcodec.c，如下所示。
```
void av_init_packet(AVPacket *pkt)
{
    pkt->pts                  = AV_NOPTS_VALUE;
    pkt->dts                  = AV_NOPTS_VALUE;
    pkt->pos                  = -1;
    pkt->duration             = 0;
#if FF_API_CONVERGENCE_DURATION
FF_DISABLE_DEPRECATION_WARNINGS
    pkt->convergence_duration = 0;
FF_ENABLE_DEPRECATION_WARNINGS
#endif
    pkt->flags                = 0;
    pkt->stream_index         = 0;
    pkt->buf                  = NULL;
    pkt->side_data            = NULL;
    pkt->side_data_elems      = 0;
}
```

av_new_packet()的声明位于libavcodec\avcodec.c，如下所示。
```
/**
 * Allocate the payload of a packet and initialize its fields with
 * default values.
 *
 * @param pkt packet
 * @param size wanted payload size
 * @return 0 if OK, AVERROR_xxx otherwise
 */
int av_new_packet(AVPacket *pkt, int size);
```
av_new_packet()的定义位于libavcodec\avpacket.c。如下所示。
```
static int packet_alloc(AVBufferRef **buf, int size)
{
    int ret;
    if (size < 0 || size >= INT_MAX - AV_INPUT_BUFFER_PADDING_SIZE)
        return AVERROR(EINVAL);

    ret = av_buffer_realloc(buf, size + AV_INPUT_BUFFER_PADDING_SIZE);
    if (ret < 0)
        return ret;

    memset((*buf)->data + size, 0, AV_INPUT_BUFFER_PADDING_SIZE);

    return 0;
}

int av_new_packet(AVPacket *pkt, int size)
{
    AVBufferRef *buf = NULL;
    int ret = packet_alloc(&buf, size);
    if (ret < 0)
        return ret;

    av_init_packet(pkt);
    pkt->buf      = buf;
    pkt->data     = buf->data;
    pkt->size     = size;

    return 0;
}

```

av_packet_from_data的定义位于libavcodec\avpacket.c。如下所示。
```
int av_packet_from_data(AVPacket *pkt, uint8_t *data, int size)
{
    if (size >= INT_MAX - AV_INPUT_BUFFER_PADDING_SIZE)
        return AVERROR(EINVAL);

    pkt->buf = av_buffer_create(data, size + AV_INPUT_BUFFER_PADDING_SIZE,
                                av_buffer_default_free, NULL, 0);
    if (!pkt->buf)
        return AVERROR(ENOMEM);

    pkt->data = data;
    pkt->size = size;

    return 0;
}
```

av_packet_free()的定义位于libavcodec\avpacket.c。如下所示。
```
void av_packet_free(AVPacket **pkt)
{
    if (!pkt || !*pkt)
        return;

    av_packet_unref(*pkt);
    av_freep(pkt);
}
```


### 4. 引用计数的内存管理-AVBuffer
AVBuffer是一个引用计数的buffer的API。

两个核心的数据结构：
- AVBuffer: 代表数据buffer本身，它对API的使用者是透明的即不是给使用方直接访问的，使用方只能通过AVBufferRef来访问。然而使用方可以进行这样的操作比如比较两个AVBuffer指针来判断两个引用是否指向相同的buffer。
- AVBufferRef: 代表一个指向AVBuffer的引用，这是调用者直接操作的对象。

创建AVBuffer的方法：
1. av_buffer_alloc()：只是用于分配一块新的buffer。
2. av_buffer_create(): 用于包装一个已有的buffer到AVBuffer中。

增j减引用：
- 增加引用：av_buffer_ref()。基于已有的引用，可以通过av_buffer_ref()创建更多的引用。
- 减少引用：av_buffer_unref()。使用来av_buffer_unref()释放一个引用，当所有的引用都释放的时候，数据会自动释放掉。

这个API和FFmpeg其余部分的约定：
1. buffer只有一个引用指向它并且不是标识为read-only的时候buffer才是可写的。
2. av_buffer_is_writable()提供了检查buffer是否可写。
3. av_buffer_make_writable()必要的时候将自动地创建一个新的可写的buffer。
3. 当然没有什么强制的限定来阻止违反这个约定的行为，但是只有按这种约定的操作才能保证安全。

说明：
1. 对buffer的引用和解引用操作是线程安全的，因此可以在多线程同时进行操作，没必要增加额外的锁保护。
2. 指向同一个buffer的不同的两个引用可能指向buffer的不同部分。比如它们的AVBufferRef.data是不一样的。

AVBufferRef的定义在libavcodec/avcodec.h，如下所示。
```
/**
 * A reference to a data buffer.
 *
 * The size of this struct is not a part of the public ABI and it is not meant
 * to be allocated directly.
 */
typedef struct AVBufferRef {
    AVBuffer *buffer;

    /**
     * The data buffer. It is considered writable if and only if
     * this is the only reference to the buffer, in which case
     * av_buffer_is_writable() returns 1.
     */
    uint8_t *data;
    /**
     * Size of data in bytes.
     */
    int      size;
} AVBufferRef;
```
AVBuffer的定义在libavutil/buffer_inner.h，如下所示。
```
struct AVBuffer {
    uint8_t *data; /**< data described by this buffer */
    int      size; /**< size of data in bytes */

    /**
     *  number of existing AVBufferRef instances referring to this buffer
     */
    atomic_uint refcount;

    /**
     * a callback for freeing the data
     */
    void (*free)(void *opaque, uint8_t *data);

    /**
     * an opaque pointer, to be used by the freeing callback
     */
    void *opaque;

    /**
     * A combination of BUFFER_FLAG_*
     */
    int flags;
};
```
av_buffer_alloc(), av_buffer_create(), av_buffer_ref(), av_buffer_unref之间的函数调用关系如下图所示：
![AVBuffer_func.png](https://upload-images.jianshu.io/upload_images/8744338-21a854f76b6e7088.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这几个函数的声明位于libavutil/buffer.h，如下所示。
```
/**
 * Allocate an AVBuffer of the given size using av_malloc().
 *
 * @return an AVBufferRef of given size or NULL when out of memory
 */
AVBufferRef *av_buffer_alloc(int size);

/**
 * Create an AVBuffer from an existing array.
 *
 * If this function is successful, data is owned by the AVBuffer. The caller may
 * only access data through the returned AVBufferRef and references derived from
 * it.
 * If this function fails, data is left untouched.
 * @param data   data array
 * @param size   size of data in bytes
 * @param free   a callback for freeing this buffer's data
 * @param opaque parameter to be got for processing or passed to free
 * @param flags  a combination of AV_BUFFER_FLAG_*
 *
 * @return an AVBufferRef referring to data on success, NULL on failure.
 */
AVBufferRef *av_buffer_create(uint8_t *data, int size,
                              void (*free)(void *opaque, uint8_t *data),
                              void *opaque, int flags);

/**
 * Create a new reference to an AVBuffer.
 *
 * @return a new AVBufferRef referring to the same AVBuffer as buf or NULL on
 * failure.
 */
AVBufferRef *av_buffer_ref(AVBufferRef *buf);

/**
 * Free a given reference and automatically free the buffer if there are no more
 * references to it.
 *
 * @param buf the reference to be freed. The pointer is set to NULL on return.
 */
void av_buffer_unref(AVBufferRef **buf);

```

av_buffer_create()的定义在libavutil/buffer.c，如下所示。
```
AVBufferRef *av_buffer_create(uint8_t *data, int size,
                              void (*free)(void *opaque, uint8_t *data),
                              void *opaque, int flags)
{
    AVBufferRef *ref = NULL;
    AVBuffer    *buf = NULL;

    buf = av_mallocz(sizeof(*buf));
    if (!buf)
        return NULL;

    buf->data     = data;
    buf->size     = size;
    buf->free     = free ? free : av_buffer_default_free;
    buf->opaque   = opaque;

    atomic_init(&buf->refcount, 1);

    if (flags & AV_BUFFER_FLAG_READONLY)
        buf->flags |= BUFFER_FLAG_READONLY;

    ref = av_mallocz(sizeof(*ref));
    if (!ref) {
        av_freep(&buf);
        return NULL;
    }

    ref->buffer = buf;
    ref->data   = data;
    ref->size   = size;

    return ref;
}
```
av_buffer_alloc()的定义在libavutil/buffer.c，如下所示。
```
AVBufferRef *av_buffer_alloc(int size)
{
    AVBufferRef *ret = NULL;
    uint8_t    *data = NULL;

    data = av_malloc(size);
    if (!data)
        return NULL;

    ret = av_buffer_create(data, size, av_buffer_default_free, NULL, 0);
    if (!ret)
        av_freep(&data);

    return ret;
}
```
av_buffer_ref()的定义在libavutil/buffer.c，如下所示。
```
AVBufferRef *av_buffer_ref(AVBufferRef *buf)
{
    AVBufferRef *ret = av_mallocz(sizeof(*ret));

    if (!ret)
        return NULL;

    *ret = *buf;

    atomic_fetch_add_explicit(&buf->buffer->refcount, 1, memory_order_relaxed);

    return ret;
}
```
av_buffer_unref()的定义在libavutil/buffer.c，如下所示。
```
static void buffer_replace(AVBufferRef **dst, AVBufferRef **src)
{
    AVBuffer *b;

    b = (*dst)->buffer;

    if (src) {
        **dst = **src;
        av_freep(src);
    } else
        av_freep(dst);

    if (atomic_fetch_add_explicit(&b->refcount, -1, memory_order_acq_rel) == 1) {
        b->free(b->opaque, b->data);
        av_freep(&b);
    }
}

void av_buffer_unref(AVBufferRef **buf)
{
    if (!buf || !*buf)
        return;

    buffer_replace(buf, NULL);
}
```







