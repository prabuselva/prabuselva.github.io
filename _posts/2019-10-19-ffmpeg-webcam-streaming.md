---
title: "Webcamera streaming with ffmpeg"
related: true
categories:
  - linux
  - hacks
tags:
  - ffmpeg
  - video
  - h264
  - camera
  - linux
toc: true  
---

This post explains the method of streaming the laptop's web camera to a raw UDP socket of same/another system.
Make sure, we have ffmpeg, mpv, ffplay installed

* Find your laptop web camera video device in the /dev/ list. (Mostly it will be /dev/video0) 

```shell
$ ls /dev/video*
/dev/video0  /dev/video1
```

* Once the video device is found, start the ffmpeg process to take the /dev/video as input and stream it to  same/another system's UDP sockets. Upon initializing the stream, we can stream it as mjpeg (Motion JPEG) or h264 video codec in mpegts/h264 containers

```shell
$ ffmpeg -r 10 -video_size 640x480 -i /dev/video0 -c:v libx264 -b:a 2048k -preset ultrafast -tune zerolatency -r 10 -f h264 udp://127.0.0.1:12345
ffmpeg version n4.2.1 Copyright (c) 2000-2019 the FFmpeg developers
  built with gcc 9.1.0 (GCC)
  configuration: --prefix=/usr --disable-debug --disable-static --disable-stripping --enable-fontconfig --enable-gmp --enable-gnutls --enable-gpl --enable-ladspa --enable-libaom --enable-libass --enable-libbluray --enable-libdav1d --enable-libdrm --enable-libfreetype --enable-libfribidi --enable-libgsm --enable-libiec61883 --enable-libjack --enable-libmodplug --enable-libmp3lame --enable-libopencore_amrnb --enable-libopencore_amrwb --enable-libopenjpeg --enable-libopus --enable-libpulse --enable-libsoxr --enable-libspeex --enable-libssh --enable-libtheora --enable-libv4l2 --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwebp --enable-libx264 --enable-libx265 --enable-libxcb --enable-libxml2 --enable-libxvid --enable-nvdec --enable-nvenc --enable-omx --enable-shared --enable-version3
  libavutil      56. 31.100 / 56. 31.100
  libavcodec     58. 54.100 / 58. 54.100
  libavformat    58. 29.100 / 58. 29.100
  libavdevice    58.  8.100 / 58.  8.100
  libavfilter     7. 57.100 /  7. 57.100
  libswscale      5.  5.100 /  5.  5.100
  libswresample   3.  5.100 /  3.  5.100
  libpostproc    55.  5.100 / 55.  5.100
Input #0, video4linux2,v4l2, from '/dev/video0':
  Duration: N/A, start: 48239.006793, bitrate: 147456 kb/s
    Stream #0:0: Video: rawvideo (YUY2 / 0x32595559), yuyv422, 640x480, 147456 kb/s, 30 fps, 30 tbr, 1000k tbn, 1000k tbc
Codec AVOption b (set bitrate (in bits/s)) specified for output file #0 (udp://127.0.0.1:12345) has not been used for any stream. The most likely reason is either wrong type (e.g. a video option with no video streams) or that it is a private option of some encoder which was not actually used for any stream.
Stream mapping:
  Stream #0:0 -> #0:0 (rawvideo (native) -> h264 (libx264))
Press [q] to stop, [?] for help
[libx264 @ 0x565467590500] using cpu capabilities: MMX2 SSE2Fast SSSE3 SSE4.2 AVX FMA3 BMI2 AVX2
[libx264 @ 0x565467590500] profile High 4:2:2, level 2.2, 4:2:2, 8-bit
Output #0, h264, to 'udp://127.0.0.1:12345':
  Metadata:
    encoder         : Lavf58.29.100
    Stream #0:0: Video: h264 (libx264), yuv422p, 640x480, q=-1--1, 10 fps, 10 tbn, 10 tbc
    Metadata:
      encoder         : Lavc58.54.100 libx264
    Side data:
      cpb: bitrate max/min/avg: 0/0/0 buffer size: 0 vbv_delay: -1

```

* Next play the stream using ffplay or mpv players as follows,

```shell
$ ffplay -f h264 -probesize 32 udp://127.0.0.1:12345?reuse=1
```

* Wait for about a 30secs or 1 min, the stream will start playing as shown below,

![Unsplash image 9]({{ site.url }}{{ site.baseurl }}/assets/images/webcam_stream.jpg)

* We can also stream to another system like below,

```shell
# You the same h264 raw strea
$ ffmpeg -r 10 -video_size 640x480 -i /dev/video0 -c:v libx264 -b:a 2048k -preset ultrafast -tune zerolatency -r 10 -f h264 udp://192.168.1.x:12345
# Go the other system with IP 192.168.x.x
$ ffplay -f h264 -probesize 32 udp://127.0.0.1:12345

# In order to stream mpeg2video
$ ffmpeg -r 10 -video_size 640x480 -i /dev/video0 -c:v mpeg2video -r 10 -f mpegts udp://192.168.1.x:12345
$ ffplay -f mpegts -probesize 32 udp://127.0.0.1:12345
```