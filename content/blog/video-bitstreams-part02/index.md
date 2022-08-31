---
title: Video Bitstreams Part02
date: "2022-08-27T21:00:00.284Z"
---

So now we have our fmp4 (see Part01), lets actually look inside the file. The tool I like to use here is [MP4Box.js](https://github.com/gpac/mp4box.js/), lets load in the fmp4 and see what is inside (click to enlarge image).
![mp4box](./mp4box.png)

What we see in the above image is the tree structure of MP4 Boxes. These Boxes are defined in a number of ISO standards and help organise the fmp4 file into relevant info. Most of the boxes hold stream information, the mdat box holds the actual bitstream data so will be much larger.

It is helpful to view the file in a hex editor, this will show how the boxes fit together in a file and shows visually what the file actually looks like. Here we have used hexl-mode in emacs.
![emacs](./emacs.png)

The right side of the hex view translates the box name with 4 bytes. We see ftyp box at the top, a moov box, then all the sub boxes like trak, tkhd and further down the tree, avc1. tkhd includes the track_id which for this stream is "1". The avc1 box obviously tells us we have a AVC (or h.264) video stream.
![avc1](./avc1.png)

Next lets look at the first mdat box, MP4Box.js will tell me it has type: mdat; size: 127403; and start: 2549.
Thats as far as we go with MP4Box.js on boxes, we will need to go into more detail with ffmpeg.

The ffmpeg demuxers organise video data into "packets" (or pkts) usually this is one frame but the documentation suggests this may not always be true.
Using the following ffprobe command we can print out the packets, a option is set to include the packet data.

```
ffprobe -show_packets -show_data -select_streams v ./sintelTrailer_fmp4_avc.mp4 > showPacketsShowData.txt
```

Comparing the first mdat box in the raw view we can see the first packet has the same data as the start of our first mdat box.

![ffmpegVsRaw](./ffmpegVsRaw.png)

The data from the next two packets continue to show the data from our first mdat box. No data is lost between the pkts.

![next2pkts](./next2pkts.png)

Its important at this point to state that its clear this bitstream is using the avcc bitstream format.
Most AVC streams are either avcc or Annex-b.

Annex-b deliminates [NAL units](https://en.wikipedia.org/wiki/Network_Abstraction_Layer) with a start code, for example, 00000001. This is often easily spotted in the hex editor.

AVCC deliminates NAL units with 4bytes indicating the NAL unit size. In the image above the the first 4 bytes of the pkt in hex will translate to the packet size - 4 (the size of the deliminator).

For example, in the below snippet of the second packet  `0000 006d = 109` this is the same value as `size=113` subtracted by 4.

```
size=113
pos=3648
flags=__
data=
00000000: 0000 006d 419a 236c 49ff a01f 768c ea00  ...mA.#lI...v...
```


5000 lines of the output have been printed below, analysing this has been interesting so I will leave it here and end Part02.

```
[PACKET]
codec_type=video
stream_index=0
pts=1024
pts_time=0.083333
dts=0
dts_time=0.000000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1091
pos=2557
flags=K_
data=
00000000: 0000 029b 0605 ffff 97dc 45e9 bde6 d948  ..........E....H
00000010: b796 2cd8 20d9 23ee ef78 3236 3420 2d20  ..,. .#..x264 -
00000020: 636f 7265 2031 3535 2072 3239 3137 2030  core 155 r2917 0
00000030: 6138 3464 3938 202d 2048 2e32 3634 2f4d  a84d98 - H.264/M
00000040: 5045 472d 3420 4156 4320 636f 6465 6320  PEG-4 AVC codec
00000050: 2d20 436f 7079 6c65 6674 2032 3030 332d  - Copyleft 2003-
00000060: 3230 3138 202d 2068 7474 703a 2f2f 7777  2018 - http://ww
00000070: 772e 7669 6465 6f6c 616e 2e6f 7267 2f78  w.videolan.org/x
00000080: 3236 342e 6874 6d6c 202d 206f 7074 696f  264.html - optio
00000090: 6e73 3a20 6361 6261 633d 3120 7265 663d  ns: cabac=1 ref=
000000a0: 3220 6465 626c 6f63 6b3d 313a 303a 3020  2 deblock=1:0:0
000000b0: 616e 616c 7973 653d 3078 313a 3078 3131  analyse=0x1:0x11
000000c0: 3120 6d65 3d68 6578 2073 7562 6d65 3d34  1 me=hex subme=4
000000d0: 2070 7379 3d30 206d 6978 6564 5f72 6566   psy=0 mixed_ref
000000e0: 3d30 206d 655f 7261 6e67 653d 3136 2063  =0 me_range=16 c
000000f0: 6872 6f6d 615f 6d65 3d31 2074 7265 6c6c  hroma_me=1 trell
00000100: 6973 3d31 2038 7838 6463 743d 3020 6371  is=1 8x8dct=0 cq
00000110: 6d3d 3020 6465 6164 7a6f 6e65 3d32 312c  m=0 deadzone=21,
00000120: 3131 2066 6173 745f 7073 6b69 703d 3120  11 fast_pskip=1
00000130: 6368 726f 6d61 5f71 705f 6f66 6673 6574  chroma_qp_offset
00000140: 3d30 2074 6872 6561 6473 3d32 3420 6c6f  =0 threads=24 lo
00000150: 6f6b 6168 6561 645f 7468 7265 6164 733d  okahead_threads=
00000160: 3620 736c 6963 6564 5f74 6872 6561 6473  6 sliced_threads
00000170: 3d30 206e 723d 3020 6465 6369 6d61 7465  =0 nr=0 decimate
00000180: 3d31 2069 6e74 6572 6c61 6365 643d 3020  =1 interlaced=0
00000190: 626c 7572 6179 5f63 6f6d 7061 743d 3020  bluray_compat=0
000001a0: 636f 6e73 7472 6169 6e65 645f 696e 7472  constrained_intr
000001b0: 613d 3020 6266 7261 6d65 733d 3220 625f  a=0 bframes=2 b_
000001c0: 7079 7261 6d69 643d 3220 625f 6164 6170  pyramid=2 b_adap
000001d0: 743d 3120 625f 6269 6173 3d30 2064 6972  t=1 b_bias=0 dir
000001e0: 6563 743d 3120 7765 6967 6874 623d 3120  ect=1 weightb=1
000001f0: 6f70 656e 5f67 6f70 3d30 2077 6569 6768  open_gop=0 weigh
00000200: 7470 3d31 206b 6579 696e 743d 3438 206b  tp=1 keyint=48 k
00000210: 6579 696e 745f 6d69 6e3d 3235 2073 6365  eyint_min=25 sce
00000220: 6e65 6375 743d 3020 696e 7472 615f 7265  necut=0 intra_re
00000230: 6672 6573 683d 3020 7263 5f6c 6f6f 6b61  fresh=0 rc_looka
00000240: 6865 6164 3d32 3020 7263 3d63 7266 206d  head=20 rc=crf m
00000250: 6274 7265 653d 3120 6372 663d 3233 2e30  btree=1 crf=23.0
00000260: 2071 636f 6d70 3d30 2e36 3020 7170 6d69   qcomp=0.60 qpmi
00000270: 6e3d 3020 7170 6d61 783d 3639 2071 7073  n=0 qpmax=69 qps
00000280: 7465 703d 3420 6970 5f72 6174 696f 3d31  tep=4 ip_ratio=1
00000290: 2e34 3020 6171 3d31 3a30 2e30 3000 8000  .40 aq=1:0.00...
000002a0: 0001 a065 8884 009f da8d e8a4 ff05 acef  ...e............
000002b0: ee9d b4d9 1fdc 8e2d f39d 01d8 71cd 7bae  .......-....q.{.
000002c0: 0000 0300 0003 0000 0300 0003 000e 1548  ...............H
000002d0: f727 763d b247 3400 0003 0000 0300 002b  .'v=.G4........+
000002e0: 2000 0024 4000 000c 2000 0005 dc00 0004   ..$@... .......
000002f0: a000 0003 03b8 0000 0420 0000 0590 0000  ......... ......
00000300: 08e0 0000 0be0 0000 1780 0000 2200 0003  ............"...
00000310: 0044 0000 0300 0003 0000 0300 0003 0000  .D..............
00000320: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000330: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000340: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000350: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000360: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000370: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000380: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000390: 0000 0300 0003 0000 0300 0003 0000 0300  ................
000003a0: 0003 0000 0300 0003 0000 0300 0003 0000  ................
000003b0: 0300 0003 0000 0300 0003 0000 0300 0003  ................
000003c0: 0000 0300 0003 0000 0300 0003 0000 0300  ................
000003d0: 0003 0000 0300 0003 0000 0300 0003 0000  ................
000003e0: 0300 0003 0000 0300 0003 0000 0300 0003  ................
000003f0: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000400: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000410: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000420: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000430: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000440: 0301 6f                                  ..o

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=2560
pts_time=0.208333
dts=512
dts_time=0.041667
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=113
pos=3648
flags=__
data=
00000000: 0000 006d 419a 236c 49ff a01f 768c ea00  ...mA.#lI...v...
00000010: 0003 0000 0300 0003 0000 0300 1665 7b6f  .............e{o
00000020: bb5a 9540 0340 d2a0 013d aa5e b925 b1d5  .Z.@.@...=.^.%..
00000030: 473e 0332 fb9a cceb 8899 5430 a709 9677  G>.2......T0...w
00000040: a04b 84ea 38f4 8761 4677 7028 c344 a537  .K..8..aFwp(.D.7
00000050: 8349 647a c109 7229 86c2 1311 3c1f 47ab  .Idz..r)....<.G.
00000060: 7b3f 0995 70e7 e188 f1e1 4cc3 f26e 19c8  {?..p.....L..n..
00000070: 40                                       @

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=1536
pts_time=0.125000
dts=1024
dts_time=0.083333
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=67
pos=3761
flags=__
data=
00000000: 0000 003f 419e 4178 8aff 0000 0300 0003  ...?A.Ax........
00000010: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000020: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000030: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000040: 0001 45                                  ..E

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=2048
pts_time=0.166667
dts=1536
dts_time=0.125000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=66
pos=3828
flags=__
data=
00000000: 0000 003e 019e 6244 5700 0003 0000 0300  ...>..bDW.......
00000010: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000020: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000030: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000040: 0145                                     .E

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=4096
pts_time=0.333333
dts=2048
dts_time=0.166667
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=71
pos=3894
flags=__
data=
00000000: 0000 0043 419a 6634 a4c1 2700 0003 0000  ...CA.f4..'.....
00000010: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000020: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000030: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000040: 0300 0003 008d 81                        .......

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=3072
pts_time=0.250000
dts=2560
dts_time=0.208333
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=68
pos=3965
flags=__
data=
00000000: 0000 0040 419e 8445 112c 5700 0003 0000  ...@A..E.,W.....
00000010: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000020: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000030: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000040: 0300 0145                                ...E

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=3584
pts_time=0.291667
dts=3072
dts_time=0.250000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=66
pos=4033
flags=__
data=
00000000: 0000 003e 019e a544 5700 0003 0000 0300  ...>...DW.......
00000010: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000020: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000030: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000040: 0145                                     .E

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=5632
pts_time=0.458333
dts=3584
dts_time=0.291667
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=71
pos=4099
flags=__
data=
00000000: 0000 0043 419a a934 a4c1 2700 0003 0000  ...CA..4..'.....
00000010: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000020: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000030: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000040: 0300 0003 008d 81                        .......

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=4608
pts_time=0.375000
dts=4096
dts_time=0.333333
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=68
pos=4170
flags=__
data=
00000000: 0000 0040 419e c745 152c 5700 0003 0000  ...@A..E.,W.....
00000010: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000020: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000030: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000040: 0300 0145                                ...E

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=5120
pts_time=0.416667
dts=4608
dts_time=0.375000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=66
pos=4238
flags=__
data=
00000000: 0000 003e 019e e844 5700 0003 0000 0300  ...>...DW.......
00000010: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000020: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000030: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000040: 0145                                     .E

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=7168
pts_time=0.583333
dts=5120
dts_time=0.416667
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=70
pos=4304
flags=__
data=
00000000: 0000 0042 419a ec34 a4c1 3700 0003 0000  ...BA..4..7.....
00000010: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000020: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000030: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000040: 0300 0003 0117                           ......

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=6144
pts_time=0.500000
dts=5632
dts_time=0.458333
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=68
pos=4374
flags=__
data=
00000000: 0000 0040 419f 0a45 152c 5700 0003 0000  ...@A..E.,W.....
00000010: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000020: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000030: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000040: 0300 0145                                ...E

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=6656
pts_time=0.541667
dts=6144
dts_time=0.500000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=66
pos=4442
flags=__
data=
00000000: 0000 003e 019f 2b44 5700 0003 0000 0300  ...>..+DW.......
00000010: 0003 0000 0300 0003 0000 0300 0003 0000  ................
00000020: 0300 0003 0000 0300 0003 0000 0300 0003  ................
00000030: 0000 0300 0003 0000 0300 0003 0000 0300  ................
00000040: 0145                                     .E

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=8704
pts_time=0.708333
dts=6656
dts_time=0.541667
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=595
pos=4508
flags=__
data=
00000000: 0000 024f 419b 2f34 a4c1 15ff 0000 0300  ...OA./4........
00000010: 0003 0010 ed01 9201 02da 2b1e 688b 66db  ..........+.h.f.
00000020: 003a 19d8 909c 9a0c 613e aba7 e814 ff9a  .:......a>......
00000030: ec3b 598d 6ae9 5b1d cd93 ccc2 b388 fdcd  .;Y.j.[.........
00000040: 35f5 de33 d191 322a 8b28 9013 229c 3ad9  5..3..2*.(..".:.
00000050: 6530 0000 79be 2c41 4b68 0923 732d feb3  e0..y.,AKh.#s-..
00000060: b8d8 19c2 fd12 9b59 d63d 0710 35df edd4  .......Y.=..5...
00000070: a8f3 ae60 002b ea10 ed18 07ac 48cd 9560  ...`.+......H..`
00000080: 3689 9b36 6cd9 b366 a980 0000 0408 0000  6..6l..f........
00000090: 0300 0888 0000 0300 2240 0000 0301 38f4  ........"@....8.
000000a0: 0000 0300 5461 2080 0000 0301 67f7 8d60  ....Ta .....g..`
000000b0: 0000 0300 13aa 77bb 2000 0003 0004 8bb6  ......w. .......
000000c0: a200 0003 0007 0000 0300 fae9 5639 aea4  ............V9..
000000d0: 06b9 7032 9375 dc1b 8096 38db 51d2 dc2f  ..p2.u....8.Q../
000000e0: 5732 00bf 8838 320a fba6 fa39 1abe edfa  W2...82....9....
000000f0: b4ae aafb 5f25 8002 c200 6f84 2012 0164  ...._%....o. ..d
00000100: 09c0 3d48 73fe c114 0456 0001 fda2 7242  ..=Hs....V....rB
00000110: ac97 6ab3 405c 047c cbbf 5a6d e5d3 63a7  ..j.@\.|..Zm..c.
00000120: 85ca 3d98 099e ef42 990f 46c0 25b1 e6e9  ..=....B..F.%...
00000130: 1c45 ce03 e8b2 7ce0 9d32 8004 d4ad ff07  .E....|..2......
00000140: c00e d237 21d7 f51c e7bf 184c 0dfd e08e  ...7!......L....
00000150: 0045 5127 064c 1b3f ecf0 8385 45d3 9820  .EQ'.L.?....E..
00000160: 50b2 55fa 5030 34f7 f004 3e44 ffbc 74b8  P.U.P04...>D..t.
00000170: 3106 5e1c 173e 7db5 8541 e14a 7bc9 c013  1.^..>}..A.J{...
00000180: 26c1 8cd8 3161 beaa a9c0 2cf3 9fd5 c4fc  &...1a....,.....
00000190: 8161 bd17 a016 6954 35e1 5d9d 2cf4 b180  .a....iT5.].,...
000001a0: fa03 642d 7d19 e6fa 8698 4472 4722 e707  ..d-}.....DrG"..
000001b0: a05c 14d8 5218 5e61 9245 8580 9a2d f9e9  .\..R.^a.E...-..
000001c0: 57ba aeea 1fe4 5053 86d5 9b91 ee13 0fdf  W.....PS........
000001d0: 8c96 592e a9e5 05aa 042c 101c 003c 08d5  ..Y......,...<..
000001e0: 5694 e9de 7575 8843 ecfe 960c 328c 0036  V...uu.C....2..6
000001f0: e8c5 5130 59bc 60d7 aa2f 71ce 0009 244e  ..Q0Y.`../q...$N
00000200: d820 66c4 6938 ecc0 55f5 8702 9910 02c5  . f.i8..U.......
00000210: ec00 6080 0cac fa34 49c3 80dd 3c80 343d  ..`....4I...<.4=
00000220: 0ccc 0723 d436 d384 3200 0dc8 2800 eb22  ...#.6..2...(.."
00000230: d3fa 78b2 bd2c 3e77 fbce 7e42 fbd5 475d  ..x..,>w..~B..G]
00000240: 8303 dfb1 ca31 2eda 85fe 8800 0003 0000  .....1..........
00000250: 0301 e3                                  ...

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=7680
pts_time=0.625000
dts=7168
dts_time=0.583333
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=194
pos=5103
flags=__
data=
00000000: 0000 00be 419f 4d45 152c 5700 0003 0000  ....A.ME.,W.....
00000010: 0300 71cf 693c 8a2a 9b01 ae82 5067 8603  ..q.i<.*....Pg..
00000020: ec4f 302f b75b b08c 3a54 15dd 488e 5b4e  .O0/.[..:T..H.[N
00000030: 4a00 342f 309b 8c9d 8214 6e22 0397 0d6e  J.4/0.....n"...n
00000040: c9da fe04 ee6b 5c5c f26b 945a 5671 a00e  .....k\\.k.ZVq..
00000050: 65e9 9ad4 b2f0 0d81 ad60 c815 6a36 5d54  e........`..j6]T
00000060: ee37 e22a 6f3b 28e9 7665 b453 8607 890a  .7.*o;(.ve.S....
00000070: 128b 809d dd54 baf4 f1f1 583d 4c4b c6bb  .....T....X=LK..
00000080: b1c1 0f01 8202 42ef 1465 a8b7 3252 7289  ......B..e..2Rr.
00000090: 329f db0f 40df 2205 6b0d 6282 abe7 c6bf  2...@.".k.b.....
000000a0: c8b7 bd4b 8503 7195 d546 bc59 cf59 1e4e  ...K..q..F.Y.Y.N
000000b0: 020b b6a2 488a 7a48 001f 5000 0003 0000  ....H.zH..P.....
000000c0: 3021                                     0!

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=8192
pts_time=0.666667
dts=7680
dts_time=0.625000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=484
pos=5297
flags=__
data=
00000000: 0000 01e0 019f 6e44 5700 0003 0000 0487  ......nDW.......
00000010: 872a c02a 18ad 130e e1a2 e3ed 516d 7b11  .*.*........Qm{.
00000020: 26a6 a326 e977 532b d7d8 0000 0300 0025  &..&.wS+.......%
00000030: b1f4 02e1 9bcd eb46 b738 62e7 3a3d f1bf  .......F.8b.:=..
00000040: cde1 684d b175 590e 8c9d 5b4e 830a afff  ..hM.uY...[N....
00000050: 1051 a1e7 659c 6c98 dd60 a489 1ae5 c62e  .Q..e.l..`......
00000060: 7169 abc9 0afc 49f1 3273 f92e ac84 6092  qi....I.2s....`.
00000070: 2706 4c0d 1321 7161 2ba6 ebf8 096c 3cd3  '.L..!qa+....l<.
00000080: 0ff4 b4ac e59a a978 0724 9905 8c76 3a2d  .......x.$...v:-
00000090: cf2c fd77 bfad 4789 d1b0 67ca 387a 6661  .,.w..G...g.8zfa
000000a0: 63c8 e2eb ffa7 8a19 d793 56c8 6be9 2f3c  c.........V.k./<
000000b0: d289 b819 3e45 3631 9504 6b4d d178 f4fc  ....>E61..kM.x..
000000c0: 9df8 b0a5 036a 99bf 83b0 2ac5 3c4a 8289  .....j....*.<J..
000000d0: 2050 df68 c1fc c1d2 35f0 8e2a 2732 f298   P.h....5..*'2..
000000e0: db8b 3b79 288d 6aea 7eea 27d7 f6ac e4bb  ..;y(.j.~.'.....
000000f0: e105 4c17 7df2 98ff 6476 7310 0e13 9ab0  ..L.}...dvs.....
00000100: 6232 927b 21c7 e138 c727 d7e7 d36b 2b8c  b2.{!..8.'...k+.
00000110: 8741 3dae 076d cf6a 0202 7811 9ec7 61d9  .A=..m.j..x...a.
00000120: 289b d2e4 5f0a f53d f6d7 5244 0bd9 cde6  (..._..=..RD....
00000130: 4266 fe60 faab b969 2c88 9a96 6d60 810a  Bf.`...i,...m`..
00000140: 38d8 b5da 7c45 62e0 455f 1ec7 fa7d 02ff  8...|Eb.E_...}..
00000150: 1e0c 1c9c 218d 35a7 0ed9 cf56 009d b512  ....!.5....V....
00000160: a2d8 67a4 9b65 8517 bd8a f48e 532a 5c49  ..g..e......S*\I
00000170: 67ca de53 14a8 48c2 f731 9bfa 4893 a389  g..S..H..1..H...
00000180: 1b70 42c3 52c7 3214 f6e2 cffe b650 c149  .pB.R.2......P.I
00000190: 0cb2 2e93 1ac0 b3f0 771a b420 f2e0 ffae  ........w.. ....
000001a0: 8e91 7ab6 3b3b 6302 7731 25d7 4686 bfc9  ..z.;;c.w1%.F...
000001b0: b494 40b9 7b21 be18 086d b7c8 c327 23eb  ..@.{!...m...'#.
000001c0: 013e 2198 c091 29c3 5d45 c29e 2120 b36c  .>!...).]E..! .l
000001d0: 3c61 d9ae ad1c a5e9 81e1 56b7 1000 0003  <a........V.....
000001e0: 0000 0ad9                                ....

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=10240
pts_time=0.833333
dts=8192
dts_time=0.666667
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1428
pos=5781
flags=__
data=
00000000: 0000 0590 419b 7234 a436 0228 1384 5700  ....A.r4.6.(..W.
00000010: 0003 0000 0300 1d1f 0792 8c7b 44f3 d84b  ...........{D..K
00000020: e22e e72f 21ff 15b0 fe43 60b4 6d93 b9de  .../!....C`.m...
00000030: d9d3 dbce 4266 5580 f9f6 2c21 393f 07e8  ....BfU...,!9?..
00000040: ca24 3382 0084 1115 ccd8 68ce 3f7d 9a96  .$3.......h.?}..
00000050: 8e3d 6c27 0141 88a7 1fba 29c9 0e99 7317  .=l'.A....)...s.
00000060: 8d32 f21c 5390 bcef 5993 3594 0360 a074  .2..S...Y.5..`.t
00000070: 672d f1e4 b7f6 a954 09c8 62e6 3313 c612  g-.....T..b.3...
00000080: 780d f8ae c400 0760 127d cef8 bbeb c746  x......`.}.....F
00000090: 039c 9b48 713d b14d 51c0 0c48 f052 7fb7  ...Hq=.MQ..H.R..
000000a0: 3d2f a890 5646 34e8 625e 4fa9 f527 6ffa  =/..VF4.b^O..'o.
000000b0: be00 b2ab 203f c01d f841 ab10 1f74 f16c  .... ?...A...t.l
000000c0: 15ec 1e45 5747 88bd ef69 60bb 8323 ec3d  ...EWG...i`..#.=
000000d0: d970 4fad 8490 4eda 2ccb f3d4 253e 64ce  .pO...N.,...%>d.
000000e0: 7fd3 a414 c930 0003 fde0 99b2 800a 4485  .....0........D.
000000f0: f5cb 0744 d9c0 7749 4532 f31a dec6 4942  ...D..wIE2....IB
00000100: 7e0e 7c28 c4b0 a304 0042 f3ec aea3 5af4  ~.|(.....B....Z.
00000110: 424c 2e81 e714 341c 73f8 0e28 6c13 a94d  BL....4.s..(l..M
00000120: a902 e45a 6f4c 27df e96b 0562 02be 15c5  ...ZoL'..k.b....
00000130: 3d56 92a9 1ed9 1f22 f0b3 0386 a79d f707  =V....."........
00000140: 2043 a954 509b 8418 a00d cacf c770 f380   C.TP........p..
00000150: 971b 5342 29f1 5a8d 8c24 3969 537c 737e  ..SB).Z..$9iS|s~
00000160: 3a1a 1a6b 70b0 e101 d430 b5d9 d941 9d81  :..kp....0...A..
00000170: 55cb ad30 3144 ce62 42af 5412 a489 ae37  U..01D.bB.T....7
00000180: 7a24 307a ca88 6cf0 b984 0805 9f59 9d84  z$0z..l......Y..
00000190: 497b ce1a 2d0a 5a0a 28ea de85 c675 9c28  I{..-.Z.(....u.(
000001a0: f92a ba74 2a70 8710 c8f1 3e15 815b 6415  .*.t*p....>..[d.
000001b0: cb66 2c09 6ca1 3c37 f30e f687 c786 6b81  .f,.l.<7......k.
000001c0: f875 4283 849e 6065 6078 0010 9ea7 423f  .uB...`e`x....B?
000001d0: 7cf2 d715 df2c 9ab8 dee4 bd67 320c ca6a  |....,.....g2..j
000001e0: 964a 5c6d f93e 5369 6842 a598 7271 1eca  .J\m.>SihB..rq..
000001f0: ad66 378d b21f fbc2 0716 8bcc 00ff 2c0d  .f7...........,.
00000200: b015 b1f7 9d46 8b00 86b3 ff03 a791 40fe  .....F........@.
00000210: ee85 28ad ddd9 0267 a9b1 10dc 0004 a985  ..(....g........
00000220: 3b78 2f45 b2fc f3d9 a343 6816 9d51 d4e8  ;x/E.....Ch..Q..
00000230: aa09 832a 2368 000a ae4e 557a c7c9 0e17  ...*#h...NUz....
00000240: 7260 a5b7 18ef 0807 e12f 4655 cbdb f294  r`......./FU....
00000250: ad3c d61e 77f0 012d fd15 4ec3 5fc1 cc57  .<..w..-..N._..W
00000260: aaa8 c4a5 3c01 4494 c924 98d1 f1f0 1089  ....<.D..$......
00000270: 3404 510d 1cd7 0c00 c977 379c ceae 3d3d  4.Q......w7...==
00000280: 8492 61b4 21ca 8032 2174 02ad 3f9f 3a5a  ..a.!..2!t..?.:Z
00000290: 2396 2c77 c6d7 f84b 4ecf 556c 0211 7920  #.,w...KN.Ul..y
000002a0: b3c9 7997 ae4b 511e 6035 b323 ea6f 56d8  ..y..KQ.`5.#.oV.
000002b0: 42ea 4406 1314 5d2b 88d8 2960 8060 7be7  B.D...]+..)`.`{.
000002c0: fbdd 3ab7 56c0 02bc 524d e89f 8f51 c136  ..:.V...RM...Q.6
000002d0: 2344 5dfa 3da8 baba 3afd 39ca ce8f 9063  #D].=...:.9....c
000002e0: 0e9a 3812 e92f 36a8 d946 5d19 adaa 4047  ..8../6..F]...@G
000002f0: 483f 2cef 78a1 5f24 83b8 7817 307e e08d  H?,.x._$..x.0~..
00000300: a307 a3e6 c6b4 d670 27c3 8630 f963 84ef  .......p'..0.c..
00000310: 0278 1c76 8cea 74ec 2400 c8dd 1801 85f7  .x.v..t.$.......
00000320: 12f0 a5fb 4e74 26b2 270b 06be c1eb ab61  ....Nt&.'......a
00000330: f0fd a277 b0d0 b066 56cf 634c 0211 605c  ...w...fV.cL..`\
00000340: c14e c61e 089c 0e44 43bd ef60 fc91 780a  .N.....DC..`..x.
00000350: 957d 76ae a37d 1422 b2ad 1889 62b3 056e  .}v..}."....b..n
00000360: a1ef d566 2f17 c29d fc25 ddbe 698f 4382  ...f/....%..i.C.
00000370: 54d6 adc2 2667 3484 984e 192e f425 3434  T...&g4..N...%44
00000380: ffe9 4b3a 5dc3 2544 3189 d15d 7044 68e9  ..K:].%D1..]pDh.
00000390: ebff a289 0096 204b 652f a329 79c0 0000  ...... Ke/.)y...
000003a0: 8f5e 53cb 2c30 5db3 3c34 d9e3 b192 0992  .^S.,0].<4......
000003b0: 9f32 00f3 d55a 5c98 3d25 c09f 47f0 087b  .2...Z\.=%..G..{
000003c0: 4890 9e7c 2dc5 fc3e 02c9 54c0 1957 c4d0  H..|-..>..T..W..
000003d0: 018d 5ec4 b9ad 2de5 3277 896d 1394 a439  ..^...-.2w.m...9
000003e0: 106b 1883 d055 e3ac d849 cd00 cf4c e64a  .k...U...I...L.J
000003f0: 30a2 b29e c2f5 27c6 aff8 2821 f0cb ca5a  0.....'...(!...Z
00000400: 0c89 b021 94a1 c47a dedd 19ae 533a 5328  ...!...z....S:S(
00000410: 1faa 676b 9245 746e 1b7b 0351 bfa6 83d1  ..gk.Etn.{.Q....
00000420: d8b9 c96f ab72 619a b756 a759 50df 3daf  ...o.ra..V.YP.=.
00000430: 953f 03ca be6c a0c0 616a ee15 75d8 1e3b  .?...l..aj..u..;
00000440: aa31 aa59 1858 a8fd 1866 a952 617d a445  .1.Y.X...f.Ra}.E
00000450: 458a 2f82 a86d f6c9 f22a 75fa 8dbd c3d8  E./..m...*u.....
00000460: 102c bc6a 27de f2d2 7ddd 0986 da08 363a  .,.j'...}.....6:
00000470: c08d 88e5 c1bc 340a 09a1 faa5 fff3 3339  ......4.......39
00000480: 9ddc 9adc 7ca6 0b6a 5c25 572b 6a34 f2d7  ....|..j\%W+j4..
00000490: 97d4 9cd7 e80b d6dc 01c2 0e94 1a16 370a  ..............7.
000004a0: 9d63 662d fc54 a77e 5ed0 51f3 f8fc ac85  .cf-.T.~^.Q.....
000004b0: d382 4912 0057 0a36 09b4 0392 f0b7 56f4  ..I..W.6......V.
000004c0: e9b5 bc05 32fe 9497 c9eb 09fd 727d 6c14  ....2.......r}l.
000004d0: 8810 18a9 dd49 1d94 16ac 40da 3615 ea3c  .....I....@.6..<
000004e0: 1b64 7af1 5634 a464 693d 0abe 2428 8bc8  .dz.V4.di=..$(..
000004f0: a93c 1c8f d660 71e7 584a 9287 341f 776e  .<...`q.XJ..4.wn
00000500: d6a2 f040 4fa7 638f 0fbb 8d96 37de ca2f  ...@O.c.....7../
00000510: a103 d361 2202 2464 2251 4cdd 9814 55e4  ...a".$d"QL...U.
00000520: 8085 6266 92a0 6cc5 3a57 d8c6 7ab4 7a17  ..bf..l.:W..z.z.
00000530: af1c b762 c0ff 5681 2aea a143 ed28 bb9d  ...b..V.*..C.(..
00000540: a615 5791 69bb 4e13 41e0 9081 8fcd 1356  ..W.i.N.A......V
00000550: c4fb b118 12b8 342c a657 7645 f005 a967  ......4,.WvE...g
00000560: fbac 23a2 000f 90b6 2625 e09b 1cc3 9bc0  ..#.....&%......
00000570: d922 70ef 4de5 3e4d 2246 2c06 11ad 4c06  ."p.M.>M"F,...L.
00000580: 2d55 1bf1 8173 ca9a b0ba 4c00 0003 0000  -U...s....L.....
00000590: 0300 26e0                                ..&.

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=9216
pts_time=0.750000
dts=8704
dts_time=0.708333
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1996
pos=7209
flags=__
data=
00000000: 0000 07c8 419f 9045 152c 5700 0003 0000  ....A..E.,W.....
00000010: 0799 b8b8 18ef d214 5d81 5471 50f4 ae7a  ........].TqP..z
00000020: cf5a ea5f 53bf ba80 96eb 9316 40e5 3018  .Z._S.......@.0.
00000030: f6cd a0bf f1f5 6f4b 2f62 13b7 8654 9a3c  ......oK/b...T.<
00000040: 76a6 a87d e4c3 e440 847f 9888 611a fe4c  v..}...@....a..L
00000050: 656e 3802 c143 4373 29d3 4db1 166d ee6c  en8..CCs).M..m.l
00000060: 87f0 d6f9 d070 e7b0 1f52 1ae0 3ca6 3591  .....p...R..<.5.
00000070: 0b4b cc1f efc2 6521 3b7d 021d fbc7 ce50  .K....e!;}.....P
00000080: 4bd6 bf6d f4c0 000a 8ded 3191 d5ec aa34  K..m......1....4
00000090: 1ca0 6a1b ec0e 57af 9fa6 281c 0fa2 994c  ..j...W...(....L
000000a0: 7815 3f0a 3db1 6f98 b97d 6e08 9bdc cd5c  x.?.=.o..}n....\
000000b0: 9756 14ee 0d14 9df3 4c1a 0e96 050f fdae  .V......L.......
000000c0: ed03 758c 7152 a744 9d74 a824 a93a 26ea  ..u.qR.D.t.$.:&.
000000d0: 0a00 7570 008b 3df7 5d22 391b 5fcf 945d  ..up..=.]"9._..]
000000e0: 0fb2 e93e f2fc e152 6fde 4525 ee25 82ed  ...>...Ro.E%.%..
000000f0: b794 7fc2 e2fa 218e b102 0067 f111 0b8c  ......!....g....
00000100: 683c 5a32 84da 2d3e b148 c603 2dc1 f51f  h<Z2..->.H..-...
00000110: bb8d 1f93 85e3 88d1 098e 7e26 53d0 5f6a  ..........~&S._j
00000120: 4330 fd2a 82f5 20a0 eea6 8853 d5f8 a68d  C0.*.. ....S....
00000130: 7040 4e25 77f6 407b 71c8 0683 e236 2f89  p@N%w.@{q....6/.
00000140: 423d d2a7 53af 2e8c b085 7a5d 4e9c eb98  B=..S.....z]N...
00000150: 1972 970a f6b2 c998 2913 983b d423 255f  .r......)..;.#%_
00000160: 9eec cbfa 86a0 d6ad 325d 035e 5c36 574b  ........2].^\6WK
00000170: f7fa fcf7 3ea6 d939 28d6 9214 e512 616e  ....>..9(.....an
00000180: 11a0 defb b74c 799a 4ade 1d4a 60e3 a01b  .....Ly.J..J`...
00000190: 300d 7cb7 1d1c 0e1e 0679 5955 ea21 e3c5  0.|......yYU.!..
000001a0: 3d38 f47f cf85 5595 04f7 94e3 c8bc 6d7e  =8....U.......m~
000001b0: 4963 566e 9287 0479 f121 c20d 7daf 5595  IcVn...y.!..}.U.
000001c0: e5b2 6d59 f5a3 3804 1cee 25da f75c 6bbf  ..mY..8...%..\k.
000001d0: 50d0 4571 862e 08e8 17b6 c415 2e20 5023  P.Eq......... P#
000001e0: e91e eaae 08ce c72f 53ce 133f 5ee8 67a0  ......./S..?^.g.
000001f0: 37df 6aba 5012 0106 1c87 8a66 9841 3bf3  7.j.P......f.A;.
00000200: 6c7f 152d 85c8 80d6 f154 fe97 c4fd fb07  l..-.....T......
00000210: 0fca 3857 cbba 3774 97f7 fc8d 3f3a c171  ..8W..7t....?:.q
00000220: 57e0 79f3 5afa 73b1 4cf7 c288 b6a5 e670  W.y.Z.s.L......p
00000230: 48a4 ada1 bb16 be3c 654c d384 9e68 6ff6  H......<eL...ho.
00000240: 8f64 3d15 84dc e02b 0385 adde d2af 9c73  .d=....+.......s
00000250: 9b59 0296 1c6d 4b3d 0fd3 f403 10e4 4053  .Y...mK=......@S
00000260: c5aa a108 d088 2788 1603 29ed 78ae a71c  ......'...).x...
00000270: 01f0 4da3 f4cc 38a5 8f01 dc27 51f8 1200  ..M...8....'Q...
00000280: e1ec 961e b9e2 59d1 4625 7629 aaea d7c6  ......Y.F%v)....
00000290: f594 37d0 0892 30a7 8142 8d17 2e8f aeaa  ..7...0..B......
000002a0: 2d4e 44b1 9a92 1099 bfbe c7f6 3d6f 82b5  -ND.........=o..
000002b0: e08f 5eb5 1705 8f4b c36b d422 520f bfb3  ..^....K.k."R...
000002c0: 9575 1f25 1496 1876 0a0f 9123 fd1c 2c97  .u.%...v...#..,.
000002d0: b516 a2d3 f223 8c48 e4ae 0761 4627 118f  .....#.H...aF'..
000002e0: cf5a 0650 0ecb cc30 9fd2 72a1 a73c 902d  .Z.P...0..r..<.-
000002f0: 9253 7adc c114 db95 1e91 8724 f43f 0f38  .Sz........$.?.8
00000300: dedd ac57 a205 3bb6 dd83 0c32 97b4 8aa5  ...W..;....2....
00000310: 2d5c 0a16 9171 21f3 adf6 57ce 5f91 81a5  -\...q!...W._...
00000320: c41a 08b8 1fd0 3efc fc31 c621 824b 1b2b  ......>..1.!.K.+
00000330: 3ab8 d59b fddb bc87 f9d4 8e95 2389 883d  :...........#..=
00000340: 080b 0e81 2a9d bc11 907a 3a70 4399 83e4  ....*....z:pC...
00000350: b218 f4af 59f4 37c0 48af 32c3 347b 047a  ....Y.7.H.2.4{.z
00000360: 396b 513a 729d 14d1 bbdd b6d2 0629 707d  9kQ:r........)p}
00000370: bbb4 358c 8c38 3765 290e 5382 26d1 a214  ..5..87e).S.&...
00000380: 3b12 ea0d 8b84 0228 fb1e 5545 c4a6 2b7e  ;......(..UE..+~
00000390: bbdd dddd dddd dddd dcf6 842b 19c1 abc2  ...........+....
000003a0: 4e8b a2dc bcd3 c7b0 dd00 6c66 f1e5 55d1  N.........lf..U.
000003b0: 7a9d e36b ab1d 2e13 a343 17a9 17e1 fd62  z..k.....C.....b
000003c0: 816d 53ab b85b 92b7 256e 4ad4 8a18 f940  .mS..[..%nJ....@
000003d0: a6fa 97ad fb5b 1cc8 e3a8 6f9f 40b8 0ba4  .....[....o.@...
000003e0: 3b90 00df dfa3 abef 85e3 d998 118e b98e  ;...............
000003f0: adb9 9d93 d8e9 4b6b b85b 92b7 2593 f6b5  ......Kk.[..%...
00000400: f04b 0f6a f58f ff9a 8026 a157 b46e d8c0  .K.j.....&.W.n..
00000410: 3dd0 ff63 6481 116d 5760 ddf5 c9d1 5ca2  =..cd..mW`....\.
00000420: 236b e81c 92b3 9b5b e74a 9e5f 0997 b3f1  #k.....[.J._....
00000430: 158c 0365 8b36 cda3 76f4 889e 0daf c949  ...e.6..v......I
00000440: c33d 8632 3c7d 7c33 4150 4def 1bd0 d5f1  .=.2<}|3APM.....
00000450: 18f9 a09a 7a56 0f88 fd75 1f22 665e bc8d  ....zV...u."f^..
00000460: e4d3 c2af f70a c678 51c5 54b7 a096 6579  .......xQ.T...ey
00000470: f323 5969 8cf2 71b9 7dba 9613 a9cf 2226  .#Yi..q.}....."&
00000480: 8a06 1c9e fe6c 9df1 c0ac 7b8a 4f88 ef3c  .....l....{.O..<
00000490: 823c a48a 2b05 8b55 066d 133d dbd0 d9bc  .<..+..U.m.=....
000004a0: 7a21 7b57 61c7 21d5 67b1 5f12 4cba 4ecf  z!{Wa.!.g._.L.N.
000004b0: b2ac 8a57 78bd b031 29f3 57f4 f6aa a6a9  ...Wx..1).W.....
000004c0: dd7e 33a9 102b f538 3c94 61cf 7f71 052a  .~3..+.8<.a..q.*
000004d0: d6cb 5f45 aaae e0c4 525b c5ad 3c14 9f3f  .._E....R[..<..?
000004e0: 1ba0 aa06 3255 0aac 2fe2 c7c3 baeb 0737  ....2U../......7
000004f0: 3095 4852 2a00 2649 74bb bca6 b69e 6b67  0.HR*.&It.....kg
00000500: 9945 2812 1f62 745e 5973 f1ac 2951 9373  .E(..bt^Ys..)Q.s
00000510: e8f7 f7ae fcb3 e71a 164b 46d5 4731 4f73  .........KF.G1Os
00000520: f2f0 cd98 7898 6dfc 647c 1a92 ee60 0055  ....x.m.d|...`.U
00000530: eff6 3def 3491 cb81 f5f3 5127 a578 63db  ..=.4.....Q'.xc.
00000540: f778 1f0c 7018 2c9d 8ac3 0778 7b44 bd47  .x..p.,....x{D.G
00000550: f9c6 1064 b4fe c0f3 d1c0 ece6 13ab 507c  ...d..........P|
00000560: 2c6f 704c 265d aa86 3b8f 1acf 2780 ebef  ,opL&]..;...'...
00000570: d694 4896 b68b 07b5 1d78 c68a 2511 924c  ..H......x..%..L
00000580: 018e ca06 3c0b 47ff 758c 9091 1fda 339f  ....<.G.u.....3.
00000590: 8bd7 14f3 86ef 2be4 5317 1f63 47af eca7  ......+.S..cG...
000005a0: a5dd fb66 2aeb 8846 3fc0 e896 dc34 c6f8  ...f*..F?....4..
000005b0: 6922 8ec5 a667 c32c 42ec 38e3 48fd 77cb  i"...g.,B.8.H.w.
000005c0: 71b0 58b3 9d7f b0b0 a4e1 5c6d 7454 3922  q.X.......\mtT9"
000005d0: f8f3 c8ad 6a1f 3609 c883 d892 f01e 86eb  ....j.6.........
000005e0: 5dfa 1f94 7e8a d8b5 07ee d67b 89a2 cd6f  ]...~......{...o
000005f0: dd5a 8b3e d2de bc93 4ca5 2ef9 8a0c c2e8  .Z.>....L.......
00000600: bc92 2acd 49e4 282e abd6 be9e f8f2 bad6  ..*.I.(.........
00000610: 636a 2176 0aeb 9bb4 03ea 5740 5232 bbca  cj!v......W@R2..
00000620: a39c 6093 64f8 3bb9 f8bd 40e9 79b4 e6ea  ..`.d.;...@.y...
00000630: c3b4 5f92 f9a4 d929 ecfe a5dc 96bb 1f84  .._....)........
00000640: e230 9b87 bf7c f0b0 545a f1b7 9eab e4e9  .0...|..TZ......
00000650: 524c 1e50 13cf 12e0 9646 8a98 2320 c3a8  RL.P.....F..# ..
00000660: 90df eb09 84e8 af3d b3a1 5ace 1f3b 96a4  .......=..Z..;..
00000670: 5f30 c250 56bd f913 eb7f 94e4 67f7 dcd3  _0.PV.......g...
00000680: 5a70 bb5a 9e81 fda8 9ddb 6bf2 c802 a6da  Zp.Z......k.....
00000690: 6059 0c0c 190d 4f7b 9b3a 6aed e33c 926e  `Y....O{.:j..<.n
000006a0: 8c9f 2fdc c7ae 3b56 ed78 f019 ee12 305a  ../...;V.x....0Z
000006b0: 3e52 5af3 0bde 072a 86e3 fbd7 2caa 40b1  >RZ....*....,.@.
000006c0: bb1e 2992 bb66 2231 b9c7 ffbd c15d ff5c  ..)..f"1.....].\
000006d0: 70d5 0e50 d44a 2fbd 623f 9df9 ab68 bb77  p..P.J/.b?...h.w
000006e0: 171e d1a5 adbd d06f fec1 e785 fd2e 9e5f  .......o......._
000006f0: 360e 5f0c 5bca f312 1cc4 e09c c8db 8daf  6._.[...........
00000700: 2758 4a11 07b5 b407 4b86 a729 dd82 c744  'XJ.....K..)...D
00000710: a12d 029c a968 9f25 0252 b62b 3e48 fb4c  .-...h.%.R.+>H.L
00000720: dea3 1e9e 34b2 1921 f365 599b 34a8 ba04  ....4..!.eY.4...
00000730: 1c37 41fc ed53 9ca0 c4a1 532f c999 c47f  .7A..S....S/....
00000740: a3ea 93b5 8546 6e01 59b2 6000 98b4 6db9  .....Fn.Y.`...m.
00000750: 6376 b875 e95c 10cf f319 5536 40e1 4e10  cv.u.\....U6@.N.
00000760: 4ea3 e227 7d3d 0891 8181 3787 1923 fe3f  N..'}=....7..#.?
00000770: e69c 7d7a a763 b208 9924 e480 0db2 c129  ..}z.c...$.....)
00000780: 3026 e284 039e 8a8a b4b3 37e4 e528 7627  0&........7..(v'
00000790: b1a2 ff31 09fb 6b3c 5274 b880 e591 735a  ...1..k<Rt....sZ
000007a0: 70ad 10f0 4f64 5430 e94e fe3c ea29 1919  p...OdT0.N.<.)..
000007b0: 7b84 4d46 7765 c9e6 f28f 854a a2d0 25aa  {.MFwe.....J..%.
000007c0: ee2d b62c 752e 0000 0300 01d3            .-.,u.......

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=9728
pts_time=0.791667
dts=9216
dts_time=0.750000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1241
pos=9205
flags=__
data=
00000000: 0000 04d5 019f b144 5700 0003 0000 07bd  .......DW.......
00000010: 2d67 6635 de49 69f3 79ab a30a b7b5 9b96  -gf5.Ii.y.......
00000020: 363f 1cf2 51fa b5d3 6065 4bd6 9ce8 d945  6?..Q...`eK....E
00000030: 55e9 32cd 6be4 8d48 bc99 b1a1 c28e 4b26  U.2.k..H......K&
00000040: 8ab6 b7dd 1f9d 4b23 2938 8d74 2960 b2be  ......K#)8.t)`..
00000050: a8b3 9c57 d221 fa5e dcb3 032a d216 b345  ...W.!.^...*...E
00000060: e377 9656 8000 0003 0001 134e 0a32 c056  .w.V.......N.2.V
00000070: 03d1 1e1e 4d75 f38c 5c30 a72f 7494 0b7b  ....Mu..\0./t..{
00000080: 4c2e dbf3 6cd0 0500 70ab a2a1 ea89 e588  L...l...p.......
00000090: 3c95 7a89 0e67 acc0 3670 e089 8f3e 6704  <.z..g..6p...>g.
000000a0: d484 0eb9 9da6 f04e e4e7 04ac 5d12 2827  .......N....].('
000000b0: 2f90 c6a5 b149 042b f07b c198 ecea 07bf  /....I.+.{......
000000c0: 0e89 81d8 293c 047b 4e77 13bb 09e0 9daf  ....)<.{Nw......
000000d0: adf0 44ef cc3a a28d ccc4 0199 6317 9518  ..D..:......c...
000000e0: e93e 1c6a 2e29 a390 9d53 294e efeb 699b  .>.j.)...S)N..i.
000000f0: 6a91 aedc 1050 0e8e 6880 3cdb 8d41 2ad7  j....P..h.<..A*.
00000100: 17be d963 59c0 9b31 ad79 b68c df0b 5952  ...cY..1.y....YR
00000110: a8d8 5a1e fb35 2933 8cd5 e87b 6438 e192  ..Z..5)3...{d8..
00000120: 3fdc 59a1 88bd 2242 98d0 8636 f4f8 d013  ?.Y..."B...6....
00000130: 230d 702f 5b41 ac28 1160 31d9 8e79 ab95  #.p/[A.(.`1..y..
00000140: 285d d333 0e2e d332 1e86 2184 303c da17  (].3...2..!.0<..
00000150: da15 4f22 e795 0f7f 01a2 2f17 5968 1ad2  ..O"....../.Yh..
00000160: 732a 9064 0f5f a96d 2494 2cac c3ce 7444  s*.d._.m$.,...tD
00000170: 9c6b a82b 7711 eb4d b768 6031 4e25 0956  .k.+w..M.h`1N%.V
00000180: ceaf eb3d 02d8 b145 22bc 4b02 b1f2 dd24  ...=...E".K....$
00000190: 3d98 a0a7 60d0 7823 5de3 0d94 ead2 4703  =...`.x#].....G.
000001a0: 6ab7 9384 0f2b c6cb b94f ea74 9248 526e  j....+...O.t.HRn
000001b0: dff9 22c2 91e2 b822 9ea5 d48b 2999 1ef3  .."...."....)...
000001c0: b579 90b5 4bef 24fc f593 bcc7 fb2c b3de  .y..K.$......,..
000001d0: 832c 3db4 38a8 75b3 4214 7968 4734 6dd5  .,=.8.u.B.yhG4m.
000001e0: bc2a d6d9 f29e 631f 9388 ac7c bb88 e3d0  .*....c....|....
000001f0: 1a8b 65fd 5864 2e70 d481 61a2 01ac c53e  ..e.Xd.p..a....>
00000200: 8c43 8d55 b2bd a652 971f 5996 087e f81f  .C.U...R..Y..~..
00000210: ceac fb90 ac07 0986 c507 62be 547b 5ca9  ..........b.T{\.
00000220: 5dba c0b2 2777 00a2 516f 1b92 6801 cf8c  ]...'w..Qo..h...
00000230: bb66 e816 1d97 61e1 83f4 33ad dc86 4593  .f....a...3...E.
00000240: 573f fef0 943b d812 c933 271b b723 99d9  W?...;...3'..#..
00000250: f263 fd53 ed37 6b4b 634d b183 e8db 0247  .c.S.7kKcM.....G
00000260: 5c20 291b 7586 e8bc ca0b 12d3 674b 5aef  \ ).u.......gKZ.
00000270: 5db7 64b8 e684 5d2a 8e15 1c01 ed24 5f4e  ].d...]*.....$_N
00000280: ec32 915a 441a 50f7 2772 82b8 fdfa c952  .2.ZD.P.'r.....R
00000290: 2c40 81aa 936e dcd5 02a9 55d4 8ed9 982f  ,@...n....U..../
000002a0: 18cc 00f7 ec2c bc02 fc3d 9236 21be 2020  .....,...=.6!.
000002b0: bba2 6a91 907e c906 1791 1a06 cc49 915f  ..j..~.......I._
000002c0: 9731 70b8 2ed9 9501 67e9 24d2 4dc6 7a87  .1p.....g.$.M.z.
000002d0: 57c3 3401 a852 6d7e 81da 0093 65f4 218b  W.4..Rm~....e.!.
000002e0: 221d 50f7 7046 1664 6dd4 acf9 d055 79e7  ".P.pF.dm....Uy.
000002f0: abee e310 5207 ca57 86bb 9fe7 4df2 db2e  ....R..W....M...
00000300: 688f 5b24 1c71 1141 47bf ef48 3b8f 16d4  h.[$.q.AG..H;...
00000310: ec6f 1b34 460e d2dc a7af 4f01 27e7 df3b  .o.4F.....O.'..;
00000320: 7175 8a72 7b16 42ca de77 767d 90b7 97c6  qu.r{.B..wv}....
00000330: 3b56 551b 89b9 ef7f 7052 0368 0091 5205  ;VU.....pR.h..R.
00000340: a027 a098 ccf1 bee9 bb9b c93a 8fa3 bb8b  .'.........:....
00000350: 7043 0616 7db1 7d60 670f 311e 69db ea0d  pC..}.}`g.1.i...
00000360: fd41 fe01 77fb eaf2 ea91 e56a 314d 39d1  .A..w......j1M9.
00000370: 6d1d 0293 8241 a6b3 422b 5b28 7df7 56b4  m....A..B+[(}.V.
00000380: 6ccd 4849 d66c 1e91 8772 af11 f1d3 8516  l.HI.l...r......
00000390: 6b9e d8c9 6f08 574f 5e0b fba4 57b8 3125  k...o.WO^...W.1%
000003a0: 6be1 7109 a300 fe18 81c5 5fe7 92b9 33a7  k.q......._...3.
000003b0: 1357 1221 197a 1144 3bd5 1d53 edf1 9a71  .W.!.z.D;..S...q
000003c0: 876b ed55 a7a4 bd32 ecd1 6c9a 4131 731d  .k.U...2..l.A1s.
000003d0: 6d82 6016 f8b3 ff0a 1e6b 3f76 a5e8 47f3  m.`......k?v..G.
000003e0: 8671 5914 4209 f8f8 345c 6744 d5e9 64a5  .qY.B...4\gD..d.
000003f0: 35f6 677f 0363 12e6 0706 5c01 5ea5 445c  5.g..c....\.^.D\
00000400: d0f7 3003 f11e ab35 2952 50f3 081f 28af  ..0....5)RP...(.
00000410: e507 1a0a 9afb 8d2e 5dfc 7dc1 1c43 e03f  ........].}..C.?
00000420: a5ff f3bf bd1c a83d beba fa60 184b d911  .......=...`.K..
00000430: c255 2126 ae3a f16e de88 82cf 775a 5578  .U!&.:.n....wZUx
00000440: be19 fa5c 9e6d 4c01 74aa b72e 0c59 4f95  ...\.mL.t....YO.
00000450: 5a45 f29d 2de6 8cda ec94 fd01 d62d 4f60  ZE..-........-O`
00000460: d700 46c8 b699 9246 a980 7568 72d4 95e9  ..F....F..uhr...
00000470: 6096 348b 79b6 7082 6d5d e221 3078 6213  `.4.y.p.m].!0xb.
00000480: c954 7153 811a 9a5f 0f2b ae5b 04b7 b1e2  .TqS..._.+.[....
00000490: 5f24 b2f0 ba0e 0427 30dd d1d0 1fdc 7eae  _$.....'0.....~.
000004a0: 5f69 b304 bc10 6a81 f287 d116 3fc2 a93e  _i....j.....?..>
000004b0: 715b f952 7e92 2f59 ee8f 027b 6049 227f  q[.R~./Y...{`I".
000004c0: 4bad 74db 18c3 7919 9918 1112 8a6a 6960  K.t...y......ji`
000004d0: 0000 0300 0003 0021 e1                   .......!.

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=11776
pts_time=0.958333
dts=9728
dts_time=0.791667
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1839
pos=10446
flags=__
data=
00000000: 0000 072b 419b b534 a43e 0308 2211 5f00  ...+A..4.>.."._.
00000010: 0003 0000 0301 06d0 1494 37a5 69a0 5752  ..........7.i.WR
00000020: 5e9a cf4c 47fc 1cde 9c6b 95fd 3d0c 2331  ^..LG....k..=.#1
00000030: a02d 6155 6683 7652 a16a bc6f 2186 0da3  .-aUf.vR.j.o!...
00000040: 74c2 3fa5 7814 e725 eadf 0000 8b2c a8b2  t.?.x..%.....,..
00000050: 1780 9739 0830 0260 af07 4390 9aa6 8f57  ...9.0.`..C....W
00000060: ac41 319f 26a1 5ccf 7f4e 6e92 bcfb 94fd  .A1.&.\..Nn.....
00000070: 5354 8840 4d33 cb0c 6ade 01b3 8be2 9712  ST.@M3..j.......
00000080: 239e 105b d2c8 107f c835 5259 1177 ac7d  #..[.....5RY.w.}
00000090: a7d4 a3cb 806e 8006 7d77 f6e8 1e28 8a76  .....n..}w...(.v
000000a0: 05b9 97ae 94ee 27de 036c 73d4 d465 1630  ......'..ls..e.0
000000b0: d803 b087 1c26 5b88 27a0 eb51 4e9f 78e3  .....&[.'..QN.x.
000000c0: 9255 e5d9 e540 63fb 00e4 1de8 ab1d 2bd4  .U...@c.......+.
000000d0: 0d2f 6309 1142 4038 6a53 0798 381f 85cd  ./c..B@8jS..8...
000000e0: 0454 7b8c 0e52 eabf 5b35 c872 651f 3c4d  .T{..R..[5.re.<M
000000f0: ed37 bab4 1c0e 3250 e3c3 b7f7 196e bdc0  .7....2P.....n..
00000100: a1f9 aa0b 1df4 321f f54f 51ef d40d 5132  ......2..OQ...Q2
00000110: d051 3e9c 0816 95f2 e15f ed0f 4b62 bfd3  .Q>......_..Kb..
00000120: cf36 4527 20ac 0dcf 3e64 0040 6fdd 0938  .6E' ...>d.@o..8
00000130: 99b0 94aa 949b 02a5 e055 a568 fd7d e47f  .........U.h.}..
00000140: c222 808e 0ff0 79d0 cdff 774c a50f e6e5  ."....y...wL....
00000150: 432f 0604 ea13 1d47 1b2d 5493 e126 10c5  C/.....G.-T..&..
00000160: daf2 e909 84fb e8c0 91bd 60b9 d2b8 ad38  ..........`....8
00000170: 2326 57e3 d4f9 e0cb 8d1f 7a99 bdd3 a3b8  #&W.......z.....
00000180: b8f3 19b4 9b4f cb76 598c 47e4 20ee 5976  .....O.vY.G. .Yv
00000190: e204 8bbf bfb6 0777 3000 2b16 e563 e315  .......w0.+..c..
000001a0: 7d11 a1e3 4627 3f1d f4a8 59ec 2e31 ffbc  }...F'?...Y..1..
000001b0: c427 4839 ae71 0ef7 9a9f f36a 2916 9977  .'H9.q.....j)..w
000001c0: 8a84 a7df b63f 99ec 76ee 4a1e 0bbf 5809  .....?..v.J...X.
000001d0: 00d5 a470 3518 119e 64ca 7985 54b9 8397  ...p5...d.y.T...
000001e0: 0054 3fa8 2a32 37d5 9fb9 8d15 34d4 fb2f  .T?.*27.....4../
000001f0: 934c d92c 7a11 8b5a cc92 5247 2c17 e7bf  .L.,z..Z..RG,...
00000200: 078d dda2 11f5 8837 dbe4 1336 0821 2a06  .......7...6.!*.
00000210: 2516 d440 5ec0 20b8 abde cd0b d3d9 8dd0  %..@^. .........
00000220: 56eb 7f8d d2a2 aaa1 01ca 9e55 88f5 60f1  V..........U..`.
00000230: e868 925c 0429 a59d 192f b9e0 60c5 a9c7  .h.\.).../..`...
00000240: 47b9 feb5 0c1e 9fd0 2581 f98b 3ddc 89f2  G.......%...=...
00000250: 6ea8 011c 90f8 a77a c23f 8a73 6181 5bbe  n......z.?.sa.[.
00000260: 92a1 71fd 2024 f78e 5e4b 7d30 2dfc 2a41  ..q. $..^K}0-.*A
00000270: d91c d27b 00b6 8e94 d027 ff0b 51a1 7e40  ...{.....'..Q.~@
00000280: 7cc9 fe51 01d8 c111 88c0 401d d27f 2dbc  |..Q......@...-.
00000290: cd80 5ff6 1219 3236 5e07 5577 5837 b07d  .._...26^.UwX7.}
000002a0: fa4d caf9 92ea b602 0d7a 26b2 55c0 c1b6  .M.......z&.U...
000002b0: 6004 7621 5ec3 e527 d584 373e 6bd2 f3a2  `.v!^..'..7>k...
000002c0: e7a4 556c fd63 9bbe ae67 0e76 7c57 d459  ..Ul.c...g.v|W.Y
000002d0: 7519 71c8 665c 95b4 c8e7 f5bb 0d7c dc15  u.q.f\.......|..
000002e0: be18 1754 55e1 ea1a 0516 ada0 6278 3ed8  ...TU.......bx>.
000002f0: 0d04 8784 6975 e157 8294 d924 acf5 7395  ....iu.W...$..s.
00000300: c85b 2630 c8e4 a68d bdb5 4dcb 41bb 5158  .[&0......M.A.QX
00000310: 7c43 e4e8 826f 3737 6bd1 5a1e d0bb e424  |C...o77k.Z....$
00000320: ca6a 7b01 a744 94e1 5ec2 460d 9944 025e  .j{..D..^.F..D.^
00000330: e340 960a 7c23 0ba0 6301 c6db f388 850b  .@..|#..c.......
00000340: 1bee 6d82 b579 7763 c46e 67e3 2f11 e969  ..m..ywc.ng./..i
00000350: f1ed ad23 63e0 9a46 5e3a 4c87 98d9 7b0d  ...#c..F^:L...{.
00000360: 4c47 0967 5643 1d61 f520 1f9c 14ca 2130  LG.gVC.a. ....!0
00000370: fef1 d216 01d1 09e6 65f0 ef93 83a6 9cd4  ........e.......
00000380: 57ef 2a94 eef8 2661 c91f 8470 9625 0e5b  W.*...&a...p.%.[
00000390: 3773 2865 92a1 5e31 0db2 d90f 324c c1fc  7s(e..^1....2L..
000003a0: 6309 a0b4 800b b9a0 e72e 17a7 43a5 294e  c...........C.)N
000003b0: 27a0 6dd1 a9de 79ee 93c4 baae e561 58c5  '.m...y......aX.
000003c0: 2e75 e916 9bb8 2028 8ba8 59c4 1875 e9f0  .u.... (..Y..u..
000003d0: 991e 9c3b 1f06 5122 3d37 11e4 0fb9 0444  ...;..Q"=7.....D
000003e0: 3731 3564 55d7 23c6 631c 09c4 92b7 6013  715dU.#.c.....`.
000003f0: 322f 9e81 23eb f634 5348 2abf 423d 1908  2/..#..4SH*.B=..
00000400: 9492 a266 81c3 0406 5d7b 3ed0 42ad fce3  ...f....]{>.B...
00000410: e5e4 ebfd fa2d 3cfd 26b5 6aa7 d628 c954  .....-<.&.j..(.T
00000420: 1402 9363 8168 c133 4938 7d58 119e 1526  ...c.h.3I8}X...&
00000430: 3884 7ce4 c177 5ef6 6598 27de 9f26 9423  8.|..w^.e.'..&.#
00000440: 7b0b c2e2 c807 b689 567a a919 ea8f d337  {.......Vz.....7
00000450: 70b4 efc7 7b0c 85fe 982d b245 5da1 03c0  p...{....-.E]...
00000460: b3d5 c7ed 8b47 cdf7 57bd 1dcd bc2d fa8b  .....G..W....-..
00000470: e4a6 4207 3055 1720 bf2f 2aac 1b53 4a78  ..B.0U. ./*..SJx
00000480: 4055 83e3 fe51 a4bd 4b20 c5ee ccef 301c  @U...Q..K ....0.
00000490: fd61 826c f290 f201 9c16 9c6f 52fb 5754  .a.l.......oR.WT
000004a0: f002 1a3f e017 4522 6c1a dff6 860d c0d6  ...?..E"l.......
000004b0: 9b2d 3b3f df28 4d70 3430 b5ee c396 0103  .-;?.(Mp40......
000004c0: a35d 98a7 8c05 d24c 0b5a ad6e a0a2 599c  .].....L.Z.n..Y.
000004d0: ef11 878c fa1f e50b 275f f7cf 427f aebd  ........'_..B...
000004e0: ef40 596e 92a8 f136 59eb b0d4 c66e 1c13  .@Yn...6Y....n..
000004f0: 2c63 9e76 77af 0cfb 19f8 78f7 69b6 9240  ,c.vw.....x.i..@
00000500: 4b31 ea99 a2c9 01cf a5fc b45c a02b 2448  K1.........\.+$H
00000510: 027a 52e0 5ba4 aa65 8188 6ebd 8f29 3765  .zR.[..e..n..)7e
00000520: 163f 36b3 52ce f18b de74 ab86 f72d fbbb  .?6.R....t...-..
00000530: 4c1a 2a7f 6be8 419a feb9 93b4 0059 a653  L.*.k.A......Y.S
00000540: c2e2 724f 7daa 2494 b09e ac82 33c8 0429  ..rO}.$.....3..)
00000550: dd1d 0049 d749 d7bf c4d1 d67a 3c4c 3753  ...I.I.....z<L7S
00000560: c5b4 cc23 28ab 07a2 d8a4 2761 f79f 50c5  ...#(.....'a..P.
00000570: 6f43 4321 71d2 31f9 fe5d 83bb 4c14 2ded  oCC!q.1..]..L.-.
00000580: ec82 c21d 5799 c5c8 4719 24d6 b766 285f  ....W...G.$..f(_
00000590: 4026 a67d 46f6 f4f9 ec20 8f5c f6b8 93ee  @&.}F.... .\....
000005a0: 969e 0545 3d4f 6a4c 2864 f8de de73 3955  ...E=OjL(d...s9U
000005b0: d781 27d7 392f c65f 565a 1514 a82f b3d6  ..'.9/._VZ.../..
000005c0: d95a c8a6 41d0 a200 c16d 27cc 6a88 cbc0  .Z..A....m'.j...
000005d0: c1d5 c869 06a3 333e ad7a bf85 1378 8d60  ...i..3>.z...x.`
000005e0: b0f9 3859 3aaf 4064 9e96 f554 23af b59c  ..8Y:.@d...T#...
000005f0: 68ba 974e 4baf 32e0 e8c8 435a 2bb8 bd1c  h..NK.2...CZ+...
00000600: 9b57 aa97 bc07 3a38 ec7a 46ba e0fa 2021  .W....:8.zF... !
00000610: d33b 7242 fff8 8cde b7ae 6017 3739 8c3c  .;rB......`.79.<
00000620: ce72 81dc cbcf 0fad 1b54 8337 3e55 d421  .r.......T.7>U.!
00000630: e139 c993 7166 2414 e425 abb2 858f e117  .9..qf$..%......
00000640: e264 753e 70d2 d9c5 e54f 61ce 6393 e4d9  .du>p....Oa.c...
00000650: 729c d814 e78c 603a 585c 4945 a4da 0ef6  r.....`:X\IE....
00000660: dc2d 0280 49b2 1d9d 6434 219a 3656 c661  .-..I...d4!.6V.a
00000670: 9312 a7c2 6215 636c 64ea 81bc 2c33 af25  ....b.cld...,3.%
00000680: ea47 2ac9 6841 21fb d2e8 735f 9959 3033  .G*.hA!...s_.Y03
00000690: f75c 85c8 c7c8 fb9a f5d9 9724 133f f878  .\.........$.?.x
000006a0: 19c3 9f2a 1a41 5243 1feb 224c 0504 a39e  ...*.ARC.."L....
000006b0: 6fe4 383e 71d2 5560 89dc b465 b22d 5c2e  o.8>q.U`...e.-\.
000006c0: 6ac9 1c05 e36d 5ac4 14d2 2660 822c f32b  j....mZ...&`.,.+
000006d0: 404e a71a c127 7e30 bfd3 c962 e4ad 770f  @N...'~0...b..w.
000006e0: f255 9f69 725b b7e2 057c 7fc9 18b9 6cd1  .U.ir[...|....l.
000006f0: c2cc 1392 d0c3 b740 d610 3461 a3c7 d35b  .......@..4a...[
00000700: 50b3 8bab a1e6 e051 a405 8d74 292a 1b5c  P......Q...t)*.\
00000710: f024 42df 4aa6 ef22 2aa6 b801 4097 09fa  .$B.J.."*...@...
00000720: 169c 5c29 6386 3637 6943 302c c0e0 80    ..\)c.67iC0,...

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=10752
pts_time=0.875000
dts=10240
dts_time=0.833333
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1714
pos=12285
flags=__
data=
00000000: 0000 06ae 419f d345 152c 5700 0003 0000  ....A..E.,W.....
00000010: 74fc eacc 017a 01cc 77cb 3f2d da86 6cfc  t....z..w.?-..l.
00000020: 90eb c8f9 1b6a 4d64 fd95 79d3 c48e f535  .....jMd..y....5
00000030: 137b 0d2e 0df1 053b c973 f629 b4a3 acf0  .{.....;.s.)....
00000040: af36 1fdf 3fb0 9e4e ec01 8dd6 6d26 fc4f  .6..?..N....m&.O
00000050: 0d79 591a 9f27 4164 d0ef 0dbe 63ab 535b  .yY..'Ad....c.S[
00000060: e80c 8b09 6e49 37f8 cff3 6106 e725 1044  ....nI7...a..%.D
00000070: 503a a721 ae15 9ce7 cc29 b35f 6214 6d8b  P:.!.....)._b.m.
00000080: 6ed5 98ac f1fe a1ec 47ec 9a5b 54a2 32af  n.......G..[T.2.
00000090: d2cc b6ed de96 7857 1847 017e 2782 a6a5  ......xW.G.~'...
000000a0: b805 88ef b5ab 77d9 ce38 a698 e2c0 c96e  ......w..8.....n
000000b0: 259e ede8 6cdd e6b3 a3bd a995 218a ae93  %...l.......!...
000000c0: 0073 4432 bfa5 5302 d8b5 fa45 0e04 7462  .sD2..S....E..tb
000000d0: 4934 94bc fdc1 f143 5055 d4a9 53a3 9694  I4.....CPU..S...
000000e0: f6a2 d45a 8b51 6a2d 45a8 b5e7 6ac1 d7cd  ...Z.Qj-E...j...
000000f0: b637 821a 6ae7 fbf6 3110 b0bd 1ee1 a676  .7..j...1......v
00000100: 0bc1 aacf 0e5f 4f78 f5b9 e69d 6ce7 0fa9  ....._Ox....l...
00000110: 1a4c 20ca dfbe bf7d 7efa fdf5 fbeb f7d7  .L ....}~.......
00000120: f190 5c5c 330b 4d7e a41e a943 d9bf b472  ..\\3.M~...C...r
00000130: 94b4 184d 115e 68cf 01c4 1499 41d4 a5ce  ...M.^h.....A...
00000140: 18f7 d7ef afdf 5fbe bf7d 7efa fdf5 fbeb  ......_..}~.....
00000150: f7d7 efb2 3981 ba5d 6c08 9045 632f f1c0  ....9..]l..Ec/..
00000160: 8788 d5a2 5822 6d06 0863 4e8e 5a5b 6cb9  ....X"m..cN.Z[l.
00000170: 4b58 1abc def8 922b 44d5 a0c1 0c69 d229  KX.....+D....i.)
00000180: bc74 b44c 99b5 9600 20d3 c491 5e14 1e2a  .t.L.... ...^..*
00000190: d469 1635 0b25 24ee 9fed ee97 8afa 2933  .i.5.%$.......)3
000001a0: d0bb 00ce 1814 455a f143 51fe d8ed 8fff  ......EZ.CQ.....
000001b0: 2b63 54ae 87b4 c065 4e01 b5ca 3899 8a75  +cT....eN...8..u
000001c0: 58a0 1e46 047e 4437 12f0 33cb eac6 924c  X..F.~D7..3....L
000001d0: e763 6750 11e5 726a 7eef ba93 3c2d b077  .cgP..rj~...<-.w
000001e0: acaa 7510 d673 fb8f 010a 0271 9fad 5c72  ..u..s.....q..\r
000001f0: af98 5d55 e7c5 0962 aed8 b9fd d072 f908  ..]U...b.....r..
00000200: 1ad4 fadc 4241 ca3e 23be a99b 9449 5192  ....BA.>#....IQ.
00000210: 5e29 ef3d 0a23 7d22 0e8b c4b5 26cb 573f  ^).=.#}"....&.W?
00000220: 679e 090f 0c9a f477 f40e 5a08 7de3 0f61  g......w..Z.}..a
00000230: 7418 65ba 0761 27b0 ff15 92f6 0b05 f928  t.e..a'........(
00000240: 256b f543 a753 6c74 0fb3 d5c9 180b d348  %k.C.Slt.......H
00000250: e5ac a28a 44b8 6810 80e9 0c1d 4093 3d93  ....D.h.....@.=.
00000260: e1de 2804 0d3e 221b 1e53 027e 47d8 572e  ..(..>"..S.~G.W.
00000270: 302a e071 be74 c621 86b2 da19 8fde 42e0  0*.q.t.!......B.
00000280: c6a0 22af 896e 4d52 0cd6 1b41 328d e9ae  .."..nMR...A2...
00000290: 24ee 42ae 008f 032a dbf0 a6ff f1f9 08b9  $.B....*........
000002a0: 945b ffa3 db0f ce8f c6d4 bffe 78d5 0249  .[..........x..I
000002b0: 68bd dece 149a c95b ecf5 4a12 88d5 99c3  h......[..J.....
000002c0: 83d3 0981 104d e921 bf22 4194 bbab 4dfd  .....M.!."A...M.
000002d0: 0171 bf3c 5a0d 32ec a682 9e47 ef6e 17d7  .q.<Z.2....G.n..
000002e0: 4b5e 4566 525f 6a21 700b ccee b7ef 447b  K^EfR_j!p.....D{
000002f0: a62b 63e9 d51e 1bad a197 8e3e 0552 82d5  .+c........>.R..
00000300: 11eb 842b 02d4 692f a7e9 d7d7 9524 2736  ...+..i/.....$'6
00000310: 958b d072 40ad 8ee4 85b8 2b4a 78b2 2e14  ...r@.....+Jx...
00000320: c014 4256 dff3 5c3b ab95 a35f fc28 5821  ..BV..\;..._.(X!
00000330: 513e 7412 3a19 2033 d347 b0b7 2f51 de5b  Q>t.:. 3.G../Q.[
00000340: d123 425c b678 c38c c459 7a6a 71de 39b5  .#B\.x...Yzjq.9.
00000350: 2b71 8821 4eeb 3616 9c4f 554d 8ce8 dce9  +q.!N.6..OUM....
00000360: 6b45 9da3 9f80 26fe aee0 7390 1077 c5fc  kE....&...s..w..
00000370: 5293 ae5a 0340 4e56 482d 496b 7e6d 6831  R..Z.@NVH-Ik~mh1
00000380: 2e88 2307 b8d1 a5a6 e150 2e70 ddb4 38b6  ..#......P.p..8.
00000390: e087 16f7 b14d 3334 ebe7 315a 5ac8 e075  .....M34..1ZZ..u
000003a0: f75f a334 7c8a 5ae1 4899 37af 0099 d61f  ._.4|.Z.H.7.....
000003b0: 6631 9d42 5aa7 909e b26e f3df 995b cae5  f1.BZ....n...[..
000003c0: 345b c292 1366 2c49 f685 05f4 c584 eae6  4[...f,I........
000003d0: d02c 64e4 1b4f d929 3a1d 20dc 3255 7591  .,d..O.):. .2Uu.
000003e0: fe81 69c4 b682 df43 55c1 73a7 4905 fcc0  ..i....CU.s.I...
000003f0: 2a30 9918 8eda 7c1e 97b8 bdb4 4a21 9ecc  *0....|.....J!..
00000400: c8fb 30ae c701 7e86 038b 718e 401d 3104  ..0...~...q.@.1.
00000410: eafe c6b7 6d39 2331 fc42 7755 5c7f 7040  ....m9#1.BwU\.p@
00000420: 4459 e864 a40f cc39 eebd 039d f250 5557  DY.d...9.....PUW
00000430: 73f5 421a 95e3 2a76 ea8d 316a 563b 2ee2  s.B...*v..1jV;..
00000440: 98ba 3531 2fff 06fe f5e1 3999 44b9 42fc  ..51/.....9.D.B.
00000450: 9a5b 1a46 3fef 5728 50af ebb8 a097 ed23  .[.F?.W(P......#
00000460: fde4 139f 6af2 4c3d 33c0 7429 6d48 0f51  ....j.L=3.t)mH.Q
00000470: a936 a3d8 9e28 3221 234f 71f2 7593 bed9  .6...(2!#Oq.u...
00000480: 209c e2fc e679 63f9 ec83 1592 6ccf bebd   ....yc.....l...
00000490: fe64 a1b1 e410 3abb 2195 ba8f 176d b6b7  .d....:.!....m..
000004a0: 2335 0dc1 fbd5 7ffc ce1d f7fb 99bd 6497  #5............d.
000004b0: 1b83 117b dbef af01 85de 1878 4848 790b  ...{.......xHHy.
000004c0: be82 8764 10db cb31 e8db c1a2 394c 130c  ...d...1....9L..
000004d0: d938 17e9 b167 dc1f fa45 a971 9abf 28df  .8...g...E.q..(.
000004e0: 0f22 0584 ae26 5dca d730 9a0d 1762 fc52  ."...&]..0...b.R
000004f0: 6c71 d852 80a2 881e ac14 516b 1908 bce2  lq.R......Qk....
00000500: 1805 3dfb 18b8 2253 3d33 b988 daaa 68ff  ..=..."S=3....h.
00000510: 1fb0 7c06 679b 2dfd 61fe 2d10 7132 856b  ..|.g.-.a.-.q2.k
00000520: 10da 9ab3 80ed 4513 725e 0db2 c783 14e9  ......E.r^......
00000530: ee53 ccb1 74d8 3622 54e9 54f6 102e f96d  .S..t.6"T.T....m
00000540: 4fd3 466b bfdc 2a59 5dfd 5447 479b d503  O.Fk..*Y].TGG...
00000550: 4a3e 4abb 571e deef 9fad 0506 f306 3508  J>J.W.........5.
00000560: 764a 0fa9 bf7c c35e 347b a23f 6634 de94  vJ...|.^4{.?f4..
00000570: 5617 41d1 d00d bcca 50f7 a159 01f4 fee3  V.A.....P..Y....
00000580: 366c d960 814e eed5 6822 0c31 2295 2e79  6l.`.N..h".1"..y
00000590: 79ed c643 a3fa 0672 4cb7 d9e0 8714 fcd0  y..C...rL.......
000005a0: e2aa 9eed e0a4 dc08 30d7 baf4 d4c6 9f04  ........0.......
000005b0: 4bb3 e893 e465 6e08 666e a1ca b366 a3a4  K....en.fn...f..
000005c0: 056e 5ca9 a1cb 549d b452 b05a 42af d7a1  .n\...T..R.ZB...
000005d0: 7c2b dd1c 9692 df4d 8faa 53ac 62b2 4546  |+.....M..S.b.EF
000005e0: a2ff d469 6180 8cf7 a5cd df37 a9f6 d702  ...ia......7....
000005f0: 7b6d 9c3b 127c a87a 71b2 e43a 2ac0 fb00  {m.;.|.zq..:*...
00000600: a908 049f b56f 055d 916d e4c2 2df4 ab55  .....o.].m..-..U
00000610: 8041 ccef 6588 fb06 51cc 63e5 85c3 bd61  .A..e...Q.c....a
00000620: bba9 0084 58ef e259 38c3 7d73 34e5 ed72  ....X..Y8.}s4..r
00000630: aeba 9c5e 9910 9812 057c 7d30 1ef7 2c3a  ...^.....|}0..,:
00000640: aeea b23c 5bef 24ca 8c2d 61e5 ed3c c540  ...<[.$..-a..<.@
00000650: 4903 e6e8 fc26 74bd e572 7e5d bca8 ba73  I....&t..r~]...s
00000660: 0ad0 3990 6c7e 6e89 ba46 c027 9efe 8944  ..9.l~n..F.'...D
00000670: a8d3 34ec 384a e9dd 7d03 0c00 ac4c b34e  ..4.8J..}....L.N
00000680: 6e42 357c eb67 6786 1858 10ad 3280 e07c  nB5|.gg..X..2..|
00000690: cb2e e601 43b4 4474 423f 47c7 dc73 4483  ....C.DtB?G..sD.
000006a0: afeb 9c2c eb33 fec6 9bea 6000 0003 0000  ...,.3....`.....
000006b0: 2560                                     %`

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=11264
pts_time=0.916667
dts=10752
dts_time=0.875000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1224
pos=13999
flags=__
data=
00000000: 0000 04c4 019f f444 5700 0003 0000 07a4  .......DW.......
00000010: 8f95 2f7c 7f0c 2dbc f760 ab1f 46b9 eb3d  ../|..-..`..F..=
00000020: 6ba9 7d4f 216f 3103 c396 2008 4a00 426d  k.}O!o1... .J.Bm
00000030: 712a df09 a471 1f44 0e9b f51a 831a 9eea  q*...q.D........
00000040: 0e02 d3d4 9150 591c 7464 9cfa 5427 96bc  .....PY.td..T'..
00000050: 94fc d4af 9da0 96f4 4543 dc4d 59fe a5c5  ........EC.MY...
00000060: 05d9 0435 8376 daf4 c96e aadb fb4d ac94  ...5.v...n...M..
00000070: 2f39 9927 1e80 2528 e13d 7c78 3f13 5637  /9.'..%(.=|x?.V7
00000080: c6da 6740 cfc0 5940 799d c17f 754e 9176  ..g@..Y@y...uN.v
00000090: 421d daf6 a49e 60a0 41b3 9bdb 0a87 7503  B.....`.A.....u.
000000a0: 2e7d c3e5 7676 d09c 098d 4b1d c90e aa5b  .}..vv....K....[
000000b0: 721e 65de bfcc 02fa 6667 7beb 832a 9531  r.e.....fg{..*.1
000000c0: d81b a5ab 5616 322f 02a4 9bad 5db3 611f  ....V.2/....].a.
000000d0: b718 fcd4 5628 e668 4f39 0a29 a1fb 8625  ....V(.hO9.)...%
000000e0: c761 f8a7 4fb2 4c89 968e 9f1c fc8e be37  .a..O.L........7
000000f0: 960d 3674 d295 56d5 8fe1 8ed3 f696 8cf7  ..6t..V.........
00000100: 6346 428a 1a31 7af3 28b1 ce8d eae3 d242  cFB..1z.(......B
00000110: 8c84 d5da 02e9 769a 1fdc 9220 2e38 7da8  ......v.... .8}.
00000120: 0026 7429 0000 3729 3b3d 6728 4bf8 e270  .&t)..7);=g(K..p
00000130: fb08 0122 c7e7 8608 a5a2 7995 db4d 9e0e  ..."......y..M..
00000140: abf3 eb1e 26db 4bd0 e1c9 676c c970 915b  ....&.K...gl.p.[
00000150: a43d 9dcc 35bf 878b e018 8f5e c4e2 796b  .=..5......^..yk
00000160: ed02 e8d4 126d 8b50 b6dd b00f f4bd d9ab  .....m.P........
00000170: b74f 6c90 eb05 8e04 eead 732e 5c23 1561  .Ol.......s.\#.a
00000180: 0d39 9c30 d9da eecd 49c6 78c5 c426 d670  .9.0....I.x..&.p
00000190: 62d5 af4b 0fa6 9210 1d26 91dc c62b 1efe  b..K.....&...+..
000001a0: 7de7 32f7 0bfd cf91 ee18 01e7 fd2f 72d7  }.2........../r.
000001b0: db51 43eb b4af 3f41 443c b3ab 5d6b bbec  .QC...?AD<..]k..
000001c0: eb68 02ba 01ba 060a 5c6e ea7d 0161 12b6  .h......\n.}.a..
000001d0: c0e2 263f 5e90 2b45 f800 4257 29e8 8f0c  ..&?^.+E..BW)...
000001e0: 369b 5c8a 7bad d567 6119 8b1d be10 74c2  6.\.{..ga.....t.
000001f0: bf7d d345 69d4 bc37 00c7 ff47 1296 9700  .}.Ei..7...G....
00000200: b6c2 4a82 027e 1de8 de32 1a29 250d ff7c  ..J..~...2.)%..|
00000210: b4f7 a039 2f5c 2d39 93b1 f432 a7dd 9eb8  ...9/\-9...2....
00000220: 8273 6b1b 188b 4489 1505 d25f e89f 46a5  .sk...D...._..F.
00000230: 6339 3b06 d8c9 1bb2 e390 d188 213e a6af  c9;.........!>..
00000240: 89d8 8335 c337 aad5 d019 b640 596d cb5c  ...5.7.....@Ym.\
00000250: 7048 4494 5a22 379d 20f0 17e1 e983 df25  pHD.Z"7. ......%
00000260: dc16 eb19 5e75 e5a8 4f88 ec61 4e81 1ac5  ....^u..O..aN...
00000270: b992 3520 e1ab 126b 32b9 6ecd 1ee2 969e  ..5 ...k2.n.....
00000280: fe17 89e5 8134 1da0 1870 05a8 778b 32ab  .....4...p..w.2.
00000290: 1e16 5f07 1f4a ab05 d641 b9f4 938b cda1  .._..J...A......
000002a0: 1287 f8de fcd7 7e67 176a 4089 1cab fb61  ......~g.j@....a
000002b0: be57 ec28 7c21 ca05 00df 8ccd 2f82 704f  .W.(|!....../.pO
000002c0: 7cc4 7706 beaa 46ed 0c53 5c9b 3bf2 374e  |.w...F..S\.;.7N
000002d0: 6b4a b2eb 6734 1644 f7ee 0d58 bcad 43fa  kJ..g4.D...X..C.
000002e0: 6099 30fa 03b1 620b 3e7f 8483 9d4a 44e8  `.0...b.>....JD.
000002f0: 0360 86f4 fd6f b190 f7e8 85e3 7a74 9ba6  .`...o......zt..
00000300: a1c5 4cab 519c 8ee5 fdf2 d790 9edc 4814  ..L.Q.........H.
00000310: 091a dd54 7f04 fd12 0cd3 9fd3 8f9c a860  ...T...........`
00000320: a2b9 bd79 7b66 a444 00f2 0ae0 865d b276  ...y{f.D.....].v
00000330: d47a a7bb a162 686d 3688 f343 6e1b d0dc  .z...bhm6..Cn...
00000340: 66dc 0f50 7d48 074f bffb 8820 ceb5 8dc1  f..P}H.O... ....
00000350: 5520 f356 6b58 b10d 34fd cd65 1cb1 4d80  U .VkX..4..e..M.
00000360: aa15 b444 2f92 1591 87bf 2fdb 8d61 c1d0  ...D/...../..a..
00000370: 96e2 d78c 22ed 4b4f afb6 7369 2da8 e0ff  ....".KO..si-...
00000380: 8753 0f29 5b4d b1f8 545b d02d 0a13 27ab  .S.)[M..T[.-..'.
00000390: 378c ebdd 146f 59ee c4fa 773c 0708 47ae  7....oY...w<..G.
000003a0: e076 875f 41f2 4eff b28d 4a27 96f4 a768  .v._A.N...J'...h
000003b0: 7fda 8c93 ea41 4b63 d65d cbed 41e6 90cd  .....AKc.]..A...
000003c0: a6b7 fa7e c874 7aa0 b743 2615 4f80 883d  ...~.tz..C&.O..=
000003d0: e7d8 2177 9df0 bf5a 65a4 f478 752c 85fd  ..!w...Ze..xu,..
000003e0: 35d0 8919 4a60 1e28 8c04 dae7 c374 9133  5...J`.(.....t.3
000003f0: 1cf4 c336 9f8d efc7 8b8f f2e2 8ee9 1fe0  ...6............
00000400: a65e fad0 bbb4 c9f5 5f92 0f6b d86b b012  .^......_..k.k..
00000410: f205 5b3d aebd c03b b839 889b 33f4 a22e  ..[=...;.9..3...
00000420: e551 ac44 7bbe d9ac 3503 848e 7824 7dea  .Q.D{...5...x$}.
00000430: 5646 2272 b4e2 d17a c0bc f62c 13da 738e  VF"r...z...,..s.
00000440: 4331 4808 61aa 5f75 b146 435e f4d8 5eb7  C1H.a._u.FC^..^.
00000450: d255 84d3 262f 7f32 6b78 cbba 35df ec16  .U..&/.2kx..5...
00000460: d181 3a54 5db9 675c 2f87 0322 f68c 088a  ..:T].g\/.."....
00000470: 4777 c619 dd06 d7a7 29b8 0908 d841 46b5  Gw......)....AF.
00000480: e3b4 2e91 48ae 862f 6cb7 5ee9 107f b66a  ....H../l.^....j
00000490: 0c76 7573 0432 1b81 4a75 f6d2 7bcd 8bf0  .vus.2..Ju..{...
000004a0: d791 f90f 1c1e 133b c8bf 7e09 e60d 827d  .......;..~....}
000004b0: 9b54 c91a 386d 7f9e ffeb 5471 bf7b d444  .T..8m....Tq.{.D
000004c0: 0000 0300 0003 0317                      ........

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=12800
pts_time=1.041667
dts=11264
dts_time=0.916667
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1923
pos=15223
flags=__
data=
00000000: 0000 077f 419b f734 a43e 0278 48a2 62bf  ....A..4.>.xH.b.
00000010: 0000 0300 0003 010e cfc9 52fc 6c89 f690  ..........R.l...
00000020: c24a c7ea ed9f ac84 ae9b 5a2d 4a63 534b  .J........Z-JcSK
00000030: 18d7 4e3d 8497 5fce 31f2 09ad e23e cad5  ..N=.._.1....>..
00000040: 8586 fc4d 69b5 ce15 299b 2d2c 9fff ebbf  ...Mi...).-,....
00000050: c383 671d f542 f87d 7c45 6eb4 26db e930  ..g..B.}|En.&..0
00000060: f1d8 4d0a 6a1e e185 e300 4830 5fdd 42a1  ..M.j.....H0_.B.
00000070: d873 80b3 7c03 4623 09d0 28e8 bbf4 ec44  .s..|.F#..(....D
00000080: a5e8 cd43 94dd b85c 3636 dbde df84 7c8e  ...C...\66....|.
00000090: 93e2 d074 663f 52ba ed8b c129 be95 279f  ...tf?R....)..'.
000000a0: deb4 adf2 bf62 62a4 8415 51b6 8eb0 8260  .....bb...Q....`
000000b0: 5706 c2cf f891 f30f ce0e f685 783d 8d46  W...........x=.F
000000c0: 66f8 a302 b2aa 295f 6894 4bc1 095c 6836  f.....)_h.K..\h6
000000d0: ef03 3aa6 5950 3367 8695 0030 1ff2 5680  ..:.YP3g...0..V.
000000e0: 1ee9 34a4 3cf5 6dad 4d9a 0323 5f96 39ac  ..4.<.m.M..#_.9.
000000f0: f73e d635 0171 2b75 d5ea cdb0 272e e793  .>.5.q+u....'...
00000100: 4390 6b48 2948 22c0 d855 cb22 a396 e800  C.kH)H"..U."....
00000110: 0fa2 6820 76e7 267d a2c5 8ed1 f291 3965  ..h v.&}......9e
00000120: 0e65 3fa3 eca8 dde1 5075 56ff 3a23 02d0  .e?.....PuV.:#..
00000130: cb90 b9d6 3d1a 2c5d 3646 74c5 e7be 2e56  ....=.,]6Ft....V
00000140: 0c17 1fb9 8ae2 738f c3ce 1150 a0c0 93c6  ......s....P....
00000150: 072d 3c22 2f75 0de4 3924 5277 c8ff 7e93  .-<"/u..9$Rw..~.
00000160: 88a1 59df d38b b182 3084 ad84 88a7 5095  ..Y.....0.....P.
00000170: 319c 3e02 d092 b7d1 60e9 b680 4fb1 9ede  1.>.....`...O...
00000180: dcc0 744e 8b7f 8e40 dac5 f488 5654 9702  ..tN...@....VT..
00000190: fafb 32d7 f750 d54e 3a67 3956 52f6 a71a  ..2..P.N:g9VR...
000001a0: f26e b5dd 5638 88d2 697c ff7f 36f0 2c54  .n..V8..i|..6.,T
000001b0: 73c2 4074 9760 d8e1 8eb6 7d73 47a3 1f3b  s.@t.`....}sG..;
000001c0: daa3 238a 941c 6fc6 a15d ecea 7b2e 1686  ..#...o..]..{...
000001d0: afb2 e9f9 54d9 a730 f31f d7b7 d883 274a  ....T..0......'J
000001e0: afd8 4979 1a5b 65f6 c885 a611 0d9a 7c14  ..Iy.[e.......|.
000001f0: 1ec3 cd40 e311 2a5f c23e 9b9c b24b 34ea  ...@..*_.>...K4.
00000200: 5422 f766 4972 ec2d 9d3d b1ea fbfd 8d38  T".fIr.-.=.....8
00000210: ec78 a666 141d ed61 e4ac 07bc 3391 943a  .x.f...a....3..:
00000220: f204 1577 9f61 13cb 6902 d2ab 8912 c7e2  ...w.a..i.......
00000230: e77e 360d 534b 2e31 7a5d 3035 d7b5 4e6a  .~6.SK.1z]05..Nj
00000240: 190d 74f5 e115 2ace a4bd a59d eecd 1c61  ..t...*........a
00000250: 3fbc fcf2 ab04 9196 1a9a 088a 89ae 7c56  ?.............|V
00000260: 8ae1 825c 8208 0602 11a0 98b8 8dbb 5976  ...\..........Yv
00000270: 2d2e 9373 0394 4d61 ddc8 b209 df1e cc35  -..s..Ma.......5
00000280: 6d8c 0861 4718 f293 69ef 4eeb 561c 2280  m..aG...i.N.V.".
00000290: 4034 3aa2 31bb f88b f95d 6663 5146 aa65  @4:.1....]fcQF.e
000002a0: a2fe 5ed4 0da8 72d6 9ad7 f3fa 073c 4359  ..^...r......<CY
000002b0: 348e a12d 195e e2ba 66e2 1ed6 6bad 196d  4..-.^..f...k..m
000002c0: bcdb d457 e92c ae3e f6dc 63de a056 9202  ...W.,.>..c..V..
000002d0: 3945 ccf7 ffd4 6f76 0c0c 49e9 46e6 8787  9E....ov..I.F...
000002e0: 55ac 6f7e 358e 400f d0ba 24b1 9b9c aed7  U.o~5.@...$.....
000002f0: c0b0 8c22 a53b cb99 9222 77f4 756b fbd2  ...".;..."w.uk..
00000300: 0822 34ef 45bf e373 4be3 bdbb 09be f532  ."4.E..sK......2
00000310: f444 5ee1 04f8 75dc ee7b 87ab dd03 5d1f  .D^...u..{....].
00000320: 4725 7c14 6d9a 9cf6 212f a094 460c 98b6  G%|.m...!/..F...
00000330: 6aea c006 57ba 2e8b 0435 f300 8346 05a2  j...W....5...F..
00000340: e12f 10fe 1189 1f73 490c ff46 eb7a f264  ./.....sI..F.z.d
00000350: c323 10ff 8177 d79a bccd 291a 09ca 0d6b  .#...w....)....k
00000360: 362c 02a2 c2a9 a98d 1bb9 9c35 5a32 d531  6,.........5Z2.1
00000370: 6b9f 03d0 7f63 c2f5 e29b ff74 5714 d7e4  k....c.....tW...
00000380: a37a affe c6d3 87bf 4bf7 e144 c250 c70d  .z......K..D.P..
00000390: 0ff6 24f4 32a0 4149 a35a a96f 7b65 b615  ..$.2.AI.Z.o{e..
000003a0: 6d65 9e70 0e54 bf57 0fed d4e7 2d49 0ac9  me.p.T.W....-I..
000003b0: b46d 41d3 5e0b e1cb 8b3e accb 4ad9 32ef  .mA.^....>..J.2.
000003c0: 244f 85c7 a22e ba6f d82d a4fb d612 2427  $O.....o.-....$'
000003d0: b276 aba4 bae3 33a7 4ac0 530e 4444 c2fb  .v....3.J.S.DD..
000003e0: fa51 7c8a f03c fa02 c5fa dd67 cb11 b892  .Q|..<.....g....
000003f0: 5f99 7dab 4d56 2a03 67ab d9ac 3ffa 4a51  _.}.MV*.g...?.JQ
00000400: ceab 1077 6005 9059 fe85 80c8 3747 4a5b  ...w`..Y....7GJ[
00000410: 4412 34f0 3343 806d 8b97 29a0 8289 c36e  D.4.3C.m..)....n
00000420: 3f4d e358 f056 8226 df0d 9776 bae7 236f  ?M.X.V.&...v..#o
00000430: 89f9 dc8e adfa 72bb 85c5 6e7b ff03 4ab4  ......r...n{..J.
00000440: 29e9 45df 86fb dc6b a08d 9d5b da14 0b4c  ).E....k...[...L
00000450: e624 074d 7ebb 9be5 3bb7 5a96 8a98 d3c5  .$.M~...;.Z.....
00000460: f1e4 d0d4 ca7f 9392 7d66 f90a d89a 1748  ........}f.....H
00000470: 5342 7661 2ded 99c7 330c 76b4 f10c 5972  SBva-...3.v...Yr
00000480: 4e7f 4f79 b0ab fdd6 93c3 33e6 ad1d 57f3  N.Oy......3...W.
00000490: 6bd9 a4bb ede6 294e bf44 d17d 92e4 acb1  k.....)N.D.}....
000004a0: 2482 3cf0 e272 5adc 6a79 97c5 92ba 56cd  $.<..rZ.jy....V.
000004b0: b2ef 8cfd 96e8 a72a c5e4 63a1 4ae3 9b10  .......*..c.J...
000004c0: 521a bde0 7cc7 d57d 4f39 455a 13f0 c3c9  R...|..}O9EZ....
000004d0: cfb5 e2c2 1951 d0cd e122 7d4e f9f1 266a  .....Q..."}N..&j
000004e0: d6a2 37b9 2dae 38a6 a2eb 79d8 4f64 cd0c  ..7.-.8...y.Od..
000004f0: 85f9 cb6b 69ce 3b3a 6e04 f9a6 4e17 cc28  ...ki.;:n...N..(
00000500: 4c3a 11fb ceab 0b82 6a84 3e0d a740 2f9c  L:......j.>..@/.
00000510: a1a2 63a6 5f0d be01 d49c 7e06 305c 2613  ..c._.....~.0\&.
00000520: c0cc 9dbd 4982 dff5 6785 59b1 c786 089d  ....I...g.Y.....
00000530: 9641 e6bf 8686 3e59 9b51 fb45 5af3 9c85  .A....>Y.Q.EZ...
00000540: d041 e3fb 5042 d740 e64b ce70 c023 eea1  .A..PB.@.K.p.#..
00000550: 8a8d 1ec3 4131 45a7 3122 f373 977d 90c6  ....A1E.1".s.}..
00000560: e320 e153 7811 8642 7b39 b868 fc96 bffd  . .Sx..B{9.h....
00000570: 1c94 277b 5ec3 513f abe6 3ba8 ac49 e4f1  ..'{^.Q?..;..I..
00000580: a5a3 d6f2 bf75 302b ee6f 255f d8f7 dd4a  .....u0+.o%_...J
00000590: 09f2 84ff b6eb 2747 9704 9f6d bec9 7290  ......'G...m..r.
000005a0: db10 ea3a b6ae 8cad 5437 fcf6 e65d 1775  ...:....T7...].u
000005b0: 15b1 59ad f94a 0e4f 89e9 4d79 baa0 fa5c  ..Y..J.O..My...\
000005c0: 568f 264c af80 6bf9 9c5b a518 50ce bf71  V.&L..k..[..P..q
000005d0: 475a f7e4 9e05 975e 2c14 a7f0 0626 3208  GZ.....^,....&2.
000005e0: 6d73 8ce1 99cf b392 3748 aee3 d724 3386  ms......7H...$3.
000005f0: 7ca7 e2e5 4312 4c23 22c7 fe66 3fda ec02  |...C.L#"..f?...
00000600: 166d e2b1 62a4 d83a ea2f 4c88 5912 a6a7  .m..b..:./L.Y...
00000610: 5ff9 cb02 8fb3 a77f 96be 0ad1 08dc db26  _..............&
00000620: b26b 7bfe 32e7 349d 9ed3 2659 5b2b 15e1  .k{.2.4...&Y[+..
00000630: 20fb 209c 0b89 50fd f5db d3ad 9508 35e8   . ...P.......5.
00000640: aea7 cf3a 7b57 733f 7e02 3f0a dd9c 0e04  ...:{Ws?~.?.....
00000650: 16a5 abf9 f8da 90c9 fee2 1686 d663 cc81  .............c..
00000660: 71f0 8d2c 9965 87a6 c1bd aa46 a703 5cab  q..,.e.....F..\.
00000670: e1d9 4954 27d4 3cf1 b214 6ade 1db0 2975  ..IT'.<...j...)u
00000680: 11a9 d230 8642 1981 207f 6713 7d8d 1ae1  ...0.B.. .g.}...
00000690: 33e0 286a 7b9a 7c24 e2b0 2e0e bd42 658c  3.(j{.|$.....Be.
000006a0: c1dc acb7 5f6a ac9e 6e80 29d0 e52c a639  ...._j..n.)..,.9
000006b0: 7125 3cc1 9e5e 7f14 f709 4578 9702 b81d  q%<..^....Ex....
000006c0: 1481 ebfa d92f f742 02ed 4ff1 7d10 3909  ...../.B..O.}.9.
000006d0: 9ae0 bdf3 22f7 49f4 0136 71e6 9368 b779  ....".I..6q..h.y
000006e0: 6eec 86bf 8d2e baaf 4c9f 87a0 f8af 5b2d  n.......L.....[-
000006f0: e7e0 6cba bb22 8481 f3e3 96db 1d18 9336  ..l..".........6
00000700: e2cc 5876 a46a 638a 992b 081f 9ae1 1669  ..Xv.jc..+.....i
00000710: 8094 4f0f 452d 57c7 9afb 5a6a 1602 c535  ..O.E-W...Zj...5
00000720: 60ff 2f98 c7bc d2f0 ea00 b7ef 8e3e 8998  `./..........>..
00000730: aec9 3e84 9ad1 5b70 4f4f 44ab 0cd1 c808  ..>...[pOOD.....
00000740: 5e22 e4b7 4c3d 246b 69d9 cd2a 5206 5078  ^"..L=$ki..*R.Px
00000750: 3c40 d661 3e95 33fc 8fcf 12bb 5418 ce9c  <@.a>.3.....T...
00000760: 56aa a924 784c bab5 8f28 a8a2 7164 0e34  V..$xL...(..qd.4
00000770: 438f 4350 58ed 0041 770a d036 810b 073c  C.CPX..Aw..6...<
00000780: 0011 30                                  ..0

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=12288
pts_time=1.000000
dts=11776
dts_time=0.958333
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=1525
pos=17146
flags=__
data=
00000000: 0000 05f1 019e 1644 5700 0003 0000 06a4  .......DW.......
00000010: feac c09a fed6 f775 168f f99d 8141 240b  .......u.....A$.
00000020: 8000 9495 7cfe 7adf 7147 db29 3373 adcd  ....|.z.qG.)3s..
00000030: 5d45 5a09 4b18 eeda 97e2 335c ac3b 3f7d  ]EZ.K.....3\.;?}
00000040: a655 153f 7543 b037 267a 2cc4 4544 1827  .U.?uC.7&z,.ED.'
00000050: e005 ec24 0b1e c433 2181 0d26 06b4 9fa1  ...$...3!..&....
00000060: def0 dc7c 27f8 e39e 0901 5c41 095c 2aef  ...|'.....\A.\*.
00000070: b728 69bb a4d2 9085 b3a2 4991 83e1 6efc  .(i.......I...n.
00000080: cc04 ed3b 5ae2 7f87 0d5a fac6 434a 1154  ...;Z....Z..CJ.T
00000090: 0103 31ee d1fb 19e0 740a e394 86fe ae3f  ..1.....t......?
000000a0: 1500 5d96 e643 ab0e 6798 ca74 4ffd e7a9  ..]..C..g..tO...
000000b0: 6ab2 169e 8c16 1beb d850 8bbd 2f37 aafb  j........P../7..
000000c0: 83db 12fd b4ff b942 0be7 c6c2 c580 7313  .......B......s.
000000d0: d01f fcdc bf0e 4209 beab ce3b bce2 94e3  ......B....;....
000000e0: 0f2c d1dd b6fb 85df d809 5be8 d299 3d2e  .,........[...=.
000000f0: 02f9 4b07 2b25 3f05 e9e7 6ec2 5b5b 6b85  ..K.+%?...n.[[k.
00000100: 8cb7 1c13 277c 1e31 c386 ead4 8e1c 64a2  ....'|.1......d.
00000110: 9d0f f8df bb92 1af1 46ee cff0 e161 0a25  ........F....a.%
00000120: c930 37f4 7989 8e47 7ed3 5f74 59cb 4300  .07.y..G~._tY.C.
00000130: a1b4 3326 262e 959a 69ad 974e 9f5a 2b92  ..3&&...i..N.Z+.
00000140: a64b 46be 562d eb32 3fbf c71a d2a3 84f4  .KF.V-.2?.......
00000150: 1bc9 5c8b efd3 aed3 2f80 8202 88c6 cecc  ..\...../.......
00000160: 2d16 4a06 408a aa0a 80b9 102e e680 d403  -.J.@...........
00000170: 9672 a2d2 6c9b 7fba cf46 eaf0 aabb c619  .r..l....F......
00000180: 8bf1 6900 da9c 0853 0407 43c6 c12c 5e00  ..i....S..C..,^.
00000190: 4ccc 8f80 2baf ed90 ed1a 5f27 3bee 2768  L...+....._';.'h
000001a0: 056d d927 fe4f 2e92 894f ee80 d40e cecf  .m.'.O...O......
000001b0: 38a2 dc3a c193 6fd1 ecb9 886a cc3f 0422  8..:..o....j.?."
000001c0: 2f53 1c8a baad 9435 024d 92a7 435a 16eb  /S.....5.M..CZ..
000001d0: 1e48 0ecc d457 49bf d560 afeb 12ea 7e1e  .H...WI..`....~.
000001e0: 1d58 d154 2f02 7bdb c6c8 a850 0200 ae68  .X.T/.{....P...h
000001f0: 934f bdef d41a d63c f240 dc20 5a7c 34b1  .O.....<.@. Z|4.
00000200: 83f7 8feb 272a 2831 a59a a53c 7346 e891  ....'*(1...<sF..
00000210: 659f b913 6910 85e6 d867 0b0b 95ac 57db  e...i....g....W.
00000220: 3b03 f619 d4c9 7e73 e423 305b 7c90 15f6  ;.....~s.#0[|...
00000230: 6642 7b8d 470b 9166 5d78 56b3 61ff 647c  fB{.G..f]xV.a.d|
00000240: 3651 ccf9 ae90 2438 b53c 0334 2ba7 4192  6Q....$8.<.4+.A.
00000250: 9442 ee94 de3d ec93 925e 1efe 3fda 7ac8  .B...=...^..?.z.
00000260: fce6 f062 8bbc 7d00 e29e 8cf3 032f f53b  ...b..}....../.;
00000270: a53f 4278 73cd 2f3e 7e88 277e 1228 f177  .?Bxs./>~.'~.(.w
00000280: 8430 25f5 db25 58c4 6580 b220 6bb9 8e0b  .0%..%X.e.. k...
00000290: 2663 602f e7da 4022 162b a2a1 9e05 b20b  &c`/..@".+......
000002a0: b909 a221 c861 8e5d e0cc f2cf dcec 0e28  ...!.a.].......(
000002b0: f271 5a23 24cf be36 e0e8 7d10 a750 e8dd  .qZ#$..6..}..P..
000002c0: 9350 a0bf 6930 631c 6674 f38e d728 c35c  .P..i0c.ft...(.\
000002d0: 8002 da96 63ad edbb 63ff fb79 72f8 e930  ....c...c..yr..0
000002e0: 19d0 1a93 361d bbeb 1e74 af4e 4f6a c3ea  ....6....t.NOj..
000002f0: 00e7 46d8 41ab ef36 7bca 1c68 02dd a8ed  ..F.A..6{..h....
00000300: 1f31 43f4 8250 e833 6083 2b41 dd8e 560f  .1C..P.3`.+A..V.
00000310: 1368 620d 2064 1a42 8fcc 5f34 e5af 791a  .hb. d.B.._4..y.
00000320: d13b 25e4 77dc 906a 88e4 df94 2ac9 f04d  .;%.w..j....*..M
00000330: 9e1a 8f52 2d7d 8afe 3293 2496 9d68 d153  ...R-}..2.$..h.S
00000340: 7d15 444a e6c5 df64 1670 2ba7 564d 0263  }.DJ...d.p+.VM.c
00000350: 75b4 7fe5 02e8 640b f627 e061 4952 8a25  u.....d..'.aIR.%
00000360: 1ed3 81eb ade6 24ae 7910 7d08 b2b6 f7cd  ......$.y.}.....
00000370: e885 5631 3bc4 5553 f606 7288 9343 6b70  ..V1;.US..r..Ckp
00000380: 1027 1c73 e278 1774 c6f7 a5d3 1c44 643c  .'.s.x.t.....Dd<
00000390: eefe 7df5 c6ef 04ed 2548 fceb 9375 6fa9  ..}.....%H...uo.
000003a0: 1173 665d 7e42 d02c 3a4a 026c fada 4981  .sf]~B.,:J.l..I.
000003b0: 6a08 3943 0b9f 29d4 be4a 3900 fcd1 eb49  j.9C..)..J9....I
000003c0: 2920 54aa 0e8c f5ac f624 35ff 5c84 d157  ) T......$5.\..W
000003d0: 207e b7d4 391a 5b1c d90d 10f2 857d e5de   ~..9.[......}..
000003e0: ceda c0a7 9ed7 5dbc 8c8f 2e20 3e21 9646  ......].... >!.F
000003f0: d05d 2cc2 2595 260a b9bf 7d86 a8d3 6190  .],.%.&...}...a.
00000400: 2f7a aa96 4ee4 bf92 c887 1362 bbca 580c  /z..N......b..X.
00000410: ce3a 82b5 46c2 658e 8eb7 c111 4061 7aed  .:..F.e.....@az.
00000420: ea32 8c50 a4dc 3b36 93f3 a21d 1f75 5e51  .2.P..;6.....u^Q
00000430: 8290 e9ef f68e 2e5d 8ec7 480d c859 5ec0  .......]..H..Y^.
00000440: e505 2823 8ec0 0448 aa7f edcb 5779 dd74  ..(#...H....Wy.t
00000450: 16d7 39c3 a7f2 d076 1967 2759 cac1 ef24  ..9....v.g'Y...$
00000460: 935a 5b4a b09a 5661 c98f 84a6 79f7 045c  .Z[J..Va....y..\
00000470: bd1e 50e9 7f43 d4cc b669 682f ef37 be08  ..P..C...ih/.7..
00000480: b5d9 12d2 5732 67f6 aa6c 4c3e 65b6 d45f  ....W2g..lL>e.._
00000490: 4764 4113 4fab 71e4 fd52 92cc aaf5 dacd  GdA.O.q..R......
000004a0: 2c96 6f14 a87c 399c f644 b250 545b f7f5  ,.o..|9..D.PT[..
000004b0: 980d 5cee f18f 1e9b 3149 7bb1 48c0 edb0  ..\.....1I{.H...
000004c0: 9281 ba93 6c41 b5d7 3855 2c96 62ce 7a8c  ....lA..8U,.b.z.
000004d0: af1c 0a14 c16c cf3c db44 3351 2c40 66ff  .....l.<.D3Q,@f.
000004e0: 6d83 1ccd 147a deab 1d15 8aad c112 a97c  m....z.........|
000004f0: b6dc 02d4 5dc6 1eaf 9be9 a167 fb70 a0a9  ....]......g.p..
00000500: bd20 834c d5f2 f6d0 2ec5 84d8 2ecd 1b31  . .L...........1
00000510: aed5 ce55 c434 7e2c 47a3 19dd 3fee fb57  ...U.4~,G...?..W
00000520: 117b 368a 3049 b5ea 395e ed40 4ef0 9505  .{6.0I..9^.@N...
00000530: 37f6 f52a 60cc bc56 efc7 ed77 7468 12d9  7..*`..V...wth..
00000540: 9eaa 5071 4511 2bc8 5f9d 0c9d 13a3 a619  ..PqE.+._.......
00000550: 20b4 2f54 b466 898a 3508 a63e 1d7a 2a63   ./T.f..5..>.z*c
00000560: 423f b121 ae8a 51aa fedc 6260 75ec 0725  B?.!..Q...b`u..%
00000570: 5422 d4d8 d8de 3eca 9405 f388 81ac 9816  T"....>.........
00000580: 8dc1 64df 92e8 a213 49e5 2bdf e907 3259  ..d.....I.+...2Y
00000590: 5d4a f822 e998 ac30 a719 e989 bee9 2577  ]J."...0......%w
000005a0: d037 14e6 7da1 061c 0d54 73f1 0a7f 441e  .7..}....Ts...D.
000005b0: 6c63 cab4 c9ad f88f 0758 3d32 7bcf 4900  lc.......X=2{.I.
000005c0: f483 ae32 2d35 835c f970 77be 0472 6d27  ...2-5.\.pw..rm'
000005d0: 57e6 8af3 61c1 7be7 8381 5a54 a74f 9637  W...a.{...ZT.O.7
000005e0: f990 b952 5e01 bc8a 88e8 66e0 0000 0300  ...R^.....f.....
000005f0: 0003 0001 6d                             ....m

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=14336
pts_time=1.166667
dts=12288
dts_time=1.000000
duration=N/A
duration_time=N/A
convergence_duration=N/A
convergence_duration_time=N/A
size=2800
pos=18671
flags=__
data=
00000000: 0000 0aec 419a 1a3d 10d8 1485 8457 0000  ....A..=.....W..
00000010: 0300 0003 021d a037 7d54 92ce 043e 4d1d  .......7}T...>M.
00000020: f9bc f73e 1aef 5e5d bac2 e784 3805 1c33  ...>..^]....8..3
00000030: fde7 d99f 0485 61c6 d2a2 f276 279e f245  ......a....v'..E
00000040: 8f3b a6cf 3f2c 7649 2c3f 940a 6812 2bd4  .;..?,vI,?..h.+.
00000050: 6f27 f76f 26ba 0350 5fd1 c260 5147 cdcf  o'.o&..P_..`QG..
00000060: 780e 66a7 0891 7d72 d732 a61f 9275 f994  x.f...}r.2...u..
00000070: b0ce cba4 5459 9eca a679 4094 bf67 9501  ....TY...y@..g..
00000080: 6d75 9c63 79b0 18c7 05e4 00e9 e253 f684  mu.cy........S..
00000090: 5f9e c998 6c96 d97c aa58 cf4b c387 bc2e  _...l..|.X.K....
000000a0: 5d9a 2e6c 616b 7ade 32ca 69f8 6539 74a7  ]..lakz.2.i.e9t.
000000b0: b50c a344 1ff0 09ca 5a57 4a51 7c66 c7df  ...D....ZWJQ|f..
000000c0: df5f 29c5 74a5 bdeb 1e75 3c38 e402 654f  ._).t....u<8..eO
000000d0: 8688 4533 59aa 1c27 76b9 e950 ee56 77bb  ..E3Y..'v..P.Vw.
000000e0: 010c 2c93 c660 7010 889b 3e62 fc0b 1513  ..,..`p...>b....
000000f0: 8dc8 bb30 e80b 2727 f584 d397 a866 08ab  ...0..''.....f..
00000100: 475b 0ede 576f b4f4 d7b7 fa95 62dd 9b0b  G[..Wo......b...
00000110: 63cf 4c6c ea56 4884 e564 ea2c 8dd1 8382  c.Ll.VH..d.,....
00000120: b112 bb2f ab74 1864 0a91 bc16 e6b5 eea4  .../.t.d........
00000130: 000a c8a0 fd85 4901 2d61 2e8b 9958 e488  ......I.-a...X..
00000140: 0193 dea7 3121 421b 02f4 01a0 42d9 47a8  ....1!B.....B.G.
00000150: 453f ed36 42ae 97c3 9a4d 65cc f9dd 33a6  E?.6B....Me...3.
00000160: 04ca d9b9 cdaf b33f e084 d862 f109 e9ca  .......?...b....
00000170: ca10 00c9 6dda 507f bf0b 046d 2b2b a3f2  ....m.P....m++..
00000180: 6440 e76d 9858 49af fc9c 5e60 bc5a cb61  d@.m.XI...^`.Z.a
00000190: 4c4b 9805 e768 8b6e 2b42 ef97 df36 9aaf  LK...h.n+B...6..
000001a0: 9da7 ff70 f240 9b68 6de0 ccb8 37b1 cb78  ...p.@.hm...7..x
000001b0: 6a0a 662b f651 a038 ee6f c3bf 818e f5d0  j.f+.Q.8.o......
000001c0: f995 e9fe 343e 428e 7f35 07c0 93be 3f74  ....4>B..5....?t
000001d0: 622a 1fab 94f8 9787 9305 35ff cbd7 8d4b  b*........5....K
000001e0: 5491 0b60 0431 64a0 1f7a 29c2 586a c00d  T..`.1d..z).Xj..
000001f0: 4239 c118 8b4f a2d8 61e7 b12c 87bc fbe9  B9...O..a..,....
00000200: 7b30 d3a3 74b8 f0d8 9ae0 bcde 275c 6e5a  {0..t.......'\nZ
00000210: f789 8fe1 8525 abad c4fe 5cec 124b 4a07  .....%....\..KJ.
00000220: cbda f106 bf94 9445 38e9 8c5f 8e17 f571  .......E8.._...q
00000230: dd90 669e 69a4 ad72 73cf 81f5 01ec f338  ..f.i..rs......8
00000240: 1f2f cb40 667e 479c faf7 5819 364c f075  ./.@f~G...X.6L.u
00000250: 35ca d196 dfca 1d40 2fb7 b3b9 68ed dd9f  5......@/...h...
00000260: 6a2a 2060 22c3 99b4 a93d 2e34 432a 79b5  j* `"....=.4C*y.
00000270: 9693 ab8c da2e b358 dade 2e0d 5e9d 1bae  .......X....^...
00000280: e0b1 788b 68fe 2630 c408 6c94 359e c4ff  ..x.h.&0..l.5...
00000290: 8c68 e513 fbb5 00fb d852 a741 3791 d0df  .h.......R.A7...
000002a0: 32e1 9aa0 59e6 90ef 4053 54d5 730a d6fc  2...Y...@ST.s...
000002b0: 3a3c 8f90 f071 d62b 608c a9df 0d98 833a  :<...q.+`......:
000002c0: 7c2b 3504 c451 ba74 a387 0aa8 077c 555e  |+5..Q.t.....|U^
000002d0: f385 8204 cafa 0505 2bc9 ab16 22b7 276c  ........+...".'l
000002e0: 7aa6 8161 3e88 fe9c 8e7c b5a3 3739 74ad  z..a>....|..79t.
000002f0: d487 e56c 0414 6a35 e52e c882 44e5 1052  ...l..j5....D..R
00000300: e75a 4fe0 b7f0 8883 ce9e 35bd 1644 6d1c  .ZO.......5..Dm.
00000310: 87e7 6599 2e6e 1fbc 59c7 c749 8088 2048  ..e..n..Y..I.. H
00000320: 90ca 7933 da4f b0b1 fcf0 7472 48b5 5406  ..y3.O....trH.T.
00000330: 2ffa d811 4389 6659 b526 6c2b 7cd5 e9c3  /...C.fY.&l+|...
00000340: 9084 4756 a59f cead 4fb1 57bf e506 fc4f  ..GV....O.W....O
00000350: d315 4aa9 0280 b1ce 1389 a745 f8e3 7b50  ..J........E..{P
00000360: 3ae4 7c0b 20c3 452b 5744 07d8 7ba1 722c  :.|. .E+WD..{.r,
00000370: 8826 31ab 8273 ec57 ffe5 fd25 24db eb66  .&1..s.W...%$..f
00000380: 9657 f00c 3193 ec91 de85 bbda ff67 ef3a  .W..1........g.:
00000390: ea73 0dda 1ec7 8cbc 5092 4d01 e394 2729  .s......P.M...')
000003a0: d2d0 fa71 c00c 08ed 0ea6 aec8 5009 a268  ...q........P..h
000003b0: 7a8a aff3 3d9b 753f e6aa b040 a843 bff3  z...=.u?...@.C..
000003c0: 40fe 249f 67e2 b936 36e0 6a70 a763 a80d  @.$.g..66.jp.c..
000003d0: 3a5f a217 ae81 b527 f9fd c885 3e5c 520a  :_.....'....>\R.
000003e0: d7d2 b9b8 a24a 7db4 bea0 d600 abd5 c34d  .....J}........M
000003f0: 0881 f24a 1477 2459 edd5 3512 aa8b d202  ...J.w$Y..5.....
00000400: 1990 cf16 7e97 c1e4 4fd5 e805 00a0 b886  ....~...O.......
00000410: 8ff6 201d b825 320c eb62 c248 c02a bbc6  .. ..%2..b.H.*..
00000420: a2a9 ddaf 00bd 2f1c e3ff b920 2bf6 3bfb  ....../.... +.;.
00000430: a94c 8cfd 4099 be38 283a e007 d0da 73b5  .L..@..8(:....s.
00000440: 4a20 8c9a 5c5a fd9e 7f96 acdc a0dc ca72  J ..\Z.........r
00000450: 43f1 c0f8 50ae bce2 887c 2a65 79bd 67f6  C...P....|*ey.g.
00000460: c653 143b 2c43 6400 f706 210c 9fba acf5  .S.;,Cd...!.....
00000470: 2048 1bed e184 1fc6 3c4d 44d0 f91c a21d   H......<MD.....
00000480: 9166 c6cb 205c ad04 503f bd00 843d 074f  .f.. \..P?...=.O
00000490: 1961 2316 7aca 19f7 6d24 ea4d f69d cf31  .a#.z...m$.M...1
000004a0: 90ab 8d0a b70e bdbe cefc 6db4 3335 2b70  ..........m.35+p
000004b0: 8ac3 9f93 6175 c605 adca a608 5d3a 446c  ....au......]:Dl
000004c0: f755 993e 11e3 0363 a91e eb52 3b4d e9a0  .U.>...c...R;M..
000004d0: 2bc2 bcc3 b0e4 f8fe 185f 3f41 d980 22a3  +........_?A..".
000004e0: 957b d95c e5be d2f1 311b 66e1 b550 c078  .{.\....1.f..P.x
000004f0: 9ea7 cef2 c1ca 9538 fe09 1b23 3e9a 445f  .......8...#>.D_
00000500: b119 513f c237 85a1 20ea 4596 719f 156d  ..Q?.7.. .E.q..m
00000510: 83e8 ac47 9587 2bf3 25b5 3ce0 d843 c01b  ...G..+.%.<..C..
00000520: ff32 cffc 5539 9d90 7d0a 7044 9aa5 54a7  .2..U9..}.pD..T.
00000530: da9e 34f1 5b22 8d8d e79d 74ae 7b27 d7de  ..4.["....t.{'..
00000540: be4e 4d30 d63d ebb8 4ed8 48b1 9529 8144  .NM0.=..N.H..).D
00000550: fd7b 6542 98e5 ea00 2fcb 95f5 1b56 f65e  .{eB..../....V.^
00000560: b5a1 fddd 9c0f ee4d 3c1d b887 283b 9f95  .......M<...(;..
00000570: 131c af1b 0f51 2839 7b24 2983 04ff 998c  .....Q(9{$).....
00000580: d0a2 d663 db66 efff a4e7 d1fe cf6f 2129  ...c.f.......o!)
00000590: c09e 206d 8d0b b0b4 ff73 d88a 6a3a 0755  .. m.....s..j:.U
000005a0: 797f 7ddc 7535 f794 9f1c ec57 1c59 d62d  y.}.u5.....W.Y.-
000005b0: c140 9f2b c677 7bc2 b857 4390 05db 985a  .@.+.w{..WC....Z
000005c0: 2ec5 baa9 ddee f84f b8df fa5a b75a a7cd  .......O...Z.Z..
000005d0: 2338 39a2 de92 11bc c114 35e8 31f8 26a4  #89.......5.1.&.
000005e0: b8a0 ee94 9bdc 8960 cdd7 42e8 4ff9 864c  .......`..B.O..L
000005f0: 07d4 6f9b 15a6 d7e5 fb3d 95b5 1417 584a  ..o......=....XJ
00000600: c741 d9f1 136d ba49 7102 bfaf 461b 1223  .A...m.Iq...F..#
00000610: 593f 2009 d27a 7f60 b4eb d9fa c46f 5b72  Y? ..z.`.....o[r
00000620: 0ab0 1f42 1303 4271 2974 6472 3da1 928e  ...B..Bq)tdr=...
00000630: c6fb dac9 81e4 d007 3aa8 4584 8275 0680  ........:.E..u..
00000640: 1077 24f0 3bd6 2fcc 6f9e dd33 0ca8 2715  .w$.;./.o..3..'.
00000650: 5fba 576e d825 5ca7 1f46 4174 a3ee 7c37  _.Wn.%\..FAt..|7
00000660: b6f1 2bbf bc88 47be df1a c523 c1ff 3fec  ..+...G....#..?.
00000670: 4981 7bd9 af3f 79ce 0a97 a6a1 ac25 3ecd  I.{..?y......%>.
00000680: ace4 34ee 31b5 e9d6 b055 cf31 3d9f 757a  ..4.1....U.1=.uz
00000690: 41a1 bdfb 7c8c 63f7 9354 3705 725e c7ad  A...|.c..T7.r^..
000006a0: 7523 f0b1 b3e3 4981 c933 5f21 b159 53c2  u#....I..3_!.YS.
000006b0: 1110 4e3f 8bdf 5fab 0079 cb8b 227b 9d29  ..N?.._..y.."{.)
000006c0: 3668 0751 6b1f ba77 8490 bda4 f2a5 e681  6h.Qk..w........
000006d0: 5936 f702 7a2d 7593 d34b 457d f2f3 5357  Y6..z-u..KE}..SW
000006e0: 60a8 caba 3056 0282 37ed 12dd dad3 1974  `...0V..7......t
000006f0: b936 cb64 cc3d 364e 08a2 26ba 3bf4 b2ad  .6.d.=6N..&.;...
00000700: 879f 7794 8e84 52f8 d38e 6ae0 1fc5 eb10  ..w...R...j.....
00000710: c401 db01 fc56 8d99 070d f722 8b20 fcdb  .....V.....". ..
00000720: 4367 7ee7 6c7e 8918 7919 13a7 b421 4bf6  Cg~.l~..y....!K.
00000730: 4cbb a826 a5f7 d2a9 718c 282c fd3d 56e2  L..&....q.(,.=V.
00000740: d74c 8018 aecd 5892 a217 73ce e733 1ff0  .L....X...s..3..
00000750: ea91 d751 c384 9379 c5d0 f764 7575 19fd  ...Q...y...duu..
00000760: e047 93d9 e49c e1f9 74a0 f4fe 7d64 f37b  .G......t...}d.{
00000770: 904c 8004 c8f3 b491 2f42 ce91 66e1 30e1  .L....../B..f.0.
00000780: 1dd0 29c4 ce9d 2c65 0262 b745 e578 b047  ..)...,e.b.E.x.G
00000790: fb18 184f e6a0 2c1f d1f3 dfd4 3485 2451  ...O..,.....4.$Q
000007a0: 25ac 187a b5a8 0162 207e afb2 f13c 1f63  %..z...b ~...<.c
000007b0: b5d9 2dcb 292e 6fa5 af7d 3403 83b1 6080  ..-.).o..}4...`.
000007c0: c938 7b3d a13f a940 e1a2 d71a fe58 37c3  .8{=.?.@.....X7.
000007d0: 5791 6955 d89a 1daa 9570 4b93 3f5f 05ed  W.iU.....pK.?_..
000007e0: c23f b0a5 8433 82e6 d702 3ec0 8000 008f  .?...3....>.....
000007f0: d4b0 ffef 6a00 a9a2 5b5e 7270 cf87 b75d  ....j...[^rp...]
00000800: ac2e 3dc4 1a2f 69a5 f3c5 7524 6055 eff1  ..=../i...u$`U..
00000810: 58ea 8026 dc3f d575 f37c 1a11 7fa2 82f0  X..&.?.u.|......
00000820: b522 394b 1c16 6d7a a804 bc44 b349 2d19  ."9K..mz...D.I-.
00000830: 022d b4bb 04ac 7754 33b3 e552 be66 2c7e  .-....wT3..R.f,~
00000840: 5447 0806 06f8 6df0 121e baf9 324f 65f7  TG....m.....2Oe.
00000850: a000 04d5 5797 fe62 35d5 9833 209b efb8  ....W..b5..3 ...
00000860: f2f2 e0fa d3be 516b adfe a0ab b703 e472  ......Qk.......r
00000870: c228 a577 2f20 82bb 3d6b c153 f501 1b27  .(.w/ ..=k.S...'
00000880: 63ae 5f4b df91 664a 3f7e 5f0e b09a 5535  c._K..fJ?~_...U5
00000890: 27b6 268f 4ad7 525f 271f 2b04 cab6 088e  '.&.J.R_'.+.....
000008a0: 15fc 8370 a8ff c30b 3224 6de8 2eeb 53db  ...p....2$m...S.
000008b0: a304 058b 7e68 b2df de70 8a92 8688 7270  ....~h...p....rp
000008c0: bc35 d04c b4c6 c179 1ace f2f1 b3bc 4af8  .5.L...y......J.
000008d0: 9f86 2a08 e0ee abb9 9948 257d 509b 488c  ..*......H%}P.H.
000008e0: d47a ecad cfca c406 f288 1a06 0ea5 70f9  .z............p.
000008f0: 4d09 1b06 be95 f904 58ae 44aa 640c 3026  M.......X.D.d.0&
00000900: e425 ecd8 666c b47f 141c eb2b 23e3 5b7c  .%..fl.....+#.[|
00000910: 0841 2c7b 107a 96f5 658f 6902 839b eace  .A,{.z..e.i.....
00000920: 7d4d 82ba 2476 6cd6 6da8 f076 5865 664e  }M..$vl.m..vXefN
00000930: d1f6 ff3f 8d24 9f2e 769c d3bc f2e2 1c3b  ...?.$..v......;
00000940: 7532 49d2 5b8d 3a87 1d9d 64a5 d717 58d8  u2I.[.:...d...X.
00000950: 5bbb 5884 12af 89a0 4b2e 3dcb a155 cb74  [.X.....K.=..U.t
00000960: 029d d374 21a5 2a9e 1684 7996 f8ea b34c  ...t!.*...y....L
00000970: 2cb0 e54d 0595 5b7d 271d b0a6 09bb f5d9  ,..M..[}'.......
00000980: de4a 5edc c1a6 e446 e594 40c1 69dd 043c  .J^....F..@.i..<
00000990: bf94 b91d 20c0 0ff0 eefd f57d 3ba9 6ad7  .... ......};.j.
000009a0: 5a83 fe08 ca83 9bab 8835 713e 8c38 5013  Z........5q>.8P.
000009b0: 2f16 d20b 7ed6 8411 a398 c6ab 1e36 9647  /...~........6.G
000009c0: aa4a 6096 eb39 e204 cf7a 837e 9947 a5af  .J`..9...z.~.G..
000009d0: b760 8657 48da 4af7 7c8b 3f49 ae95 f712  .`.WH.J.|.?I....
000009e0: 5652 5b61 a8f9 9386 8654 8a92 6012 a5d1  VR[a.....T..`...
000009f0: 4339 a71f bc1c 7804 e03a d802 b0e9 6ac4  C9....x..:....j.
00000a00: 5bf3 4993 5bf3 07e2 4ea8 5ea1 b13e ff30  [.I.[...N.^..>.0
00000a10: ad8d 337a 4943 3c63 1031 9ce0 9e74 ee55  ..3zIC<c.1...t.U
00000a20: 60da f49d 14c6 1604 b37a 0e65 adbc 0a9d  `........z.e....
00000a30: 3ca5 76cc a2ca 6e38 2fae 30e4 46eb cbf5  <.v...n8/.0.F...
00000a40: f949 74a4 41bb 56cd acc5 eb54 f291 8901  .It.A.V....T....
00000a50: 0534 7a61 f3cf a59b c90b 0719 8057 f88f  .4za.........W..
00000a60: 0c63 6959 8564 f190 0c08 2b13 531b 450f  .ciY.d....+.S.E.
00000a70: 93f7 35fb d1f2 0000 2fbf 9ad2 c6dc 6624  ..5...../.....f$
00000a80: b586 e297 b028 18ac 3f3d 8f1d a72b 453e  .....(..?=...+E>
00000a90: d08b c7ed 7d4c 7227 8e0a d645 52fc daee  ....}Lr'...ER...
00000aa0: 2ae1 cca0 ccc5 5631 500a 5549 6663 b409  *.....V1P.UIfc..
00000ab0: 1899 1bce fbd1 ba3a 472d 1275 44a3 1a09  .......:G-.uD...
00000ac0: f17e f833 5f24 d266 a5a6 c5a1 d46d 0ca7  .~.3_$.f.....m..
00000ad0: 85af 5c82 f50c 6e1f dba6 c089 f31b dae2  ..\...n.........
00000ae0: 4235 7277 6d13 3f8f 84ae 025e 0000 de81  B5rwm.?....^....

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=13312
pts_time=1.083333
dts=12800
dts_time=1.041667
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=2565
pos=21471
flags=__
data=
00000000: 0000 0a01 419e 3845 344c 5700 0003 0000  ....A.8E4LW.....
00000010: 06d1 af38 b22c 55d2 85a2 3088 7f56 9441  ...8.,U...0..V.A
00000020: 9953 6207 5f53 bfba 89cb 4f4b 5da2 94e4  .Sb._S....OK]...
00000030: 55b8 6a98 f2ca 311c fd34 478e 025c 9170  U.j...1..4G..\.p
00000040: 48a0 d223 d419 1b79 cfb6 2481 d6bc e05e  H..#...y..$....^
00000050: 8670 fae2 5495 d189 a484 6c3b db47 6f71  .p..T.....l;.Goq
00000060: 60e1 524e 0d20 138c 7874 ca63 ba8d 180c  `.RN. ..xt.c....
00000070: be38 d56b 4635 dca5 3568 6da8 15b9 a609  .8.kF5..5hm.....
00000080: e105 2ab7 22fd d666 392a b4e7 9038 623e  ..*."..f9*...8b>
00000090: 8b1a 1448 80ff 6311 5417 6ca5 ab85 c7ca  ...H..c.T.l.....
000000a0: ac1e 9e2d 1011 1414 c2d5 aa0c bfed 3302  ...-..........3.
000000b0: a63f 6f7a 544c dbdb 0815 630c 664f aeb0  .?ozTL....c.fO..
000000c0: 349d f9c0 f921 e664 dabe 7cb3 a7ba 096b  4....!.d..|....k
000000d0: 94aa 1e7b e732 f76f 6488 4583 a91e 3457  ...{.2.od.E...4W
000000e0: e8de 6e23 df4d f664 82fd 9797 8957 a686  ..n#.M.d.....W..
000000f0: 5f37 ac8b e32b 3a56 56b7 4e77 2dcb 425c  _7...+:VV.Nw-.B\
00000100: 58e1 6823 5b8b d930 4b06 b1f4 4046 9804  X.h#[..0K...@F..
00000110: 944b bf15 624a 8646 55c8 ada9 1869 5fac  .K..bJ.FU....i_.
00000120: 65d7 27c5 8d75 f3f5 6c36 6212 2842 30f1  e.'..u..l6b.(B0.
00000130: e351 56ba 74d5 7b83 7778 6a53 c707 e5cf  .QV.t.{.wxjS....
00000140: 1d37 1d07 b728 9662 7b41 a416 1876 28f8  .7...(.b{A...v(.
00000150: 3654 6517 0169 5184 180b daae 0fdc a3cc  6Te..iQ.........
00000160: eb74 95cd 8c87 41a3 47f0 de0d 6e24 c5a5  .t....A.G...n$..
00000170: 8560 8c4a 714d 5ffc 9238 9a8c bf54 9acc  .`.JqM_..8...T..
00000180: efe2 9f85 070c 0602 7c3a 7e5c 2ab2 e2ef  ........|:~\*...
00000190: fd17 9e76 803c 9a90 186d a391 fb1c c4e1  ...v.<...m......
000001a0: 47bb c771 4c15 45ab 3b2e dc91 2c77 9cb9  G..qL.E.;...,w..
000001b0: 155b 359d f074 a1c3 e8fd df7b fb53 fd22  .[5..t.....{.S."
000001c0: 903a 2da7 2822 c177 1805 8786 00e8 2396  .:-.(".w......#.
000001d0: 9c6c 500a 2c2c 7901 b5a2 dbe8 3226 f4be  .lP.,,y.....2&..
000001e0: 871b 3762 3c10 f7a8 b20e 6216 9205 5c0d  ..7b<.....b...\.
000001f0: c78b 7c4b ef7a 6696 04a6 6932 2291 538c  ..|K.zf...i2".S.
00000200: 593c b46a d016 c6d7 9033 ff71 7231 15ba  Y<.j.....3.qr1..
00000210: 9e10 8509 f49a 4098 76b2 172d 6dad bf61  ......@.v..-m..a
00000220: da45 fd7e 07b9 aa51 3156 4be4 93be 2582  .E.~...Q1VK...%.
00000230: 0423 dfeb 480f 6700 6b1f 5946 a423 7d94  .#..H.g.k.YF.#}.
00000240: 9dbb aa34 6de6 71bb 130c f37b 5433 25bd  ...4m.q....{T3%.
00000250: 3932 1349 7cde 0533 3c41 8293 454c c486  92.I|..3<A..EL..
00000260: af8f 1c36 c24b ba7a 703c e997 b534 865a  ...6.K.zp<...4.Z
00000270: c22a 739e 87b3 3b6d d57c f654 d7e8 c11a  .*s...;m.|.T....
00000280: 9305 5165 98e1 9341 8d37 8e9e 015a d782  ..Qe...A.7...Z..
00000290: 360b fa69 f67b 9976 4564 b9a0 3efa 1542  6..i.{.vEd..>..B
000002a0: 2c6a 0a20 a083 648a 7568 f299 4f16 7c91  ,j. ..d.uh..O.|.
000002b0: 75ac 4287 a77d b97a 2b70 223b a247 49c1  u.B..}.z+p";.GI.
000002c0: 7878 d210 52df ebf8 efb1 ce41 d788 79c9  xx..R......A..y.
000002d0: b622 4a2c 0be5 236f cdec 092e 3295 2be3  ."J,..#o....2.+.
000002e0: 7416 8202 91bf 3284 76aa b6e3 322c a2bb  t.....2.v...2,..
000002f0: b757 314b b40c 6765 50fc 1575 ca0f a478  .W1K..geP..u...x
00000300: 5d81 743c 1a72 7c24 cd23 8611 fe11 d206  ].t<.r|$.#......
00000310: 4cdd 6110 5fc4 b13c 743f 5124 b7ac a6ad  L.a._..<t?Q$....
00000320: b77c f521 c2a7 1397 2eb4 3823 8ed2 6a88  .|.!......8#..j.
00000330: 5fa9 6bd0 b802 26d0 8937 77fc 6d17 5354  _.k...&..7w.m.ST
00000340: 9fc8 2681 7e84 b546 86e3 e257 2846 2c99  ..&.~..F...W(F,.
00000350: 58e1 8e3e 00ae 17fe 0755 2234 5299 96ce  X..>.....U"4R...
00000360: 05ed 27f4 0af7 2cab 0b79 36f8 f6d3 082f  ..'...,..y6..../
00000370: 6e0f b9a7 402a 799e db97 e04f 56a9 eabc  n...@*y....OV...
00000380: 3949 a112 89bd cc8b f160 dd79 37ec a7a5  9I.......`.y7...
00000390: f9eb 7e58 bbec a14f 04d0 27a2 7cbf 3122  ..~X...O..'.|.1"
000003a0: cda0 858e 598c 0c02 3e76 e5e6 9752 bb00  ....Y...>v...R..
000003b0: 90f6 8b6e d60b 2679 69d0 5ef7 db1c 943d  ...n..&yi.^....=
000003c0: 7a88 7bbd 007d f941 306e 0b8c 66a1 56c9  z.{..}.A0n..f.V.
000003d0: 841f ede3 54fa 19ea 380c 9d9e 58f1 a06e  ....T...8...X..n
000003e0: f3fe 8e97 a9a4 940a 477a 5516 6ace 869d  ........GzU.j...
000003f0: 1ccf 7e13 1efc dfcb 670b ebf2 5fc2 bbd9  ..~.....g..._...
00000400: 1877 554a 8246 889c 0f3a b1e9 3f19 8806  .wUJ.F...:..?...
00000410: 525e ba36 2228 4121 97fe 981b 35e4 8ecb  R^.6"(A!....5...
00000420: edc7 c1f4 2103 b603 13e4 2c53 54a5 e4cd  ....!.....,ST...
00000430: 47d0 15e8 a33d 276f e3f4 602d 2398 3da2  G....='o..`-#.=.
00000440: 2518 9996 69e0 7913 c3b2 4e85 6326 3bc1  %...i.y...N.c&;.
00000450: 51e1 82e9 e515 a8a5 7600 258d a133 5d7b  Q.......v.%..3]{
00000460: 77e3 da63 56a6 1f03 d7b8 1e98 d14c 3e3a  w..cV........L>:
00000470: 9216 8950 8d04 5966 10be 6dc0 e3ef 1588  ...P..Yf..m.....
00000480: c971 4cb1 39f2 1482 f4d0 1b55 308b c96f  .qL.9......U0..o
00000490: e88d 6982 27ef 3fa1 7436 4ad6 3072 e50c  ..i.'.?.t6J.0r..
000004a0: 84dc 4d3c cbc3 7dd6 98c9 266a 2f5a 1156  ..M<..}...&j/Z.V
000004b0: b733 4521 58e6 b31c 6334 ffb5 8096 c90a  .3E!X...c4......
000004c0: 4dea 67b9 e17b 556d dd4d b501 b2e9 8c68  M.g..{Um.M.....h
000004d0: 8ec0 a386 6aa2 f375 014d 6b99 c9c1 b6f0  ....j..u.Mk.....
000004e0: 3e24 4737 5a59 4652 4141 429c 73b4 813f  >$G7ZYFRAAB.s..?
000004f0: 108d a95b 9a2b 4d95 714a 1852 04e2 1a22  ...[.+M.qJ.R..."
00000500: fe60 8a2f c69a 0b0e 4366 3761 3afd 7bf6  .`./....Cf7a:.{.
00000510: 8f5e e620 63cd 756f 98d3 3401 b8d7 2808  .^. c.uo..4...(.
00000520: d547 5dc9 970c cf69 93e5 8ba2 4fea 3742  .G]....i....O.7B
00000530: 8da6 806b c4fa b47a de6d b5cd 26ff 3544  ...k...z.m..&.5D
00000540: 8da0 e623 1ae9 a6df e268 67e5 2b31 9253  ...#.....hg.+1.S
00000550: 4737 966c 7457 9087 39af 79f3 2b2b e0d7  G7.ltW..9.y.++..
00000560: 5ea3 c695 dffc 66c2 34ed d97c b170 ab30  ^.....f.4..|.p.0
00000570: fe32 f098 039c 6781 394b d5e5 5799 f4ad  .2....g.9K..W...
00000580: 18de 1afa 3afd b6eb 49c7 1c14 da03 eae0  ....:...I.......
00000590: 9500 0494 7811 1ff7 8360 a952 13f4 87d8  ....x....`.R....
000005a0: bdfd 9487 aea4 8bd0 b9a4 352a a6a4 ce4f  ..........5*...O
000005b0: b8e8 2044 6fdc e79c 5a0f f46e 64c3 364d  .. Do...Z..nd.6M
000005c0: 0051 ee5a e017 6d6b e3b5 87a2 2817 d9e4  .Q.Z..mk....(...
000005d0: d29e 42ff 4bda 58c7 462e ca5f a766 c1ee  ..B.K.X.F.._.f..
000005e0: 5361 5708 66b0 c539 2d9f c8ae ed9b 3ff5  SaW.f..9-.....?.
000005f0: 1ba3 4ca8 a6ef 6b7d f400 fba8 2ec7 b8c0  ..L...k}........
00000600: 4548 a96d 51b1 7c3c d9fe 61f1 0567 4364  EH.mQ.|<..a..gCd
00000610: 39bb e21c 38ff dd4b 6c15 49e3 bbb3 d864  9...8..Kl.I....d
00000620: 2e90 f562 3ab1 e1f7 470e d7f2 5fa0 a052  ...b:...G..._..R
00000630: a217 362b 3039 17af 136e c24e 51e5 dfaa  ..6+09...n.NQ...
00000640: 8c07 cb21 3603 595e 4562 41ae c6f8 4b51  ...!6.Y^EbA...KQ
00000650: 825e 22ac ba5c bdc9 63a0 5d7f 07d6 85cc  .^"..\..c.].....
00000660: 48b1 cab6 b742 f32f dec5 b1f5 9639 c5cc  H....B./.....9..
00000670: 0d1a eb8b 17f2 6a96 8159 8369 821c 9b1f  ......j..Y.i....
00000680: 069b d6a9 312f 3689 0210 41d2 b34f ddae  ....1/6...A..O..
00000690: 8c6f 16f1 d96c 91cc d935 1e90 1013 301d  .o...l...5....0.
000006a0: 9c5e 2ce7 3166 4ae6 fd71 cb0b feca e257  .^,.1fJ..q.....W
000006b0: a94f 760e c721 b6ef a978 83ea 2652 76dc  .Ov..!...x..&Rv.
000006c0: 7ac5 d25a 746d 74f6 06a6 9d59 f5d6 21b7  z..Ztmt....Y..!.
000006d0: 6063 a79d e093 bef3 ca7d 3106 451d 1f9a  `c.......}1.E...
000006e0: f651 6b31 0d33 b421 743a 8183 c5ef 76e1  .Qk1.3.!t:....v.
000006f0: a16d bde5 b898 5590 cd36 3c42 6170 16e5  .m....U..6<Bap..
00000700: c4a9 5914 7ef2 7974 1f85 a3ee 4075 5ba8  ..Y.~.yt....@u[.
00000710: 6196 1355 e818 141b 3dbf 816a ed3f 8e6b  a..U....=..j.?.k
00000720: 2dfc 01ff a840 83ad 23e8 d7e1 a24a 2cc3  -....@..#....J,.
00000730: 6c1f 2dde 6028 9ed6 a844 a243 1c79 7d7c  l.-.`(...D.C.y}|
00000740: 7a5f dad2 5449 701d f178 dcfe c90a 6527  z_..TIp..x....e'
00000750: 810a cf31 5cea 1683 4cfb a250 a6d5 11c9  ...1\...L..P....
00000760: a18a ab82 432f c742 aee6 e468 10ca ada5  ....C/.B...h....
00000770: c4f9 2aa0 d307 0333 bd1c 0691 4084 cf0f  ..*....3....@...
00000780: 2169 9965 74ed f885 da05 8f90 0c73 de5c  !i.et........s.\
00000790: d05b 5b3b e8fb 24a6 3a43 b21b 9033 3232  .[[;..$.:C...322
000007a0: 4a6c 131c cb94 bb42 1257 ba50 87e2 f9c8  Jl.....B.W.P....
000007b0: 064d fc81 35d1 0f0f 8ef1 1678 1283 2fdd  .M..5......x../.
000007c0: 8c64 f8c5 bb2e 79cd d96e a44d a63e a2d5  .d....y..n.M.>..
000007d0: 3a36 a5fb 6938 c99c d07b da9d 773d 739a  :6..i8...{..w=s.
000007e0: d2bf f9cb 7ffa dac3 aa50 4e92 0c74 9cd1  .........PN..t..
000007f0: bdbd e536 1a39 6691 394f ed22 12fb 8305  ...6.9f.9O."....
00000800: 419d 722b 66d7 0351 138d 8c88 f8f0 2e06  A.r+f..Q........
00000810: 6f81 fa32 593d 622a ecac 96d6 a42c b2b5  o..2Y=b*.....,..
00000820: a75f 3333 17bf 6673 bfe1 92e4 29de 7cd0  ._33..fs....).|.
00000830: ff40 045e e42a e00c 1166 f729 5c37 4a09  .@.^.*...f.)\7J.
00000840: 4e53 0d0c 3e8d b86d db88 4eb3 cf22 5e1d  NS..>..m..N.."^.
00000850: bf9d 6be7 56c4 c9f0 cb18 1231 a42b 72b4  ..k.V......1.+r.
00000860: bf10 99cc 115c b271 7a36 7242 eb16 0289  .....\.qz6rB....
00000870: 744f 8f9a 823c 3383 b0cc 82a3 9112 b315  tO...<3.........
00000880: 7497 4ba9 cf87 1dd5 83c5 9c2d 8f22 903e  t.K........-.".>
00000890: fb40 c61b 7734 89b5 d41a f7d3 713b 0527  .@..w4......q;.'
000008a0: bc2b 50b3 80ca b381 c1f8 4d13 2dcd 7f5a  .+P.......M.-..Z
000008b0: 6ba3 e95e f376 a4b2 1b75 a4dc a5b0 0f9f  k..^.v...u......
000008c0: c9a3 3704 ad43 4289 84f5 4648 47e7 8a40  ..7..CB...FHG..@
000008d0: 61e5 d9b4 a228 7a69 5cea 6ab9 ccd6 9e16  a....(zi\.j.....
000008e0: 0af3 d9a0 8c26 3b37 9e16 0ad1 3ca4 558c  .....&;7....<.U.
000008f0: cd29 c7d0 cebf 2292 12a6 0cbf ef5b 2d34  .)...."......[-4
00000900: 7f58 ca2b 2c0e 059e d1f9 a02e 6e01 0f0e  .X.+,.......n...
00000910: 764c 113f 98b7 9e75 cee7 58f3 b194 6899  vL.?...u..X...h.
00000920: 0fcc eabc da37 9b5a 89a9 7153 b58e f345  .....7.Z..qS...E
00000930: f807 b8be b216 75c7 81a2 e68a 2d67 c2c7  ......u.....-g..
00000940: 05be f9be f43c 5e7b 6bba f999 8d26 be80  .....<^{k....&..
00000950: 2f59 1286 8f95 0897 9aae 1492 cc2f da6e  /Y.........../.n
00000960: 7ed1 a80a 1ab9 9dfa ec99 d541 40ed cd12  ~..........A@...
00000970: f02f 970d ceed 522c 1ba1 4d4f 476e 02eb  ./....R,..MOGn..
00000980: 771a 2a1a d49f e167 53a3 fbbc efaf 584d  w.*....gS.....XM
00000990: c36d 3649 f3c5 5253 38fe 95c6 1beb 867b  .m6I..RS8......{
000009a0: 9d16 293e 0435 06a2 0d90 d5bf 37bb 6699  ..)>.5......7.f.
000009b0: 3be0 4eec 8c23 5c9e 7cda 75b4 98eb fa36  ;.N..#\.|.u....6
000009c0: e02c 1a98 7415 0513 5795 3d1b f890 93c1  .,..t...W.=.....
000009d0: 81dc 31e4 4025 7368 7ae6 7dfe 12bf a1d4  ..1.@%shz.}.....
000009e0: b15f f9cb d1a9 72a2 4858 40b1 2f12 2dcd  ._....r.HX@./.-.
000009f0: a458 3876 1114 dff1 dbb3 842f 532c 0000  .X8v......./S,..
00000a00: 0300 0047 c0                             ...G.

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=13824
pts_time=1.125000
dts=13312
dts_time=1.083333
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=1930
pos=24036
flags=__
data=
00000000: 0000 0786 019e 5944 5700 0003 0000 07cf  ......YDW.......
00000010: c4df 1ba0 41ea 2774 0e31 b8f7 fb95 c1ff  ....A.'t.1......
00000020: 463b 89ac ffcb aaa3 4fab f10e 8fc4 b1da  F;......O.......
00000030: 7bce 716b 5555 db0e 89f1 9a1f c6b0 e25b  {.qkUU.........[
00000040: d8a9 20a2 4695 71a6 121c 8799 8894 ffe1  .. .F.q.........
00000050: 944b 50f7 9de2 a809 0552 2c8f 04a0 6d39  .KP......R,...m9
00000060: 7d8b e266 46dc c0a2 6c64 99fc 54d7 990f  }..fF...ld..T...
00000070: 2542 0129 f32a 196c cb6f 29ab f413 f58e  %B.).*.l.o).....
00000080: 6591 147f 42a9 2f6d 1515 0e66 6652 dff0  e...B./m...ffR..
00000090: eebf 5b05 c4a4 ffd6 b4e4 8742 aaee b67c  ..[........B...|
000000a0: 5baf a00d 9830 f799 b115 e5f1 8985 7e83  [....0........~.
000000b0: b69b fea4 f801 da0c efb2 336c 2bc9 3829  ..........3l+.8)
000000c0: 53d2 84ee c5f9 544f a985 7a24 6a36 e546  S.....TO..z$j6.F
000000d0: f77d c04b 6000 af9b aa41 adcb 8adc a4da  .}.K`....A......
000000e0: cb43 b644 4ffa 0e40 14d3 1e43 21ff 6a25  .C.DO..@...C!.j%
000000f0: 8320 25d1 baf5 23d9 e5dd ef97 4894 e821  . %...#.....H..!
00000100: 6b68 f748 ce43 9f79 d2cb 1e72 2d7e f04d  kh.H.C.y...r-~.M
00000110: 2862 a931 1c8e bc36 9282 982c 1c4e 8327  (b.1...6...,.N.'
00000120: ad2a 468b 6cc3 1148 d143 4dc6 5749 c273  .*F.l..H.CM.WI.s
00000130: 66dc f46f e7af f9c4 edb7 d63d b0af b5ff  f..o.......=....
00000140: 3801 7ea1 f791 edb1 05bd 3beb ff20 fc86  8.~.......;.. ..
00000150: 6156 bee6 e645 3296 2055 17e0 a32c 8d5b  aV...E2. U...,.[
00000160: e12d c0b8 b1ff 5212 251f b5bf d120 b3c0  .-....R.%.... ..
00000170: 9d83 bfeb 654e d5a3 fcc3 6854 d058 6375  ....eN....hT.Xcu
00000180: 104b 05bf cc38 0aa3 fbc7 f81b 2192 1853  .K...8......!..S
00000190: 6a45 849a 8f0a 30a7 62b7 dcaa 8c30 63a9  jE....0.b....0c.
000001a0: 9cde 8969 048b 3a68 9598 f8b7 a12d 05b6  ...i..:h.....-..
000001b0: e141 d28e c830 b195 4e4e 0920 d793 372c  .A...0..NN. ..7,
000001c0: 2766 21ed 960d 3380 fc2a dfc8 90c3 3a14  'f!...3..*....:.
000001d0: 4045 a3f1 a434 dca2 5bbf eb34 2e46 2921  @E...4..[..4.F)!
000001e0: 8b57 f631 df95 1f47 fd57 d422 bb0b 59f4  .W.1...G.W."..Y.
000001f0: 9de9 8dda 7e7d eb58 8226 22ea d9c6 90ec  ....~}.X.&".....
00000200: 8e3b 5a40 30c9 88c2 40d3 94a0 7ca5 975e  .;Z@0...@...|..^
00000210: 3571 8785 0e5b b69c ebd1 d7fb 682a 3122  5q...[......h*1"
00000220: 55d7 37fe 412e e71f 66d3 998e c6f1 b781  U.7.A...f.......
00000230: 3760 f46e 0dad b817 b0fa 146c 66ee 878b  7`.n.......lf...
00000240: ceba 9ab3 7bc8 0c9a 3011 9e9c da89 15a7  ....{...0.......
00000250: aa5b 910a 0be3 239b ebd7 9c98 b10c c7af  .[....#.........
00000260: 4623 7e95 b886 c3f8 a63e f9ae 6ee7 05cc  F#~......>..n...
00000270: 9fb8 4865 f83e 49e8 af0b 9c2a 530e 44a9  ..He.>I....*S.D.
00000280: 07e5 c412 5b13 b19c be79 1e24 24c6 9833  ....[....y.$$..3
00000290: 6f72 50e1 624c cb05 4965 d9ea 4b06 059a  orP.bL..Ie..K...
000002a0: c1da 323a dc63 ff76 0243 894c 69ed 1ace  ..2:.c.v.C.Li...
000002b0: eddf 8b52 48f3 5a76 bdd6 4946 0f89 1e8c  ...RH.Zv..IF....
000002c0: d271 100d 4dc1 9cf3 2639 ce66 cde3 7d1e  .q..M...&9.f..}.
000002d0: 4111 6b84 47bb e33d e8bd 060c 154a fdbf  A.k.G..=.....J..
000002e0: a57f 05e4 fd85 0579 9683 2f4f dea8 3774  .......y../O..7t
000002f0: 6714 4583 295c 539b d3fe 3184 dc0b ba3f  g.E.)\S...1....?
00000300: d164 a2f6 5b17 62e2 954b a47e 9abf cfb8  .d..[.b..K.~....
00000310: 2662 9839 7e7b ca0b f9e8 dc8f cfbb 2103  &b.9~{........!.
00000320: 7739 7b0e 4b19 f4d4 c4dd 6eaf 51a8 c713  w9{.K.....n.Q...
00000330: ca0a 632b d296 cfd7 2024 a982 2452 0312  ..c+.... $..$R..
00000340: 6932 d175 d4bc 2b8d c57b 27fd 068f 739a  i2.u..+..{'...s.
00000350: 95bb ab73 7d0a d17b 1132 5cef b843 8326  ...s}..{.2\..C.&
00000360: ce47 8c9a 9967 5538 2b58 5c81 cf19 0d70  .G...gU8+X\....p
00000370: 4323 6e1c 5eb7 d933 7a51 3dc6 6a4c 8763  C#n.^..3zQ=.jL.c
00000380: 9e58 8139 0999 24ac 5129 abab cb53 4892  .X.9..$.Q)...SH.
00000390: 39f9 bc8f 63f8 9e21 dd05 e5a5 a0c8 d7e2  9...c..!........
000003a0: 2c15 c494 3d65 3d1a 44d5 655c 640f 8cbc  ,...=e=.D.e\d...
000003b0: b6ba 6609 1fcb 42cc bbaf 1145 70a1 9f04  ..f...B....Ep...
000003c0: 465a f486 2a2c 4179 052b 4792 e05a 8d7b  FZ..*,Ay.+G..Z.{
000003d0: 8534 57cc 33fe 6450 cd37 a912 1948 23fd  .4W.3.dP.7...H#.
000003e0: 1b31 1639 c7a1 2212 2c7a 9e0e e260 7a6c  .1.9..".,z...`zl
000003f0: 77a0 a311 5702 d370 b427 8f4c 6c44 0341  w...W..p.'.LlD.A
00000400: b2bf 5bf4 6f92 ea83 a9de c1d0 fde3 31e7  ..[.o.........1.
00000410: 97dd fcfa 7173 db94 c535 f596 64cb 21bd  ....qs...5..d.!.
00000420: 1aa9 837b a008 24e2 e927 28b7 8615 c7f8  ...{..$..'(.....
00000430: 7605 8ad5 bbc0 f761 4036 b496 5f7f f56e  v......a@6.._..n
00000440: d885 8e71 1abb 1027 d9e2 dc8a 023e a6c2  ...q...'.....>..
00000450: 85f0 30ab 4fb7 0a3d 7ba5 17f9 38c0 6c8d  ..0.O..={...8.l.
00000460: 7b3f 007f b3e5 f17a 71d6 3aae 5112 6f40  {?.....zq.:.Q.o@
00000470: f090 3d0c 1d24 752c e7d0 59b1 5a08 b68c  ..=..$u,..Y.Z...
00000480: 5f66 de57 094c cd9a f4e0 5b5d a41b 8761  _f.W.L....[]...a
00000490: a4a2 2a57 507b 3ef2 7c22 61f4 7e57 0868  ..*WP{>.|"a.~W.h
000004a0: 1e00 b558 1037 76ce fa4d 71bd 9fb0 9840  ...X.7v..Mq....@
000004b0: 705a c6ae 6cde 3f73 df6c dcf9 5f51 71a8  pZ..l.?s.l.._Qq.
000004c0: 7033 ef69 32fd 8810 edd0 c88d 9fe9 e2dd  p3.i2...........
000004d0: 6221 944e 6d50 a103 a534 e06c b8ab 03a6  b!.NmP...4.l....
000004e0: 39eb e2a9 ad6b df62 ce62 d2d7 8b5b a4e5  9....k.b.b...[..
000004f0: daab e4ab cd24 f01f 98e6 d301 1660 783d  .....$.......`x=
00000500: adcf abc5 7a92 727a e3cf f931 529e 0663  ....z.rz...1R..c
00000510: 6440 ef7f 2789 12a9 6cf0 4ad9 3779 f72e  d@..'...l.J.7y..
00000520: 036b e380 7f90 5acf 6baf e3a6 e626 95bf  .k....Z.k....&..
00000530: 21f4 85e2 4066 56fc bbea 7e04 8701 6d38  !...@fV...~...m8
00000540: 6a18 c2a9 024c 9cd3 b03f 5644 a0d3 0267  j....L...?VD...g
00000550: f330 8745 d4c2 a6d5 926b 398e 0166 996c  .0.E.....k9..f.l
00000560: 384f 8e19 7d4c b8f6 fcbd fbda a011 222e  8O..}L........".
00000570: 9642 322c b1a9 36b0 5e03 31b0 c08f 42b0  .B2,..6.^.1...B.
00000580: 92f6 f673 fe57 9e47 78f5 6d9e b3fe d5ae  ...s.W.Gx.m.....
00000590: 753e 089b c9b7 815b 3339 bc9d c39a 39c1  u>.....[39....9.
000005a0: 19a3 facd 0c83 9216 a77a b180 5930 f945  .........z..Y0.E
000005b0: 15d1 2933 264a 0ea0 e8b3 c748 c1f9 33c7  ..)3&J.....H..3.
000005c0: 305e 7627 8b07 827d 3729 40b8 8cb7 7f62  0^v'...}7)@....b
000005d0: a13e 36d2 695e 734e ed17 8871 1ec9 fa01  .>6.i^sN...q....
000005e0: e488 bee0 ba06 a3ef 0fc8 decc 42db 0676  ............B..v
000005f0: f035 9dbf ce6c f693 f669 03a3 922e 1f28  .5...l...i.....(
00000600: b798 121f e09e 896f 871f da93 2679 c2f8  .......o....&y..
00000610: 9c7e bc1c 8629 3340 9c2d ff33 22d2 e1b8  .~...)3@.-.3"...
00000620: 9f78 edb4 9344 f6eb b558 c42b 32b8 9815  .x...D...X.+2...
00000630: a363 47f6 1558 ec14 682e af34 60b4 2042  .cG..X..h..4`. B
00000640: f14e fb0c c57b 083b 67b4 33f3 cb5f 3d5b  .N...{.;g.3.._=[
00000650: a239 a9c8 89ae c5c8 5587 110e 8454 41ed  .9......U....TA.
00000660: 7452 1d74 0c1c f703 fcce 91ce 63f2 e7c0  tR.t........c...
00000670: 1a97 4fc0 a916 6f14 64f5 725e 0141 c9ee  ..O...o.d.r^.A..
00000680: 1a3f 76ac af3c 8344 550f a249 1bd0 d1bc  .?v..<.DU..I....
00000690: 39a7 fb5a 07ee f7c4 d435 979f dc77 ec6d  9..Z.....5...w.m
000006a0: 2553 78d0 f767 d7c7 7559 183a af36 be89  %Sx..g..uY.:.6..
000006b0: 2641 e732 65a8 ec8e 7912 ecb8 ef47 217d  &A.2e...y....G!}
000006c0: 2aa2 ceb8 42d0 691b a64e abaf 7f23 1859  *...B.i..N...#.Y
000006d0: 6890 667b 0dea ca28 8c96 a68b f0a5 23a2  h.f{...(......#.
000006e0: 514d 7839 9b94 e8ce d502 bda8 6d55 a7a5  QMx9........mU..
000006f0: 2bff 02fc 18ee b08d 8692 1135 b375 1c68  +..........5.u.h
00000700: f2c9 71a1 247d 2b07 4531 a5c7 a95f 0593  ..q.$}+.E1..._..
00000710: 6a1b 0a87 7b84 ffee db52 d17e c164 35a2  j...{....R.~.d5.
00000720: ee41 8e8e eb23 7294 9325 60cb 5d1f 7935  .A...#r..%`.].y5
00000730: 9f89 74df 715a 33fc 3a4c f720 02ce 7ad6  ..t.qZ3.:L. ..z.
00000740: 9b74 1283 5876 5970 12c1 7c28 2ed6 07b0  .t..XvYp..|(....
00000750: f647 bee3 84a5 dc2e 585d 8479 0de8 8338  .G......X].y...8
00000760: 9409 21ad 0ec5 dde1 5e1b 8ec6 ad03 9a7d  ..!.....^......}
00000770: d9bf 9b27 a904 8c95 cd48 a7b1 ab72 2b1b  ...'.....H...r+.
00000780: 1da0 0000 0300 0003 037b                 .........{

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=15872
pts_time=1.291667
dts=13824
dts_time=1.125000
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=3091
pos=25966
flags=__
data=
00000000: 0000 0c0f 419a 5d34 a43e 0268 e115 ff00  ....A.]4.>.h....
00000010: 0003 0000 0300 1ca8 7a15 77ff 76f3 6b12  ........z.w.v.k.
00000020: 3903 7306 3007 0528 70fb fda3 2c97 70f9  9.s.0..(p...,.p.
00000030: f62d 6734 c85f 88e3 5913 4d5d b346 69e7  .-g4._..Y.M].Fi.
00000040: 6979 390b 6d98 ee7b 3f41 35ef 215a 5417  iy9.m..{?A5.!ZT.
00000050: f5e9 e542 255a d858 7103 f92f dd4a 2a8b  ...B%Z.Xq../.J*.
00000060: c89e 2a64 6a88 7ee3 0426 4000 1dbb 54cd  ..*dj.~..&@...T.
00000070: 3904 1e9d b7de 869d 6deb 0a7d 0050 3a69  9.......m..}.P:i
00000080: b3aa 7a0a 4c99 69ee 1305 ac7a 9400 b351  ..z.L.i....z...Q
00000090: c7c0 be5b ec27 65c7 7256 d4e1 bb13 abe6  ...[.'e.rV......
000000a0: f708 90b5 5f34 d25a 8ace 345c 6e46 018f  ...._4.Z..4\nF..
000000b0: 1051 7a03 57d8 8647 adc9 7bb7 c0a2 6027  .Qz.W..G..{...`'
000000c0: e405 ec0b 4f39 aad4 998b 2744 dfa1 ecc1  ....O9....'D....
000000d0: 8b5f 095c e163 e0b2 7326 093b 56cd 7144  ._.\.c..s&.;V.qD
000000e0: 5d64 9af8 a9c4 ffc3 2a60 cf4b 0e97 5d6f  ]d......*`.K..]o
000000f0: 63be 6bf7 f125 4fb3 d78d 99ba 5000 4b45  c.k..%O.....P.KE
00000100: 33f2 f0f2 823e e7b1 143a a145 eb19 a894  3....>...:.E....
00000110: a96d dbf4 a0fb dca4 095c 70d1 ac83 b92b  .m.......\p....+
00000120: d406 8b86 b8bb fe7e 47bc 68d1 f740 fc65  .......~G.h..@.e
00000130: 7128 c35e ca72 0221 2471 4087 76d1 cf30  q(.^.r.!$q@.v..0
00000140: 1314 e8a3 148e 8e80 8bac 933d 889e dc8a  ...........=....
00000150: 637f 2d4c 4688 c68f 6f38 c682 b2dd 77db  c.-LF...o8....w.
00000160: f72f e82d c17a b976 dedb 811b 76b2 cc4f  ./.-.z.v....v..O
00000170: 91dd 633c f8c5 8d80 ba10 02a3 b616 0c99  ..c<............
00000180: 617e 0aa3 7374 071b f1b4 0859 636f 5cdf  a~..st.....Yco\.
00000190: 5657 501c 55d5 099c eba7 fdd5 00d8 f573  VWP.U..........s
000001a0: a001 a826 eca0 9a40 4f22 4d96 462b 00ec  ...&...@O"M.F+..
000001b0: c8e1 0cb9 a4a1 1a8d 88f2 54ac 7f00 c7d8  ..........T.....
000001c0: 2ad8 f3c5 5bee acc4 daf3 b6ad 0795 161f  *...[...........
000001d0: 35a2 6149 d89c b11c 6a53 b91a 011d 115e  5.aI....jS.....^
000001e0: 98ec 44d1 2026 efdf 31c4 c3a7 45ba b075  ..D. &..1...E..u
000001f0: 196b 9c96 a2fa 63b3 245d 21bd aa8c 9cf7  .k....c.$]!.....
00000200: 8365 0ed7 af22 c45e 80a9 a98e 7ad3 43ee  .e...".^....z.C.
00000210: cba9 45a6 520f cad7 ef72 031d bb47 78cc  ..E.R....r...Gx.
00000220: 4d82 b924 3bbb de9a 4ba3 83ad 4aab 35d8  M..$;...K...J.5.
00000230: daee 6f9d 1cb8 2fb5 0315 4571 4cbf 222d  ..o.../...EqL."-
00000240: 8ed7 f653 2423 4df0 337b 50d8 4258 8f64  ...S$#M.3{P.BX.d
00000250: 17c4 5c98 9d13 31ce 435c bd43 2915 5ad9  ..\...1.C\.C).Z.
00000260: c84b 1f04 cd10 e8d6 95ce a731 ebe6 a970  .K.........1...p
00000270: 6312 f1f7 10a1 f6b7 bd07 d69b f31f 27d8  c.............'.
00000280: 67a4 6b9d 0ff3 17e9 0aba 4ac8 fa88 f107  g.k.......J.....
00000290: 1320 ac2a 08a6 a382 938d b79d f77c 8ae6  . .*.........|..
000002a0: 30f7 0beb 47b7 1cb7 6e2e 6c37 f0e0 3919  0...G...n.l7..9.
000002b0: 12a2 1e3f b1c7 6079 0901 a9b5 e7c9 5cba  ...?..`y......\.
000002c0: 554a 15bc 4b7d f4da e7c5 496f 6a7f 7c6d  UJ..K}....Ioj.|m
000002d0: a981 3a9b 3b39 ea97 8083 cf64 e27e 5228  ..:.;9.....d.~R(
000002e0: 0b91 268b bcc8 b5fb 75c0 6d8b 3c8a a388  ..&.....u.m.<...
000002f0: 1614 da41 a2d1 2a19 6836 b9a1 5ed5 814c  ...A..*.h6..^..L
00000300: 1c8a 0978 4caa 2c12 d10f 587f b4b9 5308  ...xL.,...X...S.
00000310: 6867 044f 170b 6699 ecda e87a 40fd 948b  hg.O..f....z@...
00000320: b834 6a67 1b80 c64c 8dee 9a9c 8e63 fee6  .4jg...L.....c..
00000330: abe4 34ee 5e39 3606 0623 958a 14ab 41d1  ..4.^96..#....A.
00000340: 61f4 5382 ff83 5d94 c5ac a674 1698 2b5d  a.S...]....t..+]
00000350: c113 a9ae b823 6e38 015e 25c0 7d16 9edc  .....#n8.^%.}...
00000360: ab11 c6b6 ebad aa05 8c38 3e44 ed5f 6423  .........8>D._d#
00000370: 10cc 74e9 ea42 f64d f85a 531c f16a b30e  ..t..B.M.ZS..j..
00000380: 1181 ba4b ef1c 848f e4ac b3d1 0e0f 5a50  ...K..........ZP
00000390: 5805 4a1f 933f 0fec ea96 0c8d 9317 9f0b  X.J..?..........
000003a0: e0af 39ed 9c90 47b8 ff64 4ea4 6921 fef9  ..9...G..dN.i!..
000003b0: 891c 42d2 5a78 7078 c49c 5542 527d e223  ..B.Zxpx..UBR}.#
000003c0: e791 be05 6c3c 94d3 f816 ca88 5e7a edef  ....l<......^z..
000003d0: d02d 3be1 4d9c f68c 2de9 bdf8 8856 878c  .-;.M...-....V..
000003e0: 482c d1e3 8173 90dd 9f77 46be 7bdc 5266  H,...s...wF.{.Rf
000003f0: 3201 4b0a f9a3 fb36 99b6 0fdf a8ef 8f24  2.K....6.......$
00000400: dd3c afb7 885b 89a8 4a48 e098 5a59 85f2  .<...[..JH..ZY..
00000410: fd03 22d1 1804 0e2d 166c a726 844f 4538  .."....-.l.&.OE8
00000420: 0426 4d1a b85c 38a4 adc2 df91 832d b916  .&M..\8......-..
00000430: 4bb4 6ba6 5753 19a3 fbd5 9620 b8cf 767d  K.k.WS..... ..v}
00000440: 66f8 acf7 14cd 4396 1ec6 1ad6 aae6 859a  f.....C.........
00000450: 35bb 5bd8 ab55 f7e2 441e 6d86 c41d 1882  5.[..U..D.m.....
00000460: 2923 d387 6618 b460 42eb 2583 3cca 7562  )#..f..`B.%.<.ub
00000470: 6eaa 72c0 49b1 23e2 a74d 84e3 e7fb 2395  n.r.I.#..M....#.
00000480: 1be4 f8cc 62fc f244 8608 d117 ad26 d138  ....b..D.....&.8
00000490: 47f9 9215 c51a 7764 082b f528 37cd 236e  G.....wd.+.(7.#n
000004a0: 34e3 895e 57cc 435c a57b 48e5 7184 d105  4..^W.C\.{H.q...
000004b0: 837f f1ed 947a 13da 93db 91ac 8714 30cc  .....z........0.
000004c0: 4a08 1c3f 5552 40a7 6762 1c17 9662 f2ed  J..?UR@.gb...b..
000004d0: 0c26 9c9a ed5c 50f4 c140 2466 6743 13ac  .&...\P..@$fgC..
000004e0: 843a 88e3 ca88 54a6 96a6 7445 23c2 4713  .:....T...tE#.G.
000004f0: c72e ffee 2f5f 83a3 3821 a459 2917 f487  ..../_..8!.Y)...
00000500: 5129 6ce6 2e8e 2c42 4a05 df0e 1134 c364  Q)l...,BJ....4.d
00000510: 2919 9e3d 54cc edcb bc73 ea5f 6c6b 7a71  )..=T....s._lkzq
00000520: bb78 cb4f ad4c 53c3 a347 3e1d ab50 9ec0  .x.O.LS..G>..P..
00000530: 1464 b53d 4eb0 6a21 7adf 6776 ae41 e0f6  .d.=N.j!z.gv.A..
00000540: 7a29 a910 553c abfa 5e3d 5478 ccc9 50aa  z)..U<..^=Tx..P.
00000550: ea65 5b42 7a1f 7ccf 449d 37b9 e564 6ee0  .e[Bz.|.D.7..dn.
00000560: 9484 821d 6008 f68c 2d15 eb6f 4220 32cd  ....`...-..oB 2.
00000570: 860d 2bae 409f 18a5 56b7 052c e59d 7ecd  ..+.@...V..,..~.
00000580: e9cf dde8 0385 bb60 3289 f059 dd36 9aff  .......`2..Y.6..
00000590: f0e8 3248 033b e8ce 8b4d fcb5 58cf 6c0c  ..2H.;...M..X.l.
000005a0: 24f2 737b e6f3 00a4 3ffa db6b 080f 6207  $.s{....?..k..b.
000005b0: b38f 2f13 e93f 16d9 3a0b 2ad1 12c0 52ab  ../..?..:.*...R.
000005c0: a73b b40d 71d3 612e 78ba 26bf b6fd f64b  .;..q.a.x.&....K
000005d0: 2d17 57fe b371 f005 1aad 3245 3d82 b3df  -.W..q....2E=...
000005e0: c011 00e6 ec94 31fd 8598 6b01 5856 f0f6  ......1...k.XV..
000005f0: c4c8 58a2 cc30 cb2e aa81 2f32 d2bc 67ec  ..X..0..../2..g.
00000600: 1f85 ccad 7d02 d7dc 6bf0 8bb3 4362 8c3e  ....}...k...Cb.>
00000610: efc7 2780 97be dc0a 21fa 7d9d f15e 8f65  ..'.....!.}..^.e
00000620: 2fd1 6324 acea d90d 6e20 5ac5 d76c 12cd  /.c$....n Z..l..
00000630: 56aa 054b faba 778f e8bc af42 7ba4 4690  V..K..w....B{.F.
00000640: d3be ff8e 38ad 72be 269d 8bd2 7ee2 205c  ....8.r.&...~. \
00000650: dd94 7184 5f58 80a1 5b5b a2f4 0088 b180  ..q._X..[[......
00000660: bd2f 7f05 9599 0245 9d83 8eda 6fdb 830c  ./.....E....o...
00000670: dc6d ab43 64e8 dfef b3b1 5fb4 f834 93f7  .m.Cd....._..4..
00000680: 6574 a940 13ee fd5f 1b84 60c2 166f 6a77  et.@..._..`..ojw
00000690: 109b 6470 8d3e 9a65 7641 e14a 5d34 aa39  ..dp.>.evA.J]4.9
000006a0: 8293 081a 93f8 a461 c41b 7fed b2e5 2d30  .......a......-0
000006b0: bcb3 a967 78ca e556 6282 9962 5960 f879  ...gx..Vb..bY`.y
000006c0: 5568 c0e5 2be2 449e 1910 0131 c416 6445  Uh..+.D....1..dE
000006d0: 7e15 2b26 8a1d 6e24 a927 b10a 17b9 0d81  ~.+&..n$.'......
000006e0: 1fc8 4023 70ea a582 6ab7 b836 ee61 823b  ..@#p...j..6.a.;
000006f0: 7621 b3ae c79e db87 9dfa 7ca6 904f 9ab6  v!........|..O..
00000700: 4856 3b3a 3153 9550 c818 cd7e 9a7b 3172  HV;:1S.P...~.{1r
00000710: c47e 374c b77f cdd6 7c78 5112 6de0 48f3  .~7L....|xQ.m.H.
00000720: 0076 b540 e77c fd04 1582 4d74 dd53 5db1  .v.@.|....Mt.S].
00000730: 874c 609a b561 a193 8be7 ca81 4fd5 41a7  .L`..a......O.A.
00000740: 2c9a be1e a679 0e56 508e f6f8 f05c 5d6b  ,....y.VP....\]k
00000750: efb1 c333 a366 eb05 47cc a4ab 46ed b102  ...3.f..G...F...
00000760: 8fa9 7647 d988 7b54 a3c8 199b b9de 7d26  ..vG..{T......}&
00000770: dbcd 7a97 0a25 e762 bcac 2270 127f 0cdd  ..z..%.b.."p....
00000780: 5a0d 1558 e25b 456a d530 d9c3 4d77 11fa  Z..X.[Ej.0..Mw..
00000790: 5047 9c45 f1b0 2f9a ac7f c7c7 2ce4 3d54  PG.E../.....,.=T
000007a0: 166b 9191 caae 04cc bfec 2b27 12c8 6e00  .k........+'..n.
000007b0: 7c5c 7fcd 83a1 3cd9 7eac c4e2 c5f1 6744  |\....<.~.....gD
000007c0: fb2e e7ac 84bf 1646 8329 7dbc af56 17ee  .......F.)}..V..
000007d0: a6c3 a495 cb57 8a3a 921f 6913 4b9b 733e  .....W.:..i.K.s>
000007e0: c55d bbba 93f9 9e04 c175 bf5b 1778 c254  .].......u.[.x.T
000007f0: 35dd df3f b5b1 1239 5a35 73d9 293d 244d  5..?...9Z5s.)=$M
00000800: 24e3 2cf3 f203 6707 2c6a bb00 9c8d a584  $.,...g.,j......
00000810: 7747 0797 b20d 1413 0b94 eedb 4e8a 5e0c  wG..........N.^.
00000820: 1918 d75d ead0 5883 0d39 9a16 c651 1167  ...]..X..9...Q.g
00000830: c388 0bfb ef27 f4ce 6c8d edcd a345 66fa  .....'..l....Ef.
00000840: c1ca 65ed 4028 d35a 5530 68b6 cd1b 96ba  ..e.@(.ZU0h.....
00000850: 78d9 0938 45c9 bced 4f44 a437 b662 8b48  x..8E...OD.7.b.H
00000860: 73dd 244f 2b3f 94a1 4b1e 2af0 2a12 c059  s.$O+?..K.*.*..Y
00000870: c013 71e7 4877 57ae 0771 1b06 ff52 d3b1  ..q.HwW..q...R..
00000880: 9a7a f39d b5d8 703b eed4 a762 045c c87b  .z....p;...b.\.{
00000890: d307 ee3d d861 8e12 99da a098 a4ac 33f6  ...=.a........3.
000008a0: aab6 c1d0 40f4 ce69 7875 a4f5 a69a 4a3a  ....@..ixu....J:
000008b0: fb7b 72ed 1b17 973f 770e 2003 bcdf 2abd  .{r....?w. ...*.
000008c0: 1bd6 3ebe 2763 419c 4e36 c5e7 cf5f bae1  ..>.'cA.N6..._..
000008d0: a880 5797 c26b 4133 dddd 7561 713e 8921  ..W..kA3..uaq>.!
000008e0: 007a 0a93 aa56 aec0 cbee faf3 2a98 bfca  .z...V......*...
000008f0: 945d 510b 0106 c4ed a26b 3c2e 7c7f 7a88  .]Q......k<.|.z.
00000900: a910 35b8 603f 7999 768b f2c3 5682 a1d0  ..5.`?y.v...V...
00000910: 6cb0 68c4 e580 d2c4 6040 3603 142c 1d0b  l.h.....`@6..,..
00000920: f13c b537 d935 4398 25e7 54b3 ca17 f2dd  .<.7.5C.%.T.....
00000930: ce94 0fd6 72ca 5820 6532 6f16 fe2d eaa8  ....r.X e2o..-..
00000940: 15d4 eb42 5516 b3d8 de68 4608 2cef fca8  ...BU....hF.,...
00000950: a192 21bb 9f6e 6f17 e854 7fc7 f41c 03bf  ..!..no..T......
00000960: 3364 f04d 11dc 1e20 246f d098 081e 9f6a  3d.M... $o.....j
00000970: fd2c 7658 d458 7837 416c 0c74 ac8a 98c1  .,vX.Xx7Al.t....
00000980: 1b62 1103 793f 5331 7be9 7dbd 7429 2125  .b..y?S1{.}.t)!%
00000990: 8cd5 6065 cfa5 508d 0949 5ae4 2faa 6712  ..`e..P..IZ./.g.
000009a0: 9ebf 7c31 cc08 3ffa 5905 f309 7325 ce86  ..|1..?.Y...s%..
000009b0: 3c6d 0ffa 6b76 9294 bc6e aaf9 6611 4da7  <m..kv...n..f.M.
000009c0: f350 ccd2 d9dc b3f9 d32b 0a2e f09c 47ae  .P.......+....G.
000009d0: 0942 cb4f 3544 f032 195b cab9 59e2 c2ae  .B.O5D.2.[..Y...
000009e0: 8a85 3bb0 1ba3 2cdc 1223 cf9a 533c e76d  ..;...,..#..S<.m
000009f0: b9f7 2aa2 a282 2eea 6bb9 3686 573b e61b  ..*.....k.6.W;..
00000a00: a01c 9064 84b9 fcd6 ad42 dc54 228b 250e  ...d.....B.T".%.
00000a10: 4522 742f 02e6 d99b b369 bb27 d24f 4015  E"t/.....i.'.O@.
00000a20: c042 7a71 fa4f 23a0 1f02 ef83 a9a1 58f2  .Bzq.O#.......X.
00000a30: 625f 6097 523d 381d 750e 3fae 6845 538b  b_`.R=8.u.?.hES.
00000a40: 838b 754c 3ef2 dd6b 2d56 3efb 4274 28f4  ..uL>..k-V>.Bt(.
00000a50: 0ed2 eb8d 528d 6784 7a67 19ec b332 0e3b  ....R.g.zg...2.;
00000a60: 0824 127f 50f9 f160 f8e4 f3ae 85f2 01de  .$..P..`........
00000a70: 45af 5699 b2ad cf40 4359 cda2 c5a0 483e  E.V....@CY....H>
00000a80: 1a02 f365 a3dd 4327 752e 83de 192c 0a93  ...e..C'u....,..
00000a90: a049 d0d3 0c46 dd58 a9e6 054a 8084 4c31  .I...F.X...J..L1
00000aa0: 89ea 9850 26bf bfeb 1d80 9212 32da 32b3  ...P&.......2.2.
00000ab0: c271 9dfd 217c 77c1 f87c 0ba4 110e 243a  .q..!|w..|....$:
00000ac0: fca9 6543 fae1 661a 4028 88e1 28d7 5604  ..eC..f.@(..(.V.
00000ad0: 1f73 7bde a66a d7ea af99 c436 0f62 9ec9  .s{..j.....6.b..
00000ae0: 8bb6 6966 afee 7655 d4c4 e5a3 30e3 ebde  ..if..vU....0...
00000af0: e450 c733 5500 cd9d d078 31bc db68 1ff9  .P.3U....x1..h..
00000b00: da27 ac6d 763c 8288 283d 0728 16dc 71f4  .'.mv<..(=.(..q.
00000b10: f041 2408 4f6e 39c6 6053 0c19 b8ba fd10  .A$.On9.`S......
00000b20: d0e6 4e1f 308a 964c 5ed3 3712 6937 8cff  ..N.0..L^.7.i7..
00000b30: 8777 1f0b 3868 2601 9a5a 4cea 93c2 c3fa  .w..8h&..ZL.....
00000b40: 82e3 4ef0 1b0e 603d 5296 5688 072a 601e  ..N...`=R.V..*`.
00000b50: 4bbc 7e79 66d1 f52e 98f5 f849 7c45 c9f4  K.~yf......I|E..
00000b60: 2f34 5a5b a689 c00c 8b88 63a1 5b7f 0145  /4Z[......c.[..E
00000b70: 6c6c 665c c935 3e66 f32e cdca e55c 4648  llf\.5>f.....\FH
00000b80: d065 af54 040b 082d 4412 d07f 6761 9361  .e.T...-D...ga.a
00000b90: d31e c0bf 3dc0 7bf3 8170 db0f 29ce 14ea  ....=.{..p..)...
00000ba0: b63c 3514 08b8 89e5 dae2 7ab5 0833 1896  .<5.......z..3..
00000bb0: 53ae 646b 418e 4592 79ed 86c5 49d1 9e1d  S.dkA.E.y...I...
00000bc0: 2a90 e183 498b fa88 94c6 9e3c 8215 9fa6  *...I......<....
00000bd0: 08bf 15c0 5ad3 ab47 ebd8 b8d8 b37e e04e  ....Z..G.....~.N
00000be0: 2137 2266 ff11 e02b 87ad 410e bcbc 5bef  !7"f...+..A...[.
00000bf0: 6caa 1171 cbb3 1261 2178 2002 112c 4084  l..q...a!x ..,@.
00000c00: 318c 394a 6c8c b4f0 e0e5 1219 39ce fa3e  1.9Jl.......9..>
00000c10: c241 e0                                  .A.

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=14848
pts_time=1.208333
dts=14336
dts_time=1.166667
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=2962
pos=29057
flags=__
data=
00000000: 0000 0b8e 419e 7b45 112c 5700 0003 0000  ....A.{E.,W.....
00000010: 07d0 8f44 a32b e8a6 63a2 de50 2d1e cbf6  ...D.+..c..P-...
00000020: aa5e 1d0f 5b45 82bb 4993 733d fdb7 8fc5  .^..[E..I.s=....
00000030: ee3b e549 a4fd 8c1d 3f0b 6674 b0df 6182  .;.I....?.ft..a.
00000040: 5a4a 98cd 22dc ccb2 ea4c e7fd 931a abed  ZJ.."....L......
00000050: e916 fd90 d200 e0ac 02ec b598 eb68 3769  .............h7i
00000060: 532b 9699 2419 e4fe c121 cdcb a126 36b4  S+..$....!...&6.
00000070: 5077 7865 9848 0b6f a0b7 f105 bb79 e29b  Pwxe.H.o.....y..
00000080: 1b57 03f7 d494 a451 7eea 3cc4 e0f9 1525  .W.....Q~.<....%
00000090: b4b4 6159 63ce fdea f7f9 2479 a0db 84b4  ..aYc.....$y....
000000a0: 9ec9 759f c841 0816 b836 d6cb 15ee 31c4  ..u..A...6....1.
000000b0: 228c 3dda 9264 b15b 6eea 6b7d 4034 e686  ".=..d.[n.k}@4..
000000c0: 15bd 2050 def8 c94a b6f6 3886 389a 4e48  .. P...J..8.8.NH
000000d0: 8c46 dcd1 e71b 9f3e 6919 8858 8948 a826  .F.....>i..X.H.&
000000e0: b928 7105 c5b7 a130 8763 c37f 178a b827  .(q....0.c.....'
000000f0: c698 ee67 685f 317f b923 a713 57a0 5497  ...gh_1..#..W.T.
00000100: 3cd5 e6f8 2494 6368 cb80 3eff 8914 fe7c  <...$.ch..>....|
00000110: e5e7 df71 4237 a19d 171d cc29 75ae 29e2  ...qB7.....)u.).
00000120: f26f 350f 97aa eaeb 8df3 d771 1079 43ac  .o5........q.yC.
00000130: 7e96 29b0 7135 ee3a 5bf4 364b b3cb 6a6a  ~.).q5.:[.6K..jj
00000140: ec05 683b 3df2 3059 1953 f5e8 2f77 aabf  ..h;=.0Y.S../w..
00000150: ebea 4beb 4d8f ed73 1737 5c09 80ed 8640  ..K.M..s.7\....@
00000160: a586 f01c 887b 86f0 7b5f ff59 382f 4a83  .....{..{_.Y8/J.
00000170: 8fca 5dc4 6f0c 2c64 e6af 79a9 2333 1354  ..].o.,d..y.#3.T
00000180: 0569 4c53 e477 be58 f253 656a 0658 0f3a  .iLS.w.X.Sej.X.:
00000190: 40ee 448d f33f 4d17 e411 e2c8 1c67 2ce7  @.D..?M......g,.
000001a0: 56b6 e14f b1a2 ee5c b91b bbee c2d9 84d5  V..O...\........
000001b0: 4c83 760b c125 6885 c169 1e5b dddd e046  L.v..%h..i.[...F
000001c0: 188a 6ea8 10ed 7737 4eca ed41 dfe7 1117  ..n...w7N..A....
000001d0: 3b4c 4b9a b3dd 10af d063 1266 88a0 49c7  ;LK......c.f..I.
000001e0: 552b 376b 72a5 4da3 c5eb 38f5 9d71 e26c  U+7kr.M...8..q.l
000001f0: 46af daba 6f59 eb4f 8b00 24d3 8d94 a507  F...oY.O..$.....
00000200: 79c2 68a2 0e92 ce81 e6b6 462c 0aa3 d250  y.h.......F,...P
00000210: c0b7 d4fa e9ef 8531 181a ecf7 95b6 54b1  .......1......T.
00000220: b5ba 4d05 346d 3587 9ba5 044d 662b fe20  ..M.4m5....Mf+.
00000230: 4629 4583 1064 4517 7d37 2f0e 1c2d acf6  F)E..dE.}7/..-..
00000240: bbcb 4421 6fe7 e792 0ab3 67fb 4cf8 3746  ..D!o.....g.L.7F
00000250: f4d8 bbb9 0bbe e1a1 0c78 62cd ef96 c9c6  .........xb.....
00000260: bc08 63b6 1afb f9d9 266b 7a21 5a85 0cd8  ..c.....&kz!Z...
00000270: 87b6 5e23 d4c7 fa37 b7c0 11b5 3c84 f38e  ..^#...7....<...
00000280: a51c da4a f893 d2bc 0553 fb1f 5c7e dcfb  ...J.....S..\~..
00000290: e433 2469 8207 118a 8f3d 53e2 cbc1 6c14  .3$i.....=S...l.
000002a0: c7ec 5b83 64b5 723d bb11 e36c 1f8b b10e  ..[.d.r=...l....
000002b0: 6583 68f6 b3f8 08ed e884 c8ae c6c7 64de  e.h...........d.
000002c0: c17a bdf6 2866 e9ae 1237 bf31 802e d4b6  .z..(f...7.1....
000002d0: 1534 a5c7 aacb b9de 2559 fa46 3ec1 71f6  .4......%Y.F>.q.
000002e0: 3a18 bce0 aea4 0919 6e80 53b7 7a38 94d7  :.......n.S.z8..
000002f0: 0667 76be ba42 ecca cde8 67bd c7fa 6a73  .gv..B....g...js
00000300: 0ff0 816b 33d5 99aa 4b56 315e 092b d02b  ...k3...KV1^.+.+
00000310: 50cf 0119 0776 f765 3f0d 7496 8fd7 599e  P....v.e?.t...Y.
00000320: fdff 40a2 5d58 6ec4 ca55 607c cd5b 0b47  ..@.]Xn..U`|.[.G
00000330: 07b3 4e50 1d96 bd1c debf cfe6 62e6 55fc  ..NP........b.U.
00000340: 3932 65b3 25de b418 459a 3b3a b6d9 6200  92e.%...E.;:..b.
00000350: b1f5 903a a19b 3687 b25f 2e1e 09e8 cb6d  ...:..6.._.....m
00000360: cd74 d570 17fb de93 1b54 f1d8 6efa f912  .t.p.....T..n...
00000370: 4b20 deec 9ff0 6b51 7476 c83a 6148 e3c1  K ....kQtv.:aH..
00000380: 3797 3cfe 3a3c 3ffe 8653 e048 ef94 0c71  7.<.:<?..S.H...q
00000390: 7310 cac5 8204 7368 f9bc 1022 e02d 9d32  s.....sh...".-.2
000003a0: 1a16 b83c 05ed 35c7 cd5a 92cf 28a8 3f7e  ...<..5..Z..(.?~
000003b0: a9ab ea69 0a5f 1f40 e40f abfa 7686 3391  ...i._.@....v.3.
000003c0: 7d1b c07d 3556 bef9 931c 6bf2 2fc3 cd8d  }..}5V....k./...
000003d0: 0c11 fe1b 0edb 8a4b 9f17 f84f 250d b1f9  .......K...O%...
000003e0: 9d9d 4e7b 5266 1ab1 d432 be38 0cd1 18a8  ..N{Rf...2.8....
000003f0: 0678 0c53 bc5c 3b3c d3e3 b03e 79a4 5713  .x.S.\;<...>y.W.
00000400: 6752 b9a4 9c1c 3474 a65a 3f23 cb4f a450  gR....4t.Z?#.O.P
00000410: caa4 b63d c5ce 0519 746f 7ea0 f590 dcab  ...=....to~.....
00000420: 6732 682c cf18 2ff7 5b10 753e 3f6c e312  g2h,../.[.u>?l..
00000430: d2df f199 cdc9 0c31 c71f ff6c 374d e7b8  .......1...l7M..
00000440: 3705 e59c 46f8 dd1b 6fd4 42ec 0260 1d84  7...F...o.B..`..
00000450: 4e9c efc5 620f e2ec 8cdc de5c 4b2a e515  N...b......\K*..
00000460: b07d dd87 52aa 499a cb6a 8d08 db4d 1d48  .}..R.I..j...M.H
00000470: 72d5 29a3 4d3f a3f2 a87d 2342 6c21 395e  r.).M?...}#Bl!9^
00000480: 5c82 0a7e bcf3 6ed4 87d1 3c3e 020e 6b9c  \..~..n...<>..k.
00000490: a6d6 15c1 7e21 9f7f 6b16 6f85 4341 f46f  ....~!..k.o.CA.o
000004a0: d3da ada9 45cb 6461 fc4b 3968 cf45 8809  ....E.da.K9h.E..
000004b0: 0b29 d668 2dc7 4ca8 a982 06e1 b2c3 c8ce  .).h-.L.........
000004c0: 6e3b 2503 db5a 01fe 9bb9 3d64 b315 a20d  n;%..Z....=d....
000004d0: fffc 1f75 f1f6 a0fd 797b 42d9 9522 e986  ...u....y{B.."..
000004e0: 3237 5cd7 029a 4023 907b 38cb 3cf8 55e0  27\...@#.{8.<.U.
000004f0: 0bb8 707e 6177 acd0 a5db 1f10 0acd f5e6  ..p~aw..........
00000500: a6ff 902a 80e5 30c3 96f9 0299 72ab 576a  ...*..0.....r.Wj
00000510: d411 d8cf 79c1 cf2b 43fe bade dd25 aefe  ....y..+C....%..
00000520: 61b4 64f8 e5ce 0337 b070 3c1d 3715 7384  a.d....7.p<.7.s.
00000530: a6bd 3b6d c774 705e 2b51 a7b7 e395 f851  ..;m.tp^+Q.....Q
00000540: 8a23 1654 6226 897f 20ff 4ae6 6e01 d228  .#.Tb&.. .J.n..(
00000550: 708e 4287 9b7f 0e2c 46c3 b501 5e56 7182  p.B....,F...^Vq.
00000560: e3bb f86c 4ed0 997e 88e1 2ea1 36dc 3d33  ...lN..~....6.=3
00000570: 7112 3550 955d 45b7 94e7 4a90 3abe 916e  q.5P.]E...J.:..n
00000580: cef6 bee3 64da 61e7 081e 3f0e 4310 1cc1  ....d.a...?.C...
00000590: 1cc4 53d1 6ce3 4fe1 37a2 559e 32f8 6f31  ..S.l.O.7.U.2.o1
000005a0: 3eda 2c88 faea e764 7b34 4d07 ac75 3288  >.,....d{4M..u2.
000005b0: 20da d827 1b3c 6522 ac07 d9d7 da66 be5e   ..'.<e".....f.^
000005c0: 5591 ac34 2db8 e6d9 24a9 bb7f df00 32e7  U..4-...$.....2.
000005d0: 8d7e 0e54 9e1c 0cb1 5359 3c1d 554d 9be8  .~.T....SY<.UM..
000005e0: 5b53 7192 50d0 747e f1d1 bf01 8b9f 293d  [Sq.P.t~......)=
000005f0: dc0c 3ce0 5bdf 7c98 7d83 de0a 1daf 4ce9  ..<.[.|.}.....L.
00000600: 23de 8d1a 32b3 f271 416f 5cfa 7391 3385  #...2..qAo\.s.3.
00000610: 58a2 2ba6 73dd 38c2 f37c 06e4 9620 27cb  X.+.s.8..|... '.
00000620: 94da 08ea af02 452e 2569 2d3a 05ec 8e7c  ......E.%i-:...|
00000630: ec77 9cae a98c eaaf 1dec bcd3 7374 171b  .w..........st..
00000640: 714b 361c 13e6 66e5 dc4a b823 60de a8b0  qK6...f..J.#`...
00000650: cdb8 3c96 210e 5b11 9262 dc58 4519 0910  ..<.!.[..b.XE...
00000660: 2944 e4a8 e2a7 9b75 e576 caa8 ec33 6624  )D.....u.v...3f$
00000670: b4bb 1ba7 c0c5 183b aed2 7a99 57f4 766a  .......;..z.W.vj
00000680: 6e97 65d7 f781 1510 8b8b 3df8 5d84 291a  n.e.......=.].).
00000690: a7e5 4bab 90a3 09de c02b 3353 8a20 d728  ..K......+3S. .(
000006a0: c0bf cbfb 8cc8 65b0 c223 7002 192b 50a1  ......e..#p..+P.
000006b0: 11a8 baf6 b822 7eef a4c6 e395 e093 36c5  ....."~.......6.
000006c0: 6e7d 534f 7516 dcd1 a3db b196 5aaf 5a4e  n}SOu.......Z.ZN
000006d0: 27e2 b9e9 c24f 9262 245e 3ab1 c88a 1363  '....O.b$^:....c
000006e0: 10fb 3f3d b490 30fe c7e3 03b0 4e8f 92c6  ..?=..0.....N...
000006f0: bd39 1faa ed98 741e 4335 8077 8b9e 758a  .9....t.C5.w..u.
00000700: f653 d969 a337 954b ef3e 8dd7 f080 b79f  .S.i.7.K.>......
00000710: 5c2c dd4d 43a9 9e1f 5d75 a66e 7fc0 6b95  \,.MC...]u.n..k.
00000720: daff e0af 1745 78ce bb4b e677 81ad 479d  .....Ex..K.w..G.
00000730: 0ae0 9996 ae8e b845 755c 7476 7dc5 35d2  .......Eu\tv}.5.
00000740: 4a23 f4d8 392d 6b8a efb6 00d5 0d6e a897  J#..9-k......n..
00000750: 5aa5 b7ab be6a 1c04 4719 6830 0f6c b2aa  Z....j..G.h0.l..
00000760: 9dfa cb2a 7a5b d259 4931 28dc fa51 799c  ...*z[.YI1(..Qy.
00000770: f270 c813 aee7 4573 4462 6673 de3a ae81  .p....EsDbfs.:..
00000780: d9d2 b0da dc84 56d1 5568 8861 60d0 5388  ......V.Uh.a`.S.
00000790: c0fe 744c aac3 5e30 9be3 c8e8 2e64 3a70  ..tL..^0.....d:p
000007a0: c03c f0e4 b9a4 68c9 8281 1d2b 0ff4 0e34  .<....h....+...4
000007b0: 3a7f 02e5 85be af2a 4ace e7ea 0c7a 608d  :......*J....z`.
000007c0: 0277 215b 3e45 a304 6e59 a0d1 51d6 fc08  .w![>E..nY..Q...
000007d0: 2404 7cc3 0707 b7c6 1f3e d6a0 3601 2aef  $.|......>..6.*.
000007e0: aa05 b5a5 ca4f 6041 09d8 5e38 a637 aae2  .....O`A..^8.7..
000007f0: 281c 4310 3684 9df2 a1a7 8a74 08e7 1f73  (.C.6......t...s
00000800: 7549 9b01 a13c 655a 2c6c 5d20 b458 2098  uI...<eZ,l] .X .
00000810: e42d f32c 84dc 9ef5 438e 3c4a ca24 be08  .-.,....C.<J.$..
00000820: 98ab 5267 24db 6cb1 c4cd 6ed6 0db1 991d  ..Rg$.l...n.....
00000830: 67a7 de9c 3d1f 2138 1bdf ee6d c7e5 5026  g...=.!8...m..P&
00000840: 512f 7e78 afb2 2029 9849 bc46 ea33 a3db  Q/~x.. ).I.F.3..
00000850: a65a 1ee3 cf26 8807 8a29 1280 2715 a2e9  .Z...&...)..'...
00000860: 26b9 99e3 79e3 735d d136 b484 f758 1d81  &...y.s].6...X..
00000870: d5de 38e2 073e 0714 f8e6 f97c d702 1b07  ..8..>.....|....
00000880: 3a39 6b88 95f7 45dd 5945 30cb 46fc 15e8  :9k...E.YE0.F...
00000890: a266 3893 f7b0 85d2 e943 39d9 e2d2 231e  .f8......C9...#.
000008a0: afb3 d092 db42 faab 0861 fa9f 712a e4e3  .....B...a..q*..
000008b0: 2310 2b2f f5b7 a617 2b76 47be 91bb 9fb6  #.+/....+vG.....
000008c0: 193b 6da6 5808 9155 d7e7 39d5 c836 3dd9  .;m.X..U..9..6=.
000008d0: 5932 5ac2 335b e15f 972e bbe9 07c0 386f  Y2Z.3[._......8o
000008e0: 17db e468 aae1 5600 5b0c eef2 3d8f f90e  ...h..V.[...=...
000008f0: 382a 3c92 ca92 62a0 abac af60 564e 7ca8  8*<...b....`VN|.
00000900: b72b 68fc d9a3 248d 9090 a9f5 77a4 74f2  .+h...$.....w.t.
00000910: 1219 b265 0fca 60b3 fb16 3170 a57e fab3  ...e..`...1p.~..
00000920: 7d9b fce0 28fe 338c 9cd6 a131 ccc3 db56  }...(.3....1...V
00000930: 4c73 0944 9dba b018 141f b36e 93da f8aa  Ls.D.......n....
00000940: cb04 57e8 1c4a b101 9b56 1fc8 5090 8c80  ..W..J...V..P...
00000950: ee4f 3ea1 3796 dc1d 6ffd f50b cd60 3fc1  .O>.7...o....`?.
00000960: 83d2 5488 383b fdb5 fe72 9be2 3542 3202  ..T.8;...r..5B2.
00000970: 5843 b26f bbd9 834c 80a8 23ec 4322 1b1e  XC.o...L..#.C"..
00000980: fdd9 3857 4cda 3800 0efc 30b1 5acf c829  ..8WL.8...0.Z..)
00000990: 00d7 7970 794a 431c f57f 2eab 2c2c 4279  ..ypyJC.....,,By
000009a0: fd32 2e62 74fd e585 2273 1874 cceb f826  .2.bt..."s.t...&
000009b0: 4376 77de 892f 1180 76a7 48e7 2433 1601  Cvw../..v.H.$3..
000009c0: 4ce2 f492 620b 0788 4a4a 2228 24e6 6ed6  L...b...JJ"($.n.
000009d0: e1ef 2bab d9f7 905e 883a 6b66 e175 338f  ..+....^.:kf.u3.
000009e0: d13f 32cd ce96 0bd7 cc48 1797 e373 21c6  .?2......H...s!.
000009f0: 6f4c f8cd cec4 4a96 2f28 cdbe b4c1 6a68  oL....J./(....jh
00000a00: 381c 053f 425f 8ec1 fec5 8c2c c020 f49c  8..?B_.....,. ..
00000a10: 0f92 5743 792d a4c3 23a3 128b 53b7 63df  ..WCy-..#...S.c.
00000a20: e2cc 98bb 7d3d 88ad 2bd6 b10e 05ce 4f9a  ....}=..+.....O.
00000a30: a212 2d21 8c10 5065 be23 84be a938 7a82  ..-!..Pe.#...8z.
00000a40: ff96 3883 c345 c72e 156d e98d 41ed 1d49  ..8..E...m..A..I
00000a50: 671b 7c43 3eff ad09 5e53 e8d9 93a1 2a73  g.|C>...^S....*s
00000a60: c095 ba8e afa5 4a7c 0549 198f 179a be33  ......J|.I.....3
00000a70: f40a 83f4 fadc 22fb 50b5 e500 1475 91ac  ......".P....u..
00000a80: 19e2 1958 d073 6af4 a730 0aad cefd 0715  ...X.sj..0......
00000a90: 0968 e434 cc53 ebf0 97d0 bcea 7e74 da3c  .h.4.S......~t.<
00000aa0: b97a 65ca 755d 340f 7c49 f67b cd2c 3d8c  .ze.u]4.|I.{.,=.
00000ab0: 672a a09a b43d 05c7 3e6e cc0c c4af d709  g*...=..>n......
00000ac0: ff1e 2c55 bd95 26c6 2942 3ef5 2a8e e6be  ..,U..&.)B>.*...
00000ad0: 1bf6 4de8 9655 d1f8 0a03 aafd 2967 9c98  ..M..U......)g..
00000ae0: 89d8 eb9f 8b9f d8e8 4d34 043d 9156 484c  ........M4.=.VHL
00000af0: eb0a e968 02cc 7ee5 83e9 4e87 5d2d 60dd  ...h..~...N.]-`.
00000b00: 571d 71e0 8464 ded8 54d4 068e 6214 7fd9  W.q..d..T...b...
00000b10: 5f04 bab3 5116 d078 9a76 40b5 3e9e 1443  _...Q..x.v@.>..C
00000b20: 8ee2 3151 65d6 5527 ffc5 1dd8 bd30 cc43  ..1Qe.U'.....0.C
00000b30: 179a bf15 fbc5 df8e 355c 5f8f 1b91 64b1  ........5\_...d.
00000b40: ea30 4aca e9c2 6b58 8414 bb72 458b fa96  .0J...kX...rE...
00000b50: 28f8 6d6b 72b0 a323 935f 93e6 fd92 aafc  (.mkr..#._......
00000b60: bff6 e2e6 dc56 1a6f b9dc 85c4 a183 8689  .....V.o........
00000b70: e4d9 ff3a 20e9 498a 8deb 99b8 5fa2 48eb  ...: .I....._.H.
00000b80: f90c 0e6b 1c69 9c3c 9e00 0003 0000 0300  ...k.i.<........
00000b90: 02b7                                     ..

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=15360
pts_time=1.250000
dts=14848
dts_time=1.208333
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=2176
pos=32019
flags=__
data=
00000000: 0000 087c 019e 9c44 5700 0003 0000 06f4  ...|...DW.......
00000010: 8fe1 b4df 1fc3 7042 5965 4415 71d3 3367  ......pBYeD.q.3g
00000020: 756c 0d7a e3b2 cb0e ae62 5403 1cb1 4c64  ul.z.....bT...Ld
00000030: 578d a808 8559 02ae 00a7 7a9a f4b3 c25b  W....Y....z....[
00000040: 14a1 fd3d 6e45 1bf6 c390 8330 2f65 5ee6  ...=nE.....0/e^.
00000050: 24db 3fd2 e174 6fe0 b6e8 1d69 8565 ade3  $.?..to....i.e..
00000060: b47f 6b1c 45f5 fc70 4d22 8eac 3c64 3aae  ..k.E..pM"..<d:.
00000070: daef 4960 0861 05d7 49d1 6e9f 5e71 10be  ..I`.a..I.n.^q..
00000080: f3c1 3cbf d39a 0805 2c63 4547 8d52 4849  ..<.....,cEG.RHI
00000090: 430e b421 6a63 6eb9 29f1 f74e 4f2f 7bba  C..!jcn.)..NO/{.
000000a0: 0ba1 5c2f 45b5 eadf cd67 558b b3db 97a5  ..\/E....gU.....
000000b0: 198c 9898 31a2 a5c2 8da9 2255 576e 8b7c  ....1....."UWn.|
000000c0: ff80 65cb 769e 4199 dd62 dba9 cb07 ec78  ..e.v.A..b.....x
000000d0: 5512 8d9b 162a 2b92 d356 54d6 8705 1ef5  U....*+..VT.....
000000e0: 1e50 b654 4b2d 27ed c367 7783 4d26 884f  .P.TK-'..gw.M&.O
000000f0: a3f8 9383 df9c ce49 cf1a 63eb 13a9 b375  .......I..c....u
00000100: 4bb6 36d0 1341 f319 7f0e fbf2 1463 435d  K.6..A.......cC]
00000110: 48e8 60f6 b1be 7dc5 4d69 be5f 3dad b93b  H.`...}.Mi._=..;
00000120: c6fe 3820 f459 252c d6a4 defb e476 19d3  ..8 .Y%,.....v..
00000130: b3ba 7480 6b9d 21ca 50a6 f323 88ce 6f0d  ..t.k.!.P..#..o.
00000140: 29e2 0a74 97d1 aaa6 c400 e7ae 8fa4 0cc4  )..t............
00000150: 791c a2cf 1cee 44a1 adc2 b9e1 9320 4f85  y.....D...... O.
00000160: 4540 44fb 2ef0 41f0 5dc9 1fc8 e9f4 a479  E@D...A.]......y
00000170: e0f0 3ad4 1ec3 4697 9f03 b061 5d3b 35cb  ..:...F....a];5.
00000180: 8347 cbe7 d9df 1dd1 be08 ed0c fda9 da74  .G.............t
00000190: f58f 8cce fefb 0a8d 5bcd 4fb4 be6c 69b1  ........[.O..li.
000001a0: f308 f83f 7f4b 4d23 e5ce 9b3b c0e4 0c45  ...?.KM#...;...E
000001b0: a2e2 e8e9 ba2c 21ce 7914 a9a3 98bf 888a  .....,!.y.......
000001c0: 8b07 f436 2c95 dd8f 9597 db1f 61a3 807a  ...6,.......a..z
000001d0: 5396 af3a 56e5 4a47 45ff d72d 4fc9 e62c  S..:V.JGE..-O..,
000001e0: 50cd 3449 98e4 12f1 9f60 4dc5 3a65 d1e5  P.4I.....`M.:e..
000001f0: dc7c 1707 64e5 4b9b 5408 5660 c882 17e4  .|..d.K.T.V`....
00000200: d887 7f23 1a8c 1e48 3bd1 f083 2ffa 61fa  ...#...H;.../.a.
00000210: 03ea ff2a 1260 f221 5ea2 5277 10fa f2b9  ...*.`.!^.Rw....
00000220: de42 1d98 39ab 37de fc16 6e6f 6172 ebf7  .B..9.7...noar..
00000230: ef40 5d6f b6b9 4b19 9ca2 f5a5 ee66 73bd  .@]o..K......fs.
00000240: a024 f35f 7a2e 6cd6 dbdb 8e4d 084a f888  .$._z.l....M.J..
00000250: ae83 b4fb 6565 9536 b730 bf02 3150 89f7  ....ee.6.0..1P..
00000260: 0dc2 800f d791 18df d801 57d0 bd2f a20a  ..........W../..
00000270: 1e3f 1a84 62a8 c5ab 8d96 86ee 4b95 a823  .?..b.......K..#
00000280: 6bb8 45bd a698 9bc1 b949 488c 9992 5fbd  k.E......IH..._.
00000290: 44db 371b 6997 6219 9be5 b1c7 7894 ce91  D.7.i.b.....x...
000002a0: 361d e6f5 99e3 3987 8cd1 3822 2fe9 8069  6.....9...8"/..i
000002b0: da6c bc63 27b0 4d71 837c 6ecf 8d43 68d8  .l.c'.Mq.|n..Ch.
000002c0: 88cd 35c1 4edd 03d0 8d2b 47f7 4cf9 444d  ..5.N....+G.L.DM
000002d0: 2a4a 93ea 9363 08c4 0c89 2230 9e78 502d  *J...c...."0.xP-
000002e0: e067 32ac e2ca 9f1f ddb9 692a a4f4 2500  .g2.......i*..%.
000002f0: 40b5 c20e 9244 5e2b 94eb a9be 735a 616d  @....D^+....sZam
00000300: 8e79 e09f d6ae 7ac9 a8f7 23f9 a33f b933  .y....z...#..?.3
00000310: fd69 c7c0 4f9c d391 4157 86f5 9c5b fd71  .i..O...AW...[.q
00000320: 4cbc 4121 b8ad e776 b98d bfe5 8fe1 d253  L.A!...v.......S
00000330: a429 8597 31c7 3778 7a61 da7a cdd5 23b1  .)..1.7xza.z..#.
00000340: af3a c794 9e2f 0edd d092 5fb8 ef96 356d  .:.../...._...5m
00000350: b08e ac8f 536c b25c e1db 33ab cbf8 16ad  ....Sl.\..3.....
00000360: c047 2e2d 2fb2 7dc0 11fe ff96 ee39 2d7c  .G.-/.}......9-|
00000370: 2e9e ceda 94e8 b946 08cd af3a 3715 a7e9  .......F...:7...
00000380: 069b 5631 784a e47c b63a debc 49a1 fea2  ..V1xJ.|.:..I...
00000390: 570a 392a 5976 6f5c 46e5 595a 9f19 2343  W.9*Yvo\F.YZ..#C
000003a0: 6f59 e92d 4f53 bbf0 be6c 973f 31bd fdfd  oY.-OS...l.?1...
000003b0: 733b 14ba 4e4e 6676 2ef2 be57 d027 64f2  s;..NNfv...W.'d.
000003c0: e99e f315 8351 40f5 5681 732c 10ae 72b2  .....Q@.V.s,..r.
000003d0: e57b 3d98 5fec 707d 9ecc 6afb 19a6 f17d  .{=._.p}..j....}
000003e0: ac24 f30e ad70 a33d af6e c4f0 cd0e ccc2  .$...p.=.n......
000003f0: 0e9b ce25 be5d 2819 0bb8 c4ed 3995 abfc  ...%.](.....9...
00000400: 8f7e 24ce 23a9 d7fa 93d0 bf92 2f5f 739b  .~$.#......./_s.
00000410: 4504 f520 3a60 b36a 13ae 9e84 91b7 a7b8  E.. :`.j........
00000420: 844b 1a10 cd50 18ae 91a7 63e3 00c5 0623  .K...P....c....#
00000430: a4be d30e d7c6 6b8c 61c5 8dff 9b2b 8c7f  ......k.a....+..
00000440: 51b7 6602 5558 3522 64e3 f749 95a8 dca9  Q.f.UX5"d..I....
00000450: 9a03 d7e8 871f df08 2226 a00e 52ce 66e0  ........"&..R.f.
00000460: 2240 8966 06df 570f a1e8 2b72 e645 0d8c  "@.f..W...+r.E..
00000470: b95b 58da 79e6 7b89 51eb 43dd 611f 20b8  .[X.y.{.Q.C.a. .
00000480: 56d9 86ca d67d 9b94 983a 3638 b93b d9ac  V....}...:68.;..
00000490: ff7b 9816 fee5 2928 66ee 6793 1d56 9aad  .{....)(f.g..V..
000004a0: 7c42 cef2 2812 4ca9 d6c2 e063 f576 ec53  |B..(.L....c.v.S
000004b0: 30c5 a8c8 7400 6f6b 1a2a c893 8ae6 294d  0...t.ok.*....)M
000004c0: 4cb9 0374 9976 2c95 4537 75ff cacb 6562  L..t.v,.E7u...eb
000004d0: 0b3a 3b1f fe8b bfe2 b49b c3a2 7b86 b675  .:;.........{..u
000004e0: 557a e2ff f710 c0c3 26bb 3235 23a0 b932  Uz......&.25#..2
000004f0: 2b35 95ba 1383 ce16 3116 7b56 2dff 8bb6  +5......1.{V-...
00000500: 77dc eed4 47ad 697a 84eb 761d e2b8 62d5  w...G.iz..v...b.
00000510: 991f b3f1 8ecc 3ce8 5f60 b9ac 515a 6695  ......<._`..QZf.
00000520: 46f6 fd4c c479 2650 08c3 b91c eb77 5789  F..L.y&P.....wW.
00000530: 0deb e5e7 9fbf 9c34 8cb8 178a 733c 743f  .......4....s<t?
00000540: 0ba0 e008 b8cb ee45 d77b 7588 b8fb e4be  .......E.{u.....
00000550: 64ae 9fa7 fa08 d5e5 fe25 045a ae99 9cc9  d........%.Z....
00000560: c1b1 c41a 9f2b 5986 6000 ef66 9fba fd62  .....+Y.`..f...b
00000570: a851 aa33 79ba 7f5a 67b7 26b0 7ad0 0d31  .Q.3y..Zg.&.z..1
00000580: 5fa6 747a bf19 92d4 0661 27b0 8f5c 62e9  _.tz.....a'..\b.
00000590: 2201 6077 4efd 9fd1 55da 66bd d25b 314b  ".`wN...U.f..[1K
000005a0: 9cdc faf1 ddb9 f5f3 322e 347f 5e77 3be2  ........2.4.^w;.
000005b0: 5dfa f9a1 a543 1e96 c78a c924 3390 1c5c  ]....C.....$3..\
000005c0: 4c6e bef4 7a97 6afc 0ade 0761 bbce 579a  Ln..z.j....a..W.
000005d0: d177 d465 d0af 1cf4 bae5 3114 e82b 906b  .w.e......1..+.k
000005e0: e84a 7c0c 1a45 8091 67f1 008a 66cd 7159  .J|..E..g...f.qY
000005f0: 86c4 6c40 5935 ea67 63d3 223c 796c aa18  ..l@Y5.gc."<yl..
00000600: 6b6e 1521 2f68 0e83 7fc2 c285 651e 5053  kn.!/h......e.PS
00000610: dec3 668d fa7a 7773 5d9e b9d6 3844 1291  ..f..zws]...8D..
00000620: 13fb 63d3 d075 35a1 f575 4186 06cd 615f  ..c..u5..uA...a_
00000630: 279b b544 db7a 2ea8 7067 9375 d00e 670e  '..D.z..pg.u..g.
00000640: 0697 4bde 282b bd18 5960 7543 afac 95bd  ..K.(+..Y`uC....
00000650: e78c 975e befc 48b6 b0de 22d6 bf6e b108  ...^..H..."..n..
00000660: 2baa 06fd e4d2 8bd1 7372 956f d2b9 3322  +.......sr.o..3"
00000670: 0d3d dca5 9a06 6716 faf6 d691 ee89 2f3f  .=....g......./?
00000680: 4733 7623 6c31 fa48 10a8 9274 9ee6 171b  G3v#l1.H...t....
00000690: 75fc c2a8 8ea9 ee4a c834 39aa 4aa4 bc22  u......J.49.J.."
000006a0: 8565 801d 0ae1 13fc 4687 de3d 3089 e63d  .e......F..=0..=
000006b0: ce3e 6b82 8528 3586 a7cc 2a97 a491 b1c1  .>k..(5...*.....
000006c0: cabf 7546 62bf dd55 5fef b82a 8d25 d51b  ..uFb..U_..*.%..
000006d0: ffb4 5735 3410 6d3e 0cf3 241c 34f0 3b3c  ..W54.m>..$.4.;<
000006e0: 47db 8547 caef d4cb 57e7 8af1 17a3 4a58  G..G....W.....JX
000006f0: 4fa2 1547 f60b 4c62 c8b8 ab60 7ee9 f32d  O..G..Lb...`~..-
00000700: 7730 027b 353a c2e3 b1be 8a64 0846 5328  w0.{5:.....d.FS(
00000710: 332a 1b12 0a95 d1b5 364d 5a23 a458 44a7  3*......6MZ#.XD.
00000720: 5590 8fd2 1c2f d5e3 337a b10d c2cc f8a1  U..../..3z......
00000730: ba43 e85a 0837 a080 906d aaad 8694 4d39  .C.Z.7...m....M9
00000740: 2b49 82c1 c54a 1169 6b0a 2c67 c8e4 b4e8  +I...J.ik.,g....
00000750: bbe1 af50 f476 f870 2220 d3df cc80 dbd5  ...P.v.p" ......
00000760: d468 1c5b 335a 29fc e6ad d87d 7923 089b  .h.[3Z)....}y#..
00000770: c6f4 2ae0 3bf9 f7ee f754 c6c2 43ca 8f1f  ..*.;....T..C...
00000780: 94f0 532f c3d6 5419 b7f1 efd8 45c1 83f3  ..S/..T.....E...
00000790: d43a 0baa 76a6 dca9 0d82 ddfb ee57 415a  .:..v........WAZ
000007a0: 6e26 4df7 46ec 52e5 5df0 3698 b84a 2657  n&M.F.R.].6..J&W
000007b0: 41fb f105 d9d0 d6ec 1241 9982 12d5 1bd7  A........A......
000007c0: ca22 ca0a 4c01 252e 6a0d 9d96 58ce 376d  ."..L.%.j...X.7m
000007d0: 1c81 c32c b308 615d 3a9f 9d09 2b1d 87d0  ...,..a]:...+...
000007e0: 1111 de32 4f22 9034 8bbe 21e4 f06c c748  ...2O".4..!..l.H
000007f0: 2e34 d576 2667 e252 693f 9270 bc27 bf43  .4.v&g.Ri?.p.'.C
00000800: 4bb7 da3e 280e 201e eba4 278b 9dc8 6003  K..>(. ...'...`.
00000810: 0393 f3e1 8a2d 06f6 c6ab de97 9cac c943  .....-.........C
00000820: e8ad 2f4a f794 f8ed 9838 01ca 375b 6ca0  ../J.....8..7[l.
00000830: 7127 7d70 b8a7 622f 54ca c3fe 0129 25ff  q'}p..b/T....)%.
00000840: f6ae f004 5248 f234 9a73 9ada bf95 1383  ....RH.4.s......
00000850: 1152 a581 bbac c7bf 07a3 1fb4 cb02 4f41  .R............OA
00000860: 285d 72f4 28d7 2b58 8e48 4a5b 97d8 1066  (]r.(.+X.HJ[...f
00000870: df31 10de 7d41 8000 0003 0000 0300 5941  .1..}A........YA

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=17408
pts_time=1.416667
dts=15360
dts_time=1.250000
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=3376
pos=34195
flags=__
data=
00000000: 0000 0d2c 419a 8034 a43e 0258 e115 ff00  ...,A..4.>.X....
00000010: 0003 0000 0300 183b c788 33bc 7f1c 9d00  .......;..3.....
00000020: 99cd fa44 8ed3 be58 102d 7f51 25ac a8f6  ...D...X.-.Q%...
00000030: 7e85 a032 72a6 09fd dbcb 9f00 2684 bf91  ~..2r.......&...
00000040: a55e 91b6 2398 176b 45e5 4000 2d18 3623  .^..#..kE.@.-.6#
00000050: c941 4dc6 152a 279a 8240 0596 a63b b566  .AM..*'..@...;.f
00000060: 058b 7e4d 1c22 de5d af47 133f 7e8d 0174  ..~M.".].G.?~..t
00000070: 1bea f5ea a5aa 27d7 ff0b 7122 28d4 bb88  ......'...q"(...
00000080: 0001 784b 4d06 17e4 0f10 61e1 81fc 4d79  ..xKM.....a...My
00000090: b758 9784 addf ae05 f618 79e5 bbd0 aaa0  .X........y.....
000000a0: ddfc fa19 c1ee 3e23 1125 d668 a514 1726  ......>#.%.h...&
000000b0: a630 cf1b e59b 4fa8 ba61 bfbd 3eae 5bab  .0....O..a..>.[.
000000c0: 1b04 08e9 4d94 eef2 8379 095d 4c77 e5dc  ....M....y.]Lw..
000000d0: 503a b52b dac4 f2ed 7195 5f56 8429 0f18  P:.+....q._V.)..
000000e0: 8ba1 ca34 e8cf 1a81 3c01 4009 876e dbcc  ...4....<.@..n..
000000f0: 1506 cbaa 0ca3 0816 b345 ba36 cc3d 251a  .........E.6.=%.
00000100: c66d d0d9 66da e6b8 7071 097e 2f91 4057  .m..f...pq.~/.@W
00000110: ab0c ce8e bf29 7433 cc68 0d8b fe2b e9a4  .....)t3.h...+..
00000120: 7264 6ad6 8c2e eab8 7844 226a 0b91 962d  rdj.....xD"j...-
00000130: 289c f709 f04c 453b fdd0 0ee6 ec7f 6993  (....LE;......i.
00000140: 91b5 0395 e654 258d 12b4 05a9 c0f6 ab05  .....T%.........
00000150: 28f0 3cdd 0eb5 6aa7 1436 9fe2 ca77 efdd  (.<...j..6...w..
00000160: d9f0 6c56 19fc a113 c554 dff0 1cdd 5a0d  ..lV.....T....Z.
00000170: 2db9 0fe6 bb04 d416 281c bf8a 0be4 9df0  -.......(.......
00000180: 8a32 38d7 9e7c fcb1 85fc 7d51 5237 5c5b  .28..|....}QR7\[
00000190: 68f4 f8fa 3bdf f18c 1541 c8f9 c5c2 43fd  h...;....A....C.
000001a0: 825e 752c 6005 3bb1 f611 164b b58c 80b2  .^u,`.;....K....
000001b0: b460 ed72 5213 9d89 2bfb b19e c63c 708d  .`.rR...+....<p.
000001c0: 0673 466a 16c9 bc50 d42b b945 ce25 3159  .sFj...P.+.E.%1Y
000001d0: d837 ac23 66ea 1aea f760 db68 08c9 2007  .7.#f....`.h.. .
000001e0: d505 21db c39c 2f39 b51b 2272 de7f b7d3  ..!.../9.."r....
000001f0: 6054 0a28 098f c3f0 6030 7161 340c a704  `T.(....`0qa4...
00000200: b28d 09dc fa95 85e1 2314 1813 b435 39a8  ........#....59.
00000210: 7f0a c67c 6502 56c6 2610 acb2 0dbc b4e3  ...|e.V.&.......
00000220: 4ada f03e 7580 d4b3 6179 4804 fc77 7537  J..>u...ayH..wu7
00000230: 5b30 9b0a 9f35 2ba9 a901 1a5f aae1 7c99  [0...5+...._..|.
00000240: ccc2 4965 38f8 d898 98ad acc5 2ed9 bcc0  ..Ie8...........
00000250: a22d 0bbe 36f2 a8ac 1411 7f60 e448 6cd1  .-..6......`.Hl.
00000260: 5202 02e0 880b afd8 ce49 02b4 8633 b1bf  R........I...3..
00000270: d1a6 f162 de27 e353 81d0 0a4b 4d98 0df3  ...b.'.S...KM...
00000280: 1e1d 6ec2 8bc6 dab5 7836 b087 ae9c 426a  ..n.....x6....Bj
00000290: de7d b003 29eb 7642 df01 dac7 2922 8767  .}..).vB....)".g
000002a0: 3e59 2de6 4085 b248 a80c 485d c244 c08b  >Y-.@..H..H].D..
000002b0: e1fe efa0 402b b7f8 5742 f3a9 deff f7b8  ....@+..WB......
000002c0: b12d 00b2 26ba 89e9 3c03 90ac d006 183d  .-..&...<......=
000002d0: 3bf3 87cd d422 c473 c435 ff68 7aec 8047  ;....".s.5.hz..G
000002e0: cd70 8c8a 88da 39ac 9dc5 6c30 361f 6c87  .p....9...l06.l.
000002f0: 08d0 6417 ea16 0052 8d61 51a4 5867 6b3a  ..d....R.aQ.Xgk:
00000300: 658e 33e6 421a 7b4a a590 8130 13c6 c7ec  e.3.B.{J...0....
00000310: 7312 2a61 14d9 be69 f083 59e3 3f30 3e41  s.*a...i..Y.?0>A
00000320: 2620 37de c2e8 0019 2930 9fea f7b5 954c  & 7.....)0.....L
00000330: 4812 9121 8ee2 d074 519c 4328 e99b e9a5  H..!...tQ.C(....
00000340: 8f38 e036 15ae 800f 73fc 88db a7d5 502a  .8.6....s.....P*
00000350: c653 f384 805d 1d90 da99 3cf7 6e7a e604  .S...]....<.nz..
00000360: 01a4 5c5f 402b dee9 a062 d832 8006 d613  ..\_@+...b.2....
00000370: fe29 eb40 f76d 956d 51bb d5f5 ed59 a3ed  .).@.m.mQ....Y..
00000380: 9ab0 1af3 5918 dd53 b380 3938 d335 0935  ....Y..S..98.5.5
00000390: a9c6 e7d1 266f f51d bc84 d2d1 9d01 8441  ....&o.........A
000003a0: 14ff fbe0 73cf 30fe 85ba a2b3 f32b 56cd  ....s.0......+V.
000003b0: 0c45 30c0 0000 530c 81f9 a0b4 be6e dcf7  .E0...S......n..
000003c0: eb00 ac96 937c a974 4f9b 613e 69c8 93d9  .....|.tO.a>i...
000003d0: 16ec 1660 5877 73d8 00b7 ccbe c7be fc4e  ...`Xws........N
000003e0: d727 cfdc 4358 b37f 666e 661e 6c20 09a8  .'..CX..fnf.l ..
000003f0: 9807 d0e1 4b9c 0006 0ace e47e b722 bc97  ....K......~."..
00000400: 2f3d bfe9 5079 4e5c 03de 7c8c 8bd2 f8b1  /=..PyN\..|.....
00000410: 80e9 7065 2977 ec5a 3597 6edc df84 96ea  ..pe)w.Z5.n.....
00000420: e5b6 5146 d320 3036 b4eb 3fca 7823 fe2b  ..QF. 06..?.x#.+
00000430: 74b4 c5d4 1cfe 8600 0126 1848 8b71 211d  t........&.H.q!.
00000440: 1c48 19eb 9d33 20f6 71d1 af1a 4411 fcab  .H...3 .q...D...
00000450: 3695 4324 1a05 4ea8 95d1 6634 ebae 0099  6.C$..N...f4....
00000460: 9e2c 116b 8786 3dd5 10a2 6806 9158 c05b  .,.k..=...h..X.[
00000470: b220 33c5 7896 b572 4e0b 6d51 5759 7ca8  . 3.x..rN.mQWY|.
00000480: 3c60 d462 1f50 127a 0c6d b89d e1b0 97ae  <`.b.P.z.m......
00000490: 071a 8a78 af01 1d33 048f bdef 299b 8c67  ...x...3....)..g
000004a0: 9800 7946 0020 c27b 3542 76f0 ad14 b71f  ..yF. .{5Bv.....
000004b0: 7bfd 003a fce3 f2a4 1d7a 4c78 e975 7f2b  {..:.....zLx.u.+
000004c0: b2b4 d3c7 7f96 8d31 0628 dd59 7a89 e8e4  .......1.(.Yz...
000004d0: 0a72 dea4 c973 adfc b122 a4cf 1023 fe06  .r...s..."...#..
000004e0: e54b ab87 241f 78f0 4490 be00 aeea d570  .K..$.x.D......p
000004f0: a51b 1e9b d4c2 40cc aaa5 7d7a 5d97 1c61  ......@...}z]..a
00000500: 1054 4574 86df a528 d0df 0e7e 7f6d db58  .TEt...(...~.m.X
00000510: d818 b3bf 91e3 5554 a1f6 edc6 95e1 49e1  ......UT......I.
00000520: b6db a24d 4eb8 65f0 65dd 0042 f960 9363  ...MN.e.e..B.`.c
00000530: 66bb 8860 2ff8 c5b0 00d1 8d84 dffe daca  f..`/...........
00000540: 3b21 f631 e0a7 779b 94a1 3027 8074 ff0a  ;!.1..w...0'.t..
00000550: dc39 7c76 09fa 12c5 7484 463f fe34 3e64  .9|v....t.F?.4>d
00000560: e9f1 3147 a3fc 8e25 21b9 6da2 ee22 5340  ..1G...%!.m.."S@
00000570: c65b 67d1 6ab4 a7f2 58c6 8b25 e989 53ce  .[g.j...X..%..S.
00000580: d682 dd7d 201d a3e7 ec9b f5a4 7a88 640f  ...} .......z.d.
00000590: f5b5 efd7 aa00 b345 7b76 b2d5 0878 504d  .......E{v...xPM
000005a0: 26e2 f809 f63b d062 15dc b9d1 39ec c430  &....;.b....9..0
000005b0: d907 b0c2 119d 3c83 5aad 5177 d523 83ad  ......<.Z.Qw.#..
000005c0: 49f6 4f6b aaa9 2f99 4a86 9401 50bd 042e  I.Ok../.J...P...
000005d0: 6450 8d3f ee40 be25 3d73 adc3 e042 6360  dP.?.@.%=s...Bc`
000005e0: 1bdf 39b5 7f59 5717 659a c996 caec a60d  ..9..YW.e.......
000005f0: 1f0e b84d 465c 3a70 fa77 2376 3261 a694  ...MF\:p.w#v2a..
00000600: 50e1 037d 869a 8681 6230 eeb6 82f2 13f3  P..}....b0......
00000610: 6378 9480 21d6 3c83 794c 02aa 09b6 b001  cx..!.<.yL......
00000620: 10f3 5b23 3f4e add9 78b7 b20f 3691 8112  ..[#?N..x...6...
00000630: 3913 9381 65e7 35f5 e5ea a3f3 11da ddde  9...e.5.........
00000640: 2d5d a5fb 53d1 44fb a930 cf06 3a59 955a  -]..S.D..0..:Y.Z
00000650: 98d2 45bb 2dc8 fd3c 7dbc 0d62 92d6 b49e  ..E.-..<}..b....
00000660: 6d68 4180 bccb 528a c655 0869 f145 77d1  mhA...R..U.i.Ew.
00000670: 8669 4866 dcab 60f0 9887 3b6b 6af3 a240  .iHf..`...;kj..@
00000680: 6efb 6ac0 ae5c 8f5f 291c 70b9 fa5f f72b  n.j..\._).p.._.+
00000690: 4d8b 3048 0b53 bb9e f551 e4bb 449b 834d  M.0H.S...Q..D..M
000006a0: 1780 1d23 d874 88de f267 ffba d1a3 8612  ...#.t...g......
000006b0: a624 21ca 4e7a 270f 1eb2 0364 bc47 8dd6  .$!.Nz'....d.G..
000006c0: 6e30 f9ab ebe1 3c17 8e42 95b1 ece0 7215  n0....<..B....r.
000006d0: 6f69 d14f 1166 2651 6de4 e9e1 aa9d d445  oi.O.f&Qm......E
000006e0: 4b33 4217 7b46 292b 4669 9e8e d369 7186  K3B.{F)+Fi...iq.
000006f0: 630b 871f 1eb6 7a65 63ad 1999 6ca9 f06b  c.....zec...l..k
00000700: ec31 0a36 95dc 150c 86cf 1f62 d9f7 93f2  .1.6.......b....
00000710: 75c7 448e 851d a4d7 6e2f 4e30 be00 6f49  u.D.....n/N0..oI
00000720: 0623 1b89 9a06 ac0a 03c3 09ad 2f07 35fa  .#........../.5.
00000730: fd97 2443 4ded fecf 9e0f 9da6 1119 af26  ..$CM..........&
00000740: a287 b286 ebe5 690e 7273 393f 0654 19c5  ......i.rs9?.T..
00000750: 89d5 d0f5 f19e 96a3 a4ec 4bfc 0264 214c  ..........K..d!L
00000760: 4304 21ca 9181 c993 24a0 a8e7 8844 b10d  C.!.....$....D..
00000770: c8a1 04d1 81bb 4359 6a7f 5662 474c 22a7  ......CYj.VbGL".
00000780: 4526 f29c 0b8f f343 620b 7ddb bbc4 c3f6  E&.....Cb.}.....
00000790: 5538 fc31 0e66 9654 1a16 fe1c 6573 bde5  U8.1.f.T....es..
000007a0: 2946 d8a3 7c56 1dbb 3c21 d4bd 8dcd 43ac  )F..|V..<!....C.
000007b0: 363e 495d d989 9ae3 bd7e 5a80 1e4f c08d  6>I].....~Z..O..
000007c0: 0219 2804 9ebc 7702 1b13 2ab4 ab79 bbaa  ..(...w...*..y..
000007d0: 64d8 6c9f a97f 6a79 62c3 a323 dbf2 d906  d.l...jyb..#....
000007e0: dc91 21f1 62ed ff9b e5d5 f3ad e928 8918  ..!.b........(..
000007f0: efa9 707f f210 4a05 2753 c645 5930 4075  ..p...J.'S.EY0@u
00000800: 6d18 7780 7f07 531c 410f 7794 8b6c dcb2  m.w...S.A.w..l..
00000810: 3d7b e17f 68de 50c3 2ae7 89b4 eb14 6ac0  ={..h.P.*.....j.
00000820: bc19 e0ad d832 f4bc bd32 2d76 0916 3cc2  .....2...2-v..<.
00000830: bae2 c3e4 d57f 176a 22f0 f7dd b6b6 3f50  .......j".....?P
00000840: 54af 6328 c75c c26f 6a9a 9e4f 51f9 1e81  T.c(.\.oj..OQ...
00000850: fd6c 7fd6 90ef 4b8d d477 a3e8 8be9 0fa0  .l....K..w......
00000860: caca 10d3 efe5 0b3d eab3 7877 494a b757  .......=..xwIJ.W
00000870: c96a 5d4b 8f4d 936b e0c9 3085 5cc6 ba3d  .j]K.M.k..0.\..=
00000880: bae7 b11f 648c 6e0c 2f23 4737 6f86 793f  ....d.n./#G7o.y?
00000890: f6d1 1d66 d9bc e508 dc03 7190 cc2b 59d1  ...f......q..+Y.
000008a0: 4646 f372 872c 9bf2 25ee 2112 cc42 395d  FF.r.,..%.!..B9]
000008b0: d82c 37c0 9e8d 7843 ee3b 2e96 2991 3a40  .,7...xC.;..).:@
000008c0: c545 4e37 70bc fb77 30f4 8dd9 6a4f 227b  .EN7p..w0...jO"{
000008d0: 8e6d 7f55 e042 1fed 6671 2d35 5cda 5387  .m.U.B..fq-5\.S.
000008e0: aa10 89c9 099c cace 6387 2ac3 75ca e0e5  ........c.*.u...
000008f0: f6ff a73d f804 2407 7e05 b1f0 adda 3755  ...=..$.~.....7U
00000900: 9e0f f195 25b4 a843 f73e cab1 6975 d0c8  ....%..C.>..iu..
00000910: dbb4 04b9 f10e d8fb c2a1 2937 6675 3056  ..........)7fu0V
00000920: dec9 83a5 d33e af04 efac 1683 23fc 8a67  .....>......#..g
00000930: 88e0 7385 78a0 8298 e405 dabc c5d2 1b23  ..s.x..........#
00000940: 0ae4 e20d 9b3c 29ed f1ab 47f0 c975 ee1d  .....<)...G..u..
00000950: c2b8 8186 bd37 f6fc 380c e7f0 19f4 6c0d  .....7..8.....l.
00000960: c802 3a7c 1338 e277 837b 1d4b 8a4a e0c9  ..:|.8.w.{.K.J..
00000970: a9dd a75d e230 b292 8af5 913e 5727 800d  ...].0.....>W'..
00000980: 307d 9788 479c 9004 1f3e e4be 8b56 c114  0}..G....>...V..
00000990: e8df 6494 c41f be5f bb96 2b80 fe28 d253  ..d...._..+..(.S
000009a0: 5c68 7dd3 3a16 12ff 2537 a22e fab2 2e07  \h}.:...%7......
000009b0: 7169 1ecd dcac 4637 268c f167 7bc2 eae2  qi....F7&..g{...
000009c0: fb1c 64c7 21b8 d9f2 b24e 9832 81c3 5fc4  ..d.!....N.2.._.
000009d0: 2dea 5051 563e ddbc bceb 7c6b 0848 bde5  -.PQV>....|k.H..
000009e0: 5689 d55c b79d d188 5478 ddf8 e05b 00f9  V..\....Tx...[..
000009f0: ec98 c32a c53d c42c 6c09 5a3a 625e feeb  ...*.=.,l.Z:b^..
00000a00: 10b9 634c 2373 c91a 0a9b 611f ba34 70bc  ..cL#s....a..4p.
00000a10: 47fe 3d33 df93 43e4 e22e ffde bda3 2673  G.=3..C.......&s
00000a20: b1ff 4380 6f93 85cc 69b8 fd1d 453b ff47  ..C.o...i...E;.G
00000a30: 23b4 57aa 7b5b b475 afc0 dc5d 1cd5 3945  #.W.{[.u...]..9E
00000a40: 3f8d 8718 584e db32 86e9 fe25 fbb8 90eb  ?...XN.2...%....
00000a50: b425 367e ef01 38ba dd93 a9a8 8116 b85d  .%6~..8........]
00000a60: ae02 5e54 e323 89f2 59e2 a134 2cfa c80f  ..^T.#..Y..4,...
00000a70: 7ce4 6a91 6792 73d9 fe08 a969 082b f008  |.j.g.s....i.+..
00000a80: efcf 4582 1666 84a7 6917 cae9 d6f3 155f  ..E..f..i......_
00000a90: bd78 7e97 33f0 df7c a66c d4b8 c722 9aae  .x~.3..|.l..."..
00000aa0: 215d 0446 5894 8ce4 9cdb b219 2713 3161  !].FX.......'.1a
00000ab0: aae9 5c0f 87c3 a039 0279 6d28 e489 80af  ..\....9.ym(....
00000ac0: 45fd 1e4c 8da3 925e 30c6 7549 3b1d df8d  E..L...^0.uI;...
00000ad0: b567 7fda ee8b 7a0b 0040 f8a3 a2b3 be2f  .g....z..@...../
00000ae0: d56d 7cd2 4354 e590 db68 8766 6b35 7c2e  .m|.CT...h.fk5|.
00000af0: a075 9e10 628f 7e41 77fe c7a1 dd61 9985  .u..b.~Aw....a..
00000b00: 925b a377 4122 ab53 ca13 b9b1 133b 3bdf  .[.wA".S.....;;.
00000b10: 8638 0dce 7b0c 5097 f177 263b be44 d1ab  .8..{.P..w&;.D..
00000b20: bb52 bb8c bbf5 1779 433e 4a00 72e8 87af  .R.....yC>J.r...
00000b30: 9ca8 ba82 850d c9b5 711a 8580 46a2 7695  ........q...F.v.
00000b40: 8e36 f363 dcf3 c7af dd19 0ee1 5441 a5fc  .6.c........TA..
00000b50: 82cf 9869 a483 7096 9399 fecb 54b0 dec6  ...i..p.....T...
00000b60: 505c 27f6 1c40 7734 b08e ce51 e726 2efe  P\'..@w4...Q.&..
00000b70: 6c77 f68b 383c a526 5c35 2d4b 0712 8d0d  lw..8<.&\5-K....
00000b80: 5442 c553 3143 16ae 8af3 4239 263a c6ab  TB.S1C....B9&:..
00000b90: 06ab 30c5 e127 8116 0a0b 96b6 1a08 9bfb  ..0..'..........
00000ba0: 3658 77cf 7450 5923 4f99 52c7 de90 f331  6Xw.tPY#O.R....1
00000bb0: c46a 5ede 82db ec1e ab82 0c1e 8223 126a  .j^..........#.j
00000bc0: b8b7 9e96 3d84 6a34 ce2e c1df ea59 2cb6  ....=.j4.....Y,.
00000bd0: 8260 d5ec 0457 d3d1 b259 471e c2a0 1651  .`...W...YG....Q
00000be0: c0ef 741c 8dc5 f7a1 1bde fd65 599c 29f8  ..t........eY.).
00000bf0: ea44 4e91 0814 0f0e d2b5 f457 2e55 1e9f  .DN........W.U..
00000c00: a24d f4dc 0138 fd20 a5fa 6c2c b9c9 9694  .M...8. ..l,....
00000c10: ba8c 6894 8db9 ee63 df2d 85cf 0cdd b13a  ..h....c.-.....:
00000c20: 74b8 c895 54b6 3bdc 2931 f3a6 3b17 ccf3  t...T.;.)1..;...
00000c30: 274d b2ba 6ac3 b9cf ad70 8feb 2bff 1bae  'M..j....p..+...
00000c40: f6e1 3b18 e149 1af6 19fe d3f0 cd06 5d96  ..;..I........].
00000c50: 4931 0750 d4c9 d484 1ef5 6bbf 6f19 0790  I1.P......k.o...
00000c60: ab01 b1b8 c0cd 4b6a aea5 054f ca0c eaab  ......Kj...O....
00000c70: 2d76 a9a4 e4b5 85b9 284b 6d38 e0f5 e0ff  -v......(Km8....
00000c80: 1a6c 65eb e6f1 6184 f4d0 05ac 95e4 9fd4  .le...a.........
00000c90: a291 d8c1 fee9 59f4 eae6 956b 98a3 5319  ......Y....k..S.
00000ca0: 6144 89e2 4b93 f6d9 68f5 00a9 06c1 6e20  aD..K...h.....n
00000cb0: 29f3 b5ff 2a28 6ecd a7dc b15b bf7d de86  )...*(n....[.}..
00000cc0: e0de e073 7854 a62c f160 a924 0642 4e7b  ...sxT.,.`.$.BN{
00000cd0: c7b3 f60a 23fe da83 bbf1 6974 b8d1 9fc9  ....#.....it....
00000ce0: d65d 72e3 fb7a 3483 f60f b99b 6119 0c47  .]r..z4.....a..G
00000cf0: 3069 1510 8b93 79dd 825e aeed 3e18 b279  0i....y..^..>..y
00000d00: 693c 0359 8b4d c78c d2d9 fcb8 2ee3 8451  i<.Y.M.........Q
00000d10: 4ef4 b9f6 7910 3d3d ed80 e75c 8290 69c3  N...y.==...\..i.
00000d20: a79a 0e49 2b5d 4326 01df 1ae1 861a 67f8  ...I+]C&......g.

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=16384
pts_time=1.333333
dts=15872
dts_time=1.291667
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=3249
pos=37571
flags=__
data=
00000000: 0000 0cad 419e be45 152c 5700 0003 0000  ....A..E.,W.....
00000010: 06a1 ad74 a244 3ecd 2219 e95d c740 e592  ...t.D>."..].@..
00000020: 6105 d915 79dc 08ab f4ec a310 68f5 2d79  a...y.......h.-y
00000030: 5f7f 8e25 aabd cd38 815d 5250 1d82 f124  _..%...8.]RP...$
00000040: 6381 c348 970f 814d c3f1 5697 b531 b2e2  c..H...M..V..1..
00000050: 1c65 f276 5eed dbbd 186d e186 bcaf 948b  .e.v^....m......
00000060: f272 e182 82d8 c4e4 f767 3799 bf07 f8cd  .r.......g7.....
00000070: abaf e9a4 6aa0 dccb 6f81 c1a7 9dba 3d0a  ....j...o.....=.
00000080: e08e 0ee3 c562 2d64 b858 4fe8 30b1 c9ff  .....b-d.XO.0...
00000090: 9038 16a3 725b 3aa0 8baa 4ee6 bc08 676f  .8..r[:...N...go
000000a0: e971 cd3a 6e99 df2a f06d 1df0 7b6c 8a45  .q.:n..*.m..{l.E
000000b0: a258 a1d8 0ac9 c43f 1c84 2021 99f6 3655  .X.....?.. !..6U
000000c0: 97ee ad15 9f89 8b03 2fdf 75b2 0547 7915  ......../.u..Gy.
000000d0: d58d bdb0 6591 e94d a39e 703e 0854 89b8  ....e..M..p>.T..
000000e0: 1b22 731c 1cdf f98a f93c 9003 cd3d 2866  ."s......<...=(f
000000f0: 2ef7 1bc0 4d78 f9b7 57ba 4f4d 25bb 22b3  ....Mx..W.OM%.".
00000100: e582 0cb6 f0d3 c16d 89af 10ae 3e03 1d9e  .......m....>...
00000110: e5fe 500f 9a40 7e15 0580 9cdb 6d97 0ac3  ..P..@~.....m...
00000120: 59fd 5e15 c1ec 45e4 f4f4 f443 8c4c f9ce  Y.^...E....C.L..
00000130: 2361 7399 5dc8 92c1 dfac 93bb 0605 ea28  #as.]..........(
00000140: 7190 6a06 c007 e5d4 a29b 5692 10c8 e6fb  q.j.......V.....
00000150: 2001 b7be d7da 8a4e 8c70 87b6 b0ef 798c   ......N.p....y.
00000160: be82 4f3c a4ce 57b7 5a6c cda2 a35d a840  ..O<..W.Zl...].@
00000170: b788 229a 579e bf1d 0805 db05 344b e090  ..".W.......4K..
00000180: 849f bcee 891d 9466 76e2 f88d 8b66 5cac  .......fv....f\.
00000190: c258 a75f 09aa 97a2 33cf 5738 2cb1 190d  .X._....3.W8,...
000001a0: 82d8 2065 71fb 44c5 1aaa c331 1253 b6c9  .. eq.D....1.S..
000001b0: fe1c 4202 0850 111d 4c4a 4906 6197 5e30  ..B..P..LJI.a.^0
000001c0: b284 bc5f 016b e779 61c6 4a82 b44a 734c  ..._.k.ya.J..JsL
000001d0: eb51 d7c0 25e7 4b9b e590 a6dd 9ba4 2588  .Q..%.K.......%.
000001e0: 22fb 0299 0e32 295c 7470 f9d8 3190 58b0  "....2)\tp..1.X.
000001f0: 2653 6042 79aa e15d 1d5a d7b7 455e 1111  &S`By..].Z..E^..
00000200: bbcf 4453 b019 3dc3 8444 e71a b1df 788c  ..DS..=..D....x.
00000210: 957e 5415 0b49 9237 da62 445c 6c05 eead  .~T..I.7.bD\l...
00000220: 2475 ec4c e05a 7db6 f073 5a74 ae26 bb7e  $u.L.Z}..sZt.&.~
00000230: b26b ab18 af9f a465 abef fd09 e8b4 bfb2  .k.....e........
00000240: 611b dcf7 c1e3 1455 1675 64f3 4b6b 4aec  a......U.ud.KkJ.
00000250: 93b7 f383 ca6b c146 d10b 9eb3 e4fd 604c  .....k.F......`L
00000260: 13e4 7110 f3d0 8b51 e2f5 b5cc bb02 5497  ..q....Q......T.
00000270: 42b3 7e12 7f8c fb47 93b8 5d49 afe2 41af  B.~....G..]I..A.
00000280: 295d 7592 3f54 2732 29ca 07ae 1184 32a0  )]u.?T'2).....2.
00000290: b9b2 9f89 6a09 1479 d7eb 4a64 8534 286a  ....j..y..Jd.4(j
000002a0: 06f8 a126 d861 4651 7df4 89d2 2f23 daff  ...&.aFQ}.../#..
000002b0: b7ef 92e7 3850 041d c8ba fbb2 43c9 c23c  ....8P......C..<
000002c0: a936 1a75 344e a865 f1b7 aad7 bc55 54b1  .6.u4N.e.....UT.
000002d0: bf12 37dc ca35 1dd1 5c2b bad4 d4d3 5fd3  ..7..5..\+...._.
000002e0: 5169 1d1f edb3 8529 764d 8285 a1e6 c3a5  Qi.....)vM......
000002f0: c06b 7693 a18d 0dd7 b375 9fc2 05a2 f458  .kv......u.....X
00000300: eeb9 63bd d47a 1d54 d85f 9305 6976 9ca3  ..c..z.T._..iv..
00000310: c9bc e4dd 8076 7c7f f5b7 f02b 4cb1 15eb  .....v|....+L...
00000320: 5c30 130a 9ea2 ee25 6fcf b29c 1777 a4cb  \0.....%o....w..
00000330: f28e c764 307f e656 3f3b 8c63 46b4 b75c  ...d0..V?;.cF..\
00000340: 7ab7 6e44 8b9b a251 72ae 2ab1 212b 10b7  z.nD...Qr.*.!+..
00000350: 755e 03cf a277 c3f8 c35d 92db 6f7b 05e1  u^...w...]..o{..
00000360: 963e 28ee 611c 5f9d 6bea 409a 904a 0a40  .>(.a._.k.@..J.@
00000370: 7c60 abe6 f89c b1de e949 78ad 8d0d 1da1  |`.......Ix.....
00000380: 2590 b717 f056 1caa f993 b149 449d 8297  %....V.....ID...
00000390: b351 8b11 dfdd 6851 4ff6 5b57 0723 0393  .Q....hQO.[W.#..
000003a0: ec71 1642 a6ad c32a 7390 bcc5 9396 c53e  .q.B...*s......>
000003b0: 75e1 d341 9c0e d64a 3f34 47e6 71f5 1a77  u..A...J?4G.q..w
000003c0: 42c1 6476 2ef0 7185 ffdd fceb 806f 93bf  B.dv..q......o..
000003d0: cb92 4d32 c6dc b346 e8fd 0371 492c 42cb  ..M2...F...qI,B.
000003e0: 3c1b 7351 a5d6 29e0 b2aa 3e69 8563 01b7  <.sQ..)...>i.c..
000003f0: 87f3 6dfa 797d 8132 a385 b0d4 4e3f b76d  ..m.y}.2....N?.m
00000400: 1159 8dad 14c9 0f66 dc48 22b3 b3ef d254  .Y.....f.H"....T
00000410: 3c27 a4a8 065b be28 9e10 b1eb 4587 2d50  <'...[.(....E.-P
00000420: 1a60 efa4 0a9f 3911 317e 17aa 260f b3aa  .`....9.1~..&...
00000430: 66bf 4b1d 3538 5974 cfd3 2a28 52fd 5c01  f.K.58Yt..*(R.\.
00000440: 9dfe fe25 a31d 7169 6b6a f8cd b04f 53e1  ...%..qikj...OS.
00000450: 49b1 1dac 52c0 69a2 d1ee 2189 1f78 2f4f  I...R.i...!..x/O
00000460: feb6 0a0a c1fe 4ada 8110 7863 6648 ca73  ......J...xcfH.s
00000470: 6caa 085e 20bf 95c5 a6ae 11f3 7c9a 3852  l..^ .......|.8R
00000480: 410b 0c45 a58b 9f9a 403c 9c4a e626 410a  A..E....@<.J.&A.
00000490: 6a0c abd9 4e28 8357 0a02 8abe f0de d3e4  j...N(.W........
000004a0: 6139 18f0 6bba d044 88cb 80bc 9dfd 020a  a9..k..D........
000004b0: 28d2 5103 74d8 18f7 3554 f868 0a6d 0aa8  (.Q.t...5T.h.m..
000004c0: 9427 1bef dc43 d63d bff8 5ec4 4197 bc34  .'...C.=..^.A..4
000004d0: 1fbf 7d12 1795 31e2 b138 e28d 1247 5d36  ..}...1..8...G]6
000004e0: a231 d580 4068 5f29 0b7f 6df1 99af a7e6  .1..@h_)..m.....
000004f0: 1a23 91c0 6ed8 ae38 36e2 21c3 1232 87db  .#..n..86.!..2..
00000500: 67f4 38ad d516 50e7 48fc 2f41 a729 68f6  g.8...P.H./A.)h.
00000510: 9273 862d 5ede 3f7d a7dc 7a07 2e80 2d64  .s.-^.?}..z...-d
00000520: 2afc 261d 1df8 746d 4842 6bb6 b1f6 5a8e  *.&...tmHBk...Z.
00000530: d920 6d17 5faf f05c 6e8e ec7d 3e95 7edf  . m._..\n..}>.~.
00000540: f39c 853f 9b03 497c e44f 4811 da97 2cd3  ...?..I|.OH...,.
00000550: 9318 576d 30cd b576 d47b fca8 2852 e2e8  ..Wm0..v.{..(R..
00000560: ad7a 00ad 44ac ae88 ba27 dc6d 2f8a f1ba  .z..D....'.m/...
00000570: 9888 6dd5 45f5 6758 dbf8 b683 90d2 fd2a  ..m.E.gX.......*
00000580: 5d72 66f7 d30e 60d3 95dc 5a85 a6e6 d5e9  ]rf...`...Z.....
00000590: c005 e414 fdaa 6c80 3d8f d077 969b c48a  ......l.=..w....
000005a0: dca3 565e 61f7 a2fb 4c3a 6154 8ed4 10c4  ..V^a...L:aT....
000005b0: b598 c201 7de7 0f00 1263 dbe3 af81 1bdf  ....}....c......
000005c0: 1099 5a99 e689 1739 e442 5b4d 9a97 6988  ..Z....9.B[M..i.
000005d0: ecbc f968 777a bd12 05ca 325b 2298 8db5  ...hwz....2["...
000005e0: 88fb e671 d114 4907 5a99 a631 18cd a5dd  ...q..I.Z..1....
000005f0: eca7 726e c565 9926 b519 e229 1858 5aa9  ..rn.e.&...).XZ.
00000600: 594e 3b43 49a7 8809 7efd 09eb 8bc6 4df8  YN;CI...~.....M.
00000610: 6633 264a ec32 353f 38de 5aa0 a888 190c  f3&J.25?8.Z.....
00000620: 7cbd e2b9 8af1 4ba0 0f74 13a2 ffe2 59b3  |.....K..t....Y.
00000630: b62f 13a7 cfd8 72f5 1449 77e4 4468 8d9e  ./....r..Iw.Dh..
00000640: 7136 e03b 04cb eb9d 6fab 9dde 8498 afe6  q6.;....o.......
00000650: a2ec bb0c 3c7e 179b 61bd 0b24 7f22 a987  ....<~..a..$."..
00000660: d4a7 6c2c 725a 6520 1ccb 6ec1 0ae1 b962  ..l,rZe ..n....b
00000670: a6b3 b871 afd7 5875 1b08 efc8 1a32 39f4  ...q..Xu.....29.
00000680: 1c55 6e4a e787 f1af 621c af0b cd33 6273  .UnJ....b....3bs
00000690: b407 2739 5e89 cb5f b203 732a 46b1 4bd6  ..'9^.._..s*F.K.
000006a0: 4078 4d04 05d4 5004 db23 6b0a dffe 8e32  @xM...P..#k....2
000006b0: 1426 41ca 868f ac73 e26a ef99 8c58 f30a  .&A....s.j...X..
000006c0: fae7 43eb 6ccf 0af5 e247 4697 35bc 4d23  ..C.l....GF.5.M#
000006d0: 9f02 dc70 45bc 07b1 0b6c a4c7 2bd4 1927  ...pE....l..+..'
000006e0: bca5 0c8c 0a8d a63a 6e26 e395 9ff3 b87e  .......:n&.....~
000006f0: 728c 92a3 86a4 2340 e9ac ef6f d4a0 2d9f  r.....#@...o..-.
00000700: e6b3 6766 9d3f 6578 f01b 346b 9e22 a9e7  ..gf.?ex..4k."..
00000710: 841a 4741 ed09 0a6c bc55 963e d39b 70b4  ..GA...l.U.>..p.
00000720: 5e97 7adb cd56 32a7 2b63 79f9 454c e32c  ^.z..V2.+cy.EL.,
00000730: 7ff3 b90f 6f56 2a44 a9b2 0db2 d5a7 2797  ....oV*D......'.
00000740: 8b79 f2b7 91b6 ed97 02c7 1a21 3552 8614  .y.........!5R..
00000750: 020b 5d24 34a1 3f87 f353 a900 87f1 a97a  ..]$4.?..S.....z
00000760: e747 724d 3f86 dfbe efd5 427d 8e77 98b1  .GrM?.....B}.w..
00000770: 9010 6f20 d124 6b38 3203 5e00 42a4 ad86  ..o .$k82.^.B...
00000780: d2cf d5b5 613b 9204 4f0b 9b1b f1ca f2e8  ....a;..O.......
00000790: 01b2 7482 e896 fa1d 98cc e997 a3a9 0eb9  ..t.............
000007a0: 3d87 1e87 44e3 b451 4619 b789 4b0f 3b25  =...D..QF...K.;%
000007b0: 7a96 6f0a 94eb 5f4d f620 fdb5 3b8e dd0f  z.o..._M. ..;...
000007c0: 978d e2a9 a6bf 1e9f 8287 8b70 03bb 8037  ...........p...7
000007d0: 65cb fe01 4578 0a90 0ab7 456c e2c8 e4a4  e...Ex....El....
000007e0: 1e29 ef75 b42e 2d30 d854 32f6 739c 09b5  .).u..-0.T2.s...
000007f0: 7317 2d3e 9a46 64ed f0ba aa3c e15a 18e1  s.->.Fd....<.Z..
00000800: c59a b8f5 2fe0 28e2 2933 4304 d19e 1363  ..../.(.)3C....c
00000810: 20a8 dbd3 c179 3190 4f41 aaa9 8f24 5dc2   ....y1.OA...$].
00000820: 95a5 0b84 8dfa e0b0 6e0e 58af cbcf eb6b  ........n.X....k
00000830: 93d1 ed95 51f4 16fd c651 3c33 7c23 e213  ....Q....Q<3|#..
00000840: b853 81a3 69d3 6162 4ede 43c4 cedf f565  .S..i.abN.C....e
00000850: ebd7 7568 1db0 027a f843 75f1 6856 7a7c  ..uh...z.Cu.hVz|
00000860: 61e5 31ba 549e 40d7 1502 380a 5ce1 7866  a.1.T.@...8.\.xf
00000870: a0f2 3e32 5083 1603 2de2 d49e a979 6f8d  ..>2P...-....yo.
00000880: a866 161f 051a 8bf6 f022 f4b4 c8c5 05bd  .f......."......
00000890: b603 4658 cdcd 3ae9 20c0 e9df 41be 9f32  ..FX..:. ...A..2
000008a0: ac06 88b0 df88 26ac 0142 133c c3b4 cdae  ......&..B.<....
000008b0: d7e1 2fc3 0bba 6b28 f12a 5467 e8b3 ab4d  ../...k(.*Tg...M
000008c0: 1179 a155 2518 9bae 9b93 69a8 72ce 6c6c  .y.U%.....i.r.ll
000008d0: 3958 f11e 7ecc 2aa3 5fcd 3751 9af8 a950  9X..~.*._.7Q...P
000008e0: db54 54f8 1f3a 73ed f0ef 1c64 b9b0 35e2  .TT..:s....d..5.
000008f0: 788f 6506 0c84 a0c2 11d1 910d cdf9 f4e0  x.e.............
00000900: df5a 4575 b669 dd3b 3792 c8d2 f3fd 2ee0  .ZEu.i.;7.......
00000910: 9ca9 8a7f cd84 a43c 6183 6f5e daba 80ac  .......<a.o^....
00000920: f910 d2ab e7d9 85a2 3776 2c48 0ff2 9aee  ........7v,H....
00000930: d241 7690 a2b2 e009 b5dc 648b 8dbd 1d28  .Av.......d....(
00000940: 52aa 4a6f e669 f5c9 5ea0 fc22 66f8 8e34  R.Jo.i..^.."f..4
00000950: efcc c068 f0f7 e40e 0ef1 216e 31e8 c958  ...h......!n1..X
00000960: cab4 14ad ab34 3342 edf6 a3e7 a3ff 625c  .....43B......b\
00000970: 3646 62b8 93d8 611d 5559 3541 e1f1 1fa0  6Fb...a.UY5A....
00000980: c287 2790 5037 7317 d21d 1acc 344a cef8  ..'.P7s.....4J..
00000990: dcc3 a931 5fa0 970b f208 244a c775 f50e  ...1_.....$J.u..
000009a0: bde4 a9d9 0daa a32b 05d7 1ea3 219a d28a  .......+....!...
000009b0: aa64 e4cf ce3d 9d52 91d9 46a0 77b1 ccea  .d...=.R..F.w...
000009c0: e8c3 fe1a dd71 478e a21c 4ba9 939b 2a7a  .....qG...K...*z
000009d0: 9f86 14f8 3de8 29c1 7a8f 2f24 2f17 e457  ....=.).z./$/..W
000009e0: 1a47 204d 7412 ce16 5f74 29e6 7b53 de91  .G Mt..._t).{S..
000009f0: 0d6b fab8 fb7b 6f9d f0a2 ecce 09ef 02cb  .k...{o.........
00000a00: 5719 4568 8a5b 738d 0c51 f5a2 d007 35a7  W.Eh.[s..Q....5.
00000a10: d115 e5c4 5f76 e3c8 78df a8b4 5981 f15a  ...._v..x...Y..Z
00000a20: f11b 84c5 11dc 1fd3 3ffa 313b 9386 1078  ........?.1;...x
00000a30: e87c 6c30 b463 690b 7dae 8efc 8c5c 971d  .|l0.ci.}....\..
00000a40: 1b78 5e7a d9e0 07ec 9ce4 f64c 3bfa 442d  .x^z.......L;.D-
00000a50: 14c2 5ff7 9600 19e9 6978 b00a 9579 5806  .._.....ix...yX.
00000a60: 891f f9dd 276b 11e1 bb4d 6b6c 01d4 68f8  ....'k...Mkl..h.
00000a70: c262 e2bc 5e85 2601 a763 9e52 3b1a 3a84  .b..^.&..c.R;.:.
00000a80: b768 8abe ded6 8832 e298 86e7 7a05 b922  .h.....2....z.."
00000a90: 75e8 ca64 bbbd eebb 7055 66bd 2566 d091  u..d....pUf.%f..
00000aa0: de51 9aa7 7b8e e1fe a7e5 d334 ec7d 4e64  .Q..{......4.}Nd
00000ab0: 6d51 6970 c149 31de 8e2e f6b3 4a80 9d8c  mQip.I1.....J...
00000ac0: c2db 2e9b c52a 5d62 9aff 1364 7af4 57f2  .....*]b...dz.W.
00000ad0: 49d6 0e8c bae5 5d7c d72b b52a 9b0c 5d7f  I.....]|.+.*..].
00000ae0: 4daf 7c2c 969c b990 10ff 3b42 9c15 1179  M.|,......;B...y
00000af0: ab09 9362 ce2b 7039 6baa 6136 4bf2 bd38  ...b.+p9k.a6K..8
00000b00: abb3 438b aa0f 279d 3971 2a3a 3e3f d78a  ..C...'.9q*:>?..
00000b10: 30d9 570d d1b3 6a9d 64c9 7d15 c230 f711  0.W...j.d.}..0..
00000b20: 71f7 c7cb d2f3 94d2 8531 5758 83ee 39b0  q........1WX..9.
00000b30: 0a01 d828 c187 e8d5 793b 54b3 d4a4 f268  ...(....y;T....h
00000b40: 3189 298a 0999 a2ae 0dd9 edaf 0c01 7baa  1.)...........{.
00000b50: ed13 f780 988a ee0a 75d6 8ce3 5611 3cf4  ........u...V.<.
00000b60: 7793 8cad 93d6 b2d4 3973 32a0 c7c6 1fe8  w.......9s2.....
00000b70: 4b2e 6fce fe7d 060f 7c93 2dc4 dcf3 e713  K.o..}..|.-.....
00000b80: 6eb4 797d f178 1dde 6f10 0d2b 20f6 b82f  n.y}.x..o..+ ../
00000b90: 9444 53ce ecfb 3f1b 0ee9 0daf a103 f5cd  .DS...?.........
00000ba0: c730 95e4 a981 8413 e95f 1b64 e996 8c51  .0......._.d...Q
00000bb0: 19e6 8e1f 41a7 a7e4 16c8 71d3 ab96 4861  ....A.....q...Ha
00000bc0: 8aec 4416 c44d fbdf 6632 e4e4 6ff2 6f9e  ..D..M..f2..o.o.
00000bd0: 6c41 a68c bd8a ec5e 047c 11bd 4756 d72b  lA.....^.|..GV.+
00000be0: ba77 649f fd8e 9f8b 2f8a 83d3 22cb c8fd  .wd...../..."...
00000bf0: 10db 3f54 2020 5430 6e93 7307 92f9 79c5  ..?T  T0n.s...y.
00000c00: 8171 44f0 34a7 9f4f 3827 835a 1dea 1191  .qD.4..O8'.Z....
00000c10: 3e92 fccc 93d2 17e4 632c fbe3 a816 1c2b  >.......c,.....+
00000c20: 09f5 eaef 5735 849e 732e 6ef5 fa15 3b37  ....W5..s.n...;7
00000c30: 1de7 f246 5912 e39f a27b b8a6 ef3d 3047  ...FY....{...=0G
00000c40: 20ef 5fc0 9e36 5ce2 3cc3 3d72 d824 0acd   ._..6\.<.=r.$..
00000c50: 8e12 822e c40c 88c5 d4ee 3185 5b3b fb78  ..........1.[;.x
00000c60: 4f26 fb35 776a 84af ce9c be85 76e7 e480  O&.5wj......v...
00000c70: c7eb 5fff 2884 07d4 2794 fbf1 0096 957b  .._.(...'......{
00000c80: 35b0 fec9 a038 4ee3 d212 6444 4c0d 8457  5....8N...dDL..W
00000c90: 43f1 4398 872a 1173 cc96 e5ba 0da9 760c  C.C..*.s......v.
00000ca0: f6d0 859c d725 685e 0b80 0000 0300 0016  .....%h^........
00000cb0: 50                                       P

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=16896
pts_time=1.375000
dts=16384
dts_time=1.333333
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=2345
pos=40820
flags=__
data=
00000000: 0000 0925 019e df44 5700 0003 0000 06f8  ...%...DW.......
00000010: f117 5aaa 1c99 bf00 9833 4cd7 8c05 83bf  ..Z......3L.....
00000020: c88d 4bac 91e7 461f 9b3a c059 01f7 8606  ..K...F..:.Y....
00000030: aa49 5f97 4947 ff4b b9f7 e1b9 0698 e340  .I_.IG.K.......@
00000040: 8614 e936 43e5 39bd 62d6 4271 ee02 4067  ...6C.9.b.Bq..@g
00000050: c287 40ba b5be 6bcd 57d9 de56 30d2 10dd  ..@...k.W..V0...
00000060: 5b41 27da 3603 8bf5 cbd8 9b9b e640 77cd  [A'.6........@w.
00000070: 76e7 4bd0 b1c6 25b8 69d5 f7fa 45b9 f0fb  v.K...%.i...E...
00000080: c09a 61e3 4d1f 5cc5 6ed1 22c0 0f1b c409  ..a.M.\.n.".....
00000090: b2b5 fe33 a165 1ba9 e048 91c8 87c3 382f  ...3.e...H....8/
000000a0: 3e74 5d23 6ce5 e335 f9d0 65d5 6cd3 f769  >t]#l..5..e.l..i
000000b0: 7c12 60d8 0f22 d755 25b7 c33b 6cf7 53f7  |.`..".U%..;l.S.
000000c0: 35b8 f693 0975 15b9 ad43 9e96 449c 3281  5....u...C..D.2.
000000d0: 46e1 5b4b 1f25 3f73 8f15 56b0 d780 3210  F.[K.%?s..V...2.
000000e0: 4e6b f6ba b8bf 25e9 c833 e063 899c 1119  Nk....%..3.c....
000000f0: 1e35 7afb f2c1 154b f490 9619 85c9 bb52  .5z....K.......R
00000100: 2135 fad1 7aba aa3a a7f5 e314 9aa0 30cb  !5..z..:......0.
00000110: 3739 4f5f e8b8 2b7c 203c 8119 52fc db0b  79O_..+| <..R...
00000120: c3c9 79ae c573 38e5 0f27 9ee7 96cb 7a37  ..y..s8..'....z7
00000130: eb41 094e b0b0 8915 f73b f6a9 71a2 bd71  .A.N.....;..q..q
00000140: 7172 bc11 aca0 fdb0 4955 a268 4258 8573  qr......IU.hBX.s
00000150: 99e8 3684 ffe6 d3b0 87ac 7ede efa6 32af  ..6.......~...2.
00000160: 90ec cc82 7b4e fcec 4e7e 4c43 ff40 ad43  ....{N..N~LC.@.C
00000170: e5e8 3cce 6440 d85b 7af2 5426 6558 f46e  ..<.d@.[z.T&eX.n
00000180: b796 be70 d5f1 96eb 8490 ab59 0f54 d0e4  ...p.......Y.T..
00000190: 28bb b702 bd1f 752e 9766 8bea aa74 0a09  (.....u..f...t..
000001a0: 3ac9 4641 7585 5a80 0693 f796 4c82 f0aa  :.FAu.Z.....L...
000001b0: e137 ac45 86f9 7dbf 4b35 484a aa58 b87f  .7.E..}.K5HJ.X..
000001c0: c629 94ac 2832 9741 d2a1 04ca d2c2 4448  .)..(2.A......DH
000001d0: fce7 19cf 6ff9 0c95 d23a 6934 7e12 42fa  ....o....:i4~.B.
000001e0: bbc5 d685 75ff 6f66 3285 ee84 91e2 3918  ....u.of2.....9.
000001f0: 3e21 db08 ac9d d5d5 fda4 f308 98f8 5e71  >!............^q
00000200: 843f 0c44 88e5 18b1 f346 be7c 0d9c 7826  .?.D.....F.|..x&
00000210: 04de 4760 0741 8a7c 9ba2 e6a5 6af9 8d2d  ..G`.A.|....j..-
00000220: dd19 2ddf ee2a 2ea3 a5bd 62fe 54d1 71ef  ..-..*....b.T.q.
00000230: 3c83 0859 0ef9 c288 c3cc 33b3 961b f4a2  <..Y......3.....
00000240: 4c16 dab0 a4d7 fa5f 8d0d 8305 52f8 9752  L......_....R..R
00000250: d699 5fa2 7b5c 3c24 4170 6712 89fb 546e  .._.{\<$Apg...Tn
00000260: 335e 9d43 9aa7 6e93 5fe8 e3fd 2bdd 97dd  3^.C..n._...+...
00000270: 53e6 04c7 855c df3c 24e6 4fde 44d5 7b3b  S....\.<$.O.D.{;
00000280: ff12 ef25 679d bf67 ed41 1ee0 c41e fbc5  ...%g..g.A......
00000290: 552f b216 b043 023e ddca 2eee 8758 475a  U/...C.>.....XGZ
000002a0: 92b2 6141 522f b84f 832c 1e2c 0639 d3a9  ..aAR/.O.,.,.9..
000002b0: 65d3 d7fb 53fd 24fc 78df 255c b754 29d5  e...S.$.x.%\.T).
000002c0: 741e 6258 95f0 8fb5 8d40 7621 1aaa a7d8  t.bX.....@v!....
000002d0: 6b1d bad8 720b df98 d6d5 3e64 5cf1 6f00  k...r.....>d\.o.
000002e0: c2ce c3c0 874a 1e88 75ee d7ea 43bd 6b1a  .....J..u...C.k.
000002f0: 3ff5 43d3 9347 34ee 3054 8502 9da9 fbc1  ?.C..G4.0T......
00000300: 53d9 96dd c5d3 6076 27c9 0dc6 a38e e8a3  S.....`v'.......
00000310: f410 eeaf 4532 5508 6996 c17a 5063 646f  ....E2U.i..zPcdo
00000320: 1fea fbb8 32b0 8fdc 60e9 a4df 43dd 33bd  ....2...`...C.3.
00000330: 8bb4 c1a7 77bc c61d 35a3 5fd3 c764 c43f  ....w...5._..d.?
00000340: 43a5 da7c b7e4 4d51 f0a0 d682 b0b9 1263  C..|..MQ.......c
00000350: 88fd 5649 46f6 fd9a e219 06b6 909f 9568  ..VIF..........h
00000360: 52b1 8273 8e66 39dc b134 a973 add9 8a66  R..s.f9..4.s...f
00000370: fa27 3b95 ca47 a9a2 ea40 482a bd6c 2c66  .';..G...@H*.l,f
00000380: d2c5 36ba ea4d 35f8 ca1b e738 7fd3 6112  ..6..M5....8..a.
00000390: e612 3988 3cd6 63ce 1f73 3cb8 655e 1cdd  ..9.<.c..s<.e^..
000003a0: dea8 2855 ef57 09d1 2e46 1dfe 86f0 ffc8  ..(U.W...F......
000003b0: 5df5 8e5c 3c62 b60a 7cc4 7c6f ba4d 5927  ]..\<b..|.|o.MY'
000003c0: 284c 2443 b847 c27f 2540 3439 72e4 c324  (L$C.G..%@49r..$
000003d0: 67f6 cd04 a1b1 d922 5083 f59d a37b 15e5  g......"P....{..
000003e0: 3065 3bbf 257d 93ab 3c1a 6806 2c5d 6573  0e;.%}..<.h.,]es
000003f0: 1783 2af5 42ff e007 5bd6 55a2 ce53 8528  ..*.B...[.U..S.(
00000400: cce9 c568 5056 7297 ee66 189a f305 2c8b  ...hPVr..f....,.
00000410: 55fe cf67 d1a0 50e0 a979 b93b 383d 5af5  U..g..P..y.;8=Z.
00000420: f60c 05e4 b452 43ec 1f86 f9de 408d 0176  .....RC.....@..v
00000430: 3706 6f90 7e3a bc6c 62e2 00b7 532a b9de  7.o.~:.lb...S*..
00000440: 4ffd ab7f ea93 54ac 7099 b101 9410 b1b7  O.....T.p.......
00000450: 8780 dd95 7700 5f23 a2a0 23a2 465f 2750  ....w._#..#.F_'P
00000460: b6ad 6a1e 32da 1242 fa9d cebb 427f 7fca  ..j.2..B....B...
00000470: b6ab d797 e287 a61e b73b 8b0f a247 dfeb  .........;...G..
00000480: decd 2375 6f40 343d 1e46 7bb0 ba2b 2565  ..#uo@4=.F{..+%e
00000490: 20b4 7d96 f77f a3b1 2e4b 6600 7a1b 332d   .}......Kf.z.3-
000004a0: 6731 3d8a 0f00 90a9 93ae 97e0 5e5d aa46  g1=.........^].F
000004b0: f35e 079a 504f e2ce 5127 f248 bc85 f85b  .^..PO..Q'.H...[
000004c0: cad9 edf7 bb87 a7c0 1ec7 e6e9 a3d9 9869  ...............i
000004d0: 78ca eca4 70c2 60d4 477f a012 6b78 914e  x...p.`.G...kx.N
000004e0: 1254 0b2f 691b bd2b fbfc 529e 870b a6f1  .T./i..+..R.....
000004f0: 79ea 39f3 7f8c 4288 775c 89d5 7177 d1f0  y.9...B.w\..qw..
00000500: a421 454c d743 e399 8705 03d1 0237 f2f7  .!EL.C.......7..
00000510: d035 42d3 70ba 9285 1ff9 8634 ba87 433b  .5B.p......4..C;
00000520: b268 c115 3095 d098 9906 47f8 7b9e ac2a  .h..0.....G.{..*
00000530: 764f a147 ca2a bd98 89d1 128c fba8 1720  vO.G.*.........
00000540: 576d 1601 ac1e d932 e5e2 9df0 f578 3004  Wm.....2.....x0.
00000550: 62e9 5ac3 fac2 7323 3d1a b334 d49e 9508  b.Z...s#=..4....
00000560: a456 f942 dc16 4f20 4de2 90ea 9974 62af  .V.B..O M....tb.
00000570: fdc5 b47a 5415 abd7 ee19 9e24 76e6 342e  ...zT......$v.4.
00000580: a652 020f e16d 0b91 49a6 69fb 143c 5dd6  .R...m..I.i..<].
00000590: e450 3794 cb20 57b2 98cf 801f d698 5719  .P7.. W.......W.
000005a0: 29ba 7b45 26e2 3851 fc7b f6cb 4bef 2643  ).{E&.8Q.{..K.&C
000005b0: 2d42 6bd1 780f 8eeb e404 5482 ec48 1333  -Bk.x.....T..H.3
000005c0: 23e7 2878 7aef 3e78 5969 2094 9cad ff5e  #.(xz.>xYi ....^
000005d0: c778 fb0b 5e31 3a3e eb54 8f92 81d8 dfaa  .x..^1:>.T......
000005e0: 89b7 2150 1fe1 aa07 1dd1 f49f cfd4 bdb9  ..!P............
000005f0: 2d5b 0aa9 5e0b 7885 4324 c2f3 fc07 382c  -[..^.x.C$....8,
00000600: b696 7fc9 266c 781d b45d 809b 49a4 1c26  ....&lx..]..I..&
00000610: 7a80 7f9b 3568 e068 d01b ebf8 a701 f095  z...5h.h........
00000620: a533 6c21 e914 c2dd 39d7 0f34 91e2 40cb  .3l!....9..4..@.
00000630: b1be a4e6 f9b6 821b 41da 7dcd 8cf2 5f1a  ........A.}..._.
00000640: c503 2960 1d93 c162 98b1 be1e f86b 1d69  ..)`...b.....k.i
00000650: fb4b 5469 0349 f1c7 8a25 0c8f bea5 96f2  .KTi.I...%......
00000660: e2fc 9425 0e10 8397 ea7c fbf4 d212 d932  ...%.....|.....2
00000670: 8a53 8b89 f254 75f2 c8df 8f4d c00e f952  .S...Tu....M...R
00000680: 8b90 e12a d82d 7348 ed54 7680 2f75 f615  ...*.-sH.Tv./u..
00000690: 69d0 adc1 f4f7 6a31 183f 1d09 2ae8 7446  i.....j1.?..*.tF
000006a0: 17ca 7041 7e75 771c 83ea d5f7 a4c8 c825  ..pA~uw........%
000006b0: d3d2 b524 b092 7bc9 6579 1594 77db 6439  ...$..{.ey..w.d9
000006c0: 472d aeae 45cd 488e 75ff 2203 bdf6 e738  G-..E.H.u."....8
000006d0: bfd5 9569 af8d 5bbf ab03 b848 bfee e209  ...i..[....H....
000006e0: 09ff 86a5 5db4 6b17 c099 9ad8 e681 0cd5  ....].k.........
000006f0: e7ad 1589 8347 c31f 7f0e 9f53 4fb9 696b  .....G.....SO.ik
00000700: 1d8e 8cd8 146a 35fe e77b 9581 6433 5ffe  .....j5..{..d3_.
00000710: b9f2 c5a9 7c3a fdc5 7728 d78f af8d fbaf  ....|:..w(......
00000720: 5726 5e46 6bb0 b592 c06a a9d2 f4e1 5478  W&^Fk....j....Tx
00000730: 751d b6c4 4cfc 1d2d ecc5 85fc c5ee 212a  u...L..-......!*
00000740: a988 2344 822f 0945 a242 165a f7d7 b630  ..#D./.E.B.Z...0
00000750: d9e1 1b8c 6cad 71b6 f28f 4f2d 4e2a cfd1  ....l.q...O-N*..
00000760: e77b a80c 2a9d 11f9 b1fe 2031 5758 764f  .{..*..... 1WXvO
00000770: af30 ecb9 202c 357e 3b1f 6ccd a064 48f1  .0.. ,5~;.l..dH.
00000780: 4607 6e12 9c3a 1154 13fb 6bba 5187 af19  F.n..:.T..k.Q...
00000790: e8f3 3cca 88c7 75ba d61a dfe8 46d7 bd95  ..<...u.....F...
000007a0: 1dcf 6abd 7207 7de7 3578 a27a db4d fef5  ..j.r.}.5x.z.M..
000007b0: c0f7 b3e8 b374 6243 de96 e2c6 f123 e06e  .....tbC.....#.n
000007c0: 7c25 3ccd 6fe2 7766 e283 3e2b 4421 f98a  |%<.o.wf..>+D!..
000007d0: 375a 958d 19e4 4d2e 822f b183 abd7 1791  7Z....M../......
000007e0: d6ef 1052 0846 a34d aa7b e7e4 e03f e773  ...R.F.M.{...?.s
000007f0: 66c7 f5b8 79fb 5160 77e3 4e57 9f24 ea28  f...y.Q`w.NW.$.(
00000800: 893e 1dd9 0e64 6aa5 77ba 95d9 79c5 b104  .>...dj.w...y...
00000810: 5249 214b dc70 e11f a47c 0c0d 4f2c ac88  RI!K.p...|..O,..
00000820: fe17 3c81 ad33 fdfb 0482 b1ba 8f7e 6700  ..<..3.......~g.
00000830: 1662 980f 2efa 739d 4859 39d2 4bd9 2737  .b....s.HY9.K.'7
00000840: 20ff eb3a 6527 7ec1 3854 9d67 4b46 20d2   ..:e'~.8T.gKF .
00000850: 1a69 0fb1 d22b c8e5 f296 ccf9 4d76 8051  .i...+......Mv.Q
00000860: 5b35 6251 370f 0df3 0841 f82b ce93 a05f  [5bQ7....A.+..._
00000870: 7b92 edf3 0494 992c 2bc5 a321 0357 7e7d  {......,+..!.W~}
00000880: d049 2e47 fff3 ef90 045c 2f34 6cad e287  .I.G.....\/4l...
00000890: 924a 950a 1200 0172 4126 65db c318 52d3  .J.....rA&e...R.
000008a0: 5193 b261 6517 a2c0 aee4 7b49 8d26 c60a  Q..ae.....{I.&..
000008b0: cf4f f908 1b32 0b87 9c71 9e64 c1c5 aae9  .O...2...q.d....
000008c0: 753d 3bf5 da22 b709 afb4 c141 7c8e ba48  u=;..".....A|..H
000008d0: d48b 915c b7f8 cd03 06fd 6eb7 27fe afdb  ...\......n.'...
000008e0: b31f 2c96 e366 4b14 fe1b 60be 87e0 bbe4  ..,..fK...`.....
000008f0: 31ce daf3 a36f 0511 a6d6 eaff ec1d 033b  1....o.........;
00000900: 78a9 d755 61f2 2082 2c9d 1fec 4a45 8828  x..Ua. .,...JE.(
00000910: 9563 0c45 4aa0 627b b0a9 8713 a69f 72b0  .c.EJ.b{......r.
00000920: 0000 0300 0003 000c a9                   .........

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=18944
pts_time=1.541667
dts=16896
dts_time=1.375000
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=3505
pos=43165
flags=__
data=
00000000: 0000 0dad 419a c334 a43e 0248 a115 ff00  ....A..4.>.H....
00000010: 0003 0000 0300 197b 5d54 4a1a 3447 0de0  .......{]TJ.4G..
00000020: 1b86 25d8 8980 4691 1b37 23c6 3faa d937  ..%...F..7#.?..7
00000030: 666f be8d 6fb6 e721 ffc1 c722 14ae 9b56  fo..o..!..."...V
00000040: 2f70 9ac4 7b75 b495 aa27 f447 51f3 1701  /p..{u...'.GQ...
00000050: 2f38 cd31 f105 d14d 6ed7 3ecc da6c 9390  /8.1...Mn.>..l..
00000060: 3e78 e013 0800 0ca3 7c27 d9fb 0881 a8a9  >x......|'......
00000070: 9733 8067 6408 e65e feeb b098 6239 3acd  .3.gd..^....b9:.
00000080: 0748 cefd 67f1 14a3 552a fc24 c940 9e20  .H..g...U*.$.@.
00000090: 785c 0b3d f9d6 a7e2 4f08 43b8 bdf6 35bb  x\.=....O.C...5.
000000a0: dbc9 4fe1 70c0 ccf7 1d92 60b5 847b ce37  ..O.p.....`..{.7
000000b0: 6495 9c8f 86ee 135e e95e 3423 469f fcbc  d......^.^4#F...
000000c0: 956a 068f 6c36 0d86 f9a5 5b17 d3a7 fad8  .j..l6....[.....
000000d0: 2a87 6519 805c 2716 1f5e 5dc6 6a2c f01d  *.e..\'..^].j,..
000000e0: 00a9 3ded afd0 a6b0 d7ad 22a4 a1fd b651  ..=......."....Q
000000f0: d295 36c7 fb0e 6271 5f17 11ee c6b8 4f49  ..6...bq_.....OI
00000100: 1c3c 7f69 69e7 9951 3c4f 1911 468a 893e  .<.ii..Q<O..F..>
00000110: f661 b5a8 857a fa9a b678 464a d601 ae3e  .a...z...xFJ...>
00000120: 6ba9 79f9 ec2e a7cf 3eb5 aa2d 1cfa f0e5  k.y.....>..-....
00000130: 5c91 3bcf cecc 0196 873b b217 dd9d 5a50  \.;......;....ZP
00000140: e59c 0d22 2062 e506 3d2e 89eb 3d10 37f0  ..." b..=...=.7.
00000150: fff3 3981 65dd 0b05 a415 6400 43b2 031e  ..9.e.....d.C...
00000160: 0a66 95bd 0083 8a6d 9b7b be82 3c65 dad5  .f.....m.{..<e..
00000170: ec34 2d2b 5a71 348f 09ce d3f2 4f59 aa1d  .4-+Zq4.....OY..
00000180: 108e 0006 47bb 4c6f a39d 5111 aa15 5fc5  ....G.Lo..Q..._.
00000190: 4a01 ea65 827e e656 79c6 bc33 025d 53c2  J..e.~.Vy..3.]S.
000001a0: 3962 5f75 a437 b9b1 650a 72c5 1b93 54f1  9b_u.7..e.r...T.
000001b0: a51c 3fe6 fea7 d974 0c8b 3313 8302 fc58  ..?....t..3....X
000001c0: d03f 5374 087e 9bc9 ba1d dad8 80b0 1cdc  .?St.~..........
000001d0: 099e afbc e686 75a8 af2e f577 442e fb68  ......u....wD..h
000001e0: a4f8 dbf0 d566 3422 d4ee 7eae 2929 ed7b  .....f4"..~.)).{
000001f0: b04e b64b b372 c989 a8c3 2da2 eb3a 8829  .N.K.r....-..:.)
00000200: 5178 2198 4d4d 5fed 6ad3 e816 e5f4 0555  Qx!.MM_.j......U
00000210: 1a6b 81ee e8aa 4a3c 79b2 36dd d2fe 9bc6  .k....J<y.6.....
00000220: f0c3 d701 0068 f704 dd42 814e 7ab6 0c90  .....h...B.Nz...
00000230: f788 ba38 571e 199a 3546 8ac2 8543 b17b  ...8W...5F...C.{
00000240: cad1 9f46 7d12 32db ac03 2f47 a079 4dca  ...F}.2.../G.yM.
00000250: 651a 5f49 d30f b592 3fa6 f61e d1fd ddeb  e._I....?.......
00000260: 98b9 ff47 952d a098 127d 8a49 6f2b 2301  ...G.-...}.Io+#.
00000270: e341 a78e 378b 4dde c06e 6d87 e932 fded  .A..7.M..nm..2..
00000280: 74ed 19e8 e274 4b15 4426 d150 ee6d 5618  t....tK.D&.P.mV.
00000290: 283f 1aa1 8124 2761 d8df f364 d5bc fbab  (?...$'a...d....
000002a0: b9bb 8e2f edb8 53ed c078 167b d694 f0da  .../..S..x.{....
000002b0: 8f27 01b1 c1ea cc01 291c 9d56 68d0 45a7  .'......)..Vh.E.
000002c0: 0fbf 2350 50ea 282d 0720 f0ce 5fcd 7203  ..#PP.(-. .._.r.
000002d0: 746a b0b1 eb30 ccce 93e6 27bf b54b 1b4c  tj...0....'..K.L
000002e0: aadd e2ff bc9d c91f 6cdb f31a 958b e809  ........l.......
000002f0: a44e f564 ba8d 1c74 3279 1d46 15d7 8c85  .N.d...t2y.F....
00000300: 306c 6cff 0245 0692 793e c1dd 442d 2cf3  0ll..E..y>..D-,.
00000310: 6941 9e01 eaf6 04f8 2e18 de91 ee5a 342d  iA...........Z4-
00000320: d30b c9f9 61ce 030c ce78 13ca 2c93 5fff  ....a....x..,._.
00000330: 3bc9 0299 e58a 244e a167 9056 d55a e097  ;.....$N.g.V.Z..
00000340: 3f1c ac3d 6a66 5673 05a1 e6eb f6eb 291f  ?..=jfVs......).
00000350: 06b8 af8d 1764 e297 b828 ac26 23f9 f600  .....d...(.&#...
00000360: 2e19 6be5 20f4 46d7 3c47 710b 396d c409  ..k. .F.<Gq.9m..
00000370: 0fed eec3 520a b641 7518 bc36 d376 2056  ....R..Au..6.v V
00000380: 9a57 040e ae33 f360 a5dd e2d7 20a2 cf4b  .W...3.`.... ..K
00000390: 36be 3788 dd4d f24d 24f8 4d96 f01f 6b67  6.7..M.M$.M...kg
000003a0: 6a98 64d1 437a 8b04 fdf7 f208 10d1 b10a  j.d.Cz..........
000003b0: d440 01b4 701d da62 dab6 aa4a dcb9 30a8  .@..p..b...J..0.
000003c0: 058c 2f65 492d d3e2 80ed 16c2 af1c ebc7  ../eI-..........
000003d0: 3801 2469 4dd7 a2dc 6fae 9970 e1f1 bdd6  8.$iM...o..p....
000003e0: 7bc6 0e79 4c6d 1374 d9aa d79d 0b68 5a2b  {..yLm.t.....hZ+
000003f0: bc55 5095 51db 7045 d183 380d 7798 2cac  .UP.Q.pE..8.w.,.
00000400: 13c7 6c2f cbe8 47cb 8684 e99d ab20 002e  ..l/..G...... ..
00000410: d17e c403 ecc7 1b67 a730 d115 37ba b675  .~.....g.0..7..u
00000420: 658a d969 20dc 9b9e 5b8e cbdb 60a7 65b9  e..i ...[...`.e.
00000430: d977 59d4 4c4e 5526 2867 ae28 cca2 311c  .wY.LNU&(g.(..1.
00000440: 5169 9f3c 2467 6765 a8a1 c576 e052 0760  Qi.<$gge...v.R.`
00000450: 1a93 f353 b81a b1de 6bdc 322c ff1f 607d  ...S....k.2,..`}
00000460: 6bf6 1e0b a472 79a8 5383 538a a25c 2ff2  k....ry.S.S..\/.
00000470: 3780 d708 91c3 dc2f 8681 e0e3 de40 190f  7....../.....@..
00000480: 21b8 2437 7400 93fb 9fa8 5aa7 ba55 894c  !.$7t.....Z..U.L
00000490: 1e16 883b a8dc 3222 00e7 31bb f868 dda9  ...;..2"..1..h..
000004a0: ddd9 ee77 7e01 00ee 441f 6297 7a41 57e7  ...w~...D.b.zAW.
000004b0: 82aa e824 2b5a c26f 83ed 563f 3672 75f0  ...$+Z.o..V?6ru.
000004c0: ac66 7373 7e95 3de2 a289 8d5d 4608 fb4e  .fss~.=....]F..N
000004d0: bf9e 84ee bf81 14f6 be3a ff0d e520 7ad8  .........:... z.
000004e0: 709b e182 128e 1438 fb17 8019 d4c2 e0ed  p......8........
000004f0: 6623 2304 5b59 163f 0237 eb54 ade2 8708  f##.[Y.?.7.T....
00000500: 45fd b669 ff44 69b1 1db1 26ed 1cb2 e5cd  E..i.Di...&.....
00000510: db58 680b bf83 2cbd 3422 26e6 2c4d 2493  .Xh...,.4"&.,M$.
00000520: b960 8e09 56b0 ccc6 a310 c754 e62e 1d06  .`..V......T....
00000530: f403 faca 922c 6853 54b7 6ee9 89cc 51b8  .....,hST.n...Q.
00000540: fb9c f96b 6f4b 6321 7aa0 a297 f41b 1e3b  ...koKc!z......;
00000550: ddf5 f0a7 cb1a bdec 6de5 f402 31b0 e20f  ........m...1...
00000560: 8cb9 3d69 ad04 d93c e7f6 42b6 d4a3 fdfd  ..=i...<..B.....
00000570: 3582 1101 a79d 2527 7426 18b8 3b47 c4a6  5.....%'t&..;G..
00000580: f323 09ea 80f5 cbaa 9f25 ce37 775c 1b7c  .#.......%.7w\.|
00000590: dcc9 3cc6 0087 4f9a b3c4 c175 ce7d 95f6  ..<...O....u.}..
000005a0: b213 7cad 2618 22c7 a38b 1560 7775 237b  ..|.&."....`wu#{
000005b0: c6db 4b31 381b b837 40a0 82d2 4188 9baa  ..K18..7@...A...
000005c0: 3a2c f7d0 d868 e013 a986 54ef ee6b d460  :,...h....T..k.`
000005d0: fa34 fdf5 4c42 60f7 d6e5 154a 8bab 4b87  .4..LB`....J..K.
000005e0: e34d 1488 c4af a509 5a86 7b26 5082 5e8b  .M......Z.{&P.^.
000005f0: fcbd 25ec 17a0 932f 2f07 42eb bf85 7b5a  ..%....//.B...{Z
00000600: 038a da31 6a7c 4ef8 01d5 2f87 b7ad 7e63  ...1j|N.../...~c
00000610: 2130 e2af 0305 ae51 dec7 53fb 04af e143  !0.....Q..S....C
00000620: fd4f d38d 48e7 9efc 743e 7e9c 5a40 43cc  .O..H...t>~.Z@C.
00000630: 097a 92ad 016c 0e6f 20bd d46d 2c64 4c1c  .z...l.o ..m,dL.
00000640: 4450 8d87 44c5 f07b 3fac 6fd7 aca9 9b90  DP..D..{?.o.....
00000650: f25e 153c edfe e403 81d9 a34d f960 4679  .^.<.......M.`Fy
00000660: 6d9b 7d81 9857 61b4 e224 435a 3087 ff6a  m.}..Wa..$CZ0..j
00000670: 7818 3b33 1eea 04c4 ebbf 2d15 3743 26c5  x.;3......-.7C&.
00000680: b79b d843 4efe ab11 f509 09d3 4154 57e6  ...CN.......ATW.
00000690: a28b c1bb a2d4 7494 32e7 8abf 4985 0a90  ......t.2...I...
000006a0: 68f4 623a d64e 05e2 d698 a836 5535 d361  h.b:.N.....6U5.a
000006b0: 818c c994 bae6 cf00 638b 511b bff7 3287  ........c.Q...2.
000006c0: beec 812e 2af5 5d32 4e35 9297 a343 38fb  ....*.]2N5...C8.
000006d0: 198f 81ad 8c2c e50f 5d11 8fbc bcba 4ac6  .....,..].....J.
000006e0: a3e3 b7d1 7ff0 87bb 2436 d75d 3ba9 84a5  ........$6.];...
000006f0: 5164 5d6d 28d5 c149 1c5f a13e f2d1 72cd  Qd]m(..I._.>..r.
00000700: 3689 1806 2d0d 768d 0004 9006 4527 3190  6...-.v.....E'1.
00000710: cf9c dddb c986 091b 09e0 774b 1132 c073  ..........wK.2.s
00000720: c13e 5a03 e276 3a7a f07d f4a2 879d 1cdb  .>Z..v:z.}......
00000730: 7729 6c3f a194 32fe 09cf 16c1 424c 614a  w)l?..2.....BLaJ
00000740: 2fb9 a579 ccc5 58cf f8f6 bb92 6fe0 5c70  /..y..X.....o.\p
00000750: 3be9 fd9f 7ede edca 8c4a c73e d505 048a  ;...~....J.>....
00000760: 552c 1d80 6bda 89b2 1453 914d 01f8 b0e9  U,..k....S.M....
00000770: 623c 1ecb 7c80 6ce7 d216 d2c4 ef02 19ab  b<..|.l.........
00000780: 6507 abe3 4a74 212e 42c2 109c 5ba8 2551  e...Jt!.B...[.%Q
00000790: afc8 dc13 22a5 83aa fee0 cdf7 68a7 f6a6  ....".......h...
000007a0: 3cc5 7b8a de83 0d62 e0a9 1fbf 4484 7448  <.{....b....D.tH
000007b0: 4c68 ad6d 5e26 7f01 a004 4c24 a71a af0c  Lh.m^&....L$....
000007c0: d52e 8b4c e5c9 9434 9942 dbd3 548b 348f  ...L...4.B..T.4.
000007d0: f386 80c9 5704 233e d251 ac99 959b dcdf  ....W.#>.Q......
000007e0: 9652 5170 11cf 58f7 7490 989c 3997 77c2  .RQp..X.t...9.w.
000007f0: dbc7 d07f c2e9 2ea3 bfa0 ac23 62c5 d077  ...........#b..w
00000800: 6d55 066f 5b49 13e2 4705 327f 093c b1b7  mU.o[I..G.2..<..
00000810: 9d67 cfee e98d eeef 1749 f286 d342 1de9  .g.......I...B..
00000820: 74da fa19 6e88 442d 0c14 887e 8704 0235  t...n.D-...~...5
00000830: 8cc7 9405 95fb dad0 adc3 1fd6 d964 bf13  .............d..
00000840: 2915 9437 5793 842d fb9b 5eb3 4f76 978f  )..7W..-..^.Ov..
00000850: a5dd 8dbb 1898 532a 85d1 d41f 3047 5a10  ......S*....0GZ.
00000860: 1368 81ed 8b49 f308 4123 7090 cb30 558c  .h...I..A#p..0U.
00000870: b567 d7e7 1b4d 8558 944d 28d7 640e 3f7c  .g...M.X.M(.d.?|
00000880: 0234 44da 863d 3118 f200 6f6b 743e e77e  .4D..=1...okt>.~
00000890: 33e9 6a07 15fe 821c 2206 2ad4 93ef 4877  3.j.....".*...Hw
000008a0: 9e4e a511 0514 ede5 6bed 6e3a 3c79 2073  .N......k.n:<y s
000008b0: fbd4 52ec 7207 3e71 368d cccb 6fee 01b5  ..R.r.>q6...o...
000008c0: abbb 60de 07a8 d8b8 9b81 2245 8e11 d05b  ..`......."E...[
000008d0: a8dc e36c 9543 3818 5940 6451 e890 7993  ...l.C8.Y@dQ..y.
000008e0: 751a e5b8 d90a a4f3 c3ad af8f 32cb 83ca  u...........2...
000008f0: 8e2b a0ca d646 a078 f4cb 2706 37c4 fe0d  .+...F.x..'.7...
00000900: 3329 8cf5 4bae 0986 a830 5afd a11e a7fe  3)..K....0Z.....
00000910: 4fbf 62bd 5011 c461 7eca a50c 64a0 12e9  O.b.P..a~...d...
00000920: 2d89 4b9a b58e d2af 9cc6 f6a7 8fca 7f72  -.K............r
00000930: 0cb5 14c4 feee 0c2f d198 a3f0 b729 52f0  ......./.....)R.
00000940: 7910 0e3f 62d1 6e85 01f0 583f 8046 bb55  y..?b.n...X?.F.U
00000950: 2f94 4b03 2e30 c84c 2149 cc2b 4290 d983  /.K..0.L!I.+B...
00000960: 8615 7920 7a84 c868 5159 b901 b95d ea54  ..y z..hQY...].T
00000970: b590 15d9 e542 5c0f 2dc3 160f f158 2264  .....B\.-....X"d
00000980: 49ef e894 e68d cea9 7021 4dc1 ae85 0c19  I.......p!M.....
00000990: d260 098b c73e 7d1a e7d8 ce53 808f ca1d  .`...>}....S....
000009a0: 1c1e dc15 07b0 b794 1957 b818 f4db e579  .........W.....y
000009b0: ee05 9835 e023 1420 6245 b29b 3087 f04a  ...5.#. bE..0..J
000009c0: 6b9a afbb 0014 9fce 3033 8460 b8c1 5217  k.......03.`..R.
000009d0: f272 9f37 32d0 00be 13f8 7e4c 8cc4 cc11  .r.72.....~L....
000009e0: b867 19fe e29f 4fde ad9d 83bf dc6a b1ce  .g....O......j..
000009f0: dc41 b9bd afb6 92f3 5cfd 82d8 6ab0 cd88  .A......\...j...
00000a00: 0f70 760f 7448 f574 9cc2 0c93 060b ed79  .pv.tH.t.......y
00000a10: ec63 2d3d 7ffa 7dad 53b3 0d79 5557 c7e3  .c-=..}.S..yUW..
00000a20: 7b32 e187 4f7a 0622 fb31 9981 bc6e 189d  {2..Oz.".1...n..
00000a30: 513f 7b7c ec84 ed73 5aec 8274 2d45 726c  Q?{|...sZ..t-Erl
00000a40: 9376 270a 7601 90c2 ff63 cf85 0920 1cab  .v'.v....c... ..
00000a50: 05c4 e3c3 4a90 0985 74bb 7494 76d4 2343  ....J...t.t.v.#C
00000a60: ff15 ab12 867e b7bc defd 8eac b007 6da6  .....~........m.
00000a70: d6c8 d135 7b8a 52f8 b9c5 3d0b 34d2 7ad0  ...5{.R...=.4.z.
00000a80: 18ad c40e 23fd 83de 30de d4b4 2854 5ca8  ....#...0...(T\.
00000a90: 6288 eabe af5d fffd 6eca 0d3b d7a6 e3a3  b....]..n..;....
00000aa0: 7ea1 249d 5c0a 33e2 a538 38aa 2495 ed76  ~.$.\.3..88.$..v
00000ab0: adcc 5f50 9313 f88d b805 978b bcb6 ec6c  .._P...........l
00000ac0: 5591 2ba6 8777 20da 5f3e 369a ab2c a9d7  U.+..w ._>6..,..
00000ad0: 4ca1 0027 8649 5697 4b7e cb5f 3817 9593  L..'.IV.K~._8...
00000ae0: 7dac bd7f f8d3 98ea 65d3 5c72 6f89 4df3  }.......e.\ro.M.
00000af0: 8c67 b17b 779e 131b f345 9d2d e1e3 4613  .g.{w....E.-..F.
00000b00: c2c9 1ae3 54c8 54f3 00be 0605 53a2 22be  ....T.T.....S.".
00000b10: 115f 04e9 5fae 6642 d7c6 6e5b c682 46ee  ._.._.fB..n[..F.
00000b20: abb8 bbc4 8fd8 5fdd 9002 cbfa 54b6 386c  ......_.....T.8l
00000b30: 229c 4fb1 125c b9a3 a706 0d2c 773b aac6  ".O..\.....,w;..
00000b40: 755b 2029 1f0c 6a2f 0291 7e9f e887 38c9  u[ )..j/..~...8.
00000b50: d0f3 dc55 95dc 35d0 d76b b60d dbc9 d204  ...U..5..k......
00000b60: 7726 c6d7 5b29 ceb0 92f1 7306 ba2d 823e  w&..[)....s..-.>
00000b70: a7f2 0b56 843f 7436 973a 8a41 c8bf 9e85  ...V.?t6.:.A....
00000b80: 4f56 2351 c35e 8c5e 1881 4341 f235 0dd3  OV#Q.^.^..CA.5..
00000b90: bb2e 1b0c 029d 0694 46a6 52ce fe66 1fba  ........F.R..f..
00000ba0: ffac b15a 44dc f40b bc6b b62a 4203 ce8a  ...ZD....k.*B...
00000bb0: 7465 db9c 74df 0648 8f9d 3fc9 ba02 efcf  te..t..H..?.....
00000bc0: 7d4b 36f5 d5b8 1d0b b746 c795 944e ff74  }K6......F...N.t
00000bd0: 89d5 e9b7 826e 0cdc 6eae 1e67 6b0d 12fd  .....n..n..gk...
00000be0: 31a1 736f 6344 e910 f907 3d4c 2c4f a6f4  1.socD....=L,O..
00000bf0: e40d 4d63 0a20 0ac4 014e 53f5 c789 d978  ..Mc. ...NS....x
00000c00: b77f d335 8c82 bf7e e1bb cc7d f2c2 f4ac  ...5...~...}....
00000c10: a28a da3c 1bdd 688a 034e 8593 eeb5 712f  ...<..h..N....q/
00000c20: 70a0 0674 4c8a 0992 d78f d80d 1fd1 526a  p..tL.........Rj
00000c30: 5a8c ff0d 9f98 c0d9 6300 c831 1af5 04b3  Z.......c..1....
00000c40: b46c 8d86 fcbb 6014 6c26 8fa7 9ca0 1a76  .l....`.l&.....v
00000c50: 6d4d d498 1ccd 88f8 fb83 eb00 b14c 062c  mM...........L.,
00000c60: 818c 88ae b68c f1b5 9791 ab30 f64d aef7  ...........0.M..
00000c70: 22ef eda2 093d 26aa c6cc 3f96 e3bf f5d7  "....=&...?.....
00000c80: 4936 0980 cef9 ffa2 c352 dfb9 638b 6638  I6.......R..c.f8
00000c90: efc8 271a 1598 f3ea 01ba 0de6 8bfd 6e2f  ..'...........n/
00000ca0: 2abe 81aa e877 1af3 db66 5852 c1d6 4460  *....w...fXR..D`
00000cb0: 7fa9 ea59 adbb b801 098f e674 07f6 59d7  ...Y.......t..Y.
00000cc0: d2fa bb3a 35e0 1bf9 5c00 f3ba 42ad 350d  ...:5...\...B.5.
00000cd0: c82a 2a62 104b 1a53 91e2 385c cde2 3559  .**b.K.S..8\..5Y
00000ce0: ad33 8ff5 b39a fd26 dae2 d241 34b9 f94e  .3.....&...A4..N
00000cf0: c595 28ad d423 2729 44b1 a74b ef93 118f  ..(..#')D..K....
00000d00: f442 b0be 2dbd f24b fc32 0361 3e8b e113  .B..-..K.2.a>...
00000d10: 201a 7e20 092a 7ff6 c9c1 a796 7c47 f5bd   .~ .*......|G..
00000d20: c9af dc8d e4d2 2836 76c5 2b0b fb75 d9be  ......(6v.+..u..
00000d30: 63bc f0b2 81f3 9ad1 9caa 946e e7ee 146b  c..........n...k
00000d40: b15f f4ab ac0c cea8 53d7 5531 19ab 9198  ._......S.U1....
00000d50: f15d 4f9a e46d 0cec 179a fcea 0046 17d3  .]O..m.......F..
00000d60: a2e9 af66 426e 9617 0227 3682 bc58 12ea  ...fBn...'6..X..
00000d70: 1e92 c0d6 d5b4 e464 96b0 1565 5967 73ac  .......d...eYgs.
00000d80: 622c 866e 5d9c 1683 64be bc28 d2b1 a6d5  b,.n]...d..(....
00000d90: 74fd b400 8611 1cac c0d9 db48 655e fe24  t..........He^.$
00000da0: f935 4edf 8554 6982 7ecb 07eb 25aa 1fb5  .5N..Ti.~...%...
00000db0: af                                       .

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=17920
pts_time=1.458333
dts=17408
dts_time=1.416667
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=3043
pos=46670
flags=__
data=
00000000: 0000 0bdf 419e e145 152c 5700 0003 0000  ....A..E.,W.....
00000010: 06c6 289e 247b 1c51 523f b525 71c2 3175  ..(.${.QR?.%q.1u
00000020: af63 7e0f 53ba a10e 68a9 9f36 60e6 6ef9  .c~.S...h..6`.n.
00000030: ce87 ba7c 04f3 78f0 831f 1a37 14d8 86d2  ...|..x....7....
00000040: 2f85 6ac5 266e 3b2b e4a4 f8f5 3dfc 0d6e  /.j.&n;+....=..n
00000050: 4a35 bc95 9287 18e0 13a4 2d40 16e5 308a  J5........-@..0.
00000060: 4ee4 37b1 06d9 1a96 4ed7 6005 8511 fb1f  N.7.....N.`.....
00000070: 06d3 bdc4 2250 dd92 61ad 5237 62db 52c3  ...."P..a.R7b.R.
00000080: 21f1 f6ac d259 cf67 6dab 2e63 ad77 1dd6  !....Y.gm..c.w..
00000090: 2507 6e3d a76d 161e 1601 319a 1811 b47e  %.n=.m....1....~
000000a0: 35a9 d569 cc64 35d9 8f37 1657 847d f191  5..i.d5..7.W.}..
000000b0: 6548 7482 c3f0 a06f 7dea 0597 a40c bb97  eHt....o}.......
000000c0: 80be d98c 63e9 d459 8875 5923 a7a2 3ca2  ....c..Y.uY#..<.
000000d0: 4115 d9a0 52f3 1230 00fd abfe 7bd0 0905  A...R..0....{...
000000e0: 7688 57f8 ace5 ee80 7db9 0caa c7fd 3d71  v.W.....}.....=q
000000f0: a78f ec9b 73a3 9469 de88 7c95 8fc3 1955  ....s..i..|....U
00000100: c815 a85c 4e74 fc01 8408 7d5d ba3e 7867  ...\Nt....}].>xg
00000110: ae31 186a 3dbd 8a6b 41cb 3c00 e06f 5431  .1.j=..kA.<..oT1
00000120: 707c 6188 fa23 9a68 f749 b528 2b26 8c3a  p|a..#.h.I.(+&.:
00000130: aaf6 d4a7 4333 5395 80f4 5686 ff8f 2f63  ....C3S...V.../c
00000140: 8ff1 f660 02e6 fd93 846c d01a c1d4 9cfc  ...`.....l......
00000150: c37e da18 cb29 01ce 07be 0efa cbc7 43d6  .~...)........C.
00000160: bad7 111d f3e6 1aa8 7661 a2a4 d278 df1f  ........va...x..
00000170: 0830 8cf0 1149 abe6 abab 45d2 09ef 4028  .0...I....E...@(
00000180: e14c e522 dcd4 89d0 155f b5c0 a17a 361d  .L."....._...z6.
00000190: c4bb 1adb 2167 a308 6dc2 b651 df25 c6c9  ....!g..m..Q.%..
000001a0: 97ef ceb7 65bb 346a 0e56 a41c 6034 6884  ....e.4j.V..`4h.
000001b0: 19e5 f704 5998 0e8a 223e 9832 375a df8e  ....Y...">.27Z..
000001c0: efd0 587f 1dac c97f ae09 3f54 432f 5783  ..X.......?TC/W.
000001d0: 02fd 7ea1 0162 6f8b dbb0 078f cfa9 6a59  ..~..bo.......jY
000001e0: 8541 09e3 c24f fe0e b74f 94c5 ed42 01af  .A...O...O...B..
000001f0: 8d95 4a96 cb3b c6cd 9790 0147 db3a 2f8a  ..J..;.....G.:/.
00000200: 1cb7 bbad 1806 17fe ebb3 b88f be50 152b  .............P.+
00000210: 6510 57fd 271d 27b7 c02d 258a 0c46 4a12  e.W.'.'..-%..FJ.
00000220: 4767 b588 e829 b7ba adf1 ba13 3867 9049  Gg...)......8g.I
00000230: 9d74 253f b881 6140 5ade 8e5e 3b10 bf78  .t%?..a@Z..^;..x
00000240: 1dee 9ac4 c853 e8f2 88b7 96f6 2ed9 6b2f  .....S........k/
00000250: ac2a 6d32 b849 5f2b 45f5 2e16 0404 c8a0  .*m2.I_+E.......
00000260: 6fa8 14f5 c034 2089 6b01 d52b fadb 04e6  o....4 .k..+....
00000270: 9666 878a b500 e601 f738 2f6a 07a2 07b9  .f.......8/j....
00000280: 154d 9142 378b 0999 61c2 31e3 cbb3 c296  .M.B7...a.1.....
00000290: 5204 2564 a426 25d9 aff5 9557 1ff8 8e65  R.%d.&%....W...e
000002a0: 6280 c957 9e2a f529 bf0d 7011 e692 5766  b..W.*.)..p...Wf
000002b0: 94d6 5801 b854 360c 25ba f75a 03ca 1f23  ..X..T6.%..Z...#
000002c0: a92a 47fc fb8f be2e 7b51 dd72 15aa 45fe  .*G.....{Q.r..E.
000002d0: 00b6 f5a3 027f 5d1f f3be b70d fdc5 bcba  ......].........
000002e0: 59ec 9669 40fe f7c6 5037 6fcb 739c 5791  Y..i@...P7o.s.W.
000002f0: a4b8 c9a4 7946 fcdf 626c ab6b 5f7a 7ff5  ....yF..bl.k_z..
00000300: 7b39 68d2 554d 1926 5cb6 c81a 56a5 dd32  {9h.UM.&\...V..2
00000310: 4883 eb0b 7b4f e624 e3df 53da a78b 5823  H...{O.$..S...X#
00000320: 6c2c 4a1f 7e0a ad71 83cc 71aa 57df 07a8  l,J.~..q..q.W...
00000330: a9d4 f9b6 f077 5a0e 29da e6e0 a206 8a00  .....wZ.).......
00000340: c73a 69fe 378e d73e fe22 ed87 a7c4 4c09  .:i.7..>."....L.
00000350: 2dac 12fd 90d7 7307 b5c2 ac7b cc0f 81c9  -.....s....{....
00000360: 99e2 4b0f 3ba3 def0 ad18 7727 43a9 2699  ..K.;.....w'C.&.
00000370: cdf7 9c5c ff55 891e e0b3 ab1d 24e1 1e95  ...\.U......$...
00000380: 9d26 6f95 e31b 46f2 a77e b10d 64b4 e749  .&o...F..~..d..I
00000390: c18f 4172 72be fb1f 47ca 1459 fe51 a1d0  ..Arr...G..Y.Q..
000003a0: 120a 072c d27c 9a54 9c1e f34a e7a2 2380  ...,.|.T...J..#.
000003b0: c888 7135 9868 e702 6492 bd02 7bd9 cdd3  ..q5.h..d...{...
000003c0: 81c2 f8c1 e4e1 7f73 07cb 3eeb e0d5 413c  .......s..>...A<
000003d0: 1547 1c8e 5678 0a6b d58c bfd2 7e5b ff06  .G..Vx.k....~[..
000003e0: 2bcf 4091 1a3b 8608 e876 8957 1e71 c7fb  +.@..;...v.W.q..
000003f0: e0fe 1c37 9808 27b8 c962 8726 515d 4c5e  ...7..'..b.&Q]L^
00000400: c067 1723 edeb 2448 a7f7 4c93 c041 7082  .g.#..$H..L..Ap.
00000410: 0a02 3382 45ef 0737 935e a198 82bd ecbf  ..3.E..7.^......
00000420: 9504 17fe fcb0 28c5 76aa 88f7 0712 37f5  ......(.v.....7.
00000430: 08ae 372b 9f54 f91e 075b e512 678a 88ca  ..7+.T...[..g...
00000440: d78d 6037 e137 d424 218e 9ee3 8dab 05ec  ..`7.7.$!.......
00000450: f94a 3d1b b869 df9d 1d8f f845 b1eb 2798  .J=..i.....E..'.
00000460: 7e79 6dce b1e4 9383 caf3 c77e be1a ecb1  ~ym........~....
00000470: 6600 3fa9 42ba 0ffd 7efd 5142 8064 7c18  f.?.B...~.QB.d|.
00000480: 31ea 3fbc 521e 7e39 f421 709f 6131 d7a8  1.?.R.~9.!p.a1..
00000490: 17f2 b084 516c c4e3 443b 7166 b79b 5011  ....Ql..D;qf..P.
000004a0: 7edf 1eff 281b d529 ad2d e881 d271 b8fb  ~...(..).-...q..
000004b0: bb9c 0a16 72ec 945b 0a90 de6e 0035 f57b  ....r..[...n.5.{
000004c0: 1744 88e8 2de0 9360 1b31 ab4a 0e95 b736  .D..-..`.1.J...6
000004d0: ed19 8224 9f8b 4292 aa04 c96a 68ae 4fe7  ...$..B....jh.O.
000004e0: 3373 086e 8aef 0e92 743a d51f eece 5e4c  3s.n....t:....^L
000004f0: a848 6430 6754 1936 6fd8 3728 28c3 e0a6  .Hd0gT.6o.7((...
00000500: f619 0d91 7d36 c02a 7842 385e c344 7259  ....}6.*xB8^.DrY
00000510: ef59 d662 fb3b 18c5 a91a 8a0d 33e6 a0fc  .Y.b.;......3...
00000520: b560 4f4e 9cad 36fd fdbb 6e56 793e ddb6  .`ON..6...nVy>..
00000530: bd8a 6b63 53ea 0c7e 8f26 bceb e8ce 08c6  ..kcS..~.&......
00000540: 2fd9 0489 4598 f70e 409d 3cf2 7ac0 0ec5  /...E...@.<.z...
00000550: 0be4 7aea 25ef 8859 4878 687f eaf6 3ac1  ..z.%..YHxh...:.
00000560: f55e 3bce d0bf 1c75 4104 ecdc 4cdb b339  .^;....uA...L..9
00000570: 37a6 d5ab debc 8baf 79fb ad35 4162 b117  7.......y..5Ab..
00000580: 9c4d eff0 08f4 5332 9ce4 9e91 6f90 c5e2  .M....S2....o...
00000590: 984a 8ae1 6bda 7ea5 156a c838 642f 45a1  .J..k.~..j.8d/E.
000005a0: 224f bb63 19d8 8f8a 13e8 d4c8 66ad c320  "O.c........f..
000005b0: d19e 7fed e462 f4e1 4cbe c503 37d2 191e  .....b..L...7...
000005c0: 8c65 2dd6 3bee 3134 8954 b1cb 15bc befe  .e-.;.14.T......
000005d0: 6e91 5497 2bd2 9346 4c10 ee61 4009 aa10  n.T.+..FL..a@...
000005e0: 4484 09a8 c3d6 7237 b6b1 8c30 dd7e e690  D.....r7...0.~..
000005f0: ef36 3158 95b9 14de ae45 10d4 32e4 a7a8  .61X.....E..2...
00000600: 8de1 5e4b e831 6b06 6335 9cf9 b29c 807f  ..^K.1k.c5......
00000610: 0261 06b0 1021 0323 a256 edf4 015c 9e77  .a...!.#.V...\.w
00000620: 34a2 7c0d 8b90 76c5 6dc5 4158 18c6 138f  4.|...v.m.AX....
00000630: 1d12 4632 bc10 0bbc 35cd afc6 8057 e990  ..F2....5....W..
00000640: 0a26 9d69 6305 28f7 867b 95d5 29a7 db1d  .&.ic.(..{..)...
00000650: a76d 0b81 0031 c1dd be34 fc21 d8b4 3960  .m...1...4.!..9`
00000660: 5541 40ec f306 9044 e0a7 79d4 4960 bba4  UA@....D..y.I`..
00000670: 812b 54c4 d84a 9888 9f73 6f89 5ab5 4cf6  .+T..J...so.Z.L.
00000680: 77b3 06bc 158e abf9 a225 6962 7f49 eebb  w........%ib.I..
00000690: d654 967f 2851 38a0 395d 0ffd 4221 7074  .T..(Q8.9]..B!pt
000006a0: 4c7f 37a6 8301 5210 e017 d598 8949 f38d  L.7...R......I..
000006b0: 5df9 4a05 e31b aa33 ad2c aab1 a089 8d47  ].J....3.,.....G
000006c0: 378b e491 4bbb 53ea ddca 86f9 b909 c32e  7...K.S.........
000006d0: 701b 00ba d093 d422 1bf7 5fd8 1983 8e77  p......".._....w
000006e0: 26ff a8a1 591d c8a8 773f 6358 1ee9 bff6  &...Y...w?cX....
000006f0: d4c6 7698 d205 e735 62d6 ebe9 7bff c21c  ..v....5b...{...
00000700: f752 51de 54f9 7b50 2357 f502 03e8 7231  .RQ.T.{P#W....r1
00000710: fd74 f3e5 dfe4 899b ab98 d401 e4e6 763f  .t............v?
00000720: edd2 21d2 72a9 8d06 76be b450 a3df 2fcb  ..!.r...v..P../.
00000730: 9713 109e 6e18 11e1 eb82 5955 b502 b4d9  ....n.....YU....
00000740: b3c3 26f5 831c 2fdc 0768 4a42 43ba 9911  ..&.../..hJBC...
00000750: 523c 7c3a 1765 f917 c90a 0a81 333a 6e6d  R<|:.e......3:nm
00000760: 7617 5a96 ce57 7afc 345d 4dac f730 28d5  v.Z..Wz.4]M..0(.
00000770: 6276 8368 d570 a72c 009e 9903 b6cb ca71  bv.h.p.,.......q
00000780: f507 9c61 7a99 e123 61b7 34bc 69d3 1ebc  ...az..#a.4.i...
00000790: e547 abe7 65a0 289a 146d cea1 fde1 ae8a  .G..e.(..m......
000007a0: ebc1 0204 4b76 1e81 e5f6 eca0 1fd3 5ea5  ....Kv........^.
000007b0: ce85 e8f7 2e3d 38cb a100 25d1 60fe 84a4  .....=8...%.`...
000007c0: 5cda 112f a527 81bb 1110 7c8f 209b 79a2  \../.'....|. .y.
000007d0: a17e 8513 047d b435 6b22 e8b7 0e29 9ae9  .~...}.5k"...)..
000007e0: 1fb5 24db 4667 89fa a311 ea9e 922d a272  ..$.Fg.......-.r
000007f0: 0bbb 2f44 d80e fa0a 6998 6aa3 63d4 0dac  ../D....i.j.c...
00000800: e51b 8bd5 9224 25c1 2e81 5307 a587 43d6  .....$%...S...C.
00000810: 0224 057a e6b3 29fa ae42 bd72 4df6 e997  .$.z..)..B.rM...
00000820: a312 276c 1887 39e1 ad91 49d7 524b 42be  ..'l..9...I.RKB.
00000830: 9569 977b f2b0 9ce9 a2cf 526b 4f64 d368  .i.{......RkOd.h
00000840: 5a62 0a02 2c94 da1d 9fd2 f528 d047 e3ac  Zb..,......(.G..
00000850: f181 3b43 d334 3097 4618 3f22 eace 649b  ..;C.40.F.?"..d.
00000860: be27 5289 4e98 7631 ea42 1fc0 15ec 9bec  .'R.N.v1.B......
00000870: 705a 0cca c7e7 590e 0f86 d6ba 488f 4003  pZ....Y.....H.@.
00000880: 40e9 4068 f393 00dd c9dd e2d4 f2a0 75b5  @.@h..........u.
00000890: 1533 2613 82ca 5cf9 7b3b 1ae6 fd67 0e1d  .3&...\.{;...g..
000008a0: db27 9eb1 e38a 4c65 6946 9a7a e55a 027c  .'....LeiF.z.Z.|
000008b0: d3b4 2d1c ce0f 7c64 9de7 b586 129c 39c9  ..-...|d......9.
000008c0: bc8f a775 41f1 a703 665e e198 6a1e 43e9  ...uA...f^..j.C.
000008d0: ec8a ba8b 04ab 7c2e 276a 1931 63ca e222  ......|.'j.1c.."
000008e0: be6f ca9a 796c e954 27b8 8af6 30b1 cf80  .o..yl.T'...0...
000008f0: 3527 a30a 14ec 7d17 6f1b 32e0 96ca bda1  5'....}.o.2.....
00000900: f3e0 1327 48e6 47e6 c29c 84f1 4aac 0062  ...'H.G.....J..b
00000910: 70ec ba50 85c9 3099 2115 63db c096 6862  p..P..0.!.c...hb
00000920: 63d0 dbc7 8ee3 cf62 c486 d441 6c3e 526d  c......b...Al>Rm
00000930: 7fe5 23e9 a0da c3d0 3347 ab27 4aad b034  ..#.....3G.'J..4
00000940: 8465 5a9b 1edf 227b befe 9b26 bbcb bac5  .eZ..."{...&....
00000950: 0239 095d 5b15 ef0e e710 b5f2 fed5 674e  .9.][.........gN
00000960: b761 62f2 c4f9 22d6 2ae0 5d64 55f1 88e7  .ab...".*.]dU...
00000970: f356 83fd cdae 98ae a436 5382 e744 f8ea  .V.......6S..D..
00000980: 8d1b 5bba c91c bd6c 3c5f 9fe6 9c96 c3bf  ..[....l<_......
00000990: 52d2 a9b5 c87c d94a ceb6 9889 871d 5f62  R....|.J......_b
000009a0: b1b7 382c 5e84 8bda 5691 669f 40ba e135  ..8,^...V.f.@..5
000009b0: 5738 9050 8f69 a8db 2910 06d2 3f4d 369b  W8.P.i..)...?M6.
000009c0: 7902 712c 7727 2f3b 54ad 08d7 6985 39ab  y.q,w'/;T...i.9.
000009d0: ec94 60d8 daff 4735 0494 121a 50ed b693  ..`...G5....P...
000009e0: 00c8 5d8a aaeb a5f2 2738 e0bb 218a 95a5  ..].....'8..!...
000009f0: f577 a238 7926 254a 436d e59a e360 4ec2  .w.8y&%JCm...`N.
00000a00: f72c c38c c4f0 07c1 5b99 873c b161 10be  .,......[..<.a..
00000a10: ae7a 780e ac9a d3cf 6979 06de e1f7 0468  .zx.....iy.....h
00000a20: df9d 3dfa 3e53 4b7d f0d0 f199 55fc 22e5  ..=.>SK}....U.".
00000a30: 80ec d91f db65 630e 3d1c 24b0 f3de 2498  .....ec.=.$...$.
00000a40: bf17 1e17 5445 ed42 501c 64b9 efa6 f6d3  ....TE.BP.d.....
00000a50: ac1e 3388 afe8 8838 9982 99c8 1f95 5ac3  ..3....8......Z.
00000a60: 5c28 8cc3 6456 6ea7 7bd6 07ac ab2b aaeb  \(..dVn.{....+..
00000a70: 5b1d 5a54 bce2 55d6 6c1b 302c ab57 cc72  [.ZT..U.l.0,.W.r
00000a80: 7ac6 d797 b9ef 1d0a c1b8 46f8 29f5 2f8f  z.........F.)./.
00000a90: 0850 120f da8c 93d6 2e78 d396 77e0 04cf  .P.......x..w...
00000aa0: 5ff7 24ba ac39 11cc 3f36 6f61 927a cb41  _.$..9..?6oa.z.A
00000ab0: f68f 5b77 07f5 e9b1 b363 93e9 c4fc 205a  ..[w.....c.... Z
00000ac0: ce6d 6827 6bd2 867c 068c 15f3 4832 a50f  .mh'k..|....H2..
00000ad0: d3b1 32a3 f617 dc86 303a 8f0e 4a7f a057  ..2.....0:..J..W
00000ae0: c409 a393 fc66 632f 1eea 4737 7cfb 7ec0  .....fc/..G7|.~.
00000af0: 40f9 f895 5cbc b803 5d13 5e57 7651 b05b  @...\...].^WvQ.[
00000b00: 3d4f a7be 8389 92e2 b70a 92e0 ceee d976  =O.............v
00000b10: 2bcc a308 414d 30b2 13f5 69b1 af88 940a  +...AM0...i.....
00000b20: 153a 8ed7 710a 22e5 6f8d 5ece 805c 8b6c  .:..q.".o.^..\.l
00000b30: c4c8 aeb6 ffd4 7c91 7112 5631 293a a7f7  ......|.q.V1):..
00000b40: d3ed 6fbc dc8c c67c 3578 7af0 0165 4d3a  ..o....|5xz..eM:
00000b50: 2747 5979 f7b6 5b0b a5e4 cd07 1dd1 7224  'GYy..[.......r$
00000b60: 9f79 8218 fa07 9371 4060 d4fe 3309 84da  .y.....q@`..3...
00000b70: 4be8 b0c8 ed76 0613 d4c4 226e 83e3 6ebb  K....v...."n..n.
00000b80: fc9c 682e 8a29 da42 20dd e71f ff98 8b4c  ..h..).B ......L
00000b90: 2658 e1e7 5e68 c4f0 37e7 0d4d 13ea 4676  &X..^h..7..M..Fv
00000ba0: 30bd 652f 4489 99a6 d203 b9a2 bafb 1a29  0.e/D..........)
00000bb0: 02a7 783c 261f f9f2 f724 7629 40ba c0ca  ..x<&....$v)@...
00000bc0: 2fa4 ee40 00d1 e1e8 f081 45cb 5fb5 fde1  /..@......E._...
00000bd0: c77a a899 8302 d17d 76b6 0000 0300 0003  .z.....}v.......
00000be0: 0025 e1                                  .%.

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=18432
pts_time=1.500000
dts=17920
dts_time=1.458333
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=2649
pos=49713
flags=__
data=
00000000: 0000 0a55 019f 0244 5700 0003 0000 07d0  ...U...DW.......
00000010: 8f98 4b7e 568a 802a 68af dd2e 7d88 15a8  ..K~V..*h...}...
00000020: f37f 0f39 5ee3 defc 21e8 ddda 8e86 f2ff  ...9^...!.......
00000030: 1ed0 f95a 4c1f 3a07 3049 3a40 6619 8201  ...ZL.:.0I:@f...
00000040: ab29 0350 c59b 1700 de5e d4c5 0a27 d5a7  .).P.....^...'..
00000050: 0793 1414 3ab4 c579 e516 7329 47d5 12c8  ....:..y..s)G...
00000060: 16ba 5583 4c62 9255 b586 be78 cb86 8492  ..U.Lb.U...x....
00000070: b874 560c cd82 5f95 2a48 dcda 4686 4871  .tV..._.*H..F.Hq
00000080: eacd dadf aec6 556d 8f23 bea7 6c2b bff2  ......Um.#..l+..
00000090: 3a9f cac5 fbbb 39b1 e2f0 5c2c 2de5 fa41  :.....9...\,-..A
000000a0: 3331 1cdf 5bd9 8b48 0fad 6dfe 83ef dedf  31..[..H..m.....
000000b0: fa32 2291 1de5 97a2 1c8a 8a4d 26c7 26c8  .2"........M&.&.
000000c0: 812d 2f02 23c0 e4f3 0f4b 5684 d959 6b9e  .-/.#....KV..Yk.
000000d0: cf98 b9ca 56f7 e419 183e bf3b 9597 3ca9  ....V....>.;..<.
000000e0: c579 73e9 3e33 8be7 621a 75a0 a4b2 70dd  .ys.>3..b.u...p.
000000f0: 4a29 0899 62ee 6d1a 6b27 2792 ff1d 5065  J)..b.m.k''...Pe
00000100: 1c47 1a3a ae75 e07e faac dc64 ce2b ba57  .G.:.u.~...d.+.W
00000110: 3f2a 7fdc 5f86 a204 e473 9545 1d6b 5136  ?*.._....s.E.kQ6
00000120: cbbc e8a8 b8b5 074f a7e0 373d 37e3 0c34  .......O..7=7..4
00000130: d0ee f177 c530 9aab 0ba1 6c92 a8d6 6a70  ...w.0....l...jp
00000140: a352 ab73 8565 87b0 f8e0 436d 5433 97f1  .R.s.e....CmT3..
00000150: 4aa9 44d7 7641 92e8 1ac4 54a1 db30 31a9  J.D.vA....T..01.
00000160: 0b2e 2918 32ea 2f62 d53f 4798 8cad 2702  ..).2./b.?G...'.
00000170: eaab 5c8b bcd1 f422 e339 a0a0 ba85 f64a  ..\....".9.....J
00000180: 0a9a 4138 5e98 af46 5509 8f06 9e53 356c  ..A8^..FU....S5l
00000190: 4475 ce6d bb04 04f0 4235 3bb5 2342 a2e5  Du.m....B5;.#B..
000001a0: ec48 9b8d 176e 72a1 2c8a 7a10 9616 b564  .H...nr.,.z....d
000001b0: b546 96b9 81e9 70bf 6dd5 9e7a 5135 26a4  .F....p.m..zQ5&.
000001c0: cb61 b392 af44 d150 1bcd 92db 51d4 a5e2  .a...D.P....Q...
000001d0: 6994 e99b 4636 655a 5e2e 2c66 6e76 b1fa  i...F6eZ^.,fnv..
000001e0: 8e01 91eb 3f08 d22d 76fb 890b ba3f 3851  ....?..-v....?8Q
000001f0: 3bec a1f8 b9be 7006 f614 73f7 6c07 0a5c  ;.....p...s.l..\
00000200: 934b f0c7 d3dd b79f f486 2abc 24ca 75c0  .K........*.$.u.
00000210: e344 3c51 b87f 0570 1a28 a2e8 570a bd34  .D<Q...p.(..W..4
00000220: eb59 c9ff 8cdb 4763 bd50 a9b1 2400 4a26  .Y....Gc.P..$.J&
00000230: 0461 fc49 6367 be12 a34a 2815 077f 4b10  .a.Icg...J(...K.
00000240: 65bc 7cde 840c 3656 35fb 16da 6b49 6e5e  e.|...6V5...kIn^
00000250: 7b81 44ec e153 0ab0 f067 52db ade5 e020  {.D..S...gR....
00000260: 78fd 9ad6 746e c76f cd24 a99d f7e2 8c2c  x...tn.o.$.....,
00000270: 7957 505d 3177 0c71 2fca b30c 9878 48f9  yWP]1w.q/....xH.
00000280: 30c7 feac c94f 86c6 dceb bcab 0ae3 94a1  0....O..........
00000290: 022c e890 5876 0be6 b891 e53a 8504 51fb  .,..Xv.....:..Q.
000002a0: 66e0 0460 286d be1f 5296 7af4 2519 d2cf  f..`(m..R.z.%...
000002b0: 6d3e d4e1 5132 a65a 61ad 5534 00ae df12  m>..Q2.Za.U4....
000002c0: f627 07d3 2007 2ea7 1363 b992 3ec5 93db  .'.. ....c..>...
000002d0: eeca d349 a4a6 cb8f 5801 3726 0777 4d20  ...I....X.7&.wM
000002e0: 05d5 1465 6657 45a1 9935 c403 bb1b fb64  ...efWE..5.....d
000002f0: 2394 b279 2e27 8681 a243 b00b 9268 e497  #..y.'...C...h..
00000300: 5ec0 fbec c167 d466 f440 1eb2 3052 62f3  ^....g.f.@..0Rb.
00000310: 4933 ac60 15a8 ee0f 0bdf 4588 92c5 ed98  I3.`......E.....
00000320: ee95 ce3d 9d7f 2902 4e76 33a4 7481 edc4  ...=..).Nv3.t...
00000330: 7ef2 35bb 7fd5 a7b5 5fd6 000c 33b0 e814  ~.5....._...3...
00000340: 7273 e246 37b9 a5b8 043e 9db5 5c7a b86d  rs.F7....>..\z.m
00000350: 1de2 b7a7 3171 bdcb 4aa1 3f8f 8cab cc37  ....1q..J.?....7
00000360: 5edc ee84 bb03 d2f9 3da4 f917 8c1f 5093  ^.......=.....P.
00000370: 75be e910 9693 1b43 8674 e837 f9db 5861  u......C.t.7..Xa
00000380: 9945 8791 0334 1b62 2b31 af03 e951 7b99  .E...4.b+1...Q{.
00000390: cf01 da1f cab5 ccd9 90cb 16c7 15c6 7e8e  ..............~.
000003a0: 61bf b9d7 2782 6f7c 990b fb85 da11 c597  a...'.o|........
000003b0: 2b84 78af e3fb 3ba5 d443 3871 521e 8135  +.x...;..C8qR..5
000003c0: 6b1f 7c8c b16c 9e25 db35 1c81 96c3 e032  k.|..l.%.5.....2
000003d0: 0c76 c996 11cc 862f b34b 8a89 7209 65b8  .v...../.K..r.e.
000003e0: 2aa0 09e4 89a0 363f 1793 3223 ff74 0ba9  *.....6?..2#.t..
000003f0: dac7 6475 f6f1 ce47 5977 b806 6e99 2620  ..du...GYw..n.&
00000400: 0a2f 7edf 3a1e 23c9 14a0 87fe 48af a974  ./~.:.#.....H..t
00000410: a67e 42f8 773a edc5 8477 c69f 22f1 9f7c  .~B.w:...w.."..|
00000420: 2c56 0a93 1437 c71e b3fa aaaa 343a 5d36  ,V...7......4:]6
00000430: f1db 0da1 1bc6 50b7 b0df 2a36 80a6 7bde  ......P...*6..{.
00000440: 63e5 fb85 529c 4849 b6ac 7a55 7a84 d5a2  c...R.HI..zUz...
00000450: a61f f1cb ba0d f6e8 4a59 2141 ca9c 6662  ........JY!A..fb
00000460: 8ff8 35a6 c742 9076 0bab efab f8f4 7690  ..5..B.v......v.
00000470: cf28 dfd5 cf38 7549 d5ef e2b0 d7d8 47b2  .(...8uI......G.
00000480: abef 830c 02a2 94a5 08bd e86c 8ab7 7c91  ...........l..|.
00000490: 90e8 f548 a105 4b15 516a 4451 559a 7a1f  ...H..K.QjDQU.z.
000004a0: 5de2 f3d0 1713 a036 aa2b 4f8a 5a2d 5500  ]......6.+O.Z-U.
000004b0: beaf d427 b10a a115 13eb e3a4 9d82 3b87  ...'..........;.
000004c0: 1eb4 59db f855 688d df29 96c5 a694 66e7  ..Y..Uh..)....f.
000004d0: 7f7b d9a5 64d3 229f f258 8bd4 de1c 7f92  .{..d."..X......
000004e0: 9ba8 32f2 f605 0808 eae7 9f23 0ef1 d504  ..2........#....
000004f0: 4a25 5a50 db72 7b10 045e e533 2b94 9bcc  J%ZP.r{..^.3+...
00000500: 0bc2 c5bb d9fd e96b 268a de1a d4f5 469b  .......k&.....F.
00000510: 2f6a ebb5 3a3a 59f9 ab35 c6f1 181a 66bf  /j..::Y..5....f.
00000520: 5a5c 5c54 265e 008f aa07 50f9 4cd0 6b85  Z\\T&^....P.L.k.
00000530: 04a7 6979 5b67 c53c 9a84 02e7 f5ca f91f  ..iy[g.<........
00000540: 5383 3f1e ff9e fca1 77dc cf91 c3c2 e2c3  S.?.....w.......
00000550: 7fc8 cf8f 2a15 cc5c 4713 10d0 3e10 481e  ....*..\G...>.H.
00000560: 604c 34e3 6d1d 2653 7a49 f0b5 ca3a fdcb  `L4.m.&SzI...:..
00000570: c5e0 fb9e 19d3 f968 805b 7998 b59e e018  .......h.[y.....
00000580: a360 44c4 e2aa 8b26 b9e1 a849 e645 3923  .`D....&...I.E9#
00000590: 40d4 dcf3 446b 333a bbbd 218e d7a5 3292  @...Dk3:..!...2.
000005a0: 02cb ad1e ea21 34e9 d9fa 366b 9aa7 bdb0  .....!4...6k....
000005b0: a6d4 f4e3 8795 0c46 076d 4cdc 2c8f dfcb  .......F.mL.,...
000005c0: 8907 c637 5cfc a8ec 4ac3 e107 03ec 6c39  ...7\...J.....l9
000005d0: 5ba4 90a4 330b f077 a936 2f95 fc69 ab62  [...3..w.6/..i.b
000005e0: 50bd 6732 b5c3 ba26 7899 df21 5ecb 4e1f  P.g2...&x..!^.N.
000005f0: d772 03c8 a908 9871 0b44 8476 3f09 e989  .r.....q.D.v?...
00000600: 1b06 59d6 3756 67ae 825d 8095 ca63 2684  ..Y.7Vg..]...c&.
00000610: 3655 10b9 3f53 90a4 78ef 290c f43e 32d5  6U..?S..x.)..>2.
00000620: 6efb 152d c2be 4b27 8408 e16d aafb 5a4e  n..-..K'...m..ZN
00000630: a2a4 b597 a04e 5d60 595d 5cc4 dbb1 132d  .....N]`Y]\....-
00000640: 1d18 4508 6c95 8866 7790 b9c2 b5ea 440f  ..E.l..fw.....D.
00000650: da38 2c81 9d62 4574 b3cf d379 c9d5 fb33  .8,..bEt...y...3
00000660: ebe9 c800 b4fc cbea 918d 1da4 29a7 7dd9  ............).}.
00000670: 8277 6556 e0f9 1cd5 7aa2 e2bf a34a fd91  .weV....z....J..
00000680: 662d 8007 2de6 96ab 378e 2400 3eb9 3c53  f-..-...7.$.>.<S
00000690: 8a02 b3b0 2c2b 6720 7e4b 3d8d 49be f92d  ....,+g ~K=.I..-
000006a0: c278 7fe5 9618 50d5 4c7e b749 4a28 9a6f  .x....P.L~.IJ(.o
000006b0: ba73 6426 f376 a289 88a8 5d36 eabf 5837  .sd&.v....]6..X7
000006c0: b333 db57 982d 981a d3a6 43f7 aaea 4821  .3.W.-....C...H!
000006d0: 6169 c670 086a a5c9 d6f0 a6c4 b34d ec7a  ai.p.j.......M.z
000006e0: 1b10 0d42 7efc f374 d8ed df2a eca1 685f  ...B~..t...*..h_
000006f0: 1fd8 c8c5 8658 6b7d 4e7e 9efe 61f8 4ea8  .....Xk}N~..a.N.
00000700: 16f3 7f0f 7e24 f346 aec0 c816 ba59 5ddc  ....~$.F.....Y].
00000710: daa3 c06b f798 82c9 1727 6d09 5b8d 2dd4  ...k.....'m.[.-.
00000720: 5c1c 3129 09b5 bad7 b91e 646c ee02 e153  \.1)......dl...S
00000730: 6849 fac6 e40f 3016 f2ed 717e 0ae7 52e6  hI....0...q~..R.
00000740: bc2f 4a49 92dc 1663 2ce5 7618 04d5 5713  ./JI...c,.v...W.
00000750: 5989 8056 08f9 dc09 c25f e7d5 e219 1d25  Y..V....._.....%
00000760: 3d3f 37db 0682 d88f 97b1 7c34 6089 fa3d  =?7.......|4`..=
00000770: 304a 5635 6205 788f db43 6f61 3cba e6db  0JV5b.x..Coa<...
00000780: 14f2 6ea4 c067 9faf b85f 09c7 aefa 046e  ..n..g..._.....n
00000790: 9928 3fac 82b4 55b8 5ade 8590 10c5 1fc7  .(?...U.Z.......
000007a0: a2b8 8711 ffb5 a086 9986 2743 0205 fa54  ..........'C...T
000007b0: 56d7 bc69 e335 503c 338a ddfc 496d bc59  V..i.5P<3...Im.Y
000007c0: 7706 f720 f79f 5701 128a da40 7cd7 3e70  w.. ..W....@|.>p
000007d0: fbc3 af9a eb80 c7f9 189b 7399 9939 71e3  ..........s..9q.
000007e0: 03e1 ba1a dadc f31c eb9f 5d37 ca58 5df6  ..........]7.X].
000007f0: dede e489 30b3 61e6 8bc0 e188 1c4a 35f3  ....0.a......J5.
00000800: e51b 40f5 406d 6862 a7eb 1143 d1ca 66f0  ..@.@mhb...C..f.
00000810: e896 0cd6 3ce1 73fe 2dac 9bc0 347f ad98  ....<.s.-...4...
00000820: 250b 17cf 1bd4 a35b 8b49 c106 15ab 8e26  %......[.I.....&
00000830: 5bb5 f8d3 a4d6 863d 6afa 8d04 2041 f8e1  [......=j... A..
00000840: 7377 e858 1c95 6f95 1f08 e31d f7cb 5ff6  sw.X..o......._.
00000850: 202e e47a aa8f c82f 3f42 7e56 3b38 7637   ..z.../?B~V;8v7
00000860: bf9f cacb 1712 06df b65a 6b5f 3bb3 3bf9  .........Zk_;.;.
00000870: d4d3 b673 7122 373f c9e4 d9b4 1a07 6f3c  ...sq"7?......o<
00000880: 7314 754d 4a3e d98d de92 603d 4718 3e55  s.uMJ>....`=G.>U
00000890: 7fb7 cdb6 831d 03bb b60a 5abd 8cdd fe7e  ..........Z....~
000008a0: dffa c161 a6aa 7bb4 3296 0433 db62 1809  ...a..{.2..3.b..
000008b0: f458 2a9a 2cd4 1a4a 6162 598b 6e31 7c59  .X*.,..JabY.n1|Y
000008c0: 579a 05c9 2f37 62c3 b126 9707 caef cdbc  W.../7b..&......
000008d0: ca18 6fa4 8423 77bb affd 08a1 a48b e3bf  ..o..#w.........
000008e0: c812 5979 b175 ff87 9537 2fdf 70ee 63ba  ..Yy.u...7/.p.c.
000008f0: bcba e7ed e8d4 c14f e0a1 31e9 aa50 e431  .......O..1..P.1
00000900: c0db 02b3 983c 1502 bb64 06ba 96b5 2bf6  .....<...d....+.
00000910: b8a7 f452 0379 c757 3c15 55e8 684e cd9e  ...R.y.W<.U.hN..
00000920: 40ce 46f4 c442 4e09 4545 48d3 9fcc 9e56  @.F..BN.EEH....V
00000930: 1752 9f2c 86d6 498f 13e6 bda2 ba0a 43b1  .R.,..I.......C.
00000940: dc07 2c3f 5680 f5db 8c6f f58a 5307 bb50  ..,?V....o..S..P
00000950: 291b 5df0 a9c5 becf 6cce 2d7d 67e9 7024  ).].....l.-}g.p$
00000960: 38e6 8a3b 09e1 b9a8 4244 65fb 12a9 61bf  8..;....BDe...a.
00000970: fcb8 bdd6 e05b 44fe 74bf 9fc9 4b34 3497  .....[D.t...K44.
00000980: d94e bda7 38bf 311b a920 6c91 5e63 cdd0  .N..8.1.. l.^c..
00000990: 62f4 8372 e3c8 95de c541 8109 12e6 9c16  b..r.....A......
000009a0: 5d75 ef9c 4f5a f78f cc4a 00df d472 94db  ]u..OZ...J...r..
000009b0: 9d29 bd3c e2d6 9972 85f3 a472 1b73 3a6e  .).<...r...r.s:n
000009c0: 52e2 78c0 de41 192a b7a2 a9ac 77f3 1175  R.x..A.*....w..u
000009d0: 72c6 be55 d507 ccf8 840f b8bc 220d ebee  r..U........"...
000009e0: 3af5 933b 073b 99ae 2833 c288 5201 51a6  :..;.;..(3..R.Q.
000009f0: f471 dd24 caf0 6237 009f fa9b 5891 29e5  .q.$..b7....X.).
00000a00: e66f 5f67 72f0 ae3a 3a2a 91f5 92b7 5f9b  .o_gr..::*...._.
00000a10: ef8d 4e0c 739e eae1 976b 509d 02a0 4183  ..N.s....kP...A.
00000a20: ac62 1065 2188 bc20 d196 800f 32fa c7c2  .b.e!.. ....2...
00000a30: d95c 8b29 0ceb 1222 89b5 9892 b464 ea9d  .\.)...".....d..
00000a40: 58a2 d9d6 646c 27d5 f700 a10f 6b2e 1380  X...dl'.....k...
00000a50: 0000 0300 0003 002e 20                   ........

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=20480
pts_time=1.666667
dts=18432
dts_time=1.500000
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=5205
pos=52362
flags=__
data=
00000000: 0000 1451 419b 0634 a4c1 15ff 0000 0300  ...QA..4........
00000010: 0003 001b fe17 dc25 410f e78f 0734 8571  .......%A....4.q
00000020: c763 dad3 bbfb 408c 2734 5b5d 36f2 f51a  .c....@.'4[]6...
00000030: 6edd 4980 5a7b e42b 1aec c28b 73a2 898b  n.I.Z{.+....s...
00000040: cab4 aed4 0ea7 358e b7ab e10e 522d b5e5  ......5.....R-..
00000050: 5e5f 02eb f729 77f1 83d1 1884 95b2 e7f9  ^_...)w.........
00000060: 2440 6885 221f de06 aa4d 504b a6a2 e3db  $@h."....MPK....
00000070: acbb 5a1f 3d49 232b 453c 4562 9299 07c8  ..Z.=I#+E<Eb....
00000080: 9113 fd32 01c0 950f f757 6ca0 d330 0ad2  ...2.....Wl..0..
00000090: 5ef0 f4b8 49b8 7e7c 12fe e07c 056c 10fc  ^...I.~|...|.l..
000000a0: 5a10 5e69 e358 7135 275e c7a3 51c2 8c70  Z.^i.Xq5'^..Q..p
000000b0: 1b48 643b 5d68 5600 0003 0011 58ab 277f  .Hd;]hV.....X.'.
000000c0: 4c98 4ce7 26e7 8197 25ed cf44 5e72 0c7c  L.L.&...%..D^r.|
000000d0: f4e1 16ff 3fe8 ccd8 0212 a170 0d75 0193  ....?......p.u..
000000e0: 3f6f 358c d4bc ca8b bad7 5c08 cf38 ab68  ?o5.......\..8.h
000000f0: dd0b eca2 2fc5 e302 b7c9 1208 27ce fbfa  ..../.......'...
00000100: 8767 4690 c1ef c8e1 8765 e7f4 0614 8ca4  .gF......e......
00000110: c278 2e67 5c38 b462 a295 f1cc 2434 3e9e  .x.g\8.b....$4>.
00000120: 4400 21a4 03c1 ae3a ae53 8d22 9aca 0677  D.!....:.S."...w
00000130: ded3 5d19 94ff 535b bb33 7e5b d1ea 6b56  ..]...S[.3~[..kV
00000140: 364c 09b0 a3f9 c6b7 fa58 f408 6564 ae72  6L.......X..ed.r
00000150: da34 11d5 7cb2 c070 cc70 9e06 2127 036a  .4..|..p.p..!'.j
00000160: e941 8932 8226 48cb 886b bac0 6eb7 8802  .A.2.&H..k..n...
00000170: fd57 174d 2f2a 1867 7978 39db 6097 1e78  .W.M/*.gyx9.`..x
00000180: 6550 bfe5 78d8 0ff5 4753 fc1a 2420 74a9  eP..x...GS..$ t.
00000190: 53f7 bf06 5ce6 fe28 1a6f 29e0 44c7 d615  S...\..(.o).D...
000001a0: c7f6 2801 94ed 4be1 cca8 aea5 ab4d a163  ..(...K......M.c
000001b0: e823 05bb d97f 04e1 3eee f96e 5e46 d19a  .#......>..n^F..
000001c0: ee88 647e 94e2 2012 f745 11a9 25a4 f02b  ..d~.. ..E..%..+
000001d0: 525b 8bba 1ecc 4688 c1d0 0e7b 9699 4f0d  R[....F....{..O.
000001e0: f2e6 a3fb 3bd5 0bd4 b60f 2014 46f1 70fa  ....;..... .F.p.
000001f0: e0e5 a80c 1292 f808 533e fe56 fabe 0791  ........S>.V....
00000200: e330 fc85 2780 15f1 802b 10c4 1a86 8c09  .0..'....+......
00000210: 275c 2fa6 8ed2 4ede ef3e 4a72 22eb 8854  '\/...N..>Jr"..T
00000220: 0231 7fe1 db8e e492 f2ba 9efd 2ea6 a598  .1..............
00000230: 3235 8c63 a126 0a8a 47c0 5ac5 3171 9a67  25.c.&..G.Z.1q.g
00000240: 7e4d 9f09 66cf 486d c064 e786 1742 4e89  ~M..f.Hm.d...BN.
00000250: 3d19 08b4 df8a 8186 e7eb 9f56 4141 3499  =..........VAA4.
00000260: 9bc1 8aae 9c94 8330 369e c99b e530 9de2  .......06....0..
00000270: 2d76 7736 0d93 c280 add8 1155 f940 2895  -vw6.......U.@(.
00000280: 1930 4507 258e 40a0 da66 dc66 3ba8 b5b1  .0E.%.@..f.f;...
00000290: 2c07 6eeb ffb1 3e92 e377 3719 779d bfad  ,.n...>..w7.w...
000002a0: c9c5 7b7b 62a4 101a d90d f115 db6f ae3a  ..{{b........o.:
000002b0: 5dde 52b8 974a 006f 09da 640f 44c3 e710  ].R..J.o..d.D...
000002c0: 7b98 5891 f6d2 9771 5d8a de6c 6cff 5b58  {.X....q]..ll.[X
000002d0: 4f31 6f98 f85d 023d 027f 5752 b8d1 d767  O1o..].=..WR...g
000002e0: 4d8d 2236 b19c 5993 7e34 4e25 7c74 e63f  M."6..Y.~4N%|t.?
000002f0: 0234 9f78 48f7 ffb2 360e b44a 59fb 84a4  .4.xH...6..JY...
00000300: 6c08 e332 f267 d89a 4d5a b549 ce3d 972a  l..2.g..MZ.I.=.*
00000310: 7d0e f0d7 d9f9 140c d5d0 fcca 533d 5aaf  }...........S=Z.
00000320: 853c b83f 16cf 3960 85ca 8ae6 d2a3 192b  .<.?..9`.......+
00000330: 4ad7 da52 083c c5f8 eaf9 3138 f677 a4d6  J..R.<....18.w..
00000340: dfcb 7891 9a16 7172 4d9e 7948 ea08 932a  ..x...qrM.yH...*
00000350: 7cbd a746 476b b2c9 12f5 66c6 1191 8f4e  |..FGk....f....N
00000360: f034 1bdf 820b d3d7 65bd 5a5c 8edf 7851  .4......e.Z\..xQ
00000370: ee65 3fd6 c2e0 cb3d 8d5c 1e60 acc3 dac7  .e?....=.\.`....
00000380: 8144 ccd2 aa07 e7b9 7d2f 055a 5858 e4f1  .D......}/.ZXX..
00000390: 08e4 df5a 5d10 343a bcf7 29d6 142a b429  ...Z].4:..)..*.)
000003a0: 999f fa1f 129b 6587 8209 dbc5 5959 6b62  ......e.....YYkb
000003b0: 2000 0725 0c38 13a2 d75f 0507 7c0d 1520   ..%.8..._..|..
000003c0: 9ab6 b0ba 7605 4240 18b5 67c5 6f43 6d6d  ....v.B@..g.oCmm
000003d0: 0dff db19 6046 4d9b b6c9 9ecb 405b 9507  ....`FM.....@[..
000003e0: 0142 c471 023f 6891 5542 b98f 4b43 f4ff  .B.q.?h.UB..KC..
000003f0: b122 f135 0182 abde d963 d419 36a7 19fb  .".5.....c..6...
00000400: b073 7be5 7e40 1610 6a92 3ca7 6a35 f220  .s{.~@..j.<.j5.
00000410: e8a8 f7f3 4c20 481a a170 db51 6f29 d008  ....L H..p.Qo)..
00000420: eb84 15b1 e555 fa97 c7a8 8ef9 f777 1b79  .....U.......w.y
00000430: e53d b9e5 4ecc 9cb6 e95f c626 6457 12b6  .=..N...._.&dW..
00000440: 8e94 d1b0 2321 3015 f16c 06d0 8c0f 590e  ....#!0..l....Y.
00000450: 8a9a 9968 d495 f086 796d 6a75 2ef6 4d1e  ...h....ymju..M.
00000460: 4a6d 6a0b 90f5 8347 870b 1c02 6aef 8de6  Jmj....G....j...
00000470: a0eb 08c2 29c0 3527 9a92 b3d6 61d9 cb5d  ....).5'....a..]
00000480: 1ba5 c069 cf46 623b 378f 21c7 d241 8934  ...i.Fb;7.!..A.4
00000490: ac13 02e4 51d8 f4d5 47fe 285b d0d7 526d  ....Q...G.([..Rm
000004a0: 75c5 d59e d0b5 eee9 aaae 6b3a 2cee 16be  u.........k:,...
000004b0: a03c 9525 621c 3fb3 62e1 af8e 2cad 141b  .<.%b.?.b...,...
000004c0: f6a3 7b10 0a0e dc39 dfa3 c338 301d 0f54  ..{....9...80..T
000004d0: e612 acb5 4c41 912f f733 8b97 6850 b86a  ....LA./.3..hP.j
000004e0: c753 a827 5e13 6235 8887 6add b268 ba38  .S.'^.b5..j..h.8
000004f0: db3f bf78 008b 3a61 a6e8 5a38 3c60 5787  .?.x..:a..Z8<`W.
00000500: 58ad 8af9 9c97 1efb df0f a532 1529 9c85  X..........2.)..
00000510: 55e5 38cb fe50 729a 4ac8 04b9 ee00 4ff1  U.8..Pr.J.....O.
00000520: 9548 f3d6 e1be 0582 df24 2276 b172 3274  .H.......$"v.r2t
00000530: ed39 1e3d e644 4303 571e 5d23 52d9 2646  .9.=.DC.W.]#R.&F
00000540: 2cea 4fe7 4e48 9dc6 2054 f57b 7597 716a  ,.O.NH.. T.{u.qj
00000550: 542f 534a 4807 4f83 682a 0b3f 2b6d d054  T/SJH.O.h*.?+m.T
00000560: d100 0aab 010e ae00 41da 0965 dba1 4ae6  ........A..e..J.
00000570: 3bef 4382 6388 5f3b 0a76 f09e bfef 5b2c  ;.C.c._;.v....[,
00000580: baf6 855b 249c fca3 0890 0069 7533 b044  ...[$......iu3.D
00000590: b1ce 2338 588d 18f8 4ea3 8f27 61a4 ac7f  ..#8X...N..'a...
000005a0: 05bd 37f0 7a14 01d9 f330 9801 4ad9 26d8  ..7.z....0..J.&.
000005b0: b33e e211 6c67 239a 47c8 d2ce a594 af98  .>..lg#.G.......
000005c0: 352e 0aba 911c f40a 01f5 ba6e ff74 9c74  5..........n.t.t
000005d0: e3b5 848a c478 2b1f 4f11 0641 1b13 715e  .....x+.O..A..q^
000005e0: 2ead 1eee 2125 3875 cfbf 3f05 20c3 6714  ....!%8u..?. .g.
000005f0: 13f8 c1ea 4700 1a27 9be3 7eaa 6334 8563  ....G..'..~.c4.c
00000600: 4c2a 39f9 8dfb ea1a 8903 4f4e 102d 01fe  L*9.......ON.-..
00000610: 4527 c035 e1aa 43bd 38b6 b2e9 27fc e680  E'.5..C.8...'...
00000620: ac5a 9521 111d 8fab a2ec b1fb 1e99 c624  .Z.!...........$
00000630: 9a8b 390f 012f 86f2 9356 7512 8005 1667  ..9../...Vu....g
00000640: 89b3 00a3 36fd 27a4 1752 4c44 891e 76aa  ....6.'..RLD..v.
00000650: 67ed 8e73 70cf d552 03f1 960f 48c2 aade  g..sp..R....H...
00000660: 0d8f cdfa b5dd 76c6 85d9 6e2b d87d 9b8a  ......v...n+.}..
00000670: 9913 0773 f746 9b1c f0ed d276 4841 0e0c  ...s.F.....vHA..
00000680: 5129 f31f 6d66 a07e 53d3 332c e7a0 6386  Q)..mf.~S.3,..c.
00000690: 9cfa b380 8cab 8c7b 8702 a41c 7fcd f86f  .......{.......o
000006a0: ae29 3437 1eb0 d698 7f60 3649 7c92 b864  .)47.....`6I|..d
000006b0: 568a bdf3 3a6a 8428 ddc9 18ac 3b5f 191e  V...:j.(....;_..
000006c0: b9db e659 7dec d081 f3a3 b125 84a4 df49  ...Y}......%...I
000006d0: 85a0 4225 cfa1 0012 1596 a755 dfd0 e9fa  ..B%.......U....
000006e0: 7bd2 0cf0 5b45 118d e192 fd89 e13a 3f9a  {...[E.......:?.
000006f0: 62ef a1e1 eb53 d125 29bc 4f96 0bcf 8ff5  b....S.%).O.....
00000700: 1dbe fa0b 18e8 a265 ef0d 1751 29ab 0c34  .......e...Q)..4
00000710: 5a58 88a9 d748 1564 7062 3bac 1292 b401  ZX...H.dpb;.....
00000720: 9642 c153 45ae 02dc 8ac8 5ef5 7d63 cae0  .B.SE.....^.}c..
00000730: 113a 59be 7d0e aa58 7666 437b fe4a e287  .:Y.}..XvfC{.J..
00000740: 6ffc bb74 65d3 6875 f21f ec32 2729 a143  o..te.hu...2').C
00000750: dcbf de1f 8441 09da 2b65 d623 04e6 5d14  .....A..+e.#..].
00000760: 5ef7 136c 452c 2d57 2a47 1391 b8d8 2e5d  ^..lE,-W*G.....]
00000770: ae15 29f9 7b29 41af 4063 4d02 b40a 5f38  ..).{)A.@cM..._8
00000780: eee7 790b 2b56 d2ac db2b 5642 0cc5 db6d  ..y.+V...+VB...m
00000790: 43ec 98d4 1b64 e76c e4e2 24b3 448b a54c  C....d.l..$.D..L
000007a0: fc53 066c ddf4 296b 5971 529a c926 1621  .S.l..)kYqR..&.!
000007b0: a965 0f2e e67d 3833 26dc 2a57 68be bf35  .e...}83&.*Wh..5
000007c0: 2b2e e06a 70b3 16c5 89b7 a590 3b91 1048  +..jp.......;..H
000007d0: ae1e 692c ea06 dff0 ef28 db81 1c3a 0769  ..i,.....(...:.i
000007e0: b9bc d0f7 79be ad29 bedb ee60 5704 7e31  ....y..)...`W.~1
000007f0: a72a 5635 d04b 1159 fbee 2fc2 f72f 8697  .*V5.K.Y../../..
00000800: 01ad eae6 e6fb a323 951e 485e 86d6 8ea6  .......#..H^....
00000810: badf 0c43 7136 cc8b ecfe 843b 9651 7f7a  ...Cq6.....;.Q.z
00000820: ce27 3f0a 9221 41e7 cbf4 de70 e3a4 cfd4  .'?..!A....p....
00000830: 46db 7d19 8844 5053 594f 8474 8c29 0332  F.}..DPSYO.t.).2
00000840: 7bbb c4a1 cf4c 45a1 7c29 c11c c185 bc7f  {....LE.|)......
00000850: 19f3 c7eb 25a1 7671 90c2 8176 b694 b2cd  ....%.vq...v....
00000860: 7571 a629 98b0 9704 e9d1 4489 2d2f 90ef  uq.)......D.-/..
00000870: dc36 3f73 8162 128f 9226 6be7 ee51 6aec  .6?s.b...&k..Qj.
00000880: 8d62 5fc5 18a8 aed1 e780 3617 0316 7276  .b_.......6...rv
00000890: fd87 cd75 9a56 9652 51ff e934 d20c a426  ...u.V.RQ..4...&
000008a0: b431 77ed a6de 8359 3c8d 829e de9f 98c5  .1w....Y<.......
000008b0: a462 7db2 243b 1293 d939 1933 bfb1 60ca  .b}.$;...9.3..`.
000008c0: 4950 70b7 f100 4401 03e8 6dd7 c233 fc62  IPp...D...m..3.b
000008d0: 177d e2ae f28e 23b9 867f f03f 776d 461d  .}....#....?wmF.
000008e0: a5ec 02a5 c4ad 3566 7a91 e1eb c702 ab7c  ......5fz......|
000008f0: 7154 43a5 1f39 f9d9 b318 3186 f6b6 21ae  qTC..9....1...!.
00000900: 1ec1 62bc a57a 0f9a 979d 5080 bc09 e391  ..b..z....P.....
00000910: 79c6 1288 0dfe 066d 5816 6360 4bef e5ed  y......mX.c`K...
00000920: b1a7 9a48 db60 53aa ca81 5db1 ca19 483b  ...H.`S...]...H;
00000930: c9de 18d8 9858 609b 21c2 3245 9ecf 5ea1  .....X`.!.2E..^.
00000940: 71e9 3383 dccf 5e40 e48e dc52 7e5c 31f0  q.3...^@...R~\1.
00000950: fa21 99c3 3147 3a0a 20fd 3022 2e02 f006  .!..1G:. .0"....
00000960: a432 4a66 c7d9 4ef3 274a 2c2e 8c86 918c  .2Jf..N.'J,.....
00000970: 5401 ec9b 47aa be6b 85a1 0b0f a4ca 2089  T...G..k...... .
00000980: 7b87 efbf 2f61 35b4 a337 2bec 5dc1 e421  {.../a5..7+.]..!
00000990: 8eb8 5bda 1198 afe5 2f55 8df5 e08e 5476  ..[...../U....Tv
000009a0: f8cc 4947 4f08 f9c3 6bf1 9861 231f f58d  ..IGO...k..a#...
000009b0: e0c9 4f6b bbb0 4efe 830f 802d 474f d746  ..Ok..N....-GO.F
000009c0: 7eb3 81f4 e835 7fed f472 6e6f f82b 7878  ~....5...rno.+xx
000009d0: fac5 8118 abba 9a34 8199 dd6e 630a b3ba  .......4...nc...
000009e0: 3a29 7e66 b8ea 3e16 7c56 448a b977 3e5e  :)~f..>.|VD..w>^
000009f0: 932e b4cd bca9 cf55 be42 5f09 3a29 7f60  .......U.B_.:).`
00000a00: db3d cdb9 1d77 df9f 0aef d151 9824 ac92  .=...w.....Q.$..
00000a10: 4004 154a 931d 2f90 1b4b fc13 49f1 baba  @..J../..K..I...
00000a20: c5d7 32a9 fc9e 1c49 79dd d5e5 466b 2bdf  ..2....Iy...Fk+.
00000a30: d8f0 6d26 f9bb 1d55 4750 e322 f3b9 eab0  ..m&...UGP."....
00000a40: 1b6b 9c9a 5bc8 4030 2b86 61a2 4f59 a7f1  .k..[.@0+.a.OY..
00000a50: b350 26e5 cdf6 308a fdd1 8b48 d365 3462  .P&...0....H.e4b
00000a60: 1637 08cc b7d0 3d74 4ebc 728a 06e6 8bf3  .7....=tN.r.....
00000a70: 4139 1a0f c9d4 681c 467e 4462 91fe 8771  A9....h.F~Db...q
00000a80: 1c9c e53c 5340 899b 63c8 9be4 133e 65f2  ...<S@..c....>e.
00000a90: b890 e18b 317d f91e 416c fc5b 2549 d18f  ....1}..Al.[%I..
00000aa0: 8c1b 935a 91ed b316 65dc 6f98 9b7c 6b8b  ...Z....e.o..|k.
00000ab0: 2382 934a 44d5 0364 a824 99f7 f82b 2d58  #..JD..d.$...+-X
00000ac0: e4ff ac51 0fde 6fc9 2047 971a 05e7 a616  ...Q..o. G......
00000ad0: 0911 bea6 363d cfc4 ec59 c4f5 6080 1c69  ....6=...Y..`..i
00000ae0: 6c4c f8bb ac19 e0ed 9219 b2d5 862e 58f3  lL............X.
00000af0: 88b9 e196 1c65 cc09 5273 7324 4649 735b  .....e..Rss$FIs[
00000b00: e090 5ac0 a24e b8b0 c322 c990 d787 64ba  ..Z..N..."....d.
00000b10: e635 89e7 64a4 88c0 047b 8510 069f 00b2  .5..d....{......
00000b20: 52b0 d6be ae4b 84de bec9 5c46 1356 34d4  R....K....\F.V4.
00000b30: 8d6a 9087 9860 9bb6 a498 1110 adf5 bc5f  .j...`........._
00000b40: cc0a 6701 d319 1a32 917b 4deb 512a cedd  ..g....2.{M.Q*..
00000b50: 650a 481e bcd5 1cab 0f52 5524 cfdc f2d0  e.H......RU$....
00000b60: 16bb f904 0cc0 6d56 bf62 73d2 1718 c8a8  ......mV.bs.....
00000b70: fdbf 6630 0d6f dc4d b349 4bee 66a2 e7ba  ..f0.o.M.IK.f...
00000b80: ac13 c02a 5e00 e6e6 4e97 aaae 30eb 2c97  ...*^...N...0.,.
00000b90: 1d05 1fa0 94ad 7236 8466 c0c6 699c e324  ......r6.f..i..$
00000ba0: 7e7b e8b9 dad1 3f82 d389 d2c1 3dc5 9792  ~{....?.....=...
00000bb0: 8604 5060 854d 4705 916c 5084 e48e f6fe  ..P`.MG..lP.....
00000bc0: f078 c4d3 5a6c 7892 d90f ecc9 8ec0 7550  .x..Zlx.......uP
00000bd0: 83ed 92c7 f2d0 11a6 bcc4 3801 9cf0 20cf  ..........8... .
00000be0: 9a50 f55e 6405 46ef eb09 08ff c6b5 0e7e  .P.^d.F........~
00000bf0: 6224 a702 58ac 3bce 401d f692 cc59 ff8d  b$..X.;.@....Y..
00000c00: fb88 9269 f2f5 586c 3015 47c1 9a8a 0df8  ...i..Xl0.G.....
00000c10: 5c03 8133 1c9b 75c8 0032 b28b 206c 039c  \..3..u..2.. l..
00000c20: 4129 d0a5 d222 d3d2 5755 352f 96cb 2ab3  A)..."..WU5/..*.
00000c30: 550f 30c2 c996 e9cd 50bd f0ef 9262 3f99  U.0.....P....b?.
00000c40: 1f07 a096 3829 15fa c507 492b 4527 94e4  ....8)....I+E'..
00000c50: b3df 33e7 f1cc 4dff f974 c0c6 e72f 91ff  ..3...M..t.../..
00000c60: 4ee1 2542 0a8e eea6 00b1 c683 9868 84f2  N.%B.........h..
00000c70: 3386 22b4 0616 1fe1 9b68 0127 c43a 8f00  3."......h.'.:..
00000c80: 9416 f0a4 2a73 1a54 0009 565a e51a eecf  ....*s.T..VZ....
00000c90: 6561 47c7 c07c 1e86 f74f 0b58 0d53 3fc3  eaG..|...O.X.S?.
00000ca0: fc1c 5978 4404 b5b8 e6b7 05f7 37d3 3c73  ..YxD.......7.<s
00000cb0: 75db 4615 6dba de61 a0e6 5de9 7853 c333  u.F.m..a..].xS.3
00000cc0: 5268 3952 bf67 8379 85bf 11f0 82d3 d754  Rh9R.g.y.......T
00000cd0: 9d03 8c33 6729 538f 1e94 5086 d84f a1b8  ...3g)S...P..O..
00000ce0: 50dc 9f02 68d0 1ad9 9beb 98ce 44a8 567b  P...h.......D.V{
00000cf0: 1a25 eee2 7495 4a2c 5361 fdc6 5e20 503f  .%..t.J,Sa..^ P?
00000d00: 1dbc 36f7 8d77 6417 d756 29f8 09e6 e459  ..6..wd..V)....Y
00000d10: cdb0 490c f6a8 99e8 92a4 1642 8133 d323  ..I........B.3.#
00000d20: aa58 fce2 ce15 6f0f 3d07 b336 c592 13db  .X....o.=..6....
00000d30: e195 6aab f4d9 c917 7e4d b50a f757 3e62  ..j.....~M...W>b
00000d40: 1590 f3c1 7a58 40c9 36a5 ec16 480a 9672  ....zX@.6...H..r
00000d50: 4740 0b91 516b b6cf 694c 6caa f495 cfb1  G@..Qk..iLl.....
00000d60: f087 fbfb ddb5 055f 131c 2c7f 6e43 61a1  ......._..,.nCa.
00000d70: 76a7 dfb3 066e 79ec 1033 b3e6 b5fd 1ba3  v....ny..3......
00000d80: 71e8 aa7e 34fc dabc b85c 1b44 3612 63ba  q..~4....\.D6.c.
00000d90: c035 adf5 07eb 9f28 e264 9dc1 5c3f ea92  .5.....(.d..\?..
00000da0: 70f4 b21c ed39 0e19 410d 31b8 a2d3 cbe4  p....9..A.1.....
00000db0: 7f2b 4806 6364 cfae 613c 68ef 8549 152c  .+H.cd..a<h..I.,
00000dc0: ee89 d0da 37cd 91e4 fcbf 138a 2c1a 4332  ....7.......,.C2
00000dd0: a9b6 f30b eaf0 1d0f c76c a517 f74a 6b13  .........l...Jk.
00000de0: d8a9 2f6d a1bf 4ce9 74c0 be43 34a9 fce6  ../m..L.t..C4...
00000df0: 6a88 660b a3df c34f 9be1 5cbd cec7 bc29  j.f....O..\....)
00000e00: dbb1 628a 55c1 1752 8f1f 899b be7b 00ef  ..b.U..R.....{..
00000e10: 92c3 bb68 8008 0abd c309 848d 5b4f 602c  ...h........[O`,
00000e20: e2dd ad71 b274 39af 4f18 45aa c28b d90b  ...q.t9.O.E.....
00000e30: 3d46 e22d dc89 4f2b bf92 d404 ee5f 056f  =F.-..O+....._.o
00000e40: c759 c418 3c06 4205 4d57 8b5f fd21 55fa  .Y..<.B.MW._.!U.
00000e50: d2c2 a89b 96a1 b44c f395 404f 7f15 85d3  .......L..@O....
00000e60: ddc2 68ad b563 ba26 3fe6 ec89 925a 755c  ..h..c.&?....Zu\
00000e70: 01c4 3e2f db4a 2894 e4a5 ca60 8f45 1790  ..>/.J(....`.E..
00000e80: fc8d 3222 398e 492e 515b 32da 1ceb 719c  ..2"9.I.Q[2...q.
00000e90: 832a cb25 6a1a e964 96c9 682c d7ec 9dc3  .*.%j..d..h,....
00000ea0: a97a 2939 06a6 13c8 96dc c3a2 84c3 a514  .z)9............
00000eb0: 77a1 c79f 57d6 fab7 16b3 d34c 25a2 2672  w...W......L%.&r
00000ec0: fd73 c507 e3dc c1f8 3d58 0034 8ad4 8f89  .s......=X.4....
00000ed0: aaa3 5dd4 fb14 f396 e221 5511 9d6b 91b3  ..]......!U..k..
00000ee0: 249e 34ff 5ed0 68f2 be2b 8335 33ad 3546  $.4.^.h..+.53.5F
00000ef0: 7b40 5e94 2504 5c20 151d 6f3a 3d04 0656  {@^.%.\ ..o:=..V
00000f00: 5e1d 4759 2bc7 fb1e 3d77 3e84 c24d b05c  ^.GY+...=w>..M.\
00000f10: af26 cd21 d819 a70a d068 ae02 9764 9ec6  .&.!.....h...d..
00000f20: edc8 092e 64ea ed1a 1297 a3ba 54b3 c015  ....d.......T...
00000f30: e62d b6f0 af70 5e3f d19a 659e a172 6d63  .-...p^?..e..rmc
00000f40: 7de3 243c ca40 79c9 e023 6900 a691 e178  }.$<.@y..#i....x
00000f50: b0b2 71f9 e00e fc7a ec7f 06d5 9842 05e8  ..q....z.....B..
00000f60: 257f 09b3 4e51 916d 7bd4 9f00 3352 9e01  %...NQ.m{...3R..
00000f70: 8d40 1793 3674 c314 bc56 5a83 3a91 07c5  .@..6t...VZ.:...
00000f80: d128 3731 d791 6c52 e563 3947 6f85 2abf  .(71..lR.c9Go.*.
00000f90: d0cc 90f4 aa63 40c7 fa17 bc21 6a3a 70b9  .....c@....!j:p.
00000fa0: 6d00 c65e d19d ebdb eafa 2585 ad99 4105  m..^......%...A.
00000fb0: 50b0 201d bdc3 a1b6 b323 507f 000b 548a  P. ......#P...T.
00000fc0: cc8f 2a5a a9ce 9a28 26f0 da58 f277 24f0  ..*Z...(&..X.w$.
00000fd0: 824f 3b40 00d1 4c65 e87b 1fe6 1d4a 9cbc  .O;@..Le.{...J..
00000fe0: 012f f508 bede 6b08 4500 6eb5 3540 493b  ./....k.E.n.5@I;
00000ff0: d5c5 e009 33ff 9e94 0938 569f d1e3 a02b  ....3....8V....+
00001000: a980 ac34 47cb a39c 2fcd 195a b217 6dea  ...4G.../..Z..m.
00001010: bafe dfbd 69ac 9718 f034 4366 d88f 5765  ....i....4Cf..We
00001020: 065b 1ccf ec76 462c 85eb e026 2a7c 2276  .[...vF,...&*|"v
00001030: 0303 9813 d916 53fb c2ee 2f12 aa58 77fb  ......S.../..Xw.
00001040: 0e8c 8604 032b c58b 86f7 93a6 e596 ce70  .....+.........p
00001050: ea0c 286e a547 53f4 0b22 7518 12cb bc6a  ..(n.GS.."u....j
00001060: d87d 6af5 958b 8077 3974 ba55 018e 93da  .}j....w9t.U....
00001070: 4ff1 bde1 cf4c 52a2 f701 89d6 b652 296d  O....LR......R)m
00001080: 2f9e b7de 0802 20c1 df7b d3ec 72e6 57de  /..... ..{..r.W.
00001090: d313 8514 c722 5d95 49d0 e72d fa05 7a82  ....."].I..-..z.
000010a0: d294 5a1b 31c2 bd36 6176 eb79 b05b 9b85  ..Z.1..6av.y.[..
000010b0: 9069 a9bc b0c3 08c6 25d0 8b11 93ec 630c  .i......%.....c.
000010c0: 3bcb ad63 95d0 8016 634e ec95 be6f 91e9  ;..c....cN...o..
000010d0: 1915 b038 8e4d 008e 25a2 852f 3641 4ca3  ...8.M..%../6AL.
000010e0: a794 8f1f bfd6 4783 3316 9b66 2548 e4a6  ......G.3..f%H..
000010f0: a29f 3b0e 05c8 03c3 f48a 02b3 29bd 4178  ..;.........).Ax
00001100: f384 4b4c a4b0 311a ace7 1776 e881 476a  ..KL..1....v..Gj
00001110: c9b1 cbac 9ebd 4d40 c6bc 8768 9f2a 0053  ......M@...h.*.S
00001120: 8652 92d4 d26c 8007 3e23 b623 4061 8f09  .R...l..>#.#@a..
00001130: 62f1 e389 0388 fdc6 9a27 a735 0893 f34d  b........'.5...M
00001140: 31d6 61ac 5fe6 5a48 0ced 371c aa34 5b5b  1.a._.ZH..7..4[[
00001150: b84d ee5b ba77 79d8 d94d bfdf df7d ee01  .M.[.wy..M...}..
00001160: 5092 405a 3095 ac02 f534 d54c c90f 8fda  P.@Z0....4.L....
00001170: cbea 1c1c 3854 33ef 1ef2 9ea1 667a 54fa  ....8T3.....fzT.
00001180: 3fa5 282f 220e 6173 c7c7 a007 0da9 90b7  ?.(/".as........
00001190: 8abf b75a 028d fdb0 5aca 9c6b ecdd 357e  ...Z....Z..k..5~
000011a0: 3f33 2a87 8f23 fe1d 6c96 e0a0 ef76 a4ec  ?3*..#..l....v..
000011b0: 9a10 edbc 431a 049d 2a46 d22a 5c62 b75c  ....C...*F.*\b.\
000011c0: 1f41 13e9 fcb0 8527 33a5 c1fe 0944 7d71  .A.....'3....D}q
000011d0: f837 0a22 cd26 2426 12f9 25eb 7a7e 08af  .7.".&$&..%.z~..
000011e0: 93e5 f6a1 7f07 b21a 4cce d9de 8653 f1e3  ........L....S..
000011f0: 5544 886d 1dfd 2276 9e60 2066 ae60 b957  UD.m.."v.` f.`.W
00001200: 3a74 1297 8eda 842c 0712 55b4 0a64 1337  :t.....,..U..d.7
00001210: 6ffb 9ab8 c7ec 2203 13ed d27f 94cb bff8  o.....".........
00001220: 622e 1ebb 4d74 e176 17bf bdf1 20ec 6bf5  b...Mt.v.... .k.
00001230: 6625 72fc 6a2b 8e63 7c6f 20ef e6c6 39e3  f%r.j+.c|o ...9.
00001240: 8e12 66d6 3228 41a5 1d3a 4fa4 15fd b463  ..f.2(A..:O....c
00001250: ff95 1c9d a3ea 3e19 9f9a 426b c9f1 cd87  ......>...Bk....
00001260: 9d5f 0c7e 70f8 421e 7c6c 1864 7820 a523  ._.~p.B.|l.dx .#
00001270: 69e6 24b5 8cdf 8988 b2ff 0487 e7ad 148d  i.$.............
00001280: 68a4 803e 4a49 87b0 ca85 2cc6 034f bf14  h..>JI....,..O..
00001290: e9f3 05ee 1d58 6751 63bb da10 1d11 706f  .....XgQc.....po
000012a0: 3de1 b741 6fb8 f18c 6fbd f094 4873 f52c  =..Ao...o...Hs.,
000012b0: a794 3623 ea7b 4ea4 9cb4 9cd2 07fe 66a9  ..6#.{N.......f.
000012c0: e5a0 5bc8 0af6 36d5 b1cc 94a4 fae3 886c  ..[...6........l
000012d0: 06f8 5cdc 3782 3797 b7e7 da80 eb5b 2a9a  ..\.7.7......[*.
000012e0: 39f0 88fb 9597 9cd5 9081 a5b3 845b c1cf  9............[..
000012f0: 4fae bfcb fc37 f829 a1f9 963d 59ce 6a16  O....7.)...=Y.j.
00001300: fbc6 b903 6fb0 4bf6 9e0c 9b69 7a6b 2648  ....o.K....izk&H
00001310: 253f fdf0 7dc0 62f9 44f6 b60a 60fc 1e79  %?..}.b.D...`..y
00001320: 426a 04bb 4813 4615 d82e e379 f6f0 5e05  Bj..H.F....y..^.
00001330: ce90 e82d 7d02 a16b d9a4 671d 4bfd fd25  ...-}..k..g.K..%
00001340: dc7e aa47 ae1a 36d2 133f f437 bf7a ad6d  .~.G..6..?.7.z.m
00001350: 1bcc 686d e7b1 01d7 cb1b c928 48ba 4020  ..hm.......(H.@
00001360: f186 236b 1140 e511 1586 2531 7cbb a810  ..#k.@....%1|...
00001370: 645b d0d9 c110 f420 7576 962f 5543 4a46  d[..... uv./UCJF
00001380: 2dbd 4e46 d895 7e76 9ec5 2f82 143b eddf  -.NF..~v../..;..
00001390: 4ed4 906a 83e1 9089 a308 cffc 0c9b 0335  N..j...........5
000013a0: 00fd eda0 34f9 3614 e8f2 8403 d257 0252  ....4.6......W.R
000013b0: c00f 759d ab7f b281 fc56 bccf afdf eda5  ..u......V......
000013c0: f8d5 a907 9978 e3f6 7f2f e7a2 8797 724d  .....x.../....rM
000013d0: 859c c822 bd61 77f0 178b fe9a b942 eb78  ...".aw......B.x
000013e0: e68f 7b7c 9c8a 846c 1892 25bd 2ae3 12e5  ..{|...l..%.*...
000013f0: d4e6 4d01 e7b4 515d 580b ff0d cbd5 fd0a  ..M...Q]X.......
00001400: 527f 0cc6 c6cc 0ef2 d7fa e4ad 24ae 10ae  R...........$...
00001410: e47f 965c 0a6c d671 30a2 45a8 0ab1 418a  ...\.l.q0.E...A.
00001420: bdfd b019 10d6 ac6b 9e3c 5f6f 2382 be87  .......k.<_o#...
00001430: 9ffa 1eba 8157 511f 6c56 c4f4 7144 bd68  .....WQ.lV..qD.h
00001440: 0a26 002d 58d9 6cf3 2dce 3898 aa64 05e0  .&.-X.l.-.8..d..
00001450: 4443 6e02 87                             DCn..

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=19456
pts_time=1.583333
dts=18944
dts_time=1.541667
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=3128
pos=57567
flags=__
data=
00000000: 0000 0c34 419f 2445 152c 5700 0003 0000  ...4A.$E.,W.....
00000010: 0774 ffa8 2af7 1c91 7c52 145b 152b d78b  .t..*...|R.[.+..
00000020: fe11 0419 41b6 ab9b 7f76 f2c8 823e d29f  ....A....v...>..
00000030: 01c0 58b5 dcb0 075f 8b5e 6439 4693 6afd  ..X...._.^d9F.j.
00000040: e772 bd37 4fa4 6498 86ce 6023 19fe 3a70  .r.7O.d...`#..:p
00000050: def6 0ac5 5174 bff8 fbc7 0f60 cbb5 c96a  ....Qt.....`...j
00000060: 3a89 dd05 bbb9 af21 1bef 48d3 0c82 297a  :......!..H...)z
00000070: 4b07 24cf 10aa 3f7b 96ab 2543 45fe b1a6  K.$...?{..%CE...
00000080: 34bc 7979 53c5 b59e b656 9472 958a 1982  4.yyS....V.r....
00000090: 0932 c9cd cb0b e283 ca9a 5660 fc69 24bc  .2........V`.i$.
000000a0: 03b8 9a50 366a 7052 caa5 267c b1da d3a3  ...P6jpR..&|....
000000b0: 3a10 4d40 9de9 70a5 de90 a832 69ac 15fb  :.M@..p....2i...
000000c0: ac5a 42ad 7ef2 1ae8 2bf1 65f4 7f05 aa5f  .ZB.~...+.e...._
000000d0: 374a 8208 bb91 3d5c 50bc 31c8 b0b5 3129  7J....=\P.1...1)
000000e0: db6f 163a 59bb 4290 0a70 6b15 4119 3fbf  .o.:Y.B..pk.A.?.
000000f0: cfe6 182f ac2b b4a7 5338 42e2 0eed 90bf  .../.+..S8B.....
00000100: 8a30 d0a0 5f12 c5bb bc2d b5ed f9dc 18ca  .0.._....-......
00000110: 50db cdb8 687d 531b 43a6 c3e8 3129 2ced  P...h}S.C...1),.
00000120: 491c cdcb d2a6 452c 5c87 c951 971d a00d  I.....E,\..Q....
00000130: 79a3 46f2 2245 3788 cd39 cf0f f673 2432  y.F."E7..9...s$2
00000140: bfb2 bea6 1b08 c4e3 ac26 48bf faee b828  .........&H....(
00000150: 7975 6a2f aeed 69d7 b3b5 3e51 c7fe b100  yuj/..i...>Q....
00000160: 6630 c52a 4d55 8664 fb03 da8f bec8 9433  f0.*MU.d.......3
00000170: fb47 3922 49a5 4034 039b 9794 64a6 e5bc  .G9"I.@4....d...
00000180: f348 1651 1a1a 605c 4a23 e0d6 7a35 fe58  .H.Q..`\J#..z5.X
00000190: 42b1 ec22 3889 6633 a8e2 e633 6321 7edc  B.."8.f3...3c!~.
000001a0: 036a bdab de2e 18d2 8a4c 584b 1e1a cd90  .j.......LXK....
000001b0: 98cb 92e7 9de0 26c1 c289 75a9 e53c d0c6  ......&...u..<..
000001c0: 53b1 872c 12f3 6874 673a b48f bacc 7761  S..,..htg:....wa
000001d0: a703 ddef 33bf 0580 5f37 ead6 f692 17ed  ....3..._7......
000001e0: ad79 d321 f8bd 7224 dfd0 9e02 1657 72a4  .y.!..r$.....Wr.
000001f0: e6af 5349 acf5 e1ea 47e6 e300 fd88 5855  ..SI....G.....XU
00000200: 395a ca6a e7ce 1ba4 5337 432a 5e35 bd60  9Z.j....S7C*^5.`
00000210: 44c1 8a8a c579 16af 678e 7e22 5c22 ac76  D....y..g.~"\".v
00000220: fe75 f694 82f8 2427 fc8f 22b5 e765 32ac  .u....$'.."..e2.
00000230: 9c6c efd8 dc37 2772 ccb8 f8ea 054b 8021  .l...7'r.....K.!
00000240: 94a6 4f4c 4835 174e 5d5f cdd2 6c40 0da0  ..OLH5.N]_..l@..
00000250: 0747 0c8a 6888 9d7a 55b6 8295 c0d4 e587  .G..h..zU.......
00000260: 2be1 4a53 7bbe 5c26 fdaa 9f9a 3f14 be3f  +.JS{.\&....?..?
00000270: 0ec7 7eea 685e 7b4d cf5c 6708 3dac a5ff  ..~.h^{M.\g.=...
00000280: 9b3f 5a3e af14 e4ab 911b 87c2 cefa 48cd  .?Z>..........H.
00000290: c6fd f064 7bc2 2fe9 2300 001f 629b 8563  ...d{./.#...b..c
000002a0: da73 ed66 37c0 9cc0 a6c8 e3e8 6985 8da3  .s.f7.......i...
000002b0: 680f 3ff1 fad1 9f4d d711 8bf1 a9e6 fcb4  h.?....M........
000002c0: ca29 ad58 85e5 db2e 17c5 fdbd 20fa 9412  .).X........ ...
000002d0: 5b85 e6b3 bff6 8613 b727 83ef 156e 0484  [........'...n..
000002e0: 174a ba43 e322 ea07 3db7 25ef 3c50 e5b3  .J.C."..=.%.<P..
000002f0: 7dea 2c42 b839 bf87 08fa 0672 17f2 6218  }.,B.9.....r..b.
00000300: a359 6353 e09b c7d5 bb2c 6617 24f4 91e4  .YcS.....,f.$...
00000310: fc8e e079 570b 1fb3 1612 e7f9 c8b6 a69d  ...yW...........
00000320: 5400 a956 d2f9 ae1f c727 3b1c 4dd8 bf84  T..V.....';.M...
00000330: af8b 58c6 49c4 3d7e 3904 90b0 ecdc 7d50  ..X.I.=~9.....}P
00000340: 0d80 6d78 d2ec 4edf 402f fc36 37b1 4f4f  ..mx..N.@/.67.OO
00000350: aa77 5a1b 725c 8298 329e e04f 93e4 17d7  .wZ.r\..2..O....
00000360: 8902 fa64 869b b13f cf6c 4325 acbb 38e6  ...d...?.lC%..8.
00000370: 2c28 1f82 cf98 f9fe bc5f b150 2d78 b2bb  ,(......._.P-x..
00000380: bab0 6076 d759 3b8b c6bc 8193 ec2f 5507  ..`v.Y;....../U.
00000390: 71b4 78cb eba5 461b 7ee9 ff68 86d7 c363  q.x...F.~..h...c
000003a0: ec2b 5608 da78 ed1e b06b 65ee 04e0 67c9  .+V..x...ke...g.
000003b0: e7de 5120 d399 54ae 7cce 65b7 e768 b708  ..Q ..T.|.e..h..
000003c0: e485 7f28 6e7b 5b4c 5234 6966 addd fdda  ...(n{[LR4if....
000003d0: 625b 0769 bf9c 1253 d2c9 bdfc bf29 59c6  b[.i...S.....)Y.
000003e0: 0ee3 3cc3 9a3a f0d2 3f74 3f92 d172 c8eb  ..<..:..?t?..r..
000003f0: df32 b3b8 feb3 06b8 fda2 26b1 c4b2 bb06  .2........&.....
00000400: cd3f e959 349c d2a7 0a7a 49bf 9243 3d57  .?.Y4....zI..C=W
00000410: 233c bfb7 4f74 b67f 80ad 212d 8b0f e01f  #<..Ot....!-....
00000420: 013f b4f8 98f1 125f 862e b011 2c14 6a0d  .?....._....,.j.
00000430: 7118 202d b6b6 058a 149e 6877 9a7f 18a1  q. -......hw....
00000440: de27 cc40 b18c 3b80 81c3 0b08 88b3 76b7  .'.@..;.......v.
00000450: b2a8 0098 62d8 b03a c471 7a6c d26e 4906  ....b..:.qzl.nI.
00000460: c948 3088 5a65 2d70 3158 ed94 e38c 06c0  .H0.Ze-p1X......
00000470: 8808 5975 a4bf 2682 8b97 21b0 b029 2e77  ..Yu..&...!..).w
00000480: a72d a46e 3f82 25ac d4ca 00d3 edb1 b030  .-.n?.%........0
00000490: 05ff db47 a8c9 5de0 8c78 3dbb d50a 5f63  ...G..]..x=..._c
000004a0: 4f8e 6a93 a5f3 586b 1d90 d1a5 fba9 3e6a  O.j...Xk......>j
000004b0: eec7 982c c110 b8c1 a83a 0a49 3361 dde7  ...,.....:.I3a..
000004c0: 5a12 44ce 4b3c 61e6 a34e 8afb d798 3828  Z.D.K<a..N....8(
000004d0: a207 f3f0 d74e 5a2b 05ee 050a ac0f fb99  .....NZ+........
000004e0: 9821 0d12 f488 06bf 452d 5d66 f408 df58  .!......E-]f...X
000004f0: 9f4f a966 d393 b07c 0a1f 862c 21ab 06cf  .O.f...|...,!...
00000500: 3faa 4629 2ef5 12b2 4525 c374 d0de 5c71  ?.F)....E%.t..\q
00000510: 402d 65f3 01f6 616f da11 b22b 3f5e d9ac  @-e...ao...+?^..
00000520: 75fd 9444 115f 50d9 8014 fa98 2dfe 6ccd  u..D._P.....-.l.
00000530: 9530 7ae1 7b89 29a3 1761 71d5 71b1 6e05  .0z.{.)..aq.q.n.
00000540: 7947 1813 e1c9 3b5d 957d 5717 bc34 25bc  yG....;].}W..4%.
00000550: aca9 42a3 68a1 69bc 3821 1869 917b 082d  ..B.h.i.8!.i.{.-
00000560: efd9 9264 a3b2 b52e af5b 2170 652f cbd4  ...d.....[!pe/..
00000570: 5f36 a066 7b79 e657 e360 1322 1688 ea9b  _6.f{y.W.`."....
00000580: 7b9b 9b68 7b93 f78c bacb fefa c884 1555  {..h{..........U
00000590: 5af9 be63 ce2a 0a0f 2f4d 23c9 5fa3 d357  Z..c.*../M#._..W
000005a0: a823 de7f ee61 90d2 a2e3 6502 8d09 ffb7  .#...a....e.....
000005b0: baac cf9f fd39 0562 6208 e6e5 5bee 71b1  .....9.bb...[.q.
000005c0: d156 b11a b7fe 8b5e 1118 6733 7898 27db  .V.....^..g3x.'.
000005d0: 16b3 1016 f4fe d27b 812e a973 e37d a49f  .......{...s.}..
000005e0: ef17 f9c9 055f 6232 ed3e 4701 e008 0b2e  ....._b2.>G.....
000005f0: 51cf f7c5 edc3 6747 28d7 531b dd39 62b6  Q.....gG(.S..9b.
00000600: 0023 eac0 655f 6dc3 74ef 767e 6b0e 6d56  .#..e_m.t.v~k.mV
00000610: 6f76 5264 d635 7881 beab a49c da55 4a58  ovRd.5x......UJX
00000620: 0f60 ace6 567a 1758 033d bf0f 1392 73bf  .`..Vz.X.=....s.
00000630: 707e 3bcf c9ae a254 47cd a2f3 c112 aef9  p~;....TG.......
00000640: cf38 7ee0 2486 b3f4 b277 0dff 5667 70a8  .8~.$....w..Vgp.
00000650: 2fdc 1fb2 c6ac 0be0 a777 e5d1 670b bb58  /........w..g..X
00000660: bfed 2d0f c174 01ad e8df fcb1 0f90 e2a8  ..-..t..........
00000670: cc1f 0ec2 0dc3 d586 eb10 8377 1b21 3de9  ...........w.!=.
00000680: b737 b5f1 c5cc 2504 e813 17ed a3c3 b436  .7....%........6
00000690: 6de9 0ae5 dd76 1969 3450 5670 19d8 8a26  m....v.i4PVp...&
000006a0: 6e41 702d 7307 f749 363a 2938 a90f 783c  nAp-s..I6:)8..x<
000006b0: 9c3b 28c5 6cd2 efb8 4f7b acc0 8e36 0f00  .;(.l...O{...6..
000006c0: 1808 1f06 5066 f521 47c7 b589 b039 5690  ....Pf.!G....9V.
000006d0: b409 0851 b84d 57d6 8477 feba cd34 f6a7  ...Q.MW..w...4..
000006e0: 77fb bed0 cc4e 814f a155 fb13 5c5b b2ca  w....N.O.U..\[..
000006f0: 4268 e96d 06b8 57fc f145 5af7 a4a5 dec6  Bh.m..W..EZ.....
00000700: 6996 16d2 6a05 a70a a75b 6564 a6e5 e614  i...j....[ed....
00000710: 32aa e110 24d8 b1ae 23bc dac1 5d57 096f  2...$...#...]W.o
00000720: ac3e 4d0a edd2 c8cb bf56 5e39 2f8e 1168  .>M......V^9/..h
00000730: 4529 071e cad9 541e 93e3 fa44 86aa 63b6  E)....T....D..c.
00000740: 0f79 c458 f2bf d864 9211 98eb c7f5 0b5d  .y.X...d.......]
00000750: 90b8 774c cd1b b855 b177 eb9e dee6 f173  ..wL...U.w.....s
00000760: 0fd7 532e 2a2f e779 c4e0 284d e52c 21e5  ..S.*/.y..(M.,!.
00000770: f415 99ae a2bd e366 c458 3967 f1fa 66e7  .......f.X9g..f.
00000780: 739f 81e2 a1d9 aa39 e144 0338 a16d 6c2a  s......9.D.8.ml*
00000790: 6e4e 92da 3f5c 4a5b 7878 f481 6231 cb13  nN..?\J[xx..b1..
000007a0: c799 842f f33d 5ca2 65e4 30be 7a2d 11c6  .../.=\.e.0.z-..
000007b0: 4fb0 1d27 647f a897 180a 3941 58dd 1cd4  O..'d.....9AX...
000007c0: 76be 8702 3efa 8d11 c086 0657 1043 948f  v...>......W.C..
000007d0: e034 1f2c b6d4 b338 199c 420b 35c5 cd3e  .4.,...8..B.5..>
000007e0: a397 a4ed fa08 9e66 4981 5ce7 44eb 9766  .......fI.\.D..f
000007f0: c6b0 3eea 47f0 c53a e588 43fb 0292 adbf  ..>.G..:..C.....
00000800: 8296 fa64 0485 d8f6 89e2 9c69 04e6 5ff0  ...d.......i.._.
00000810: ded7 be05 ad35 3cf4 e3d0 e381 f8e1 be25  .....5<........%
00000820: d31f 5d6b a2e5 6958 0bae 9271 90f3 5eb8  ..]k..iX...q..^.
00000830: 4bda 8342 8e3a f08a 4a9b 1719 0832 2226  K..B.:..J....2"&
00000840: 45af 0023 56a3 cd64 a71f c473 b099 7692  E..#V..d...s..v.
00000850: 6be5 c73b 0f5a d998 5a4b 8e4a 24ab c2dd  k..;.Z..ZK.J$...
00000860: 1cae b2b3 2751 3be2 a098 1e24 3c90 f955  ....'Q;....$<..U
00000870: 51db 123e 4c5d 4689 2803 ae3c db36 5f83  Q..>L]F.(..<.6_.
00000880: e1e1 7335 db5f 0585 2dcb ffc6 ac0a b6a2  ..s5._..-.......
00000890: 7c3f 3f5e 4817 60ca 31ec fa2a 315d b36f  |??^H.`.1..*1].o
000008a0: cd31 8566 aefd 1a1a a7b2 3caf 448c 2010  .1.f......<.D. .
000008b0: 4e7b a27b a0b0 c813 6ee6 b63d 66fa 33fe  N{.{....n..=f.3.
000008c0: 02b0 4dfd 66bf 54e8 c4c8 a1ed e016 7a83  ..M.f.T.......z.
000008d0: 4502 bf9e ee5f bee3 7470 cd09 aee8 7c44  E...._..tp....|D
000008e0: c068 1df4 0e71 0e72 fcec 3b37 a613 4ede  .h...q.r..;7..N.
000008f0: dbd9 f8eb 1fa5 bb95 9f8c 14e4 f8da 0d35  ...............5
00000900: 333c d3ef 9ab3 9033 d825 2136 85c1 cf40  3<.....3.%!6...@
00000910: bbb0 4692 a7f6 c513 b913 f61d 22fe 4f83  ..F.........".O.
00000920: c362 1e6a 25bb e5de ce2d 33bf e3da c48c  .b.j%....-3.....
00000930: 1911 9fd6 cad9 ec6b ddbe 2d53 6fde 0e75  .......k..-So..u
00000940: 9a18 0f15 299f ea50 8f79 a04a c70c 958e  ....)..P.y.J....
00000950: bf96 7202 36f8 ccda 7875 a327 2fde d296  ..r.6...xu.'/...
00000960: d30a 737d 9e90 abcb e86c 088c f073 1769  ..s}.....l...s.i
00000970: fcc2 27a7 bc53 45ee 42c6 e0ef 69e8 670d  ..'..SE.B...i.g.
00000980: e6e1 e136 fb91 433c 8f32 14b7 aee6 495a  ...6..C<.2....IZ
00000990: 7483 7995 786b b36a c8fc 4bd4 6db0 4426  t.y.xk.j..K.m.D&
000009a0: df0a e387 5496 9134 b682 da97 7e63 4d28  ....T..4....~cM(
000009b0: 0b0d 5028 055d bd0c d801 5d86 f2d1 fb83  ..P(.]....].....
000009c0: 64a4 7560 3f56 d48c 6c4e 819d 8511 7782  d.u`?V..lN....w.
000009d0: 0921 26d0 e5f7 548b 26d3 b2e7 bfe5 8feb  .!&...T.&.......
000009e0: 5d4d 4c72 8410 d473 2e52 26d7 30f3 571f  ]MLr...s.R&.0.W.
000009f0: c6de f1e0 5ef5 7518 18e3 55dc c8e6 fe41  ....^.u...U....A
00000a00: 27e3 c00b 5c60 677c 5423 75a9 9d3c 7222  '...\`g|T#u..<r"
00000a10: 26c1 664e 5e0d 5373 5827 c410 5f2e 0209  &.fN^.SsX'.._...
00000a20: 53c9 d52c 74ef edd1 822e a5e8 f669 1e24  S..,t........i.$
00000a30: 02e8 9885 cc96 a4e9 7cb2 1630 af20 2835  ........|..0. (5
00000a40: 810c c82b 3f6c fbf3 1921 89ee ac4a 2cf4  ...+?l...!...J,.
00000a50: 3029 1c03 614b 3b4a 7887 27aa 7557 7b3e  0)..aK;Jx.'.uW{>
00000a60: 3116 7937 9f32 9be9 0fa0 5f56 52b7 05e7  1.y7.2...._VR...
00000a70: 0a4e ca60 5b7c 6f29 104f 5692 13b7 3bd3  .N.`[|o).OV...;.
00000a80: f288 7d14 4eea fd6c 6992 703a 20a9 3eaa  ..}.N..li.p: .>.
00000a90: 52d4 0c81 dd5d 624c 1b51 04f8 f873 068e  R....]bL.Q...s..
00000aa0: faf7 5d48 70cb 4da9 97c1 4e92 2a67 9d93  ..]Hp.M...N.*g..
00000ab0: cfba dbef 7d07 a204 efde 8c5c d036 ea74  ....}......\.6.t
00000ac0: a1b6 4400 27fd d58d 6eae ad13 309c e4df  ..D.'...n...0...
00000ad0: 5508 3f49 0938 b8f0 3137 7a85 4e0b fd9b  U.?I.8..17z.N...
00000ae0: 3a71 6ca1 2789 63a5 9c13 7568 b446 4eb0  :ql.'.c...uh.FN.
00000af0: 3f8c c080 5d26 5084 1994 2819 7407 0da9  ?...]&P...(.t...
00000b00: b33d 0348 ff1c af07 5b4b 2575 2cb4 f9db  .=.H....[K%u,...
00000b10: ba01 449b d2b7 bcab 8fc6 b82f 2e64 0d1c  ..D......../.d..
00000b20: db89 316c 2521 1a15 2992 6e96 5a7c c0db  ..1l%!..).n.Z|..
00000b30: 5cd4 3f8c b5d3 ed5b 172a f600 72a9 163a  \.?....[.*..r..:
00000b40: 7ffb 0839 5517 f4df 4b26 3b33 8c5a 6e67  ...9U...K&;3.Zng
00000b50: 310b 5856 636b fe75 ee0b 84de 0eb2 ebee  1.XVck.u........
00000b60: c068 bb63 30cc 5028 8589 5df3 9a49 66cd  .h.c0.P(..]..If.
00000b70: 3a33 f160 42b0 4e96 40f0 7773 b793 6d9f  :3.`B.N.@.ws..m.
00000b80: 1d48 fd69 1b8c 96d3 0650 793d 1361 7039  .H.i.....Py=.ap9
00000b90: 2bb2 01e8 f665 d64d fb8c c8bf 28b4 ab0f  +....e.M....(...
00000ba0: 0833 bdc9 1b77 e90d e75e 89ae 370a 6dea  .3...w...^..7.m.
00000bb0: 3e86 d392 7a59 9ac7 d898 f9f5 b5c8 e451  >...zY.........Q
00000bc0: 261d 09d8 fd68 0a94 97f2 a9e8 2205 9d35  &....h......"..5
00000bd0: 5041 6d00 a552 2cd9 c5d1 f676 00c6 1b56  PAm..R,....v...V
00000be0: 81f0 db81 e8b4 a88d c938 71ab 643e cbe0  .........8q.d>..
00000bf0: a963 4aca 38e5 62d5 278e 166f a69f c710  .cJ.8.b.'..o....
00000c00: 434b 840b d302 9931 12e6 5b8c d8d9 c49c  CK.....1..[.....
00000c10: 2379 c4a3 f202 80df 383f 113f 0f90 da42  #y......8?.?...B
00000c20: 0b0e b360 aca6 0f1a a439 47e7 8914 5c00  ...`.....9G...\.
00000c30: 0003 0000 0300 1771                      .......q

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=19968
pts_time=1.625000
dts=19456
dts_time=1.583333
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=2596
pos=60695
flags=__
data=
00000000: 0000 0a20 019f 4544 5700 0003 0000 07db  ... ..EDW.......
00000010: b20c 2805 1c61 eea2 b765 b32d 1582 ba4b  ..(..a...e.-...K
00000020: 35f7 8815 62b6 bee6 0cfe 58d5 e2b4 8112  5...b.....X.....
00000030: 5da4 2040 5b87 802f 3eca 6200 0277 580c  ]. @[../>.b..wX.
00000040: f671 3000 0434 a38a cce0 0002 17b9 e430  .q0..4.........0
00000050: 1b98 7c32 cea0 6fbe db1c 34ac 85fa 0a80  ..|2..o...4.....
00000060: 2a7d 5dde 974e d20f fdec c606 f7d3 ece9  *}]..N..........
00000070: f258 9895 7133 eeed 6227 0219 dd42 51a4  .X..q3..b'...BQ.
00000080: 1102 0bc5 5126 a50c b1c2 c890 da80 af4a  ....Q&.........J
00000090: ca13 9799 aaa6 ade3 571d a0f9 a3b1 3924  ........W.....9$
000000a0: f99f a14d 48bf daf9 9d1d e57d f231 e5ea  ...MH......}.1..
000000b0: 34fc 95dd bddc 1051 8591 9fa7 cad0 75fd  4......Q......u.
000000c0: 8f68 6f0b 7165 140b c02a da52 9e3e b07b  .ho.qe...*.R.>.{
000000d0: 036b d4c6 dd0d f691 3e5d 8e50 ce81 43d7  .k......>].P..C.
000000e0: 4f74 aca2 99b7 6f25 bfe9 2663 0e89 b7b2  Ot....o%..&c....
000000f0: 7b7e ad81 98ab 208d abb4 ae4f e586 d99c  {~.... ....O....
00000100: 6317 55f9 5196 d05b 6a75 ca22 c8c6 3535  c.U.Q..[ju."..55
00000110: 1ea9 e23b c981 d0b2 e159 4a75 2263 70c1  ...;.....YJu"cp.
00000120: c75b eacc 829d 4cff 3b9c ec8d d35f 4a92  .[....L.;...._J.
00000130: 3dcd 9674 bca0 a14b 5804 485c a2d7 afde  =..t...KX.H\....
00000140: d599 f15d 8692 efb7 72e0 fd2b b06e 931f  ...]....r..+.n..
00000150: 73d5 bcc2 24aa ed92 3b53 4149 c0ab 0cda  s...$...;SAI....
00000160: 3008 9588 4660 41c7 e301 904d 01fd e8d9  0...F`A....M....
00000170: 8867 7ab0 6b72 20f1 5147 2ace d3b1 9ea5  .gz.kr .QG*.....
00000180: 3f81 69f0 d366 4722 de85 2b9b 1dab 8146  ?.i..fG"..+....F
00000190: f702 ee9f 4245 cc0e 3b09 387a 1b23 bf0f  ....BE..;.8z.#..
000001a0: d239 5698 d405 f2e7 1717 0e9b 67f3 4a52  .9V.........g.JR
000001b0: 641d 2fb1 4de0 fab5 57c9 a356 2acc 1447  d./.M...W..V*..G
000001c0: 1898 27e1 f8c2 b7e6 a1d3 b5e6 4967 bfb7  ..'.........Ig..
000001d0: 431b afca 4821 4e72 f75a f35f 9a30 27e9  C...H!Nr.Z._.0'.
000001e0: 6499 5ffd 61f9 9e19 4e5c ede5 ad84 b1d5  d._.a...N\......
000001f0: 143e 5da3 9d4a 8ee1 64a2 7e64 c917 4e70  .>]..J..d.~d..Np
00000200: b93a 1365 6873 728d 9bdc 14c7 116d 8d1f  .:.ehsr......m..
00000210: 5c19 addc e5da af50 2b11 b558 55dd a1e4  \......P+..XU...
00000220: 8d13 1c4f df99 40bf 0e3e 03bb d65f 7229  ...O..@..>..._r)
00000230: c4d8 6d7a 3e6d c5b5 b11a 5608 afba 9bb3  ..mz>m....V.....
00000240: b92d 25d2 b850 c8d9 1f2a a7f0 b8f9 382e  .-%..P...*....8.
00000250: 6613 3b79 7fb3 107d ebbb 5a51 f6d3 27e4  f.;y...}..ZQ..'.
00000260: 43cd e4d0 fe3c aaa1 4276 209a b58e 5b9e  C....<..Bv ...[.
00000270: 13d8 2d38 3cd6 8ceb 83db e0e4 6c79 ed7e  ..-8<.......ly.~
00000280: ebe1 1cf8 5490 ec31 2502 8597 e7a2 3437  ....T..1%.....47
00000290: d611 9831 7483 49e3 32d1 d949 5fa5 cda9  ...1t.I.2..I_...
000002a0: 93c3 e115 2e6b bd98 4013 2dcb f1e6 003f  .....k..@.-....?
000002b0: 26de 1027 75a9 0d0e 82b3 649e 3a9e c47b  &..'u.....d.:..{
000002c0: 1410 5d92 173e 336d b5f1 1145 b0b0 87b4  ..]..>3m...E....
000002d0: 1a21 d942 46ef 4cd4 faa8 2046 d4f4 5ead  .!.BF.L... F..^.
000002e0: 7bfe 7bed 0d30 02f4 ffd4 b6a9 1cde d0ba  {.{..0..........
000002f0: a478 6c4e 155f 0a9c 3ad3 bd09 58ec 5c14  .xlN._..:...X.\.
00000300: 22ba a509 ceaf a114 f6ef d5c4 a436 b767  "............6.g
00000310: b159 626c d28a af60 2d1f d674 5c64 d1f3  .Ybl...`-..t\d..
00000320: a390 45c5 bbe6 b754 ca97 f7e7 8f0d 5ba5  ..E....T......[.
00000330: df90 9072 02ed a9a4 eaaf 6d52 ae57 8d21  ...r......mR.W.!
00000340: 820d 9d22 b80a 3c1e 51fd bb87 b1ab 5364  ..."..<.Q.....Sd
00000350: c70f 8659 8916 509c dcd6 1679 2099 c102  ...Y..P....y ...
00000360: 26f8 1736 3618 fd37 30c8 e25d 1f1e 3c69  &..66..70..]..<i
00000370: 703d 5482 07e8 97d5 622e e722 842f 4a8c  p=T.....b.."./J.
00000380: 4fe5 7b67 7f9f 1d44 19dd b53d 631c 9b6b  O.{g...D...=c..k
00000390: 3f78 0164 28a2 2c2b ea0d c395 a065 ba9e  ?x.d(.,+.....e..
000003a0: 0c6c 282d 968c be6f caed ba93 8a49 3852  .l(-...o.....I8R
000003b0: 0d0f debf abfd bbaf 6ec3 e982 d281 5001  ........n.....P.
000003c0: c0cd 448b 13d2 9fd5 28db bd5a 24a6 03cf  ..D.....(..Z$...
000003d0: 606d 2573 2869 5d57 eecf 01d3 24cb a5d8  `m%s(i]W....$...
000003e0: 70a0 039e 04a9 7f14 429d 8424 a3bd fa03  p.......B..$....
000003f0: dc87 62cd d193 fb92 0d4d 5a7d 1a58 d003  ..b......MZ}.X..
00000400: 6220 b5a0 f3f7 82a2 7a9c f071 0442 421d  b ......z..q.BB.
00000410: 4cbb 3fda 3c2c d9fa c2ab cfea 8fc9 72e2  L.?.<,........r.
00000420: 854e af60 9b40 caf5 2b4d 8123 e6c5 4d3b  .N.`.@..+M.#..M;
00000430: 7339 d358 da80 675d 7bd2 746b ceab 39df  s9.X..g]{.tk..9.
00000440: a7e6 d77e 2c82 f99d 7f56 9697 7290 84d5  ...~,....V..r...
00000450: 8d33 77ea 8b0b 34f4 1477 e832 8459 661c  .3w...4..w.2.Yf.
00000460: 917d b2c8 99ee 8060 88bf 6d43 4921 7b08  .}.....`..mCI!{.
00000470: 6ab2 afdc 3499 d73d 9d15 e8a5 c95e a5ef  j...4..=.....^..
00000480: de4f 5dc5 6302 c6cc 52af 4063 921f c98b  .O].c...R.@c....
00000490: 84a9 7dbf fdfb 049c 994c 02ac dd48 faa5  ..}......L...H..
000004a0: 88c1 0b5b 0731 a87e 1d97 52e5 d8db 5019  ...[.1.~..R...P.
000004b0: 40f1 63d9 e4b2 3425 f3c1 90b6 a805 a666  @.c...4%.......f
000004c0: 22f0 bb1c 130c 8cc6 ec24 0c2a 2237 cfc8  "........$.*"7..
000004d0: d1a9 0a82 b83c ac1e 6745 ed13 c7be 793b  .....<..gE....y;
000004e0: 906d 1a4c ab42 c7ae bfca 0692 fe15 dcd7  .m.L.B..........
000004f0: ecba b941 586f 569c f67d 4833 41d8 9c19  ...AXoV..}H3A...
00000500: 2f48 0f54 1aa6 f146 2bbc e86b 7c7f abdc  /H.T...F+..k|...
00000510: 2c30 3d06 f97a 9167 7ebd 60e0 2cbc 2889  ,0=..z.g~.`.,.(.
00000520: 92f7 1c5b 5650 482c fce4 4b23 bf52 0980  ...[VPH,..K#.R..
00000530: 6d2d 93ab 73db d402 a445 6f5f 0484 1371  m-..s....Eo_...q
00000540: 0297 68bb aaf4 59f8 f386 ab5e 819e 419d  ..h...Y....^..A.
00000550: 8acb 598b 5035 af0c 1805 2d08 6866 06d4  ..Y.P5....-.hf..
00000560: e877 6a0f 472c 08e8 4ef5 bf86 cdf4 7e2f  .wj.G,..N.....~/
00000570: a25c c914 ff46 3d7d bde3 f2e7 a64c 4806  .\...F=}.....LH.
00000580: eed6 1da0 70d8 d4a8 f164 6d43 4c82 8042  ....p....dmCL..B
00000590: 6418 1a36 da4d 16ea 72ca a635 c3a9 b4aa  d..6.M..r..5....
000005a0: 0cf0 d8d7 74ee aab1 ee03 cb42 96a9 55b1  ....t......B..U.
000005b0: 5a43 8314 4210 99c8 d738 304c e153 789f  ZC..B....80L.Sx.
000005c0: 1490 0531 495c affd bbc2 0407 a5a4 4136  ...1I\........A6
000005d0: 12ee fd6d 8994 2868 4e02 2ba1 ed42 2a2b  ...m..(hN.+..B*+
000005e0: 0d9e b4fb 0231 be4b bff7 3fa8 95db c980  .....1.K..?.....
000005f0: d013 aeb6 7651 5b55 f2bc 6de6 b536 9d9b  ....vQ[U..m..6..
00000600: d67f 285b b176 fd93 ad04 c38e eade 163c  ..([.v.........<
00000610: 64a4 445f eade be45 7eb2 4586 41ab f03f  d.D_...E~.E.A..?
00000620: dad2 1917 8002 5ac9 784a 5146 ee97 3707  ......Z.xJQF..7.
00000630: 0fed 2a27 859a 6493 7c7a 1a61 748d 252a  ..*'..d.|z.at.%*
00000640: da38 548a 0dfb ddd4 0eb7 a287 f263 aa56  .8T..........c.V
00000650: 74d0 7f89 2a58 75eb dcbf a3a5 378a eaa5  t...*Xu.....7...
00000660: 90a8 8e8d 2c56 fe46 8860 6c2e ec11 e1b5  ....,V.F.`l.....
00000670: a7fe afc3 51c7 c9d2 a601 4476 3063 20ed  ....Q.....Dv0c .
00000680: 2ea3 12c6 988d 049d 72ba 9c04 a50a b137  ........r......7
00000690: e4a4 20ea 4a10 9d65 a3ce 3daf de30 3b11  .. .J..e..=..0;.
000006a0: c1c9 64a8 1442 2567 3441 332e 611b c045  ..d..B%g4A3.a..E
000006b0: 0f97 81b3 6c78 ba82 5aad 5a0b 4a05 cd03  ....lx..Z.Z.J...
000006c0: a6ca aea2 8205 002d 25e3 e03d a153 a346  .......-%..=.S.F
000006d0: 7381 8b46 6b6c b7d7 d6b0 fbbd 5ff0 b3f6  s..Fkl......_...
000006e0: 44bd 3528 de32 3219 91f5 926a 7fc3 3b30  D.5(.22....j..;0
000006f0: 4327 f259 1c2f f8a0 cb56 fa59 2bd1 f006  C'.Y./...V.Y+...
00000700: 24cf c9d6 9d20 ea3d b554 2986 7278 8a74  $.... .=.T).rx.t
00000710: f23f 59b2 894a 4cc0 c3d9 3114 a028 8717  .?Y..JL...1..(..
00000720: bc16 a9e5 984b 1a34 befc 8d0c 1b7d 7042  .....K.4.....}pB
00000730: 3373 9ec6 e484 4e48 b67f a36b 01aa f484  3s....NH...k....
00000740: 14c3 f004 0928 fbe5 9413 534f 9bd8 21dc  .....(....SO..!.
00000750: 3858 a05b 03e1 3eb0 9a46 af26 72d2 a947  8X.[..>..F.&r..G
00000760: 6bb9 b26f 3ff1 d41f 4ad2 4ec3 c1b3 1bb6  k..o?...J.N.....
00000770: c703 cd62 3985 ab29 2762 ddec 9009 bd43  ...b9..)'b.....C
00000780: 1eb0 2b7f 56a5 6b0c fc90 01c3 103e 2e31  ..+.V.k......>.1
00000790: 6fb6 1881 e1ab 9278 5e3e 141f ce31 eeff  o......x^>...1..
000007a0: 5afa a655 416a 1b69 4fb4 3a16 b52b b191  Z..UAj.iO.:..+..
000007b0: 7a9d a2b2 4182 e1e5 fff7 6c34 f112 7dd7  z...A.....l4..}.
000007c0: ef37 42b4 bf13 1764 e4e7 1414 db75 1e0f  .7B....d.....u..
000007d0: b724 3332 254e 2238 aedc e8e3 bc39 3b7a  .$32%N"8.....9;z
000007e0: b03b b64e dbcf 9f1d bd54 2b33 1cc8 16a4  .;.N.....T+3....
000007f0: 3c59 7c1f 09dd 56f7 81f1 2159 04b5 efac  <Y|...V...!Y....
00000800: dcd1 f893 5ff5 9783 7072 0313 c4c7 9538  ...._...pr.....8
00000810: 6c5f c97e c4ac a9fe 7ae9 9777 44e4 2b9f  l_.~....z..wD.+.
00000820: c9fd 915b 7caa f7f0 7d9c 3fb0 d8cf 9b8d  ...[|...}.?.....
00000830: f7e9 3fa0 d2d6 c52a 43b5 8524 b147 9d28  ..?....*C..$.G.(
00000840: 4229 c0c6 6004 3e41 8fdf 7746 d220 6cb6  B)..`.>A..wF. l.
00000850: 974c cb9e c22a 48c6 c2d3 bb46 1456 38ff  .L...*H....F.V8.
00000860: 889c 0abe c170 5dee 6e7c 0438 de35 3eb3  .....p].n|.8.5>.
00000870: aed4 7252 2d3e 582d 82b3 7e84 4c4c daad  ..rR->X-..~.LL..
00000880: 17d3 06d0 b8cd e27b d20c 0001 1014 6e94  .......{......n.
00000890: d6d7 3209 d2e5 545d 206c bac4 84da 7158  ..2...T] l....qX
000008a0: e5e3 b8c1 326d 4fb6 e1ab 417a 97f6 4c72  ....2mO...Az..Lr
000008b0: 8cbe ce33 0f1b 9c04 f6d3 2f90 ae77 83a4  ...3....../..w..
000008c0: 69d5 9029 9520 c774 22cf cba2 8f3e 90e4  i..). .t"....>..
000008d0: 2b50 ffaa 4bd9 5a7f 76cb 3e36 4ca2 9285  +P..K.Z.v.>6L...
000008e0: 91e8 bd03 fa86 8685 7042 34e3 64f7 1f05  ........pB4.d...
000008f0: 9810 2949 93ef fd92 5505 9673 e256 512d  ..)I....U..s.VQ-
00000900: affc ea12 4237 c070 05f0 f73c 44c6 2aa5  ....B7.p...<D.*.
00000910: a972 f346 b72d 5e0d 40c0 7bb2 c2ca 0f02  .r.F.-^.@.{.....
00000920: 0c84 5851 627a 7484 b184 e9fa ce36 61ab  ..XQbzt......6a.
00000930: 1076 be5c 3209 e8a4 e941 0ecd ffc9 d61a  .v.\2....A......
00000940: fb72 9152 bce3 efb4 edb9 73e5 073b 2a7f  .r.R......s..;*.
00000950: 2dd1 66b2 e451 03a6 d219 d60d 427f b218  -.f..Q......B...
00000960: 412e 97ae 7a8b 33a6 dd11 602a f117 0e39  A...z.3...`*...9
00000970: f15d e1a4 419d 092e 8449 e23c 9d7e d3d7  .]..A....I.<.~..
00000980: d3f2 f969 bbf6 ee07 62eb 6989 1204 dfd1  ...i....b.i.....
00000990: c40b 33db 3fd4 0a9d fc2a 1be0 7290 1509  ..3.?....*..r...
000009a0: cd55 0306 ad2c 4b92 5fd7 c348 1685 d932  .U...,K._..H...2
000009b0: 1c8e eb29 2a89 7c1f 98bf d8e5 848c 1732  ...)*.|........2
000009c0: 1842 8935 4d06 4170 4d47 e555 39eb 6651  .B.5M.ApMG.U9.fQ
000009d0: 4cbe bc63 c373 9234 1371 c591 2020 b4d4  L..c.s.4.q..  ..
000009e0: 9e19 b9ce b928 ca4e 3299 0310 5b7c c4f4  .....(.N2...[|..
000009f0: 49bd e6da d065 3506 97e4 b4b6 a872 c1ae  I....e5......r..
00000a00: 251c a573 ccd5 7b3c c575 d827 f250 9ad0  %..s..{<.u.'.P..
00000a10: c2f9 5259 368d fc95 dfc2 326c 0000 0300  ..RY6.....2l....
00000a20: 0003 012f                                .../

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=22016
pts_time=1.791667
dts=19968
dts_time=1.625000
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=5938
pos=63291
flags=__
data=
00000000: 0000 172e 419b 4934 a4c1 15ff 0000 0300  ....A.I4........
00000010: 0003 001c cbf7 371f 0208 7f3c 77ed c345  ......7....<w..E
00000020: ca17 4f66 9d23 1772 007e 37db fbb7 91e7  ..Of.#.r.~7.....
00000030: 144a ec6c b76a 1fd0 8572 fba7 d9ad 54c8  .J.l.j...r....T.
00000040: 8adf f057 1e33 e136 27a4 34dc 4f58 5e3f  ...W.3.6'.4.OX^?
00000050: f35e 1f76 5e85 2e34 4c39 ee38 406c 893c  .^.v^..4L9.8@l.<
00000060: 757a 6d3a a357 4a0c 5347 a9fa aace 9178  uzm:.WJ.SG.....x
00000070: e5fb 206d 961a ccc6 0185 bbf1 8d63 0316  .. m.........c..
00000080: b6e4 9e12 6feb 4982 135f 47f0 2b5c 003b  ....o.I.._G.+\.;
00000090: 0965 937b 2de5 943b 5408 f021 0495 fe7b  .e.{-..;T..!...{
000000a0: 0003 2017 0181 27bd 98ec d96b ad6c 1bde  .. ...'....k.l..
000000b0: ddcd 201a af6c 4d57 51ad 49d1 0c95 78a9  .. ..lMWQ.I...x.
000000c0: 9bb6 07a6 df10 576f 0b00 3ada faab bdef  ......Wo..:.....
000000d0: ac14 38af e3f3 d4ee abaa 063f 7d32 1ca1  ..8........?}2..
000000e0: e0f7 ca02 0345 bca6 a9c2 ac20 48ac 8c42  .....E..... H..B
000000f0: 3db6 44cd b3da 9e18 5cb8 3f24 e0f4 2756  =.D.....\.?$..'V
00000100: 4409 40f6 73f3 353c 0572 e901 571f e482  D.@.s.5<.r..W...
00000110: 4431 e8d0 1e17 608e 0360 02ab 6e78 ea30  D1....`..`..nx.0
00000120: 80eb a2e5 c4e6 b60c 4c4c 835c ef42 0abe  ........LL.\.B..
00000130: 07c9 16dc aa4c aa38 d518 680b eda4 bf37  .....L.8..h....7
00000140: ee16 20a2 91e3 84a0 b600 6769 f118 9219  .. .......gi....
00000150: bf30 9036 aeae cb85 004e adce 183e 15c2  .0.6.....N...>..
00000160: d6e0 eebb 513f c03d 33b8 bde0 e50c fe2b  ....Q?.=3......+
00000170: 7928 d6c9 ed32 c524 51c0 064c cdfe 5464  y(...2.$Q..L..Td
00000180: 985c 175e 8b81 6576 87f6 f088 eebf ebd7  .\.^..ev........
00000190: 462f e950 4d00 d972 29b3 1aa7 d7f2 9873  F/.PM..r)......s
000001a0: 046d 05d7 9d7b dabb 3463 0f0e c561 9b20  .m...{..4c...a.
000001b0: adbb 1e50 fbb5 aa20 1b05 592a b69c 89d1  ...P... ..Y*....
000001c0: 7f1a ad04 426f 4f59 defc d081 f42c 8703  ....BoOY.....,..
000001d0: 2695 9a8d 06a4 b3e3 37ca 0d4c 0424 519e  &.......7..L.$Q.
000001e0: 1cd5 9eb7 83d6 55de 0259 2db9 5389 a37e  ......U..Y-.S..~
000001f0: 6b0b 9966 c01b 526c 9ee7 a83e 19b6 8e67  k..f..Rl...>...g
00000200: 4b89 6134 d015 dfa1 cb49 3f42 421d 8290  K.a4.....I?BB...
00000210: ce74 9b49 8673 c1f8 6800 51ba 2288 af60  .t.I.s..h.Q."..`
00000220: 4897 4724 3db3 e0f9 4135 ed5b b1f3 3711  H.G$=...A5.[..7.
00000230: 5fe0 123a 36cc 642b 96fa dc56 1e37 6dc8  _..:6.d+...V.7m.
00000240: dab5 a42a 30a3 cfcb 135d 3e60 2c6f 292c  ...*0....]>`,o),
00000250: 673d 334b 3971 d208 cd91 9911 2c64 16ca  g=3K9q......,d..
00000260: b005 a372 d3dc c3df b2ca d2f8 4a20 1fa0  ...r........J ..
00000270: 5767 d2c0 2b4d 6aa4 fbe1 dcf2 cd35 1746  Wg..+Mj......5.F
00000280: 819e 5d39 e1a6 23f4 762d 057b 6c5a 919e  ..]9..#.v-.{lZ..
00000290: 4117 e1c1 74fe 3d40 0793 7c4e 8006 0431  A...t.=@..|N...1
000002a0: 6622 8c0a 2179 0b3d 3e27 7177 3560 a6b3  f"..!y.=>'qw5`..
000002b0: 1453 ec00 ff89 2552 91a5 dbb5 fbf6 291d  .S....%R......).
000002c0: b21e e9bc 37e1 162e 343b eae1 197a e648  ....7...4;...z.H
000002d0: 76be 7919 4eb0 9241 5ddf 0162 bc0d 0338  v.y.N..A]..b...8
000002e0: 6cc9 f923 e24f d596 bba6 40b8 ac3a 4b1d  l..#.O....@..:K.
000002f0: e745 93fb 645f 3471 c7e5 c3ca d6b2 fdcb  .E..d_4q........
00000300: 9a2f c024 f88b d3eb 96c6 35ec 2995 7717  ./.$......5.).w.
00000310: 7285 bd9f 9053 b3e5 16cb f15c 932d 1dcb  r....S.....\.-..
00000320: 88e5 b6a3 7fcc 710b 2a2d 2d2f 6de9 dd97  ......q.*--/m...
00000330: 40f2 2abb 1c21 f76d 5502 4bbb a60a 9f02  @.*..!.mU.K.....
00000340: 9ad4 95b3 bcbc 961c 2a0a 37a3 b55a bef2  ........*.7..Z..
00000350: c432 92c7 f9dd dbf8 b986 39e5 29ef 4bc0  .2........9.).K.
00000360: 7724 d385 9eaf e6f9 ff26 56b2 02e8 43d1  w$.......&V...C.
00000370: fafe 988a 54be 9339 04cc 2203 2d43 8c96  ....T..9..".-C..
00000380: 66a8 03c2 60b6 8ac5 832f fb12 1764 0566  f...`..../...d.f
00000390: c774 8a78 f174 6509 120a 5516 06d8 cb21  .t.x.te...U....!
000003a0: c441 f889 409b 0031 7c65 141e 7d94 7b64  .A..@..1|e..}.{d
000003b0: 292d 99bd fb2b affb 2119 b823 bea1 7047  )-...+..!..#..pG
000003c0: 646d 8f2b 6626 d27d 1a50 146c 0dd8 940e  dm.+f&.}.P.l....
000003d0: 847b 4b61 9fe8 b3ac 2d1c f6ec 2bea a0d2  .{Ka....-...+...
000003e0: 9f5b a383 fcea e825 a0ff 050f 271d c68f  .[.....%....'...
000003f0: 20b1 da1b d366 9f06 f2f5 380f deba 12a8   ....f....8.....
00000400: bafc c610 42a3 bc90 30cd 170f 32b7 2827  ....B...0...2.('
00000410: 0f90 88d7 911e 5288 04b6 29be 057c 3309  ......R...)..|3.
00000420: 30ac a507 8489 56d8 113c c3f5 0e67 f8da  0.....V..<...g..
00000430: d1fb c068 7567 f9e5 d808 6a68 eb07 775c  ...hug....jh..w\
00000440: 8312 5cd4 299e c650 a71e 828f 4151 39bc  ..\.)..P....AQ9.
00000450: 674c 07ea 7f6b 13c2 c444 d095 9ed6 f94f  gL...k...D.....O
00000460: cc2f bca2 293e 4674 ab1b 09bf 6438 a08d  ./..)>Ft....d8..
00000470: cd4d 6952 4733 fb07 101a 530e a1d7 3468  .MiRG3....S...4h
00000480: ee86 80a0 94af 3650 5da9 e851 3250 0c31  ......6P]..Q2P.1
00000490: cfa4 40d8 c26a 3bb1 1df3 3391 2986 e16d  ..@..j;...3.)..m
000004a0: 1f24 323d 09fb 3d02 ae77 30b9 156e db60  .$2=..=..w0..n.`
000004b0: c97d 9522 f856 21eb f472 9d0a f109 e393  .}.".V!..r......
000004c0: 2071 0e44 4b55 625d 0574 f9cf 02c6 098a   q.DKUb].t......
000004d0: ea69 84d1 7366 76eb ce28 6e67 b797 bc7e  .i..sfv..(ng...~
000004e0: 59d1 56f5 e7ec 6ed5 81b7 dbce 09f0 6c0c  Y.V...n.......l.
000004f0: f368 e510 afc8 02bf 50f6 9c3b 9490 882a  .h......P..;...*
00000500: bc27 1617 af42 e909 77f6 64f6 9542 a2a3  .'...B..w.d..B..
00000510: fe15 a85c 046b 1804 c0ca 3eb6 a924 f3a6  ...\.k....>..$..
00000520: 7334 731c 4330 4a07 4ef1 0797 3e98 cf95  s4s.C0J.N...>...
00000530: 30c6 c610 8c8d 817c ea13 30fc 0bdf 2a96  0......|..0...*.
00000540: b21f 6b88 aa79 0eae d417 4a92 9dbd aa86  ..k..y....J.....
00000550: d518 bc26 dc33 1372 e4d5 36d6 67a7 ab9a  ...&.3.r..6.g...
00000560: 46ff 7356 769a 12d5 f15c 6fdc 6a0e 5472  F.sVv....\o.j.Tr
00000570: 941c 90e4 c0dc db70 e668 bbf6 c9c9 6231  .......p.h....b1
00000580: 4a0d 32e9 2aca 1e26 8fde 0a56 6da9 dc23  J.2.*..&...Vm..#
00000590: a606 0ba1 9fab 8ea8 3080 1a31 0753 6ee7  ........0..1.Sn.
000005a0: aa90 4873 7803 a01b 5e80 eefe adf5 7a2d  ..Hsx...^.....z-
000005b0: c53f d982 14eb efbd b6ea 26a8 1a1d 84ca  .?........&.....
000005c0: 7bc5 44fc 4a51 8d71 a205 9129 b453 ac70  {.D.JQ.q...).S.p
000005d0: 032c 377b d0e2 6010 db29 90fe 9032 a857  .,7{..`..)...2.W
000005e0: e0a0 e4c2 5eef 7f0b f92f 384c e70b fdfe  ....^..../8L....
000005f0: 6da4 cc6a f900 a86c 6661 21de 1931 5b14  m..j...lfa!..1[.
00000600: 391a ab0d 0336 e080 7418 13a6 faae 2105  9....6..t.....!.
00000610: b603 e9c4 8e1a 42a4 1909 8bf4 d873 43ff  ......B......sC.
00000620: 87cc 0806 804b a3d5 dd80 ba26 a5bd de18  .....K.....&....
00000630: 4da2 16b0 2a54 f946 9e94 9982 96dc ad63  M...*T.F.......c
00000640: 0540 6584 9f0b 81f7 cf42 8c02 f39a 9638  .@e......B.....8
00000650: bfab 7a75 5155 8091 588a dad6 ada9 834e  ..zuQU..X......N
00000660: dc51 f98e 5eba 8e73 9f03 f2b1 e200 074f  .Q..^..s.......O
00000670: 362a 189d f02b 95ec aa6c 0e5c 770a bbc7  6*...+...l.\w...
00000680: eaed f0da e0f5 0002 cbae a115 a852 41cc  .............RA.
00000690: 3384 f4cd 1641 65c6 be39 7e43 aeca 9f57  3....Ae..9~C...W
000006a0: 6b8a de42 c207 9909 db36 2c18 49c1 cee6  k..B.....6,.I...
000006b0: 10f9 338f 8e2a 17ce 2f4c faf2 0bd8 bfac  ..3..*../L......
000006c0: 2036 7ff7 7000 91fa 5200 3e39 31db 9293   6..p...R.>91...
000006d0: 6135 5248 b768 4002 43ec 35f5 aeea c901  a5RH.h@.C.5.....
000006e0: adaa 413b a5ef 7c17 2c09 fb29 6ac8 4aac  ..A;..|.,..)j.J.
000006f0: c9c6 ed0e cf40 31a4 05d9 f60d 4348 abf5  .....@1.....CH..
00000700: be55 6d6b 991f d793 4840 5f83 33f7 c386  .Umk....H@_.3...
00000710: 0bef bcc8 e3f1 c35d 1e6d 8978 3864 6af8  .......].m.x8dj.
00000720: 56ba 2637 647d 05e4 63de c006 f46c d3a7  V.&7d}..c....l..
00000730: 3aa2 476e 6938 4a76 04f5 d472 224a a207  :.Gni8Jv...r"J..
00000740: 7a27 1345 8bd3 9e18 18df 07a0 3d21 2b9c  z'.E........=!+.
00000750: 17ef ad9a 4cad ef89 940e e663 9cf6 8724  ....L......c...$
00000760: 3415 5901 7a31 e21e e52d 06e4 2918 12b1  4.Y.z1...-..)...
00000770: 73d6 b197 0ba8 03d0 e1d2 c34c e959 2b78  s..........L.Y+x
00000780: 668d 7371 be94 6226 3ebf 0afc 8742 6afe  f.sq..b&>....Bj.
00000790: 6fcc 7169 92b2 b356 3f8f 026d e423 4118  o.qi...V?..m.#A.
000007a0: ae5b b95f b12b 70b7 2158 79fd 3810 decc  .[._.+p.!Xy.8...
000007b0: 30e2 aeb3 d7aa 9cf7 3a6b 2d9b 037f 7086  0.......:k-...p.
000007c0: 82f4 0396 265c e90b 1f72 cbe0 736d 8b06  ....&\...r..sm..
000007d0: 3f29 4ad5 fc30 a315 bf7f b7a3 7689 9550  ?)J..0......v..P
000007e0: f8d1 15f7 1585 5f50 dbec 70ff 3ec4 dae3  ......_P..p.>...
000007f0: 4365 e9ba 51a3 4b9c 008c 9045 ee92 c61f  Ce..Q.K....E....
00000800: 6ee0 e623 f35b 7fae 0eb7 793a 5754 5e67  n..#.[....y:WT^g
00000810: 2bf6 aa0f 3452 d70c cad1 c6e8 de2e 481a  +...4R........H.
00000820: 24a6 0e3a 0d0b 8c27 d690 d574 8c7b 2b1f  $..:...'...t.{+.
00000830: 5bde 8b8c fcf7 def6 68f2 0df9 514d 01da  [.......h...QM..
00000840: 2a28 7582 5593 13c3 449a e1b3 ff48 142e  *(u.U...D....H..
00000850: 5fa8 8b86 ac9c 198f e250 bf43 6276 0c40  _........P.Cbv.@
00000860: d6bd 4537 1179 7750 bf03 ff99 5e02 ba52  ..E7.ywP....^..R
00000870: 4b74 a122 04e0 ed16 78b4 2696 1887 d5c8  Kt."....x.&.....
00000880: 97b1 f18a 9e53 b6e0 9b1a e13c 7c00 8a46  .....S.....<|..F
00000890: ed36 5cad af92 2ac0 4dcd 0d96 22bd 28b1  .6\...*.M...".(.
000008a0: 0056 b59d 4693 070f d55b 448b fe91 d4c1  .V..F....[D.....
000008b0: 5e4a e98e 6682 7796 b498 d340 1214 5fad  ^J..f.w....@.._.
000008c0: 7e10 3bb3 1f76 61e7 9909 7636 f4cc 5b5a  ~.;..va...v6..[Z
000008d0: e008 5790 4762 85b9 7d4a b7d3 0bb6 0ee6  ..W.Gb..}J......
000008e0: 1e7c 8d40 8216 8807 5308 5b13 ac85 b3c1  .|.@....S.[.....
000008f0: e879 df07 bb90 f0d2 8434 f03c dbc6 c552  .y.......4.<...R
00000900: 7e56 b04f 216f a6cf 780b 58ea f903 185f  ~V.O!o..x.X...._
00000910: 7e1a b6ea 1cc0 a6eb 1787 5594 4c6e ec0a  ~.........U.Ln..
00000920: f2d9 0941 2953 86b1 b33c 9efe 0b60 2473  ...A)S...<...`$s
00000930: f72d cc2a 848c d35b 8775 f056 8674 f8ff  .-.*...[.u.V.t..
00000940: e26b be75 841a 1cac 046f 98be d9a4 ef90  .k.u.....o......
00000950: 8f29 6a6c 1810 b8ca 60f7 0fd3 1ab5 f412  .)jl....`.......
00000960: 1b19 7c94 8023 084f da66 b46a 0d4d 3945  ..|..#.O.f.j.M9E
00000970: 1f6c ba5c 6fdf 8124 133a 4764 e044 3855  .l.\o..$.:Gd.D8U
00000980: 68b7 4dce 6475 f8d5 155d 38c6 05c8 e536  h.M.du...]8....6
00000990: 82b2 4b96 626b f64f e229 0a10 3bd0 0537  ..K.bk.O.)..;..7
000009a0: 2522 7cf6 8210 e4bc cdca 46b9 a8eb 7d58  %"|.......F...}X
000009b0: be68 c627 99c6 a91e cf7b 63e0 d741 95aa  .h.'.....{c..A..
000009c0: 90f7 e976 1efc 962a 36f9 56bd 2803 7d4f  ...v...*6.V.(.}O
000009d0: 5a86 69e7 af4e afa8 9721 e509 f1d6 8033  Z.i..N...!.....3
000009e0: afcb df79 e6f4 5365 2ca8 60f6 a669 05e3  ...y..Se,.`..i..
000009f0: a7ed e381 635e 86dc 3ff1 874b 6417 3302  ....c^..?..Kd.3.
00000a00: 984b f781 5907 019f 535b 87ee 3a30 7c3e  .K..Y...S[..:0|>
00000a10: 61cd 737b 6d73 1b89 776b da20 2d12 d379  a.s{ms..wk. -..y
00000a20: 65d1 defa 17a6 920c ef25 1bb6 d5ac 72b8  e........%....r.
00000a30: 4ba4 ca38 cc00 b2c1 47df 1896 3fb7 84c7  K..8....G...?...
00000a40: 56fa df58 a007 0ca5 961c 5520 a2aa a544  V..X......U ...D
00000a50: a10d d7be 166b fd3b da58 ecde eb7b e001  .....k.;.X...{..
00000a60: 5270 40d3 ede2 e367 93de 1be7 0953 f30f  Rp@....g.....S..
00000a70: 9dde 9c80 bf69 de2f 8945 b802 0b4b f434  .....i./.E...K.4
00000a80: 6d37 2171 2356 bde1 df82 6c73 8d21 a367  m7!q#V....ls.!.g
00000a90: e24c be60 601c ad71 262a efea 127b 1970  .L.``..q&*...{.p
00000aa0: 2ebe 662a dd20 553f 5b59 4f73 4584 6edc  ..f*. U?[YOsE.n.
00000ab0: 4bb2 da0b 175f 0ccb 80f4 e5f9 dc2e cabc  K...._..........
00000ac0: cdc1 ca69 8205 8eae f8b2 1db8 bd88 5217  ...i..........R.
00000ad0: 5873 0f4a 7dce edf8 87a1 2806 d89a af03  Xs.J}.....(.....
00000ae0: d6b9 7553 37b3 26f2 85ba 13db af73 e802  ..uS7.&......s..
00000af0: 5ca0 051c b1ac 0a19 a015 ccad 2fa7 5636  \.........../.V6
00000b00: 1839 d833 09a0 8fdd b76e 9467 67f1 c1db  .9.3.....n.gg...
00000b10: e9b7 c819 d00c 4a4c 020b b778 9c36 684a  ......JL...x.6hJ
00000b20: 56cd c1b7 55a6 979c 77f2 822b e81e 3eff  V...U...w..+..>.
00000b30: 4a2d d2b4 5b47 fbd3 944f 6fa1 e110 e210  J-..[G...Oo.....
00000b40: 6073 51e9 aa30 bfef 46c7 1cd7 0c09 fcab  `sQ..0..F.......
00000b50: 8f05 6196 5d31 f612 d64b f308 4459 70b3  ..a.]1...K..DYp.
00000b60: 2df4 dfa5 07fc f0f4 4826 50c9 1226 dd32  -.......H&P..&.2
00000b70: 64cf ba1a f184 8e0b 1a7a 2611 e058 62e0  d........z&..Xb.
00000b80: 8d3c e31e 6f9b 4b8b 0e5c 2da3 f846 875a  .<..o.K..\-..F.Z
00000b90: 29c3 5c30 5859 f5f3 a11e 3b84 9ebf a40b  ).\0XY....;.....
00000ba0: 9c55 76f7 0cec 346c f1eb b5e4 2903 3303  .Uv...4l....).3.
00000bb0: 2fa7 1b81 5427 f264 e92e 7541 260b 9498  /...T'.d..uA&...
00000bc0: 0a05 32ec 13e0 2f50 f72f 18ee ff2a 27e6  ..2.../P./...*'.
00000bd0: 65f2 02fd 73d0 7b52 b70f e801 dc8b 5336  e...s.{R......S6
00000be0: 88ae efe2 b4cf 9dac bbea 8f93 b14b 80cc  .............K..
00000bf0: ad16 181b f920 ed19 50e5 cea0 be5e e4b5  ..... ..P....^..
00000c00: dce0 f4a8 5023 a76e 03be 4adc de9b 2de1  ....P#.n..J...-.
00000c10: 6063 d1bb db8f 54b0 42a2 6015 b0bd 4f49  `c....T.B.`...OI
00000c20: 279b c7a2 a897 eb62 9c34 1e5b a601 b3f6  '......b.4.[....
00000c30: 8fc1 0b92 d65b b544 5b3c 7d27 341a f808  .....[.D[<}'4...
00000c40: a746 eaef c214 db20 fd72 4676 009e afc0  .F..... .rFv....
00000c50: 74d4 a875 e4b7 5718 719f 7a79 cf07 d45c  t..u..W.q.zy...\
00000c60: 3e15 ef0e bbb2 adb4 1c05 f53d 50f4 b480  >..........=P...
00000c70: 85a9 98ba 5753 883e d7f7 10ee c2ab 159b  ....WS.>........
00000c80: b23a b548 b922 b2f1 afe7 3072 091c 4a06  .:.H."....0r..J.
00000c90: d042 307e f26c 56cd 226d 85dc 8708 e269  .B0~.lV."m.....i
00000ca0: f552 02db 7f47 d9b7 ac42 d7df 0568 6c4a  .R...G...B...hlJ
00000cb0: d1c9 08ec 4e00 879c 9433 3677 2e06 ddb5  ....N....36w....
00000cc0: fdf1 9ba3 6048 91f4 5c61 5d1e dc51 03a7  ....`H..\a]..Q..
00000cd0: dcb5 c68c 446a 783d 1e43 d7a4 0b04 5d0d  ....Djx=.C....].
00000ce0: b15e 7f10 c990 b480 ad45 f2f6 a94d b75d  .^.......E...M.]
00000cf0: 1b04 8ffc 7293 6719 9519 dcb6 145c 1b3a  ....r.g......\.:
00000d00: 7d1f 8fbd 91c2 6f00 803e 9f84 eb23 0dd9  }.....o..>...#..
00000d10: fe1e 2fd5 99ab ba8d 5995 3ec0 b037 89fd  ../.....Y.>..7..
00000d20: a64f 8a17 13ea 8fbd 4b7e 810d 0907 5a0b  .O......K~....Z.
00000d30: 91e2 9afe 8f9b 3da5 14f0 a3cd a4ed 1c7c  ......=........|
00000d40: b2dc f4b3 934f 6a5d c78e 63b9 b2b5 6014  .....Oj]..c...`.
00000d50: fe64 1264 e18c a8dd e701 7f3b 230f 1c3a  .d.d.......;#..:
00000d60: 3b38 e226 f5f3 32af 9120 0d42 948c eda1  ;8.&..2.. .B....
00000d70: 7fca 806f f03e dbd6 ab75 2874 3193 ee77  ...o.>...u(t1..w
00000d80: 66fd f2f7 48fd c432 c8ad 940e 584d 069e  f...H..2....XM..
00000d90: 6184 28fa d532 3bd0 f729 d5c1 a63b d2f0  a.(..2;..)...;..
00000da0: aa1e 38d6 f80d 3931 0043 5304 099e afc7  ..8...91.CS.....
00000db0: 386b 80a6 31f4 93f1 8f7a cef1 fc55 45d7  8k..1....z...UE.
00000dc0: 5b30 a30a 069e e3bb 0887 fd2c 1103 0a5e  [0.........,...^
00000dd0: a3cb 6963 a80c e478 aeaa 8f19 d42a 180f  ..ic...x.....*..
00000de0: e5b9 dabe 9894 998d 055e ed2a 88d3 74d5  .........^.*..t.
00000df0: 66b7 0814 8a0f 9ce6 841f d4e5 7080 39dc  f...........p.9.
00000e00: 9673 97b7 5bd8 6282 dba6 8f99 46db 3006  .s..[.b.....F.0.
00000e10: cfc7 e5ad 3b06 db63 eb88 105f 9286 76ca  ....;..c..._..v.
00000e20: 92f8 2c92 4d75 75e9 b183 a19f 9620 ba21  ..,.Muu...... .!
00000e30: b3f5 fde1 22dd 39b1 f1eb f0b8 007c d934  ....".9......|.4
00000e40: 900a 2572 4135 2ec0 8143 c493 f7f4 3af6  ..%rA5...C....:.
00000e50: 0d17 d07a a1c9 05fa feb5 38a0 46c1 8f67  ...z......8.F..g
00000e60: 195e 981f 8649 daaf bed1 1b98 33bb bd4e  .^...I......3..N
00000e70: 4443 e630 d9c8 dc0c ce1b 292c b1ae 21d1  DC.0......),..!.
00000e80: 491c 06e8 215d 63b2 4156 21ed 19e0 5434  I...!]c.AV!...T4
00000e90: 8370 2c17 fc06 f358 cf20 65e9 e6ac de1d  .p,....X. e.....
00000ea0: 4fd5 cdb1 9074 2f8f 34d7 94ca fe94 f5c9  O....t/.4.......
00000eb0: 23e2 9be8 ba25 27ad 006f d358 52ad 62b5  #....%'..o.XR.b.
00000ec0: da36 efd7 db4b d25b b892 0b08 d1df 63f6  .6...K.[......c.
00000ed0: 7c14 604b 89d6 3dbe 9c92 55cb 892f 4c87  |.`K..=...U../L.
00000ee0: 4021 95e0 e86e 1a1b 9d73 7105 55c1 1035  @!...n...sq.U..5
00000ef0: b131 4755 06aa 0eb9 e630 4bc3 ec64 38d8  .1GU.....0K..d8.
00000f00: 108f f33f 6992 2ab7 03b0 ae96 ebd4 7b16  ...?i.*.......{.
00000f10: 9412 2eb5 6c0c 7faf b6f5 ddca cd5d 0919  ....l........]..
00000f20: 4abe d7a9 833c 6ec2 7552 a8eb d368 c424  J....<n.uR...h.$
00000f30: 8e05 c95f 32b2 9b12 819c f6fc 5928 bf68  ..._2.......Y(.h
00000f40: 4f9c a717 9971 41d0 4b87 0cc2 ae7b 48af  O....qA.K....{H.
00000f50: 0712 4866 7cd3 648a c3ea c721 99e4 6841  ..Hf|.d....!..hA
00000f60: 05d5 0f28 ee15 f41b 7372 2f8d 5c12 ba91  ...(....sr/.\...
00000f70: 2f55 7e31 f173 2483 b89b 5297 9f6c 15a1  /U~1.s$...R..l..
00000f80: 5dfc f283 1d17 f8f7 f54c 191c dadf 28fd  ]........L....(.
00000f90: e854 e93e 0b4c 13af 9f32 0060 3d7c 3e16  .T.>.L...2.`=|>.
00000fa0: 48f4 c297 3e94 3404 5b55 7e46 eca1 1e4b  H...>.4.[U~F...K
00000fb0: 68c5 40df e216 38b8 67b0 21d2 6bce 220b  h.@...8.g.!.k.".
00000fc0: 9d37 8918 baea 9660 6b3c 6ec7 6fe9 e30d  .7.....`k<n.o...
00000fd0: 2613 24f1 b542 75fc 18a4 bacc 2996 1253  &.$..Bu.....)..S
00000fe0: 7f57 9819 47fb cd43 ec2c 8abc af42 22ca  .W..G..C.,...B".
00000ff0: 7671 8550 d1f5 8abb 7476 0f59 104d c41f  vq.P....tv.Y.M..
00001000: 24d2 ce44 fa7b 0106 e99f f270 38ca f71d  $..D.{.....p8...
00001010: a993 396b 3f15 b2b6 a123 25bb 3056 1879  ..9k?....#%.0V.y
00001020: c784 4c4a 0a99 ed2a 5851 5b1a df98 7c5b  ..LJ...*XQ[...|[
00001030: d247 b3ec 0e9c 4752 dd42 2972 874a b31e  .G....GR.B)r.J..
00001040: 8067 dc66 65a5 4030 f5e9 48a8 9c1f 84b7  .g.fe.@0..H.....
00001050: 5739 84e5 130f d620 0bd3 8ae2 1f04 12f6  W9..... ........
00001060: dde3 8cbe da0b becc 7172 8672 713b 247a  ........qr.rq;$z
00001070: 995c a918 5939 8a43 ac66 3695 52e7 4dc2  .\..Y9.C.f6.R.M.
00001080: 9c05 05de 78be 20f3 04f5 4c38 47f4 8b7b  ....x. ...L8G..{
00001090: 590f 26f1 03a4 1a93 c5de f2bb e7b7 a71d  Y.&.............
000010a0: 948b 5e16 d2c3 7947 4403 a3f4 10f4 b298  ..^...yGD.......
000010b0: dd01 e4ee de16 047e 7f0f eea6 f245 a612  .......~.....E..
000010c0: d56b 82cd cd72 9180 5a6a 6587 082e b46c  .k...r..Zje....l
000010d0: 2175 0e45 8c05 57b6 f656 e216 0ff3 1902  !u.E..W..V......
000010e0: f64d 9d3f 2c76 635c a775 20a2 316a 3c01  .M.?,vc\.u .1j<.
000010f0: c606 a941 c83e 7b38 2d15 b3e9 30bd f02b  ...A.>{8-...0..+
00001100: 2920 392e 17c3 9fe6 6de9 bde0 7fc8 50ca  ) 9.....m.....P.
00001110: bede e2cf f010 c5c8 16aa 5234 d9bd cd50  ..........R4...P
00001120: b282 9dc7 ea5c f8c7 edaf 2ed7 b7d4 ba40  .....\.........@
00001130: b808 5274 028b 15cf e113 f99a 0318 701a  ..Rt..........p.
00001140: f0d6 fb16 670e 53d2 857a e118 22b2 d2ef  ....g.S..z.."...
00001150: 0661 4d1d 89dc e4db 9fa2 d9ee 322a 3d9b  .aM.........2*=.
00001160: dc39 fece a51f 4c4b 0365 014d cdf6 876c  .9....LK.e.M...l
00001170: 2c2e c6ec 7482 c467 1776 a481 8b76 2dae  ,...t..g.v...v-.
00001180: 287f 7dbb 43e8 61b9 8b1a 8e8d a056 2a21  (.}.C.a......V*!
00001190: 9aeb 0285 0bf8 4d02 777f 9552 608f 5e15  ......M.w..R`.^.
000011a0: 6a7f 5f1a b94e 4f92 3a86 1873 dc9a de11  j._..NO.:..s....
000011b0: 3140 4a15 708c f85b a4d5 7036 8c7a 62aa  1@J.p..[..p6.zb.
000011c0: 3b70 1814 9547 5cb9 3058 6fb1 6cb1 3b80  ;p...G\.0Xo.l.;.
000011d0: 3038 349b 85b2 4554 cd72 3f7b 5f9e b1a8  084...ET.r?{_...
000011e0: 8f5e 4ebb e34b aafb 4ea2 c917 dffe 18d3  .^N..K..N.......
000011f0: 1437 83c5 ef49 97a4 ad6b 1072 114e 8ca0  .7...I...k.r.N..
00001200: 12b2 d991 0b89 c2e1 570f 2937 944b 4ed6  ........W.)7.KN.
00001210: 5b94 05d2 d21d 3d98 a349 dcd8 3b28 ae50  [.....=..I..;(.P
00001220: 658c 3b0a fce6 6368 6196 6940 5ca4 2b9f  e.;...cha.i@\.+.
00001230: cb00 ca19 38d6 4a8c eebb c8d0 cf06 e24b  ....8.J........K
00001240: 3102 7dfe 8675 09d1 fd82 72f5 8fae 0e4e  1.}..u....r....N
00001250: 94cb 65e5 28f8 bf6c f431 a9eb 600a 2ed0  ..e.(..l.1..`...
00001260: e7c0 f72a 8497 220b 059d 5e21 106d 816f  ...*.."...^!.m.o
00001270: 4f28 8593 7232 3993 9d89 0a61 4686 fe0b  O(..r29....aF...
00001280: 710c e073 ce78 dc80 033a d2d6 daa7 e19e  q..s.x...:......
00001290: bdc4 11c6 1abd 04f5 eb0b e8e1 0578 87e8  .............x..
000012a0: f000 f6cb 9e47 3d4e 9072 a391 b4e7 9622  .....G=N.r....."
000012b0: 2c40 7e7c 5a23 6f84 6387 df89 fd3c deca  ,@~|Z#o.c....<..
000012c0: 7c10 9732 712f 037a 7ffc 8c98 9ad0 dcad  |..2q/.z........
000012d0: 39ec 57a9 c112 d157 f608 1f77 996e e3ee  9.W....W...w.n..
000012e0: 403a aee1 cce0 6357 537d 4685 fff5 da1c  @:....cWS}F.....
000012f0: 2d6f e22d e76a 6d88 6efb f83f 401c a67f  -o.-.jm.n..?@...
00001300: 8033 2c9d 23a3 c380 281e 0a95 f421 6b3c  .3,.#...(....!k<
00001310: 0f43 a1b3 1d56 4303 aa90 b1a1 3894 deb3  .C...VC.....8...
00001320: 7ad3 aec2 7680 2db3 5850 cfe0 1c4f 6ef2  z...v.-.XP...On.
00001330: c401 a051 2f4b 6386 fc7d da8a 8058 fe03  ...Q/Kc..}...X..
00001340: 80a5 9ed8 a56c 51d4 cfa0 12f5 617d b93e  .....lQ.....a}.>
00001350: d0af 62d6 db94 aab2 049f cd2a dadf 95e2  ..b........*....
00001360: 6d85 e1ca 17c1 29d1 5f84 bbb5 4aa9 b6c8  m.....)._...J...
00001370: 7612 8812 27ca 800d 8dce 2b8b 866b 5318  v...'.....+..kS.
00001380: dc54 0026 35cb 7dfa 01cd 4a5a 7914 1873  .T.&5.}...JZy..s
00001390: 01c1 c404 7ebb 6d17 e8c5 5087 c901 349c  ....~.m...P...4.
000013a0: 7637 5280 c3e8 2e8b 8617 f17c e551 8fb1  v7R........|.Q..
000013b0: 620c 9f64 ba9e 4164 8244 6647 71cb 8c7e  b..d..Ad.DfGq..~
000013c0: 4b17 8e15 ff3b 3fa6 cf7f 4dcb 8837 21bb  K....;?...M..7!.
000013d0: 2aa6 14bd 7282 acd7 da8e dda6 0164 c30c  *...r........d..
000013e0: 97d3 d9f5 63d7 f10c e342 0480 5717 f341  ....c....B..W..A
000013f0: 8a50 d99d 730f ed5b 3d5d 6d3e 18f1 a9b5  .P..s..[=]m>....
00001400: 026c 0a6d e6b6 f038 6a41 03f6 d717 a63c  .l.m...8jA.....<
00001410: 50e9 ef13 7203 3456 cb8c dd1d 1df2 034c  P...r.4V.......L
00001420: 9a72 5ca8 1556 78dc 04f3 ebd8 1b0f 3e33  .r\..Vx.......>3
00001430: 91a5 ea2a df1d b389 9cf5 2e9e b4b1 dd57  ...*...........W
00001440: f5fe 0329 ec4a ec59 17a1 b91d 5e2b 9903  ...).J.Y....^+..
00001450: 2fed 0ce0 4bcd 2da2 059e df27 b5ed 83ad  /...K.-....'....
00001460: 3db7 5d50 a102 1b60 a894 ab28 5c00 e9a8  =.]P...`...(\...
00001470: f8e8 5f9a ba3b ac14 9d96 4136 4312 a1c2  .._..;....A6C...
00001480: 1bf1 88fd dcc9 ec91 e089 3d05 58f9 9370  ..........=.X..p
00001490: c0fe e94b 7be1 1962 72d0 00d8 338f be63  ...K{..br...3..c
000014a0: 68d1 6a02 f824 ab9c 0ed2 0f08 5bb9 61d1  h.j..$......[.a.
000014b0: a9d8 8767 c295 7a1b 9066 8909 8b66 01bc  ...g..z..f...f..
000014c0: e60b 1dc6 5a28 5a9c cb8b c620 d5dc c282  ....Z(Z.... ....
000014d0: 747d 2c47 f23b f8a1 cbad 4ac1 4cd1 c6b9  t},G.;....J.L...
000014e0: c388 cc02 0101 9772 5815 4a4f b8ff c0b6  .......rX.JO....
000014f0: ead2 b314 8cdb 2588 3781 5376 0848 f933  ......%.7.Sv.H.3
00001500: bb7a 171f d18e af29 ff95 20cc ad83 dd19  .z.....).. .....
00001510: 0215 d50a 9aa7 84eb 1539 b3e1 7b29 b32e  .........9..{)..
00001520: 32bf a040 bf91 0aa4 b27c c5e8 d004 c3d1  2..@.....|......
00001530: 6de4 828c 70c0 3ccf 5a93 5f9a 982f 5c01  m...p.<.Z._../\.
00001540: f5f0 415a 7a86 12a2 401b e3a1 af2a 2c51  ..AZz...@....*,Q
00001550: 5fee 495d 7a43 c216 c37a 87e6 d290 eb4b  _.I]zC...z.....K
00001560: d359 1105 6988 1d33 4309 7af3 0b27 768d  .Y..i..3C.z..'v.
00001570: 44a9 6b37 a2e7 2f25 e6c3 f0e2 ff8c e7c6  D.k7../%........
00001580: 5d3d f4e1 60b5 33a4 612d 5785 e6d2 81af  ]=..`.3.a-W.....
00001590: d222 bb32 97ab 4273 6187 f73d fdf2 4506  .".2..Bsa..=..E.
000015a0: b399 d591 b632 910f f99b 7c7d 15a6 652f  .....2....|}..e/
000015b0: adb3 d364 6005 f12b 543b 009f 483c 85fc  ...d`..+T;..H<..
000015c0: 9624 e57f 084a 8208 b858 b2c7 24a8 e246  .$...J...X..$..F
000015d0: 54c2 0193 58db 3ea7 bace 02cb f950 b6ca  T...X.>......P..
000015e0: ee43 c5b3 f0fd c6ba 08d5 4a75 32af 996d  .C........Ju2..m
000015f0: 4a09 24a3 65a5 15c6 a103 d2fb 216a b9a2  J.$.e.......!j..
00001600: aa88 4352 6a3e 0ae9 2899 e4f6 67c3 ed44  ..CRj>..(...g..D
00001610: 6594 98f6 b038 40ad 2961 57ca 4790 3a28  e....8@.)aW.G.:(
00001620: f22c 64fd eb70 debc ccb3 8cd3 b858 c280  .,d..p.......X..
00001630: c3f7 ba61 f65c 9c91 e04e 4a68 2133 f704  ...a.\...NJh!3..
00001640: 01b4 8944 0aa5 1974 8032 6070 8e78 c8b5  ...D...t.2`p.x..
00001650: b829 836c 7bc3 2acc acc9 6f08 d089 3554  .).l{.*...o...5T
00001660: c583 199d 8f76 a002 905a 61b0 fd7f 37ab  .....v...Za...7.
00001670: f1cb fefc 9b7f 4cf9 45c1 1c09 e3db b645  ......L.E......E
00001680: cc0e 4777 191c 7772 c7a8 9972 16de d78b  ..Gw..wr...r....
00001690: f31f 7c83 0b46 a329 eda9 dcec 4de0 2421  ..|..F.)....M.$!
000016a0: ee6b dcaa 2bad 9483 1ad5 8728 4c73 ca07  .k..+......(Ls..
000016b0: eb47 74fd 90c8 4b3a ce40 5c85 d428 8dcf  .Gt...K:.@\..(..
000016c0: 335c 4a79 394f 154e 3764 94ef 591f 6f04  3\Jy9O.N7d..Y.o.
000016d0: a438 d1d4 20a4 6226 9ed8 16ad e429 183f  .8.. .b&.....).?
000016e0: ec4f 4159 1b3e 19ce 0eb6 00b1 01c5 6184  .OAY.>........a.
000016f0: c712 172c 72a4 b049 276a 0c39 ef70 fa0d  ...,r..I'j.9.p..
00001700: 903c cfa5 5991 0718 851a df0f 0b52 0eb0  .<..Y........R..
00001710: bec4 4057 088a e853 fdec da2e 32c0 27ac  ..@W...S....2.'.
00001720: e7a4 8803 c51c 0aa0 4041 ea0d 506a 837c  ........@A..Pj.|
00001730: 9581                                     ..

[/PACKET]
[PACKET]
codec_type=video
stream_index=0
pts=20992
pts_time=1.708333
dts=20480
dts_time=1.666667
duration=512
duration_time=0.041667
convergence_duration=N/A
convergence_duration_time=N/A
size=3514
pos=69229
flags=__
data=
00000000: 0000 0db6 419f 6745 152c 5700 0003 0000  ....A.gE.,W.....
00000010: 07bd 57c6 16f0 4c18 c595 0b1f 14dd 1e2a  ..W...L........*
00000020: 94e8 5832 89f2 ce6a 798a 2f58 dcce 9776  ..X2...jy./X...v
00000030: e1df f6b4 1068 30a1 0ee0 0a07 b2a4 65d1  .....h0.......e.
00000040: 5fe8 1000 0596 e002 52d8 15ba 21bc 7875  _.......R...!.xu
00000050: 8b5f 240f 0af5 ad9e 9379 b1bb ed05 a63d  ._$......y.....=
00000060: c534 0992 581e aedd f0e5 58c7 d300 2197  .4..X.....X...!.
00000070: 63c1 ec46 eb66 a8a7 edf0 048d 5983 c486  c..F.f......Y...
00000080: 63a7 2db6 676d 0aa2 6d08 c107 07d1 2e42  c.-.gm..m......B
00000090: 5da6 8de3 bd76 1df9 c8df fb93 68b0 b6d1  ]....v......h...
000000a0: bb89 6733 e7a3 206e df1a aae4 97b7 1183  ..g3.. n........
000000b0: b345 95aa a2b7 eeaf 4827 836d 4fd4 db6b  .E......H'.mO..k
000000c0: e582 986f 4718 306c d580 b167 8be9 993b  ...oG.0l...g...;
000000d0: 6612 2da0 15d8 e913 b0d9 34d1 dee9 69a0  f.-.......4...i.
000000e0: 30b3 313d 1df8 fdf6 1467 42a9 562f f9ce  0.1=.....gB.V/..
000000f0: 6297 4106 56d6 22b8 835d ff67 51c3 16bc  b.A.V."..].gQ...
00000100: 63c1 f6e4 0345 a583 033a 0839 b315 436b  c....E...:.9..Ck
00000110: 23af 1ee6 6572 f112 3085 14f1 b779 6551  #...er..0....yeQ
00000120: 5c2d 3f99 1b7f cbe3 45ee 7cfe fbbf 7efe  \-?.....E.|...~.
00000130: cf8d 86dd 3f0f 8eea 7c7c 0b38 9a14 cfd9  ....?...||.8....
00000140: a747 a65d 740a 45aa 5e49 fa10 5370 8ce9  .G.]t.E.^I..Sp..
00000150: d41f 5f00 10cc de3d 0f45 3df9 adb9 07de  .._....=.E=.....
00000160: f10b f1ce a031 4bcd 0c1c 5c46 ccb6 c6d6  .....1K...\F....
00000170: 4bf8 ee3e 1fce 55cb 92de f598 18d2 fd0b  K..>..U.........
00000180: 3bae 792a 9a03 a3b7 7ec1 e979 a120 cca0  ;.y*....~..y. ..
00000190: 030e f2fb 5ad2 5422 4d1a b2df d1dd 3dc3  ....Z.T"M.....=.
000001a0: b6cb 12b2 88f7 5830 e008 842c b0a3 90b4  ......X0...,....
000001b0: 7f6b 794f 0f7f dfdd d091 9a96 fd93 38ba  .kyO..........8.
000001c0: c8b1 fc9d 78a5 854a 7921 edcc 8bfb 4d6e  ....x..Jy!....Mn
000001d0: 2d7a 9a49 02b1 806a dddd 907d 213e 3dae  -z.I...j...}!>=.
000001e0: 1d5e 88f8 a968 1d92 3bea d2d4 b3e1 8490  .^...h..;.......
000001f0: c1b7 008f eda4 aba7 4c80 f7eb 1f7b 1f82  ........L....{..
00000200: dd0d 8632 7b07 e774 8b98 20d8 a763 1c92  ...2{..t.. ..c..
00000210: 138c 1025 fd41 f4d4 506e f37d 03e4 1b9f  ...%.A..Pn.}....
00000220: 7815 2cb9 35d9 57ad 91ac 90b8 051c fa1b  x.,.5.W.........
00000230: 57a1 b247 cbed bc23 67ac 9f02 793e efe9  W..G...#g...y>..
00000240: 64da 25de 8a88 34c8 96ea 0bf1 43f1 3308  d.%...4.....C.3.
00000250: 24f4 5145 6d67 4431 99f3 a349 dde9 2c25  $.QEmgD1...I..,%
00000260: 3a5c 7fb0 fcf6 15e3 2377 1bc4 004c 4ff8  :\......#w...LO.
00000270: 6297 10d2 c8db cdd2 c71d 46fb bc97 68b9  b.........F...h.
00000280: c433 d649 d3c9 0d34 9b22 3f05 11cb 430f  .3.I...4."?...C.
00000290: a075 3913 2e5b b74e 5ed9 a2a4 3a5b c894  .u9..[.N^...:[..
000002a0: d6f0 42fa 9487 5057 2c14 cee6 b7a9 d2a9  ..B...PW,.......
000002b0: b67c eed7 0ccb adbb 9e76 fce6 30a1 93d4  .|.......v..0...
000002c0: 383a 5a4e bd8c 16c1 c4e2 cc3a 1ccf 2c12  8:ZN.......:..,.
000002d0: a244 9b51 e015 29b1 63a2 bfed 2244 6507  .D.Q..).c..."De.
000002e0: 71c2 fd73 0f1d 7e7c 033d d7f0 b9c2 3b0a  q..s..~|.=....;.
000002f0: b451 e8f0 f9b3 25cf e5d8 7aaf 4805 aa6b  .Q....%...z.H..k
00000300: 4021 9bf4 ffca 0c1c 6a4f 0928 bf5f 6c07  @!......jO.(._l.
00000310: 0ae9 58a7 9034 7ce4 fd1f 5970 96bd f452  ..X..4|...Yp...R
00000320: 6217 0ba8 cb27 5c8b f7b4 69bb 4fec 59e7  b....'\...i.O.Y.
00000330: 069d 433f 1be3 4685 ac47 e810 cb52 8b82  ..C?..F..G...R..
00000340: d211 9d65 5712 3ac8 bdf1 e39a 2682 524a  ...eW.:.....&.RJ
00000350: 636b 2cef 0d0f f8a3 6cdc 44fd 63bd ed69  ck,.....l.D.c..i
00000360: eed0 b651 0b64 6084 4db5 946a 645f a09f  ...Q.d`.M..jd_..
00000370: eb11 b01d d650 b6e1 2043 4eac de34 22a0  .....P.. CN..4".
00000380: a87e b95f 3131 354c 64cb 0511 6373 b420  .~._115Ld...cs.
00000390: 648f c007 376b 2761 217c a66b a697 1b4b  d...7k'a!|.k...K
000003a0: f6d9 152e ed39 82fa d7e1 450d f698 1217  .....9....E.....
000003b0: 5ff1 9715 a2df bee7 6ece 8f6a c2ef b52b  _.......n..j...+
000003c0: 8f39 7a4a 6717 150e d68e 8a57 b631 d987  .9zJg......W.1..
000003d0: 51b6 2bfb 5871 bf7c 798b 23c0 aa97 1c54  Q.+.Xq.|y.#....T
000003e0: 9b73 acce 45f7 24e6 b137 d1e1 d9d6 3c21  .s..E.$..7....<!
000003f0: a206 f6e9 728c f928 93a7 1637 9ff8 adc9  ....r..(...7....
00000400: ad7b ea33 4637 2356 2737 7332 7c5a ac9a  .{.3F7#V'7s2|Z..
00000410: bed9 b261 d5c3 134f 1b87 3e2c 67be 0ae7  ...a...O..>,g...
00000420: c0f3 5860 35f6 9047 bc9b 550a 328b 809e  ..X`5..G..U.2...
00000430: 5d5f 80f6 a8bc e26c 0d31 a942 c29b b7f3  ]_.....l.1.B....
00000440: a47a fd60 7482 f9f3 7a6f 366e 35d6 fff7  .z.`t...zo6n5...
00000450: f418 a95b 8db6 8222 8580 46bd f918 918d  ...[..."..F.....
00000460: 4bcd d043 ef57 6461 f119 e9b9 402a 8017  K..C.Wda....@*..
00000470: 6938 3249 663f f2e2 6cec 9987 e63e 0ff8  i82If?..l....>..
00000480: 9699 22eb 4197 a537 fcb5 60d5 2a3e df13  ..".A..7..`.*>..
00000490: 23b5 b05b 895c d31f ff52 73d5 018a d6a9  #..[.\...Rs.....
000004a0: 82b3 28b5 e24e a04f 8c3a 878c 2121 63bf  ..(..N.O.:..!!c.
000004b0: e343 8fb8 2aca 75e6 b781 729c 751d edb1  .C..*.u...r.u...
000004c0: 22a5 defb 83e2 30b9 6ec0 fef4 d7d0 4465  ".....0.n.....De
000004d0: 876f 481f 19ab c328 15d9 0efe 7649 d379  .oH....(....vI.y
000004e0: 840d 1524 c46f c855 4184 2503 39c0 1b4f  ...$.o.UA.%.9..O
000004f0: 6913 682b 7007 4310 81be 740a 6a40 17d5  i.h+p.C...t.j@..
00000500: f35f 232e 413a 6359 45d8 12d4 56e2 fe02  ._#.A:cYE...V...
00000510: 2bed 4479 f913 cfa4 04ee 352a a676 2cc1  +.Dy......5*.v,.
00000520: 44c0 3bc4 11c6 5b9c e142 dc9b 37e3 8978  D.;...[..B..7..x
00000530: e715 45b0 dc25 f5e1 356c 70af d094 8c5e  ..E..%..5lp....^
00000540: 2008 2e80 30ce a821 f489 e8ee 91d6 67ad   ...0..!......g.
00000550: 4134 73c8 b3f0 4fd5 93e4 9ad4 4a3a 9368  A4s...O.....J:.h
00000560: 4f86 9efc 2d99 65da 4c45 376f 8da0 2488  O...-.e.LE7o..$.
00000570: 2a54 83b3 4a8f 6433 6ddb 6f6e 6c67 d50d  *T..J.d3m.onlg..
00000580: 226c 3b7d 75a6 2894 f7e0 bcd1 f434 bb6c  "l;}u.(......4.l
00000590: 0a77 1d6c a9d7 ec11 77fd 82bf 8e10 f80a  .w.l....w.......
000005a0: bf84 302b 2acc 181a abc0 0f08 2b52 dfed  ..0+*.......+R..
000005b0: 9ee3 e89f 91e4 5ec3 6a65 ef7c 68cd a7aa  ......^.je.|h...
000005c0: b03b 47fa 04ce 95d9 2e92 54b2 1aaf 0629  .;G.......T....)
000005d0: db74 2506 c2dc 5ebf 5b67 cb0f 1319 2e41  .t%...^.[g.....A
000005e0: 2e3f f4ec d596 5e5a 8bb1 bad9 c505 e486  .?....^Z........
000005f0: bdc7 a5ae 4d65 1f54 8644 8901 bb80 39db  ....Me.T.D....9.
00000600: b81f 386c 918c 4617 23e8 d388 6b38 7d96  ..8l..F.#...k8}.
00000610: 9f57 bdf9 03fe 0856 a876 58b9 f3d3 d007  .W.....V.vX.....
00000620: 2b8b 499f 1e02 e34c 25ec 8c3f eedb 6545  +.I....L%..?..eE
00000630: 46e4 069e e13f 3cf3 932d ddac b628 2d44  F....?<..-...(-D
00000640: 18ae a9b0 50e6 4421 3b65 fbdd 1dbb 1edb  ....P.D!;e......
00000650: 2482 e167 fa77 0895 aa7e af4e 77b4 c634  $..g.w...~.Nw..4
00000660: af87 d225 e33d adbf 5956 ed49 9e27 33ce  ...%.=..YV.I.'3.
00000670: 890b 5de6 807d 885d 7d7d 0a64 f63f 12db  ..]..}.]}}.d.?..
00000680: 4a0f 3794 f9de 01ef 6cb2 fbf6 2f42 9c59  J.7.....l.../B.Y
00000690: c953 22bf 85ee b574 d754 0a8e 159f 7abf  .S"....t.T....z.
000006a0: 39c3 0179 9dc0 aa43 aa47 c325 bb94 fea9  9..y...C.G.%....
000006b0: 105f e6ca 1589 6393 b8fc 5c32 633b 1f59  ._....c...\2c;.Y
000006c0: 1ab4 5ab2 4136 8b58 8cca 9208 e1bc 69a2  ..Z.A6.X......i.
000006d0: 9c42 60c3 c02f 6dd1 ff9d d0ad e921 cdf7  .B`../m......!..
000006e0: b99f 919e b4f1 e2fb 846d 322f 562a 3058  .........m2/V*0X
```


