# 1、编码  Simplest\_FFmpeg\_Video\_Encoder
#### YUV像素数据 --> 编码 --> 视频码流（H264，MPEG2，VP8等等）

下面是用FFmpeg编码视频的流程图。
使用该流程，不仅可以编码H.264的视频，而且可以编码MPEG4/MPEG2/VP8等等各种FFmpeg支持的视频。
图中蓝色背景的函数是实际输出数据的函数。浅绿色的函数是视频编码的函数。

![](https://github.com/Qiaomumer/Images/blob/master/%E8%A7%86%E9%A2%91%E7%BC%96%E7%A0%81.jpg?raw=true)

流程中各个函数的意义：

    av_register_all()：                      注册FFmpeg所有编解码器。

    avformat_alloc_output_context2()：       初始化输出码流的AVFormatContext。

    avio_open()：                            打开输出文件。

    av_new_stream()：                        创建输出码流的AVStream。

    avcodec_find_encoder()：                 查找编码器。

    avcodec_open2()：                        打开编码器。

    avformat_write_header()：                写文件头（对于某些没有文件头的封装格式，不需要此函数。比如说MPEG2TS）。

    avcodec_encode_video2()：                编码一帧视频。
                                             即将AVFrame(存储YUV像素数据) 编码为AVPacket(存储H.264的码流数据)。
    
    av_write_frame()：                       将编码后的视频码流写入文件。
    
    flush_encoder()：                        输入的像素数据读取完成后调用此函数。用于输出编码器中剩余的AVPacket。
    
    av_write_trailer()：                     写文件尾（对于某些没有文件头的封装格式，不需要此函数。比如说MPEG2TS）。


# 2、纯净版 只 `libavcodec`
仅使用libavcodec（不使用libavformat）编码视频的流程如下图所示。

![](https://github.com/Qiaomumer/Images/blob/master/encode_pure_only_libavcodec.png?raw=true)

流程图中关键函数的作用如下所列：

    avcodec_register_all()：     注册所有的编解码器。
    avcodec_find_encoder()：     查找编码器。
    avcodec_alloc_context3()：   为AVCodecContext分配内存。
    avcodec_open2()：            打开编码器。
    avcodec_encode_video2()：    编码一帧数据。


两个存储数据的结构体如下所列：

    AVFrame：                    存储一帧未编码的像素数据。
    AVPacket：                   存储一帧压缩编码数据。


# 3、对比

只使用`libavcodec`的“纯净版”视频编码器和使用`libavcodec`+`libavformat`的视频编码器的不同。

（1）	下列与libavformat相关的函数在“纯净版”视频编码器中都不存在。

    av_register_all()：              注册所有的编解码器，复用/解复用器等等组件。
                                     其中调用了avcodec_register_all()注册所有编解码器相关的组件。
    
    avformat_alloc_context()：       创建AVFormatContext结构体。
    
    avformat_alloc_output_context2()：初始化一个输出流。
    
    avio_open()：                    打开输出文件。
    
    avformat_new_stream()：          创建AVStream结构体。
                                     avformat_new_stream()中会调用avcodec_alloc_context3()创建AVCodecContext结构体。
    
    avformat_write_header()：        写文件头。
    
    av_write_frame()：               写编码后的文件帧。
    
    av_write_trailer()：             写文件尾。

（2）	新增了如下几个函数

    avcodec_register_all()：         只注册编解码器有关的组件。

    avcodec_alloc_context3()：       创建AVCodecContext结构体。

可以看出，相比于“完整”的编码器，这个纯净的编码器函数调用更加简单，功能相对少一些，相对来说更加的“轻量”。