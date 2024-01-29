---
layout: post
title:  "Video streaming for dummies"
date:   2024-01-28
categories: how-to tools
---

This week, I put together a video and streamed it to a few hundred viewers. I had some basic AV experience from high school, but I'd never run a stream before, so I learned a lot along the way. This post runs through those learnings, the setup I ended up with, and the tools I found useful along the way.

## The objective

First, I needed to put together the video itself. It would incorporate slideshows backed with music, video clips, and WhatsApp voice notes. Due to some early indecision around how I'd broadcast it, I needed to be able to stream it to Facebook, Youtube, and Zoom[^1]. 

## Preparing the video

### Getting videos from YouTube

The first step was finding background music for the video. I got mine straight from YouTube (this blog has no comment section, so all you audiophiles can't be mean to me). You'll get many results if you Google "download youtube video", but have little confidence that those sites won't just give me `ransomware.mp3.exe``.

As such, I used [yt-dlp](https://github.com/yt-dlp/yt-dlp) instead. It's a command line tool that downloads videos from YouTube. To download [this (copyright-free)](https://www.youtube.com/watch?v=yX-mmvu9J7U) track, you'd run:

```
$ yt-dlp https://www.youtube.com/watch?v=yX-mmvu9J7U
```

### Extracting audio

You *could* pass the `-x` flag to `yt-dlp` to download the videos as audio files. If you've already got them, however, you can use [FFmpeg](https://ffmpeg.org/) (a media transcoding tool) to convert them to the audio format of your choice:

```
$ ffmpeg -i youtube-video.mp4 youtube-audio.mp3
```

### Loudness normalization

We'd gathered WhatsApp voice notes from several of the attendees that needed to be included in the video. As is to be expected, the audio levels of the clips varied *wildly*. Conveniently, [Audacity](https://www.audacityteam.org/) has a tool to normalize the loudness (measured in [*LUFS*](https://en.wikipedia.org/wiki/LUFS)) of audio clips:

![Audacity loudness](/assets/images/2024-01-28-stream-setup.md/audacity-louadness.png)

LUFS are similar to decibels, but they take into account that we (humans) perceive the volume of sounds differently depending on their pitches. Very low-pitches, for examle, *sound* much quieter to humans. Each red line below shows the how many decibels a sound needs to be to have the same loudness at each pitch:

![LUFS](/assets/images/2024-01-28-stream-setup.md/lufs.svg)

## Putting together the video

If you need to put a bunch of media together in a video, Microsoft's [Clipchamp](https://clipchamp.com/en/) is great. It's free, easily accessible, and has all the functionality you'll need. My biggest gripe with it is its lack of structured media management. Once you import media to Clipchamp, it's all lumped together in the sidebar, sorted by filename. 

I got around the problem by writing a little Python script[^2] to rename files before I imported them. That way, they were grouped together in the sidebar.

## Streaming

### Streaming platforms and protocols

Streaming platforms like YouTube[^3], Facebook, and Twitch use the [Real-Time Messaging Protocol (RTMP)](https://en.wikipedia.org/wiki/Real-Time_Messaging_Protocol) to receive your video. When you open the streaming console for each service (on their web sites), you'll find the RTMP endpoint to which you'll send your video and a *stream key*, which both identifies the stream as yours _and_ acts as a secret to authenticate you (it's like a username and password in one):

![Stream key](/assets/images/2024-01-28-stream-setup.md/stream_key.png)

### OBS

Open Broadcaster Software (OBS) is a popular tool for video streaming tool and is what you'll use to send your stream to the services via RTMP. By default, it only supports streaming to a single platform at a time, so [obs-multi-rtmp](https://github.com/sorayuki/obs-multi-rtmp/releases/) adds a plugin to let you stream to multiple at once! Drop your stream key and the RTMP endpoints into the plugin and you're ready to go.


![Multi RTMP](/assets/images/2024-01-28-stream-setup.md/stream_rtmp_diagram.png)

### Streaming to zoom

Unlike streaming services, Zoom doesn't support RTMP as a video input. Its only supported video inputs are cameras, screen shares, and Zoom's built-in media playback (which plays a video file on the call).

Luckily, OBS has a *Virtual Camera*, which takes OBS' output and lets your PC treat it like a webcam. Once OBS is installed, the virtual camera automatically shows up in the list of cameras in Zoom.

![Multi RTMP with zoom video](/assets/images/2024-01-28-stream-setup.md/stream_rtmp_zoom_video_diagram.png)

We're not quite done yet, though; the virtual camera doesn't give us *audio*. For that, we use OBS' *audio monitoring*. While running a stream, audio monitoring lets you play your stream's audio through your headphones or speakers (so that you can monitor it) We're going to ~~ab~~use that functionality to get audio into Zoom.

![OBS audio monitor](/assets/images/2024-01-28-stream-setup.md/obs_audio_monitor.png)

[Virtual Audio Cable](https://vb-audio.com/Cable/) is a free tool that creates a new audio input and output device in Windows. Anything you play into the output device is immediately played straight back through the input device. By using the virtual output as a monitoring device in OBS and the virtual input as the microphone in Zoom, we get our audio in the call!

![Multi RTMP with zoom audio and video](/assets/images/2024-01-28-stream-setup.md/stream_rtmp_zoom_full_diagram.png)

### Stop Zoom from "helping"

The last steps here are to ask zoom kindly to stop making things worse. By default, your video is mirrored (so any text will be backwards) and noise suppression is enabled (which makes music inaudible).

Un-flipping your video is straightforward enough. Just untick the box:

![Zoom mirror](/assets/images/2024-01-28-stream-setup.md/zoom_mirror.png)

To fix the noise cancellation, you have to pinky promise Zoom that you're a musician:

![Zoom sound toggle](/assets/images/2024-01-28-stream-setup.md/zoom_sound_toggle.png)

Doing so adds this button to your zoom calls. Clicking it and enabling "original sound" disables audio processing entirely:

![Zoom sound toggle](/assets/images/2024-01-28-stream-setup.md/zoom_sound.png)

## Final thoughts

Everything went really well. For your first stream (and maybe for every stream), make a checklist. I made one and it made me much more confident that I'd set everything up before going live.

Tools like [StreamYard](https://streamyard.com/dashboard) are really great alternatives to OBS if you don't want to fuss with configuration (which is a perfectly reasonable reason not to use OBS), but OBS' open source community and plugins make it a really powerful tool if you do feel like diving in.

## Feet note

[^1]: If you're considering doing something like this, I would recommend streaming only. Having Zoom in the mix makes things much more complex.

[^2]: Here's the script, in case it's useful to you (it actually ended up above just above this because footnote formatting is weird).

```
import shutil
import os

dirs = [d for d in os.listdir() if os.path.isdir(d)]

for d in dirs:
    current_dir = os.getcwd()
    files = [f for f in os.listdir(f"./{d}")]
    i = 0
    for f in files:
        ext = f.split('.')
  
        # Copy everything to a new (indexed) filename .
        # Files in ./foo/ will become ./foo/foo-1, ./foo/foo-2
        shutil.copyfile(f"./{d}/{f}", f"./{d}/{d}-{str(i)}.{ext[-1]}")
        i = i + 1
```

[^3]: If you're going to be streaming to YouTube, make sure to sign up for YouTube live early; it takes 24 hours for your account to be allowlisted to stream.

[![Hits](https://hits.seeyoufarm.com/api/count/incr/badge.svg?url=https%3A%2F%2Fbrydonleonard.github.io%2Fhow-to%2Ftools%2F2024%2F01%2F28%2Fstream-setup.html&count_bg=%2379C83D&title_bg=%23555555&icon=&icon_color=%23E7E7E7&title=hits&edge_flat=false)](https://hits.seeyoufarm.com)