---
title: ffmpeg常用命令
date: 2023-07-23 22:29:00
categories:
  - 杂项
tags:
  - 视频处理
  - 视频压缩
  - ffmpeg
  - 视频分割
  - 视频合并
  - 显卡支持
description: '本文主要总结和展示了ffmpeg这个强大的媒体处理工具的常用功能和命令。文章列出了ffmpeg执行这些操作的具体命令,如视频采集摄像头命令,音频和视频转换命令,对视频截图命令,为视频添加水印命令等等。文中所有的ffmpeg命令都进行了示例和解释,便于读者理解记忆。最后,文章给出了一些ffmpeg的高级用法,如使用滤镜处理视频,对GIF动图处理等。'
cover: https://s2.loli.net/2023/07/24/uxe2mkvdq64o5EH.webp
---

### 音频视频合并

```shell
ffmpeg -i 视频文件名 -i 音频文件名 -codec copy 输出MP4 文件名
```

### mp4视频拼合

```shell
ffmpeg -i 1.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 1.ts ffmpeg -i 2.mp4 -vcodec copy -acodec copy -vbsf h264_mp4toannexb 2.ts ffmpeg -i "concat:1.ts|2.ts" -acodec copy -vcodec copy -absf aac_adtstoasc output.mp4
```



### 启用显卡支持

```bash
ffmpge -hwaccel cuvid
##### GPU:
  .\ffmpeg-win64-v4.2.2.exe -c:v h264_cuvid -i "E:\\Downloads\\0112\\The Tai-chi Maste 太极张三丰.mp4"
      -i "E:\\GitSpace\\DeWaterMark\\watermark\\watermark1.png"
      -i "E:\\GitSpace\\DeWaterMark\\watermark\\watermark2.png"
      -filter_complex "overlay=enable='lte(t,20)':x=(W-w)/2:y=H-h:,overlay=enable='gt(mod(t,31),20)':x=W-w:y=0"
      -c:v h264_nvenc -b:v 1500k -bufsize 1500k "D:\\Downloads\\output\\output.mp4" -y

##### CPU:
 .\ffmpeg-win64-v4.2.2.exe  -i "E:\\Downloads\\0112\\The Tai-chi Maste 太极张三丰.mp4"
      -i "E:\\GitSpace\\DeWaterMark\\watermark\\watermark1.png"
      -i "E:\\GitSpace\\DeWaterMark\\watermark\\watermark2.png"
      -filter_complex "overlay=enable='lte(t,20)':x=(W-w)/2:y=H-h:,overlay=enable='gt(mod(t,31),20)':x=W-w:y=0"
      -b:v 1500k -bufsize 1500k "D:\\Downloads\\output\\output.mp4" -y

```

### 添加水印

* movie滤镜详解

  ```bash
    ffmpeg -i inputfile -vf  "movie=masklogo,scale= 60: 30[watermask]; [in] [watermask] overlay=30:10 [out]" outfile
  参数说明：
  marklogo:添加的水印图片；
  scale：水印大小，水印长度＊水印的高度；
  overlay：水印的位置，距离屏幕左侧的距离＊距离屏幕上侧的距离；mainW主视频宽度， mainH主视频高度，overlayW水印宽度，overlayH水印高度
  　　左上角overlay参数为 overlay=0:0
  　　右上角为 overlay= main_w-overlay_w:0
  　　右下角为 overlay= main_w-overlay_w:main_h-overlay_h
  　　左下角为 overlay=0: main_h-overlay_h
      面的0可以改为5，或10像素，以便多留出一些空白。
  ```

* 定时文字水印

  ```bash
  ffmpeg -re -i test.mp4 -vf "drawtext=fontsize=60:fontfile=lazy.ttf:text='{localtime\:%Y\-%m-%d %H-%M-%S}':fontcolor=green:box=1:boxcolor=yellow:enable=lt(mod(t\, 3)\, 1)" out.mp4
  ```

* 添加一个图片水印

  ```bash
  ffmpeg -i test.mp4 -vf "movie=logo.jpg[wm];[in][wm]overlay=30:10[out]" image_out.mp4
  ```

* 添加四个图片水印

  ```bash
  ffmpeg -i in.mp4 -i logo.png -i logo.png -filter_complex "overlay=5:5, overlay=x=W-w:y=5" in_out_mul_watermark.mp4
  ```
