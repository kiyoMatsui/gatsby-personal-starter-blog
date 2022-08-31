---
title: Video Bitstreams Part01
date: "2022-08-26T21:00:00.284Z"
---

Recently I have been reminded about the phrase “A little knowledge is a dangerous thing”. It is probably obvious to many how this applies to working with technology, especially as often we only have an abstract understanding of many of the topics we need to familiarise ourselves with. In practice we end up in a “Danger Zone” of not knowing we should better understand the key pieces of technology we are responsible for. Often, we only find out later and have to reactively fix something. With experience you can start to make best guesses on what areas need to be explored in the hope that you can do better. But Dunning Kruger tells us we will often not know that we should have known something until it’s too late.

Until recently in my career as a video engineer, I have been spared really needing to dig into the details of compressed video bitstreams. With everything moving to web players, like Shaka-player and Dash.js, video data is appended to the SourceBuffer and most of the playback failures are elsewhere, it felt like I could put this aside and prioritise learning other things. 

Until now…

For reasons I won’t bore you with, it is time to learn about the video bitstream, so I have decided to document this learning provess in this blog. By bitstream I am talking about AVC(h264) and HVEC(h265) video bitstreams that are often found inside mp4 files. I will be using what I can find online, both information and tools that will be part of the analysis. For the remainder of this blog post, I will set up my analysis prerequisites.

All my work for this will be in `~/bistreamTesting` directory on my KDE neon 5.24 Laptop, using the repository ffmpeg.

I have decided to use the [1080p Sintel trailer](https://media.xiph.org/) as raw video data that I will then encode into AVC(h264). This is a nice and short video so analysing it will be easier. I will also include the flac stereo they have provided.


I encoded this content using ffmpeg and the options are based on guidance from recent reference books written by Jan Ozer. These references have been useful in working out what current good practice encolding parameters are for AVC/HVEC content. This content should therefore be “reflective of what can be found out there” (*Danger Zone alarms start ringing*).

```
ffmpeg -i ~/bistreamTesting/sintel_trailer-audio.flac -i ~/bistreamTesting/sintel_trailer_2k_1080p24.y4m -c:a aac -b:a 128k -ar 48000 -c:v libx264 -s 1920x1080 -crf 23 -r 24 -g 48 -bf 2 -keyint_min 48 -sc_threshold 0 -refs 2 -profile:v main -level:v 4.0 -preset faster -pix_fmt yuv420p -movflags frag_keyframe+empty_moov -tune psnr -report -f MP4 sintelTrailer_fmp4_avc.mp4
```

[Here](./sintelTrailer_fmp4_avc.mp4) it is. This is a fragmented mp4 file with AVC video and AAC audio, in the next part we will be analysing this encoded video file. Below is the output from ffprobe.


```
kiyo@kiyo-prox14amd:~/bistreamTesting$ ffprobe sintelTrailer_fmp4_avc.mp4 
ffprobe version 4.2.7-0ubuntu0.1 Copyright (c) 2007-2022 the FFmpeg developers
  built with gcc 9 (Ubuntu 9.4.0-1ubuntu1~20.04.1)
  configuration: --prefix=/usr --extra-version=0ubuntu0.1 --toolchain=hardened --libdir=/usr/lib/x86_64-linux-gnu --incdir=/usr/include/x86_64-linux-gnu --arch=amd64 --enable-gpl --disable-stripping --enable-avresample --disable-filter=resample --enable-avisynth --enable-gnutls --enable-ladspa --enable-libaom --enable-libass --enable-libbluray --enable-libbs2b --enable-libcaca --enable-libcdio --enable-libcodec2 --enable-libflite --enable-libfontconfig --enable-libfreetype --enable-libfribidi --enable-libgme --enable-libgsm --enable-libjack --enable-libmp3lame --enable-libmysofa --enable-libopenjpeg --enable-libopenmpt --enable-libopus --enable-libpulse --enable-librsvg --enable-librubberband --enable-libshine --enable-libsnappy --enable-libsoxr --enable-libspeex --enable-libssh --enable-libtheora --enable-libtwolame --enable-libvidstab --enable-libvorbis --enable-libvpx --enable-libwavpack --enable-libwebp --enable-libx265 --enable-libxml2 --enable-libxvid --enable-libzmq --enable-libzvbi --enable-lv2 --enable-omx --enable-openal --enable-opencl --enable-opengl --enable-sdl2 --enable-libdc1394 --enable-libdrm --enable-libiec61883 --enable-nvenc --enable-chromaprint --enable-frei0r --enable-libx264 --enable-shared
  libavutil      56. 31.100 / 56. 31.100
  libavcodec     58. 54.100 / 58. 54.100
  libavformat    58. 29.100 / 58. 29.100
  libavdevice    58.  8.100 / 58.  8.100
  libavfilter     7. 57.100 /  7. 57.100
  libavresample   4.  0.  0 /  4.  0.  0
  libswscale      5.  5.100 /  5.  5.100
  libswresample   3.  5.100 /  3.  5.100
  libpostproc    55.  5.100 / 55.  5.100
Input #0, mov,mp4,m4a,3gp,3g2,mj2, from 'sintelTrailer_fmp4_avc.mp4':
  Metadata:
    major_brand     : isom
    minor_version   : 512
    compatible_brands: isomiso2avc1iso6mp41
    encoder         : Lavf58.29.100
  Duration: 00:00:52.29, start: 0.000000, bitrate: 1618 kb/s
    Stream #0:0(und): Video: h264 (Main) (avc1 / 0x31637661), yuv420p, 1920x1080 [SAR 1:1 DAR 16:9], 1489 kb/s, 24 fps, 24 tbr, 12288 tbn, 48 tbc (default)
    Metadata:
      handler_name    : VideoHandler
    Stream #0:1(und): Audio: aac (LC) (mp4a / 0x6134706D), 48000 Hz, stereo, fltp, 127 kb/s (default)
    Metadata:
      handler_name    : SoundHandler
```




