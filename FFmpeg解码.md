# 1、解码的基本流程

#### 1.1 定义

**容器(container)**：视频文件本身被称为容器(container)，容器的类型决定了信息被存放在文件中的位置。（AVI和Quicktime就是容器的例子。）

**流(streams)**：表示“一连串的通过时间来串连的数据元素”。

**帧(frames)**：流中的数据元素。

**编码器(codec)**：每个流是由不同的编码器(codec)来编码生成的。（Divx 和 MP3 就是编解码器的例子）

**包(packets)**：从流中被读出来的一段数据，它包含了一段可以被解码成方便我们最后在应用程序中操作的原始帧的数据。从技术上讲一个包可以包含部分或者其它的数据，但是 ffmpeg 的解释器为了实现上的方便，保证了我们得到的 包 Packets 包含的是完整的frame(帧)。

#### 1.2 概括：

1. 从`video.avi` 文件中打开视频流 `video_stream` 
2. 从 `video_stream` 中读取 `packet` 到 `frame（帧）`中
3. 如果这个`frame（帧）`还不完整，跳到2
4. 对这个`frame（帧）`进行一些操作
5. 跳回到2

常规流程如下：

```c
av_register_all();

AVFormatContext *ic = avformat_alloc_context();
 
//主要负责连接媒体服务器，以及读取码流的头信息
if (avformat_open_input(&ic, url, NULL, NULL) < 0) {
    return -1;
}
 
if (avformat_find_stream_info(ic, NULL) < 0) {
    return -1;
}
 
AVPacket avpkt;
av_init_packet(&avpkt);
 
while (!abort_request) {
    int ret = av_read_frame(ic, &avpkt);//主要负责每次读取一帧数据，包括解协议和解封装
    if (ret < 0) break;
    // processing
}
 
av_free_packet(&avpkt);
```



