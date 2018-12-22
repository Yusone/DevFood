# RGB、YUV像素数据处理



### 1. 分离YUV420P像素数据中的Y、U、V分量

如果视频帧的宽和高分别为w和h，那么一帧YUV420P像素数据一共占用`w*h*(3/2)` Byte的数据。其中前`w*h` Byte存储Y，接着的`w*h*1/4` Byte存储U，最后`w*h*1/4` Byte存储V。

即YYYYUV存储方式。找到对应的位置解出来就ok。

下边代码运行后，将会把一张分辨率为256x256的名称为001_yuv420p.yuv的YUV420P格式的像素数据文件分离成为三个文件：

> output_420_y.y：纯Y数据，分辨率为256x256。
>
> output_420_u.y：纯U数据，分辨率为128x128。
>
> output_420_v.y：纯V数据，分辨率为128x128。

注意：本文中像素的采样位数一律为8bit。由于1Byte=8bit，所以一个像素的一个分量的采样值占用1Byte。

```c
/** 
 * Split Y, U, V planes in YUV420P file. 
 * @param url  Location of Input YUV file. 
 * @param w    Width of Input YUV file. 
 * @param h    Height of Input YUV file. 
 * @param num  Number of frames to process. 
 */ 
int simplest_yuv420_split(char *url, int w, int h,int num){  
    FILE *fp=fopen(url,"rb+");  
    FILE *fp1=fopen("output_420_y.y","wb+");  
    FILE *fp2=fopen("output_420_u.y","wb+");  
    FILE *fp3=fopen("output_420_v.y","wb+");  
  
  	//申请一个w*h*3/2大小的缓冲区，每次读入数据，然后在将对应位置分为YUV3个分量
    unsigned char *pic=(unsigned char *)malloc(w*h*3/2);  
  
    for(int i=0;i<num;i++){  
        fread(pic,1,w*h*3/2,fp);  		  //每次读取 w*h*3/2 长的数据
        fwrite(pic,1,w*h,fp1);   		  //Y: 读取的数据中，位于(0,w*h*1)的数据为Y
        fwrite(pic+w*h,1,w*h/4,fp2);  	  //U:读取的数据中，位于(w*h*1,w*h*1.25)的数据为U
        fwrite(pic+w*h*5/4,1,w*h/4,fp3);  //V:读取的数据中，位于(w*h*1.25,w*h*1.5)的数据为U
    }  
  
    free(pic);  
    fclose(fp);  
    fclose(fp1);  
    fclose(fp2);  
    fclose(fp3);  
  
    return 0;  
}  
```

调用方式：

```c
simplest_yuv420_split("001_yuv420p.yuv",256,256,1); 
```

### 2. 分离YUV444P像素数据中的Y、U、V分量

类似于分离420P，只是位置排布为YUV。不是420P的YYYYUV

### 3. 将YUV420P像素数据去掉颜色（变成灰度图）

Y分量值不变，只需要将U、V分量设置成128即可。

为什么是128？这是因为U、V是图像中的经过偏置处理的色度分量。色度分量在偏置处理前的取值范围是(-128,127)，这时候的无色对应的是“0”值。经过偏置后色度分量取值变成了(0,255)，因而此时的无色对应的就是128了。

### 4. 将YUV420P像素数据的亮度减半

将图像的每个像素的Y值取出来，分别进行除以2就可以了。

图像的每个Y值占用1 Byte，取值范围是(0,255)。

### 5. 将YUV420P像素数据的周围加上边框

图像的边框的宽度为border，本程序将距离图像边缘border范围内的像素的亮度分量Y的取值设置成了亮度最大值255。

![原图](http://img.blog.csdn.net/20160117233158395)

![加边框后](http://img.blog.csdn.net/20160117233215510)



### 6. 将RGB24格式像素数据转换为YUV420P格式像素数据

下边程序实现了RGB到YUV的转换公式：

**Y=  0.299\*R + 0.587\*G + 0.114\*B**

**U= -0.147\*R - 0.289\*G + 0.463\*B**

**V=  0.615\*R - 0.515\*G - 0.100\*B**

在转换的过程中有以下几点需要注意：

> 1. RGB24存储方式是Packed，YUV420P存储方式是Packed。
> 2. U，V在水平和垂直方向的取样数是Y的一半



```c
unsigned char clip_value(unsigned char x,unsigned char min_val,unsigned char  max_val){
	if(x>max_val){
		return max_val;
	}else if(x<min_val){
		return min_val;
	}else{
		return x;
	}
}

//RGB to YUV420
bool RGB24_TO_YUV420(unsigned char *RgbBuf,int w,int h,unsigned char *yuvBuf)
{
	unsigned char*ptrY, *ptrU, *ptrV, *ptrRGB;
	memset(yuvBuf,0,w*h*3/2);
	ptrY = yuvBuf;
	ptrU = yuvBuf + w*h;
	ptrV = ptrU + (w*h*1/4);
	unsigned char y, u, v, r, g, b;
	for (int j = 0; j<h;j++){
		ptrRGB = RgbBuf + w*j*3 ;
		for (int i = 0;i<w;i++){
			
			r = *(ptrRGB++);
			g = *(ptrRGB++);
			b = *(ptrRGB++);
			y = (unsigned char)( ( 66 * r + 129 * g +  25 * b + 128) >> 8) + 16  ;          
			u = (unsigned char)( ( -38 * r -  74 * g + 112 * b + 128) >> 8) + 128 ;          
			v = (unsigned char)( ( 112 * r -  94 * g -  18 * b + 128) >> 8) + 128 ;
			*(ptrY++) = clip_value(y,0,255);
			if (j%2==0&&i%2 ==0){
				*(ptrU++) =clip_value(u,0,255);
			}
			else{
				if (i%2==0){
				*(ptrV++) =clip_value(v,0,255);
				}
			}
		}
	}
	return true;
}

/**
 * Convert RGB24 file to YUV420P file
 * @param url_in  Location of Input RGB file.
 * @param w       Width of Input RGB file.
 * @param h       Height of Input RGB file.
 * @param num     Number of frames to process.
 * @param url_out Location of Output YUV file.
 */
int simplest_rgb24_to_yuv420(char *url_in, int w, int h,int num,char *url_out){
	FILE *fp=fopen(url_in,"rb+");
	FILE *fp1=fopen(url_out,"wb+");

	unsigned char *pic_rgb24=(unsigned char *)malloc(w*h*3);
	unsigned char *pic_yuv420=(unsigned char *)malloc(w*h*3/2);

	for(int i=0;i<num;i++){
		fread(pic_rgb24,1,w*h*3,fp);
		RGB24_TO_YUV420(pic_rgb24,w,h,pic_yuv420);
		fwrite(pic_yuv420,1,w*h*3/2,fp1);
	}

	free(pic_rgb24);
	free(pic_yuv420);
	fclose(fp);
	fclose(fp1);

	return 0;
}

```



[参考: 雷霄骅博客](http://blog.csdn.net/leixiaohua1020/article/details/50534150)





