---
title: ffmpeg
date: 2018-12-18 22:35:03
tags:
---
### 1.使用语法
#### 1.1 语法

- 解释:
> mp4中的h264编码，而h264有两种封装：
> 一种是annexb模式，传统模式，有startcode，SPS和PPS是在ES中；另一种是mp4模式，一般mp4、mkv、avi会没有startcode，SPS和PPS以及其它信息被封装在container中，每一个frame前面是这个frame的长度，很多解码器只支持annexb这种模式，因此需要将mp4做转换；在ffmpeg中用h264_mp4toannexb_filter可以做转换；所以需要使用-bsf h264_mp4toannexb来进行转换；

```php
命令格式：
    ffmpeg -i [输入文件名] [参数选项] -f [格式] [输出文件]
    ffmpeg [[options][`-i' input_file]]... {[options] output_file}...
    1、格式：
    (1) h264: 表示输出的是h264的视频裸流
    (2) mp4: 表示输出的是mp4的视频
    (3)mpegts: 表示ts视频流
    如果没有输入文件，那么视音频捕捉（只在Linux下有效，因为Linux下把音视频设备当作文件句柄来处理）就会起作用。作为通用的规则，选项一般用于下一个特定的文件。如果你给 –b 64选项，改选会设置下一个视频速率。对于原始输入文件，格式选项可能是需要的。缺省情况下，ffmpeg试图尽可能的无损转换，采用与输入同样的音频视频参数来输出。
    
    2. 视频选项
    (1) -b:v/b bitrate: 视频比特率(建议用-b:v)
    (2) -vn: 视频过滤
    (3) -vcodec/c:v codec : 强制使用视频编解码器,copy为使用原编解码器
    (4) -s size(1920*1080): 设置像素大小
    (5) -pix_fmt format(网络一般采用yuv420p这个视频空间): 设置像素格式(用ffmpeg -pix_fmts查看支持,如YUV,RGB,NV等),如果无法选择选定的像素格式，则将打印警告并选择编码器支持的最佳像素格式.
    (6) -r rate(24fps): 设置视频帧率
    
    3. 音频选项
    (1) -b:a/ab bitrate(320k): 音频比特率(不指定默认为128k)[建议用-b:a]
    (2) -ac channels(2): 设置音频声道(默认与源相同,1:单声道,2:双声道,立体声)
    (3) -ar rate(44100/48000): 设置音频采样率(如果不指定,默认为原音频的采样率)
    (4) -an: 音频过滤
    (5) -acodec/c:a codec : 强制使用音频编解码器,copy为使用原编解码器

    4.公用选线
    (1) -f fmt (input/output): 强制输入或输出文件格式,通常会自动检测输入文件的格式，并从输出文件的文件扩展名中猜出格式，因此在大多数情况下不需要此选项。 
```
#### 1.2 例子
```php
比如一个avi文件，想转为mp4，或者一个mp4想转为ts。 
ffmpeg -i input.avi output.mp4 
ffmpeg -i input.mp4 output.ts

//提取音频
ffmpeg -i test.mp4 -acodec copy -vn output.aac 
ffmpeg -i test.mp4 -acodec aac -vn output.aac
ffmpeg -i test.mp4 -acodec aac -acodec copy -vn output.aac
上面的命令(三个等效)，默认mp4的audio codec是aac,如果不是，可以都转为最常见的aac。 

//提取视频(无声音)
ffmpeg -i test.mp4 -vcodec copy -an output.mp4


```

#### 1.3 常用封装及编码
MP4封装: H264视频编码 + AAC音频编码
WebM封装: VP8视频编码 + Vorbis音频编码(Google方案,youtube正在使用)
OGG封装: Theora视频编码 + Vorbis音频编码(开源)

#### 1.4概念
```php
//视音频技术主要包括以下几点:
封装技术 + 视频压缩编码技术 + 音频压缩编码技术 + 流媒体协议技术

//封装格式
avi,flv,mp4,mkv,rmvb等
//视频编码
h264(AVC)
//音频编码
aac
```

#### 1.5 资源
[mediaInfo](https://mediaarea.net/en/MediaInfo/Download/Windows)

#### 1.6.1 视频播放器播放视频步骤
![](https://img-blog.csdn.net/20140201120523046?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQvbGVpeGlhb2h1YTEwMjA=/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/SouthEast)
1. 解协议(如果播放本地文件则不需要解协议)
采用rtmp协议传输的数据,经过解协议后,输出flv格式的数据
2. 解封装
作用是讲输入的封装格式的数据,分离成为音频压缩编码数据和视频压缩编码数据,封装格式有很多种,如TS,MP4,MKV,RMVB,FLV,AVI等,他的作用就是将压缩编码的音频数据和视频数据按照一定的格式封装到一起,例如flv格式数据,经过解封装后,输出H.264编码的视频码流和AAC编码的音频码流
3. 解码音视频
作用是将压缩编码的视音频数据,解码成为非压缩的视音频原始数据,音频压缩编码标准包含AAC,MP3,AC-3等,视频压缩标准则包含H.264,MPEG2,VC-1等,通过解码,压缩的视频数据输出成为非压缩的颜色数据,例如YUV420P,RGB等等;压缩的音频数据输出成为非压缩的音频抽样数据,例如PCM数据
4. 音视频同步
同步解码出来的视音频数据,并将视音频数据送至显卡和声卡播放出来

### 2. 通用选项
#### 2.1 选项
```php
-h: 查看帮助