* 添加两个水印 并指定位置和开始消失时间
  ```bash
  # 添加两个水印 第一个显示在下中 显示时间0-20秒 第二个显示在 右上, 显示时间20-31秒
  ffmpeg  -i intput.mp4 -i watermark1.png -i watermark2.png -filter_complex "overlay=enable='lte(t,20)':x=(W-w)/2:y=H-h:,overlay=enable='gt(mod(t,31),20)':x=W-w:y=0" output.mp4 -y
  # 同上 添加显卡支持
  ffmpeg  -hwaccel cuvid -i intput.mp4 -i watermark1.png -i watermark2.png -filter_complex "overlay=enable='lte(t,20)':x=(W-w)/2:y=H-h:,overlay=enable='gt(mod(t,31),20)':x=W-w:y=0" -vcodec h264_nvenc output.mp4 -y
  ```
* 添加两个水印，10秒交替出现
  ```bash
  ffmpeg -i /root/test.mp4 -i /root/videoProcessing/youtube/test.png -i /root/test.png 
    -filter_complex “overlay=x=if(lt(mod(t,20),10),10,NAN ):y=10,overlay=x=if(gt(mod(t,20),10),main_w-273,NAN ) :y=main_h-113,subtitles=/root/test.srt :force_style=‘Fontsize=14’” /root/test3.mp4
    
    添加两个水印，overlay=x=if(lt(mod(t,20),10),10,NAN ):y=10,overlay=x=if(gt(mod(t,20),10),main_w-273,NAN ) 这两个使用了函数，代表是交替出现水印。
    mod(t,20)代表当前时间对20进行取模；
    lt(a,b)表示的是a<b，则为true
    if(true,a,b)表示的是如果为true，则返回a，否则返回b
    
    ps:如果需要在程序中进行命令行的拼接，一定要记得转义，否则会报错。
  ```
### 视频分割

```bash
ffmpeg -v quiet -y -i S4.mp4 -vcodec copy -acodec copy -ss 00:00:00 -t 00:07:12 -sn S01E01.mp4 -ss 表示视频分割的起始时间，-t 表示分割时长
```

### rm 转mp4

```bash
ffmpeg -i 郭德纲.rmvb  -vcodec libx264 -c:v h264 -c:a aac 郭德纲.mp4 
无声 -an : 不编码音频 -vcodec : 设置视频的编码，我这里使用的是x264 -b : 这个是码率 -f : 强制使用格式 -y : 自动输y确认
ffmpeg -i 没事偷着乐HD1024高清国语.rmvb -c:v libx264 -strict -2 没事偷着乐HD1024高清国语.mp4
```

### flv 转MP4

```bash
ffmpeg -i 【AE教程】月有阴晴圆缺，这个特效，推荐你一定要看！.flv  -vcodec libx264 -c:v h264 -c:a aac 月.mp4
```

##### 批量处理视频

```bash
for %%G in (*.rmvb) do ffmpeg -i "%%~G" -c:v h264 -c:a aac "%%~nG.mp4" for %%G in (*.rm) do ffmpeg -i "%%~G" -c:v h264 -c:a aac "%%~nG.mp4"
```

### 其他格式转MP3脚本

```
@echo off & title
cd /d %~dp0
for %%a in (*.m4a) do (
 echo 正在转换：%%a
 ffmpeg -i "%%~sa" -y -acodec libmp3lame -aq 0 "output\%%~na.mp3"
)
pause
```

### MP4批量转换文件

```shell
@echo off

::在下方设置要处理的视频或音频格式，这里列出了一些主要的视频格式
set Ext=*.avi,*.mp4,*.wmv,*.flv,*.mkv,*.rmvb,*.rm,*.3gp,*.ts

md output

echo 开始视频转换

::在下方设置输出格式，这里输出为mp4，可自行更改
for %%a in (%Ext%) do (
	echo 正在转换：%%a
	ffmpeg -loglevel quiet -i "%%a" -f mp4 "output\%%~na.mp4" -y
)

echo 转换完成

pause
```

### 视频压缩

1. ###### 查看视频参数

   * ![](https://s2.loli.net/2023/07/23/jiJu5TCVg28LvlH.png)

2. ```python
   ffmpeg -i 1274.mp4 -b 600k 12.mp4
   -b    数据比特率，每秒传输的数据流量大小（kb/s），这个命令里设置的比特率是600k 也就是原来视频的八分之一
   ```

‍

