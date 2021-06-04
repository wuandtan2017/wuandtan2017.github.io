---
title: FFMpeg命令分类
date: 2021-06-04 13:24:41

---


本文主要介绍FFmpeg的命令分类及基本的使用，具体请参照《FFmpeg从入门到精通》

![FFmpeg命令分类](https://img-blog.csdnimg.cn/20210603225216334.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)  

***

## 1.信息查询命令

![基本信息查询命令列表](https://img-blog.csdnimg.cn/20210603225327708.jpg?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)

## 2.录制命令

参数说明 
-f 设备驱动
-i 设备编号

#### 采集摄像头：
根据采集过程中终端中的提示信息按照规定的格式播放。 我的摄像头默认是640x480，25帧，像素格式是yuyv422，使用ffplay播放需要设定正确的参数

ffmpeg -f video4linux2 -i /dev/video0 out.yuv

#### 采集桌面： 
ffmpeg -f x11grab -framerate 25 -video_size 1920x1080 -i :0.0 out.mp4

#### 采集音频 
ffmpeg -f alsa -i hw:0 out.wav

## 3.分解与复用命令

ffmpeg -i in.mp4 -vcodec copy -acodec copy out.flv

-an 不要音频数据
-vn 不要视频数据

## 4.处理原始数据命令

#### 提取yuv数据：

ffmpeg -i input.mp4 -an -c:v rawvideo -pix_fmt yuv420p out.yuv


-c:v 对视频编码，用原始视频进行编码

#### 提取PCM数据：

ffmpeg -i input.mp4 -vn -ar 44100 -ac2 -f s16le out.pcm

-ar 音频采样率
-ac 声道数
-f 数据存储方式

## 5.滤镜命令
#### 视频尺寸修改滤镜 

ffmpeg -i in.mp4 -vf crop=in_w-200:in_h-200 -c:v libx264 -c:a copy out.mp4

## 6.裁剪与合并
#### 裁剪 

ffmpeg -i in.mp4 -ss 00:00:00 -t 10 out.mp4


-ss 裁剪起始时间
-t   裁剪时长
#### 合并 

ffmpeg -f concat -i inputs.txt out.flv


-f concat 对后面的文件进行拼接 inputs.txt 内容为 'file filename'格式。file内容如下，和裁切片段一个目录下，采用相对路径
![file文件内容](https://img-blog.csdnimg.cn/20210604124948452.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)

## 7.图片与视频命令互转
#### 视频转图片


 ffmpeg -i outlydy2.mp4 -r 1 -f image2 image-%3d.jpeg


-r 1 1秒钟一张图片
-f image2图片格式 

![转图片结果](https://img-blog.csdnimg.cn/2021060412573361.png)
#### 图片转视频 

ffmpeg -i image-%3d.jpeg out.mp4 ffplay -r 1 hello.mp4


## 8.直播推/拉流


#### 推流

ffmpeg -re -i out.mp4 -c copy -f flv rtmp://127.0.0.1/live/livestream


-re 按时间戳读取文件

#### 拉流 

拉到本地保存成多媒体文件 

ffmpeg -i  rtmp://127.0.0.1/live/livestream -c copy live.flv

ffplay直接观看 

ffplay rtmp://127.0.0.1/live/livestream












