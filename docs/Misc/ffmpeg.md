# FFmpeg 常用功能教程

## 基本的格式转换

```shell
ffmpeg -i input.flv output.mp4
# 设定编码方式
-vcodec libx264
# 设定输出视频的码率
-b 4M
# 设置分辨率（width x height）
-s 1920x1080
-vf scale=640:-1
# 设置高度，同时强制宽度能除尽 2（否则会失败）
-vf scale=-2:480
# 输出帧率（不会改变视频的速度）
-r 29.97
# Constant Rate Factor, 取值范围，0~51，0 是无损，数字越大越不清晰
# 一般 18 被认为是几乎视觉无损的
-crf 25
# 静音 禁止画面
-an -vn
# 隐藏 ffmpeg 编译信息
-hide_banner
```

## 音/视频剪辑

```shell
# -ss 开始时间，形如 hh:mm:ss[.xxx]，下同
# -to 结束时间
# -t 持续时间（优先于 -to）
ffmpeg -ss 0:1:30 -t 0:0:20 -i video.mp4 -vcodec copy -acodec copy output.mp4
# 总帧数
ffmpeg -ss 0:1:30 -vframe 200 -i video.mp4 output.mp4
```

## 旋转/翻转视频

### 旋转视频

https://www.ostechnix.com/how-to-rotate-videos-using-ffmpeg-from-commandline/

如果需要对视频进行重新编码（速度较慢），则可以使用`-vf`相关的指令：

```shell
# 顺时针旋转 90°
ffmpeg -i input.mp4 -vf transpose=1 -c:a copy output.mp4
# 旋转 180°
ffmpeg -i input.mp4 -vf "transpose=1,transpose=1" output.mp4
```

其中，transpose 可以为：

* 0 - 沿左上右下对角线转置（逆时针旋转 90°，然后上下翻转）
* 1 - 顺时针旋转 90°
* 2 - 逆时针旋转 90°
* 3 - 沿右上左下对角线转置（顺时针旋转 90°，然后上下翻转）

还可以通过修改视频的 metadata 来快速旋转视频：

```shell
# 顺时针 90
# 逆时针为正数
ffmpeg -i input.mp4 -c copy -metadata:s:v:0 rotate=-90 output.mp4
# 保留原始视频的所有 metadata（比如拍摄信息等）
ffmpeg -i input.mp4 -map_metadata 0 -metadata:s:v rotate=90 -codec copy output.mp4
```

### 翻转视频

https://duxyng.wordpress.com/2013/04/07/rotateflip-video-with-ffmpeg/

```shell
# 水平/垂直翻转
ffmpeg -i input.mp4 -vf hflip output.mp4
ffmpeg -i input.mp4 -vf vflip output.mp4
# 同时翻转（相当于旋转 180°）
ffmpeg -i input.mp4 -vf "hflip,vflip" output.mp4
```

## 加速/减速

```shell
# 视频和音频需要分别使用不同的 filter 进行处理
ffmpeg -i video.mp4
       -filter:v "setpts=0.5*PTS"
       -filter:a "atempo=2.0"
       output.mp4

# 视频借助 setpts 来实现加减速
# 0.5 为加速 2 倍，2.0 则为减速 0.5 倍
# 加速之后会丢帧，可以通过手动设置输出帧率为 2 倍来避免
# 如果原始视频的帧率为 25
ffmpeg -i video.mp4 -filter:v "setpts=0.5*PTS" -r 50 -an output.mp4

# 音频借助 atempo 来实现加减速，并不会改变音调
ffmpeg -i video.mp4 -filter:a "atempo=2.0" -vn output.mp4
# 范围只能是 0.5~2.0，但是可以叠加多个 atempo 来实现超出范围的变速效果
# 下面的方法可以加速 4 倍
ffmpeg -i audio.mp3 -filter:a "atempo=2.0,atempo=2.0" output.mp3
```

## 合并视频与音频

https://superuser.com/questions/277642/how-to-merge-audio-and-video-file-in-ffmpeg

目前并没有找到直接对两个视频分别取视频和音频（比如使用 `-an` 和 `-vn`）并合并的方法，似乎只能先将其中一个转为音频格式（推荐使用 `aac`编码的 `m4a` 格式），然后再用下面的方法进行合并

```shell
# 最基本的方法
ffmpeg -i video.mp4 -i audio.mp3 -c:v copy output.mp4
# 不编码，只是合并为 .mkv 文件
ffmpeg -i video.mp4 -i audio.mp3 -c copy output.mkv
# 取两个之中最短的作为总时长
ffmpeg -i video.mp4 -i audio.mp3 -shortest output.mp4
# 如果使用 FFmpeg 自己的 aac 编码，需要带上 -strict 一项，其中 experimental 还可以简写为 2
ffmpeg -i video.mp4 -i audio.wav -c:v copy -c:a aac -strict experimental output.mp4
```

## 视频与连续图片

```shell
# 将视频转为连续图片
ffmpeg -i video.mp4 thumb%04d.jpg
# 按照指定 FPS 截取图片
ffmpeg -i video.webm -vf fps=1 thumb%04d.jpg
# 提取第 7 秒的一张图
ffmpeg -i video.mp4 -ss 00:00:07.000 -vframes 1 thumb.jpg
# 提取第 7 秒之后的连续三帧
ffmpeg -i video.mp4 -ss 00:00:07.000 -vframes 3 thumb%04d.jpg
# 视频转为 gif
ffmpeg -i video.mp4 -t 12 -pix_fmt rgb24 -s 960x540 output.gif
# 视频转为高质量 gif
ffmpeg -t 12 -i input.mp4 -vf "fps=24,scale=960:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 output.gif
```

[视频转为连续图片](https://www.bugcodemaster.com/article/extract-images-frame-frame-video-file-using-ffmpeg)

```shell
# 将连续图片转为视频
ffmpeg -r 60 -f image2 -s 1920x1080 -i pic%04d.png -vcodec libx264 -crf 25 -pix_fmt yuv420p test.mp4
```

## 视频转 GIF

[视频转为高质量 GIF](https://superuser.com/questions/556029/how-do-i-convert-a-video-to-gif-using-ffmpeg-with-reasonable-quality)

```shell
# 简易方法，但是图片质量一般
ffmpeg -i input.mp4 -r 12 output.gif
# 输出高质量图片
ffmpeg -i input.mp4 -vf "fps=12,scale=320:-1:flags=lanczos,split[s0][s1];[s0]palettegen[p];[s1][p]paletteuse" -loop 0 output.gif
```