-y: 覆盖输出文件(如果文件存在不提示是否覆盖)
-ss position: 搜索到指定的时间 [-]hh:mm:ss[.xxx]的格式也支持
-t duration: 设置纪录时间 hh:mm:ss[.xxx]格式的记录时间也支持
```
#### 2.2 例子
```php
下面的命令，可以从时间为00:00:15开始，截取5秒钟的视频。 
ffmpeg -y -ss 00:00:15 -t 00:00:05 -i input.mp4 -vcodec copy -acodec copy output.mp4 
ffmpeg -ss 00:00:15 -y -t 00:00:05 -i input.mp4 -vcodec copy -an output.mp4 //忽略音频
-ss表示开始切割的时间，-t表示要切多少。上面就是从15秒开始，切5秒钟出来。
```

### 3. 码率控制
```php
码率控制对于在线视频比较重要。因为在线视频需要考虑其能提供的带宽。

那么，什么是码率？很简单： 
bitrate = file size / duration 
比如一个文件20.8M，时长1分钟，那么，码率就是： 
biterate = 20.8M bit/60s = 20.8*1024*1024*8 bit/60s= 2831Kbps 
一般音频的码率只有固定几种，比如是128Kbps， 
那么，video的就是 
video biterate = 2831Kbps -128Kbps = 2703Kbps。

那么ffmpeg如何控制码率。 
ffmpg控制码率有3种选择，-minrate -b:v -maxrate 
-b:v主要是控制平均码率。 
比如一个视频源的码率太高了，有10Mbps，文件太大，想把文件弄小一点，但是又不破坏分辨率。 
ffmpeg -i input.mp4 -b:v 2000k output.mp4 
上面把码率从原码率转成2Mbps码率，这样其实也间接让文件变小了。目测接近一半。 
不过，ffmpeg官方wiki比较建议，设置b:v时，同时加上 -bufsize 
-bufsize 用于设置码率控制缓冲器的大小，设置的好处是，让整体的码率更趋近于希望的值，减少波动。（简单来说，比如1 2的平均值是1.5， 1.49 1.51 也是1.5, 当然是第二种比较好） 
ffmpeg -i input.mp4 -b:v 2000k -bufsize 2000k output.mp4

-minrate -maxrate就简单了，在线视频有时候，希望码率波动，不要超过一个阈值，可以设置maxrate。 
ffmpeg -i input.mp4 -b:v 2000k -bufsize 2000k -maxrate 2500k output.mp4
```

### 4. 视频格式转化
#### 4.1 常规
```php
比如一个视频的编码是MPEG4，想用H264编码，咋办？ 
ffmpeg -i input.mp4 -vcodec[或-c:v] h264 output.mp4 
相反也一样 
ffmpeg -i input.mp4 -vcodec[或-c:v] mpeg4 output.mp4

当然了，如果ffmpeg当时编译时，添加了外部的x265或者X264，那也可以用外部的编码器来编码。（不知道什么是X265，可以 Google一下，简单的说，就是她不包含在ffmpeg的源码里，是独立的一个开源代码，用于编码HEVC，ffmpeg编码时可以调用它。当然 了，ffmpeg自己也有编码器） 
ffmpeg -i input.mp4 -c:v libx265 output.mp4 
ffmpeg -i input.mp4 -c:v libx264 output.mp4
```

#### 4.2 mp4与m3u8互转
- MP4存在问题
```php
1.当视频时长比较长的时候，mp4的关键帧元素往往很大，需要加载很长时间才能开始播放，
  网速不好的情况缓冲加载就要20多秒的时间，客户早已急不可耐。
    
2.当用户打开一个视频播放的时候，浏览器会持续请求下载mp4文件直到下载完成，就算是
  用户暂停视频播放浏览器也会持续这种下载状态，如果这个视频文件是500M则会请求服
  务器下载500M文件，是1G则会不停下载1G，给服务器硬盘和宽带造成很大浪费和压力。

