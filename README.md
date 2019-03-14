# ezcap-linux
This page describes how to get a ForwardVideo EZCap USB Video Grabber device working on Linux, for use with VLC and other things too I suppose.

This was done some time ago, so some things may be a little bit out of date, but the general information will still be correct.

## Background
The EZCap USB Video Grabber is a seemingly, cheapo no-brand, I would go so far as to say in my opinion "crappy", USB video capture device that is used to grab video (and audio) from an analogue video camera.

I acquired one of these as part of a bird-box nest-cam project.

Of course this device worked on Windows but did not work immediately on Linux. To add insult to injury the device does work on Mac OSX, but only after you pay more Apple Tax for no good reason for some bespoke software.

## The Problem

Plugging the device into a Linux PC does *not* cause the device to be auto-detected. If it worked, a Video4Linux video device such as "/dev/video0" would be created and playable using VLC. Sadly no such device was created.

This caused me a lot of headaches to make it work, and I threw the towel in a few times, but eventually I got it fully working...

## Making It Work for Video4Linux

This device uses an EM2860 chip-set, which seems to be generally supported quite well in the Video4Linux project. Since there are many devices using that chip-set, the idea was to try and trick the OS into recognising this particular device as one that was supported.

Starting with "lsusb":

```
Bus 002 Device 005: ID eb1a:5124 eMPIA Technology, Inc.`
```

The trick is therefore this:

```
sudo modprobe em28xx card=64
echo eb1a 5124 | sudo tee /sys/bus/usb/drivers/em28xx/new_id
```

Using a "card" type of "64" was somewhat arbitrary - the idea was to pick something generic - and indeed other card types may also work. These card identifiers are listed in the Video4Linux source code.

After executing those two commands above, the result of executing "dmesg" is:

```
[46753.005870] Linux video capture interface: v2.00
[46753.017123] usbcore: registered new interface driver em28xx
[46757.493158] em28xx: New device  USB VIDBOX FW Audio @ 480 Mbps (eb1a:5124, interface 0, class 0)
[46757.493163] em28xx: Video interface 0 found: isoc
[46757.493380] em28xx: chip ID is em2860
[46757.628967] em2860 #0: i2c eeprom 00: 1a eb 67 95 1a eb 24 51 50 00 20 03 8c 28 6a 22
[46757.628980] em2860 #0: i2c eeprom 10: 00 00 24 57 06 02 00 00 00 00 00 00 00 00 00 00
[46757.628990] em2860 #0: i2c eeprom 20: 02 00 01 00 f0 10 01 00 00 00 00 00 5b 00 00 00
[46757.629000] em2860 #0: i2c eeprom 30: 00 00 20 40 20 80 02 20 01 01 00 00 00 00 00 00
[46757.629010] em2860 #0: i2c eeprom 40: 00 00 00 00 00 00 00 00 00 00 00 00 00 c4 00 00
[46757.629020] em2860 #0: i2c eeprom 50: 00 a2 00 87 81 00 00 00 00 00 00 00 00 00 00 00
[46757.629030] em2860 #0: i2c eeprom 60: 00 00 00 00 00 00 00 00 00 00 22 03 55 00 53 00
[46757.629040] em2860 #0: i2c eeprom 70: 42 00 32 00 2e 00 30 00 20 00 56 00 49 00 44 00
[46757.629050] em2860 #0: i2c eeprom 80: 42 00 4f 00 58 00 20 00 46 00 57 00 28 03 55 00
[46757.629060] em2860 #0: i2c eeprom 90: 53 00 42 00 20 00 56 00 49 00 44 00 42 00 4f 00
[46757.629070] em2860 #0: i2c eeprom a0: 58 00 20 00 46 00 57 00 20 00 41 00 75 00 64 00
[46757.629080] em2860 #0: i2c eeprom b0: 69 00 6f 00 00 00 00 00 00 00 00 00 00 00 00 00
[46757.629090] em2860 #0: i2c eeprom c0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
[46757.629100] em2860 #0: i2c eeprom d0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
[46757.629109] em2860 #0: i2c eeprom e0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
[46757.629119] em2860 #0: i2c eeprom f0: 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00 00
[46757.629131] em2860 #0: EEPROM ID = 1a eb 67 95, EEPROM hash = 0x806f2156
[46757.629133] em2860 #0: EEPROM info:
[46757.629135] em2860 #0:       AC97 audio (5 sample rates)
[46757.629136] em2860 #0:       500mA max power
[46757.629138] em2860 #0:       Table at offset 0x24, strings=0x288c, 0x226a, 0x0000
[46757.629141] em2860 #0: Identified as Easy Cap Capture DC-60 (card=64)
[46758.015717] saa7115 0-0025: saa7113 found @ 0x4a (em2860 #0)
[46758.782583] em2860 #0: Config register raw data: 0x50
[46758.806434] em2860 #0: AC97 vendor ID = 0x83847652
[46758.818419] em2860 #0: AC97 features = 0x6a90
[46758.818423] em2860 #0: Sigmatel audio processor detected(stac 9752)
[46759.289961] em2860 #0: v4l2 driver version 0.2.0
[46761.220200] em2860 #0: V4L2 video device registered as video0
[46761.220205] em2860 #0: V4L2 VBI device registered as vbi0
[46761.220207] em2860 #0: analog set to isoc mode.
```

Checking the "dev" directory also shows that a new device "/dev/video0" *was* created. So far, so good.

## Making It Work for VLC

Of course it did not work right away in VLC, playing the "v4l2://" MRL showed nothing at all. It did work immediately with the ancient "xawtv" application so I knew it was worth persevering...

After some experimentation it turns out that it was necessary to explicitly specify the video width and height - the maximum dimensions supported by the device are 720x576, so:

```
vlc --v4l2-dev /dev/video0 --v4l2-standard PAL --v4l2-input 0 --v4l2-width 720 --v4l2-height 576 v4l2://
```

Lo and behold, the video appears.

Input 0 is Composite, input 1 would be S-Video.

### Making Audio Work

Up to this point, the video is working, but no audio.

This is a simple matter of selecting the right audio input device. The correct value for this will change depending on what other audio cards are in the system, but for me this was "hw:2,0", so "--input-slave=alsa://hw:2,0", and:

```
vlc --v4l2-dev /dev/video0 --v4l2-standard PAL --v4l2-input 0 --v4l2-width 720 --v4l2-height 576 --input-slave=alsa://hw:2,0 v4l2://
```

### Summary

Right now this is not ideal, but works well enough (both video and audio) for me, and of course now I can use my camera feed in my bespoke vlcj applications.

#### Working VLC capture settings

![VLC capture settings](https://github.com/caprica/ezcap-linux/raw/master/etc/vlc-capture.png)

#### VLC advanced capture settings

![VLC advanced capture settings](https://github.com/caprica/ezcap-linux/raw/master/etc/vlc-capture-advanced.png)

#### VLC on Linux playing the video from the camera

![VLC snapshot](https://github.com/caprica/ezcap-linux/raw/master/etc/vlc-capture-snapshot.png)

In this picture the camera is in IR mode since it is night time, a colour picture is available during the day.

## Games Console Capture
                     
It is even possible to use this device to capture video and audio from video games consoles - it's not great, well, it was kinda cheap to be fair, but it works.

Example settings for VLC that can capture footage from an XBox (Playstation is bestest by the way) games console:

```
vlc                         \
--v4l2-standard=PAL_60      \
--v4l2-input=0              \
--v4l2-audio-input=1        \
--v4l2-width=1280           \
--v4l2-height=720           \
--v4l2-aspect-ratio=16\:9   \
--no-v4l2-controls-reset    \
--live-caching=30           \
--input-slave=alsa://hw:2,0 \
v4l2://
```

Note that with these VLC settings if you save the footage it will be as an uncompressed AVI and the resulting file size will be very large, e.g. 1Gb per minute of footage captured. You can of course set your own transcoding options to make smaller files.

Note also that the "live-caching" setting is crucial, as without it the captured footage will lag the games console output at an unacceptable rate.

The quality of the captured footage is decent, but it is analog so is not optimal. The following image shows a snapshot from VLC captured this way:

![Games console capture](https://github.com/caprica/ezcap-linux/raw/master/etc/games-console-capture.png)

The point here is to show how it can be done on a Linux with a cheap capture device, it is recommended of course to use a dedicated games console capture device that can natively record and encode 1080p60 video, and have HMDI pass through to your TV.

# Final Notes

Would I recommened this capture device?

No way.

But it was cheap and necessary for a project that I was doing at the time.

A digital IP camera solution is going to be much, much, better than this.

