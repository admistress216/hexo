---
title: ffmpeg
date: 2018-12-18 22:35:03
tags:
---
### 1.使用语法
#### 1.1 语法
```php
命令格式：
    ffmpeg -i [输入文件名] [参数选项] -f [格式] [输出文件]
    ffmpeg [[options][`-i' input_file]]... {[options] output_file}...
    1、参数选项：
    (1) -an: 去掉音频
    (2) -vn: 去掉视频
    (3) -acodec: 音频选项， 一般后面加copy表示拷贝
    (4) -vcodec:视频选项，一般后面加copy表示拷贝
    (5) -cv: 设置视频编码器(外部编码器使用,内部也可使用此参数)
    2、格式：
    (1) h264: 表示输出的是h264的视频裸流
    (2) mp4: 表示输出的是mp4的视频
    (3)mpegts: 表示ts视频流
    如果没有输入文件，那么视音频捕捉（只在Linux下有效，因为Linux下把音视频设备当作文件句柄来处理）就会起作用。作为通用的规则，选项一般用于下一个特定的文件。如果你给 –b 64选项，改选会设置下一个视频速率。对于原始输入文件，格式选项可能是需要的。缺省情况下，ffmpeg试图尽可能的无损转换，采用与输入同样的音频视频参数来输出。
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

#### 4.2 mp4转m3u8
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
//如果视频不为mp4格式，需先将视频转码为mp4，可使用如下命令进行转换
ffmpeg -i 本地视频地址 -y -c:v libx264 -strict -2 转换视频.mp4
//将mp4格式转换为ts格式(-vbsf:流过滤器的音频过滤选项)
ffmpeg -y -i 本地视频.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 转换视频.ts
//将ts文件进行切片
ffmpeg -i 本地视频.ts -c copy -map 0 -f segment -segment_list 视频索引.m3u8 -segment_time 5 前缀-%03d.ts
//其中segment 就是切片，-segment_time表示隔几秒进行切一个文件，上面命令是隔5s，你也可以调整成更大的参数。
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