```
- m3u8优势
```php
m3u8是苹果公司开发的一项新型播放格式，这种播放格式支持目前市面的windows、androis、
ios设备主流的浏览器，同样的视频文件既可以在flash环境播放，又能在无flash的html5环境
播放，它的优势还不止于此，它可以实现多种码率在不同网速下的自动切换，网速好自动切换高清
晰度视频，网速慢自动播放低清晰度文件，还可以实现流加密（视频文件本身加密）、分段下载播
放、任意时间点拖拽播放、随机视频文件广告插入等等优势
```
- mp4转m3u8

```php
例一:

//如果视频不为mp4格式，需先将视频转码为mp4，可使用如下命令进行转换
ffmpeg -i 本地视频地址 -y -c:v libx264 -strict -2 转换视频.mp4
//将mp4格式转换为ts格式(-vbsf:流过滤器的音频过滤选项)
ffmpeg -y -i 本地视频.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 转换视频.ts
//将ts文件进行切片
ffmpeg -i 本地视频.ts -c copy -map 0 -f segment -segment_list 视频索引.m3u8 -segment_time 5 前缀-%03d.ts
//其中segment 就是切片，-segment_time表示隔几秒进行切一个文件，上面命令是隔5s，你也可以调整成更大的参数。
```

```php
例子二:

ffmpeg -i "翟天临.mp4" -vcodec libx264 -ss 0 -t 00:03:30.46 -pix_fmt yuv420p -b:v 1.5M -r 25 -bufsize 1M -keyint_min 50 -g 50  -ac 2 -ab 128k -vf scale="trunc(oh*a/2)*2:min(720\,ih)" -bsf:v h264_mp4toannexb -hls_time 5 -max_muxing_queue_size 9999 -hls_list_size 0 -hls_segment_filename haha/v-%5d.ts haha/haha.m3u8

解释:
ffmpeg -i "$inputFile" 
-vcodec libx264                                                 //设置编码器
-ss 0 -t $maxTime                                               //转码时间
-pix_fmt yuv420p                                                //设置像素格式
-b:v 1.5M                                                       //比特率
-r 25                                                           //帧率
-bufsize 1M                                                     //码率控制缓冲器大小(一般与视频码率一致)
-keyint_min 50 -g 50                                            //gop设置
-ac 2                                                           //双声道
-ab 128k                                                        //音频比特率
-vf scale="trunc(oh*a/2)*2:min(720\,ih)"                        //视频过滤器
-bsf:v h264_mp4toannexb                                         //流的比特流过滤器,用这个选项的原因见最上方解释,用-bsfs来查看所有流过滤器
-hls_time $hls_time                                             //设置目标段(target segment)的长度(以此时间剪切),默认2秒
-max_muxing_queue_size 9999                                     //当转码音频和/或视频流时，ffmpeg将不会开始写入输出，直到它为每个这样的流有一个数据包,应设置足够大
-hls_list_size 0                                                //设置播放列表的最大数量,如果设置为0,则列表文件包含所有段,默认5
-hls_segment_filename $outpath/v-%5d.ts                         //设置段文件名
$outputFile 
> /data0/service/mts2/logs/sh_$jobId.log 2>&1                   //标准输出,标准错误均到日志
```

- m3u8转mp4
```php
ffmpeg -i {$path}/total.ts 
-c:a aac                                                        //设置音频转码器
-c:v copy                                                       //设置视频转码器
-async 1                                                        //音频同步方法,只有音频流的开始被校正，而没有任何后校正
-movflags faststart -pass 2 
-y                                                              //输出文件存在则直接覆盖
-f mp4 
{$path}/mp4.mp4  
2>/dev/null
```


### 5. 过滤器的使用(过滤器的原理为帧过滤)
#### 5.1 参数
```php
-vf: 视频过滤器
-af: 音频过滤器
-lavfi/filter_complex: 复杂的过滤器
```

#### 5.2 例子
```php
//ps: 如果540不写，写成-1，即scale=960:-1, 那也是可以的，ffmpeg会通知缩放滤镜在输出时保持原始的宽高比
ffmpeg -i input.mp4 -vf scale=960:540 output.mp4 
//图片贴在视频上(左上角)
ffmpeg -i input.mp4 -i iQIYI_logo.png -filter_complex overlay output.mp4
//右上角： 
ffmpeg -i input.mp4 -i logo.png -filter_complex overlay=W-w output.mp4 
//左下角： 
ffmpeg -i input.mp4 -i logo.png -filter_complex overlay=0:H-h output.mp4 
//右下角： 
ffmpeg -i input.mp4 -i logo.png -filter_complex overlay=W-w:H-h output.mp4
```

