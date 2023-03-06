[【FFmpeg 分P教学】转码、压制、录屏、裁切、合并、提取 … 统统不是问题。_哔哩哔哩_bilibili](https://www.bilibili.com/video/BV1Ft411s7Xa?p=1)

ffmpeg -h 打开帮助菜单

## 1.转换格式

ffmpeg -i input.mov output.mp4

-i 指定输入文件

## 2.提取mp4视频

​	ffmpeg -i in_file_name.mp4 -vcodec copy -an outfile.mp4

- -an 静音

## 3.提取音频

​	ffmpeg -i in.mp4 -vn -acodec copy a.m4a

## 4. 合并音视频

​	ffmpeg -i a.m4a -i v.mp4 -c copy out.mp4

## 5.音频转码

ffmpeg -i in.flac -acodec libmp3lame -ar 44100 -ab 320k -ac 2 out.mp3

- -acodec 指定音频编码器(可不输入，自动识别)

- libmp3lame mp3音频编码器

- -ar 44100 设置音频采样率(可不输入使用默认值原音频的采样率)
- -ab 320k 指定音频比特率(可不输入使用默认值180k)

- -ac 2 设置声道数为2声道(可不输入使用默认值原音频的声道数)

## 6.视频压制

ffmpeg -i in.webm -s 1920x1080 -pix_fmt yuv420p -vcodec libx264 -preset medium profile:v high -level:v 4.l -crf 23 -acodec aac -ar 44100 -ac 2 -b:a 128k out.mp4

- -s 缩放视频尺寸
- -pix_fmt yuv420p 设置视频颜色空间 ffmpeg -pix_fmts查看具体参数
- -vcodec libx264 设置视频流的编码器
- -preset 编码器预设。影响编码算法精度与运行速度。参数:ultrafast superfast veryfast faster fast medium slow slower veryslow placebo
- -profile:v 指定编码器配置
- -level:v 对编码器配置的限制(一般1080p视频就用4.1)
- -crf 码率控制模式。crf为恒定速率因子模式。参数可从0~51中任选，数字越小，质量越高，0为无损画质，一般在18~28内选
- -r 30 设置视频帧率每秒三十帧 

