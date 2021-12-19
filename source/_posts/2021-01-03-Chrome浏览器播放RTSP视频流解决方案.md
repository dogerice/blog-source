---
title: Chrome浏览器播放RTSP视频流解决方案
date: 2021-01-03 01:19:19
tags:
- FFmpeg
- RTSP
- RTMP
categories:
- tools
---

记录一个在高版本chrome浏览器中播放RTSP视频流的方法
<!-- more -->

背景：海康威视的监控系统推送的视频流是RTSP协议，在IE浏览器上可以通过VLC等插件来播放，但chrome浏览器从46版本开始不再支持，试了许多前端组件基本都只能播放HLS或RTMP的流，最终选择RTSP转RTMP的方式解决这个问题

方案：将RTSP视频流通过FFmpeg转换为RTMP协议的流，并推送到RTMP流媒体服务器上（基于Nginx搭建），最后前端使用video.js来播放RTMP视频流

# 安装FFmpeg

官网下载最新安装包http://ffmpeg.org/download.html

解压`tar -xjvf ffmpeg-4.4.2.tar.bz2`
编译安装

```
cd ffmpeg-4.4.2
./configure --enable-shared --prefix=/usr/local/ffmpeg --disable-yasm
make
make install
```

执行`ffmpeg -version` 出现类似如下内容表示安装成功，提示ffmpeg命令不存在的话就把ffmpeg的bin目录配到环境变量里

```
ffmpeg version 4.1.3 Copyright (c) 2000-2019 the FFmpeg developers
built with gcc 4.4.7 (GCC) 20120313 (Red Hat 4.4.7-23)
configuration: --prefix=/usr/local/src/ffmpeg
libavutil   56. 22.100 / 56. 22.100
libavcodec   58. 35.100 / 58. 35.100
libavformat  58. 20.100 / 58. 20.100
libavdevice  58. 5.100 / 58. 5.100
libavfilter   7. 40.101 / 7. 40.101
libswscale   5. 3.100 / 5. 3.100
libswresample  3. 3.100 / 3. 3.100
```

# 搭建RTMP流媒体服务器

下载并解压Nginx与[nginx-rtmp-module](https://github.com/arut/nginx-rtmp-module)安装包

```shell
mkdir /usr/local/nginx
cd /usr/local/nginx
wget https://github.com/arut/nginx-rtmp-module/archive/master.zip
wget http://nginx.org/download/nginx-1.12.2.tar.gz
 
tar -zxvf nginx-1.12.2.tar.gz
unzip master.zip
```

安装Nginx和RTMP模块前安装依赖库

```shell
yum -y install gcc-c++ 
yum -y install pcre pcre-devel  
yum -y install zlib zlib-devel 
yum -y install openssl openssl-devel
```

编译安装

```shell
cd /usr/local/nginx/nginx-1.12.2/
 
./configure --add-module=/usr/local/nginx/nginx-rtmp-module-master
make
make install
```

验证

```shell
cd /usr/local/nginx/sbin/
./nginx -t
```

出现类似如下内容表示安装成功

```
nginx: the configuration file /usr/local/nginx/conf/nginx.conf syntax is ok
nginx: configuration file /usr/local/nginx/conf/nginx.conf test is successful
```

修改Nginx配置文件添加如下内容，注意rtmp{}与http{}为同级不要放错位置

**/usr/local/nginx/conf/nginx.conf** 

```
rtmp{
        server{
                listen 1935;
                chunk_size 4000;
                application live{
                        live on;
                        record off;
                }
        }
}
```

启动Nginx

`/usr/local/nginx/sbin/nginx -c /usr/local/nginx/conf/nginx.conf`

# 推流测试

开启ffmpeg推流

```shell
ffmpeg -i "rtsp://10.16.4.80:554/pag://10.16.4.80:7302:37021200001310015420:0:MAIN:TCP?cnid=100001&pnid=1&auth=50" -vcodec copy -acodec copy -f flv "rtmp://10.16.3.232:1935/live/115"
```

-i：源视频，这里是rtsp的视频流地址

-vcodec copy ：视频编码选项，copy表示不变

-acode copy ：音频编码选项，copy表示不变

-f flv ：视频格式，使用flv格式

"rtmp://10.16.3.232:1935/live/115" 即推向刚才搭建的rtmp服务器上，1935和live是上一步骤在配置文件中配的监听端口和appname，后面的115我们可以自己定义

开始推流后就可以测试视频有没有推成功了，这里推荐一个播放器软件 VLC Media Player

VLC播放器中  媒体=》打开网络串流=》网络 填写刚才推的URL rtmp://10.16.3.232:1935/live/115

![image-20210103012924127](image-20210103012924127.png)

点击播放，出现视频画面了说明推流成功

# 用video.js实现页面播放

编写一个播放视频的测试页面，需要用到video.js

**video.html** 

```html
<!DOCTYPE html>
<html lang="en" dir="ltr">
  <head>
    <meta charset="utf-8">
    <title>video.js</title>
    <link href="https://unpkg.com/video.js@6.11.0/dist/video-js.min.css" rel="stylesheet">
    <script src="https://unpkg.com/video.js@6.11.0/dist/video.min.js"></script>
    <script src="https://unpkg.com/videojs-flash/dist/videojs-flash.js"></script>
    <script src="https://unpkg.com/videojs-contrib-hls/dist/videojs-contrib-hls.js"></script>
  </head>
  <body>
    <video id="my-player" class="video-js" controls>
        <source src="rtmp://10.16.3.232:1935/live/115" type="rtmp/flv">
        <p class="vjs-no-js">
          not support
        </p>
    </video>
    <script type="text/javascript">
      var player = videojs('my-player',{
        width:600,
        heigh:400
      });
    </script>
  </body>
```

页面上出现视频控件并能正常播放画面表示成功