![](https://github.com/Qiaomumer/Images/blob/master/FFMpeg解码流程.jpeg?raw=true)



#### 1.3 代码详解

```c
//具体代码查看 An_ffmpeg_and_SDL_Tutorial.c (Finder搜索)
#include <avcodec.h>
#include <avformat.h>
...

int main(int argc, const char * argv[]) {
    
    
/*===================================打开文件======================================*/
    
    av_register_all();
    /*av_register_all()
     这里注册了所有的文件格式和编解码器的库，所以它们将被自动的使用在被打开的合适格式的文件上。
     注意 你只需要调用 av_register_all() 一次，因此我们在主函数 main() 中来调用它。*/
    
    
    
    AVFormatContext *pFormatCtx;
  	//AVFormat(容器Container)的Context，保存Container信息的总控结构
    

  	// Open video file
    if(av_open_input_file(&pFormatCtx, argv[1], NULL, 0, NULL)!=0)
        return -1; // Couldn’t open file
  ♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️/*av_open_input_file()  
	 av_open_input_file(&pFormatCtx, argv[1], NULL, 0, NULL),
     我们通过第一个参数来获得文件名。
     这个函数读取文件的头部并且把信息保存到我们给的 AVFormatContext 结构体中。
     最后三个参数用来指定特殊的文件格式，缓冲大小和格式参数，但如果把它们设置为空 NULL 或者 0， libavformat 将自动检测这些参数。
     
我们来看看av_open_input_file()都做了些什么：
int av_open_input_file(AVFormatContext **ic_ptr, const char *filename,
                       AVInputFormat *fmt,
                       int buf_size,
                       AVFormatParameters *ap)
{
    ......
    if (!fmt) {
        fmt = av_probe_input_format(pd, 0);// guess format if no file can be opened 
    }
   ......
    err = av_open_input_stream(ic_ptr, pb, filename, fmt, ap);
   ......
}

这样看来，只是做了两件事情：
1). 侦测容器文件格式
2). 从容器文件获取Stream的信息

这两件事情，实际上就是调用特定文件的demuxer以分离Stream的过程:

具体流程如下:

av_open_input_file
    |
    +---->av_probe_input_format从first_iformat中遍历注册的所有demuxer以
    |     调用相应的probe函数
    |
    +---->av_open_input_stream调用指定demuxer的read_header函数以获取相关
          流的信息ic->iformat->read_header

这个函数只是检测了文件的头部，所以接着我们需要检查在文件中的流的信息*/
    
  
  
  
♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️/*av_find_stream_info()
AVFormatContext获取Stream的信息
这个函数为 pFormatCtx->streams 填充上正确的信息。*/
    if(av_find_stream_info(pFormatCtx)<0)
        return -1; // Couldn’t find stream information
    

  /*
  av_open_input_file()
  av_find_stream_info()
  当这两步执行成功后，媒体信息就已经成功保存在了 ffmpeg 相关的结构体成员变量pFormatCtx中了
  */
  
    // 打印出解析到的媒体信息
    dump_format(pFormatCtx, 0, argv[1], 0);
    /*现在 pFormatCtx->streams 仅仅是一组大小为 pFormatCtx->nb_streams 的指针，所以让我们先跳过它直到我们找到一个视频流。*/
    

    
    int i;
    AVCodecContext *pCodecCtx;
    
    // Find the first video stream
    videoStream=-1;
    for(i=0; i<pFormatCtx->nb_streams; i++)
      	//穷举所有的流，查找其中种类为CODEC_TYPE_VIDEO
        if(pFormatCtx->streams[i]->codec->codec_type==CODEC_TYPE_VIDEO) {
            videoStream=i;
            break;
        }
    
    if(videoStream==-1)
        return -1; // Didn’t find a video stream
    
    // 指向流中所使用的关于编解码器信息的指针
    pCodecCtx=pFormatCtx->streams[videoStream]->codec;
    /*流中关于编解码器的信息就是被我们叫做“编解码器上下文”(codec context)的东西。
     这里面包含了流中所使用的关于编解码器的所有信息，现在我们有了一个指向它的指针。
     但是我们必需要找到真正的编解码器并且打开它:*/
    
    
    AVCodec *pCodec;
    
    // 查找对应的解码器
    pCodec=avcodec_find_decoder(pCodecCtx->codec_id);
    if(pCodec==NULL) {
        fprintf(stderr, ”Unsupported codec!\n”);
        return -1; // Codec not found
    }
    
    // 打开编解码器
    if(avcodec_open(pCodecCtx, pCodec)<0)
        return -1; // Could not open codec
        
    /*pCodecCtx->time_base 现在已经保存了帧 率的信息。
     time_base 是一个结构体，它里面有一个分子和分母 (AVRational)。
     我们使用分数的方式来表示帧率是 因为很多编解码器使用非整数的帧率(例如 NTSC 使用 29.97fps)。*/
    
    
/*===================================保存数据======================================*/        
    /*现在我们需要找到一个地方来保存帧:*/
    AVFrame *pFrame;
    
    // Allocate video frame
    pFrame=avcodec_alloc_frame();
    
    /*因为我们准备输出保存 24 位 RGB 色的 PPM 文件，我们必需把帧的格式从原来的转换为 RGB。
     FFMPEG 将 为我们做这些转换。
     在大多数项目中(包括我们的这个)我们都想把原始的帧转换成一个特定的格式。
     让我们先 为转换来申请一帧的内存。*/
    
    // 为解码帧分配内存
    pFrameRGB=avcodec_alloc_frame();
    if(pFrameRGB==NULL)
        return -1;
    /*即使我们申请了一帧的内存，当转换的时候，我们仍然需要一个地方来放置原始的数据。
     我们使用 avpicture_get_size 来获得我们需要的大小，然后手工申请内存空间:*/
        
    uint8_t *buffer;
    int numBytes;
    
    // Determine required buffer size and allocate buffer
    numBytes=avpicture_get_size(PIX_FMT_RGB24,
                                pCodecCtx->width,
                                pCodecCtx->height);
    
    buffer=(uint8_t *)av_malloc(numBytes*sizeof(uint8_t));
    
    /*av_malloc 是 ffmpeg 的 malloc，用来实现一个简单的 malloc 的包装，这样来保证内存地址是对齐的(4 字节 对齐或者 2 字节对齐)。
     它并不能保护你不被内存泄漏，重复释放或者其它 malloc 的问题所困扰。*/
    /*现在我们使用 avpicture_fill 来把帧和我们新申请的内存来结合。
     关于 AVPicture 的结构:AVPicture 结构体是 AVFrame 结构体的子集——AVFrame 结构体的开始部分与 AVPicture 结构体是一样的。*/
    
    // Assign appropriate parts of buffer to image planes in pFrameRGB
    // Note that pFrameRGB is an AVFrame, but AVFrame is a superset
    // of AVPicture
    avpicture_fill((AVPicture *)pFrameRGB, buffer, PIX_FMT_RGB24, pCodecCtx->width, pCodecCtx->height);
    

/*===================================读取数据======================================*/ 

  /*我们将要做的是通过读取包来读取整个视频流，然后把它解码成帧，最后转换格式并且保存。*/
    int frameFinished; 
    AVPacket packet;
    i=0;
	//不停地从码流中提取出帧数据
    while(av_read_frame(pFormatCtx, &packet)>=0) {
        // 判断帧的类型，对于视频帧调用:avcodec_decode_video()
        if(packet.stream_index==videoStream) {
            avcodec_decode_video(pCodecCtx, pFrame, &frameFinished, packet.data, packet.size);
            
            // Did we get a video frame?
            if(frameFinished) {
                
                // Convert the image from its native format to RGB
                img_convert((AVPicture *)pFrameRGB, PIX_FMT_RGB24,
                            (AVPicture*)pFrame, pCodecCtx->pix_fmt,
                            pCodecCtx->width, pCodecCtx->height);
                
                // Save the frame to disk
                if(++i<=5)
                    SaveFrame(pFrameRGB, pCodecCtx->width, pCodecCtx->height, i);
            }
        }
        
        // Free the packet that was allocated by av_read_frame
        av_free_packet(&packet);
  
♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️♠️/*av_read_frame()

av_read_frame() 实际做了：
	从文件中读取packet，从Packet中解码相应的frame;
	从帧中解码;
	if(解码帧完成)
		do something();
        	
我们来看看如何获取Packet,又如何从Packet中解码frame的。
av_read_frame
    |
    +---->av_read_frame_internal
        |
        +---->av_parser_parse调用指定解码器的s->parser->parser_parse函数，
        以从raw packet中重构frame

avcodec_decode_video
    |
    +---->avctx->codec->decode调用指定Codec的解码函数
   
上面过程实际上分为了两部分：

一部分是解复用(demuxer),然后是解码(decode)
使用的分别是：
av_open_input_file()      ---->解复用
av_read_frame()           ---->解码   
avcodec_decode_video() 
*/

/*av_read_frame() 读取一个Packet并保存到 AVPacket中。后边通过av_free_packet()释放这些数据。
avcodec_decode_video() 把packet转换为frame。然而当解码一个packet的时候，可能得不到关于frame的信息。因此，当我们得到下一帧的时候，avcodec_decode_video() 为我们设置了帧结束标志 frameFinished。
最后，我们使用 img_convert() 函数来把帧从原始格式(pCodecCtx->pix_fmt)转 换成为 RGB 格式。要记住，你可以把一个 AVFrame 结构体的指针转换为 AVPicture 结构体的指针。最后，我们 把帧和高度宽度信息传递给我们的 SaveFrame 函数。
*/
    }

    /*一旦我们开始读取完视频流，我们必需清理一切:*/
    // Free the RGB image
    av_free(buffer);  /*使用 av_free 来释放我们使用 avcode_alloc_fram 和 av_malloc 来分配的内存*/
    av_free(pFrameRGB);
    // Free the YUV frame
    av_free(pFrame);
    // 解码完后，释放解码器
    avcodec_close(pCodecCtx);
    // 关闭输入文件
    av_close_input_file(pFormatCtx);
    
    return 0;
}
    
/*===================================SaveFrame======================================*/ 

   /*现在我们需要做的是让 SaveFrame 函数能把 RGB 信息写入到一个 PPM 格式的文件中。我们将生成一个简单 的 PPM 格式文件，请相信，它是可以工作的。*/
void SaveFrame(AVFrame *pFrame, int width, int height, int iFrame) { FILE *pFile;
    char szFilename[32];
    int y;
    // Open file
    sprintf(szFilename, ”frame%d.ppm”, iFrame); pFile=fopen(szFilename, ”wb”); if(pFile==NULL)
        return;
    // Write header
    fprintf(pFile, ”P6\n%d %d\n255\n”, width, height);
    // Write pixel data
    for(y=0; y<height; y++) fwrite(pFrame->data[0]+y*pFrame->linesize[0], 1, width*3, pFile);
    // Close file
    fclose(pFile);
}
/*我们做了一些标准的文件打开动作，然后写入 RGB 数据。我们一次向文件写入一行数据。
 PPM 格式文件的 是一种包含一长串的 RGB 数据的文件。
 如果你了解 HTML 色彩表示的方式，那么它就类似于把每个像素的颜色 头对头的展开，就像 #ff0000#ff0000.... 就表示了了个红色的屏幕。
 (它被保存成二进制方式并且没有分隔符，但是 你自己是知道如何分隔的)。
 文件的头部表示了图像的宽度和高度以及最大的 RGB 值的大小。*/


```

sprintf(s, "%d", 123);  //把整数123打印成一个字符串保存在s中（将格式化的数据写入字符串）



#### 1.4 FFMpeg 中断阻塞的网络线程

> avformat_open_input 主要负责连接媒体服务器，以及读取码流的头信息
>
> av_read_frame 主要负责每次读取一帧数据，包括解协议和解封装

都有可能会出现**耗时很长或者阻塞**的情况，我们需要一个中断机制，在等待超时或者退出播放的时候，就可以轻松中断掉这个阻塞过程。

ffmpeg 提供了一个很简单的回调机制，即注册一个自定义的回调函数，用于外部中断阻塞的网络操作，用法如下所示：

```c
static int custom_interrupt_callback(void *arg) {
    if (timeout || abort_request) {
        return 1;
    }
    return 0;
}
 
AVFormatContext *ic = avformat_alloc_context();
ic->interrupt_callback.callback = custom_interrupt_callback;
ic->interrupt_callback.opaque = custom_arg;
```

当自定义的回调函数返回 1，则会产生中断。

因此，我们可以在等待超时或者退出播放器的时候，**将 timeout 或者 abort_request 置为 1 来达到中断当前**的网络阻塞过程的目的。



# 2、simplest\_ffmpeg\_player基本流程

使用libavformat(解封装) 和 libavcodec(解码) 两个库完成了视频解码工作
纯净版 仅仅使用 libavcodec(解码) 就行了。见 --> pure\_only\_libavcodec

### 2.1 解码流程：

![](https://github.com/Qiaomumer/Images/blob/master/simplest_ffmpeg_player%E8%A7%A3%E7%A0%81%E6%B5%81%E7%A8%8B.jpg?raw=true)


`视频数据->YUV` 解码关键步骤如下：

    {
     AVFormatContext * pFormatContext;   存储视音频封装格式中包含的信息
     AVCodecContext  * pCodecContext;    编解码方式信息
     AVCodec * pCodec;                   解码器
     }
     
    1、av_register_all();     
                                        注册了和编解码器有关的组件：硬件加速器，解码器，编码器，Parser，Bitstream Filter。以及复用器，解复用器，协议处理器
    
    2、pFormatContext = avformat_alloc_context();  
                                         AVFormatContext 存储视音频封装格式中包含的信息
    
    3、avformat_open_input(&pFormatContext,filepath,NULL,NULL)     
                                         打开媒体过程
    
    4、avformat_find_stream_info(pFormatContext,NULL)     
                                         给每个媒体流（音频/视频）的AVStream结构体赋值 。   
                                         它其实已经实现了解码器的查找，解码器的打开，视音频帧的读取、解码等工作。
                                         换句话说，该函数实际上已经“走通”的解码的整个流程。
    
    5、pFormatCtx -> streams[i] -> codec -> codec_type
                                        pFormatCtx -> streams[i] : 视音频流
                                        pFormatCtx -> streams[i] -> codec -> codec_type : 编解码器的类型
        
    6、pCodecContext = pFormatContext -> streams[videoindex] -> codec    
                                        编解码方式信息
    
    7、pCodec = avcodec_find_decoder(pCodecContext -> codec_id);     
                                        解码器
    
    8、avcodec_open2(pCodecContext, pCodec,NULL)       
                                        初始化一个视音频编解码器的AVCodecContext
    
    9、av_read_frame(pFormatContext, packet)      
                                        读取码流中的音频若干帧或者视频一帧。输出packet
    
    10、avcodec_decode_video2(pCodecContext, pFrame, &got_picture, packet);
                                        解码一帧视频数据。
                                        输入一个压缩编码的结构体AVPacket，输出一个解码后的结构体AVFrame。
                                        若不能解码，got_picture_ptr==0。	
### 1.2 SDL显示YUV图像的流程图：

![](https://github.com/Qiaomumer/Images/blob/master/simplest_ffmpeg_player%E6%98%BE%E7%A4%BA%E6%B5%81%E7%A8%8B.jpg?raw=true)

SDL实际上是对底层绘图API（Direct3D，OpenGL）的封装，使用起来明显简单于直接调用底层API。

`YUV->显示器` 显示关键步骤如下：
​    
     [初始化]
    SDL_Init():                 初始化SDL。
    SDL_CreateWindow():         创建窗口（Window）。
    SDL_CreateRenderer():       基于窗口创建渲染器（Render）。
    SDL_CreateTexture():        创建纹理（Texture）。
    
    [循环渲染数据]
    SDL_UpdateTexture():        设置纹理的数据。
    SDL_RenderCopy():           纹理复制给渲染器。
    SDL_RenderPresent():        显示。

#  2、pure\_only\_libavcodec 基本流程

![](https://github.com/Qiaomumer/Images/blob/master/pure_only_libavcodec%E7%BA%AF%E5%87%80%E7%89%88%E8%A7%A3%E7%A0%81%E6%B5%81%E7%A8%8B.jpg?raw=true)

释义：
流程图中关键函数的作用如下所列：

    avcodec_register_all()：     注册所有的编解码器。
    avcodec_find_decoder()：     查找解码器。
    avcodec_alloc_context3()：   为AVCodecContext分配内存。
    avcodec_open2()：            打开解码器。
    avcodec_decode_video2()：    解码一帧数据。

有两个平时“不太常见”的函数：

    av_parser_init()：           初始化AVCodecParserContext。
    av_parser_parse2()：         解析获得一个Packet。


两个存储数据的结构体如下所列：

    AVFrame：                    存储一帧解码后的像素数据
    AVPacket：                   存储一帧（一般情况下）压缩编码数据

`AVCodecParser` 用于解析输入的数据流并把它分成一帧一帧的压缩编码数据。
比较形象的说法就是把长长的一段连续的数据“切割”成一段段的数据。他的核心函数是`av_parser_parse2()`。
​    
    int av_parser_parse2(AVCodecParserContext *s,  
                     AVCodecContext *avctx,  
                     uint8_t **poutbuf, int *poutbuf_size,  
                     const uint8_t *buf, int buf_size,  
                     int64_t pts, int64_t dts,  
                     int64_t pos);  
    其中,
    poutbuf指向解析后输出的压缩编码数据帧，buf指向输入的压缩编码数据。
    如果函数执行完后输出数据为空（poutbuf_size为0），则代表解析还没有完成，还需要再次调用av_parser_parse2()解析一部分数据才可以得到解析后的数据帧。
    当函数执行完后输出数据不为空的时候，代表解析完成，可以将poutbuf中的这帧数据取出来做后续处理。

# 3、对比
只使用`libavcodec`的“纯净版”视频解码器 和 使用`libavcodec`+`libavformat`的视频解码器的不同：

（1） 下列与libavformat相关的函数在“纯净版”视频解码器中都不存在。

    av_register_all()：          注册所有的编解码器，复用/解复用器等等组件。其中调用了avcodec_register_all()注册所有编解码器相关的组件。
    avformat_alloc_context()：   创建AVFormatContext结构体。
    avformat_open_input()：      打开一个输入流（文件或者网络地址）。其中会调用avformat_new_stream()创建AVStream结构体。
    avformat_new_stream()：      中会调用avcodec_alloc_context3()创建AVCodecContext结构体。
    avformat_find_stream_info()：获取媒体的信息。
    av_read_frame()：            获取媒体的一帧压缩编码数据。其中调用了av_parser_parse2()。

（2） 新增了如下几个函数。

    avcodec_register_all()：     只注册编解码器有关的组件。比如说编码器、解码器、比特流滤镜等，但是不注册复用/解复用器这些和编解码器无关的组件。
    avcodec_alloc_context3()：   创建AVCodecContext结构体。
    av_parser_init()：           初始化AVCodecParserContext结构体。
    av_parser_parse2()：         使用AVCodecParser从输入的数据流中分离出一帧一帧的压缩编码数据。

（3） 程序的流程发生了变化。
在`libavcodec`+`libavformat`的视频解码器中，使用`avformat_open_input()`和`avformat_find_stream_info()`就可以解析出输入视频的信息（例如视频的宽、高）并且赋值给相关的结构体。
因此我们在初始化的时候就可以通过读取相应的字段获取到这些信息。

在`libavcodec`“纯净”的解码器则不能这样，由于没有上述的函数，所以不能在初始化的时候获得视频的参数。
“纯净”的解码器中，可以通过`avcodec_decode_video2()`获得这些信息。
因此我们只有在成功解码第一帧之后，才能通过读取相应的字段获取到这些信息。