---
title: FFmpeg编译与依赖库安装
date: 2021-06-03 22:04:30
tags:
-ffmpeg
---
# 前言

刚开始学音视频开发，环境的配置研究了几天在几个虚拟机上都尝试了一下，终于把刚开始能踩的坑都踩遍了，遇到了各种问题，经过我的收集整理，把所有遇到的问题都放进来，后面再遇到问题还会继续更新的，作为个人以后参考用，如果写的有错误请各位大佬在评论区指正！感谢很多人帮助了我，希望自己也可以帮助更多的人。
我是Ubuntu18.04的版本，不知道是不是版本问题，下的版本好多配置和包都没有，手动配置了好久。

# 前期准备工作
省得之后文件权限问题，都是血和泪的教训啊，记不清在哪个错误的目录下 chmod -R -777 结果sudo su打不开了，为了防止和我一样的错误，先设置一下默认进入root用户，及时备份虚拟机。
具体的看这篇，很详细了，感谢这位老哥
https://blog.csdn.net/qq_39591507/article/details/81288644
以root身份登陆以后打开shell是这样的
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420150510445.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)下一步，打开根目录下opt文件夹，新建一个lib文件夹，之后编译的库放里面，后面全部编译完成以后把这个文件夹备份一下
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420150955569.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)里面我已经编译好了，下载的库资源全部放这里面。
# 安装yasm
直接在刚刚的lib文件夹下面打开终端
wget http://www.tortall.net/projects/yasm/releases/yasm-1.3.0.tar.gz
tar xvzf yasm-1.3.0.tar.gz
cd yasm-1.3.0
./configure
make && make install

PS：//默认的./configure 配置是安装在usr/local/lib里面，后面ffmpeg不放进去，前期的一些必要的组件直接configure没问题。
# 安装nasm（2.13以上版本）
和上面一样
wget https://www.nasm.us/pub/nasm/releasebuilds/2.14.02/nasm-2.14.02.tar.bz2
tar xvf nasm-2.14.02.tar.bz2
cd nasm-2.14.02
./configure
make 
make install
# 安装其他依赖
apt install cmake -y
apt install pkg-config   //后面编译x264和x265需要

# 编译x264（只编译静态库）
x264下载地址：
http://ftp.videolan.org/pub/videolan/x264/snapshots/
tar xvf x264-snapshot-20191024-2245-stable.tar.bz2
cd x264-snapshot-20191024-2245-stable
./configure --enable-static --prefix=../x264 --enable-pic 
make -j
make install
//make -j4就是开启4个并行编译，make -j不加数字就是默认全部核心数，编译速度差不多要快一半
##编译x265（只编译静态库）
x265下载地址:
http://ftp.videolan.org/pub/videolan/x265/
tar xvf x265_3.2.tar.gz
//**和x264编译不一样，264直接在根目录下编译，265要进入这个目录**
cd x265_3.2/build/linux/        

cmake -G "Unix Makefiles" -DCMAKE_INSTALL_PREFIX="../../../x265" -DENABLE_SHARED:bool=off ../../source
make -j
make install
为了避免后面的问题，我们先进行如下的设置
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153019777.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)
进入265这个目录，编辑一下这个文件
按如图添加依赖
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420153143863.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)
# 编译SDL2
编译之前检查一下Ubuntu是否声音输出是否正常，我的是不正常的，基本各种疑难杂症都给我遇见了，不过问题不大，参考一下别人的解决方法。
https://blog.csdn.net/multimicro/article/details/82528730
确保有声音再进入下一步
我需要用到ffplay, 需要再下载SDL的源码，http://libsdl.org/release/
我的版本是SDL2-2.0.14
在编译之前先安装
sudo apt-get install libx11-dev
sudo apt-get install xorg-dev
不然会无法渲染SDL displa
如运行ffplay时，有些机器上会出现
Could not initialize SDL - No available video device
(Did you set the DISPLAY variable?)
说明系统中没有安装x11的库文件，因此编译出来的SDL库实际上不能用。

需要先编译安装SDL，和上面一样，直接默认编译安装

tar zxvf SDL2-2.0.8.tar.gz

cd SDL2-2.0.8

./configure

make

make install

# 编译ffmpeg
编译ffmpeg是最后一步，但是前面任何一步配置错误，回去修改以后，ffmpeg都要重新编译。

ffmpeg直接去官网下载，我下的是4.3.2版本的
解压以后还是在lib文件夹里面，我在官网下载的提取出来还套了一个文件夹，直接移到外面来，如图，不然没法直接执行下面的命令
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420155055789.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)在当前页面打开终端执行
export PKG_CONFIG_PATH=$PKG_CONFIG_PATH:../x264/lib/pkgconfig:../x265/lib/pkgconfig

./configure --enable-shared --enable-nonfree --enable-gpl --enable-pthreads --enable-libx264 --enable-libx265 --prefix=../ffmpeg 

make -j
make install

编译完以后lib文件夹多了这三个

![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420165525566.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)

# 编译ffmpeg
编译完成后，进入/etc/profile中将ffmpeg加入到环境变量（在文件最后加上export PATH=/opt/lib/ffmpeg/bin:$PATH）
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420163102945.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)进一步，把ffmpeg的库加入/etc/ld.so.conf中
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420163524174.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)
# 测试是否安装成功
执行ffmpeg -devices
![在这里插入图片描述](https://img-blog.csdnimg.cn/20210420160325660.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L3d1Z2VidWN1bw==,size_16,color_FFFFFF,t_70)我已经解决了，如果需要alsa驱动的需要装一下alsa驱动

之前的问题是编译完了ffmpeg发现没有alsa音频驱动，关于如何安装alsa音频驱动建议看这篇
https://blog.csdn.net/keepingstudying/article/details/7373246

安装完了重启，然后重新编译ffmpeg
再测试一下ffplay命令，
ffplay xxx.MP4
如果可以正常播放视频,声音正常就ok，否则需要重装SDL2，然后重新编译ffmpeg


主体参考了云天之巅博主的，感谢！
具体不清楚的可以看云天之巅博主的视频
https://www.bilibili.com/video/BV1Sz411v7Wm

 [1]: https://blog.csdn.net/sinat_41559158/article/details/80363649
 [2]: http://blog.yundiantech.com/?log=blog&id=35

---

