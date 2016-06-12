Usage
=====

Command description
-------------------


Please [install](installation.md) the tools before trying to use it.

* **somagic-init**.
Load the firmware and will try to connect to the easycap device
* **somagic-capture**.
Record video, output the raw stream in stdout 
* **somagic-audio-capture**.
Record audio, output the raw stream in stdout 
* **somagic-both**.
Record video and audio, output the video in stdout and the audio in stderr

The format of the raw video and audio depends on the options your choose.
Your can find more information using man page (or in the examples below)
```bash
man somagic-capture
```


Live visualisation of the video
-------------------------------

dependencies
* mplayer

###### PAL format
```bash
#kill every process lost (could be usefull if the script is killed)
sudo killall -9 somagic-capture

# init the somagic driver
sudo somagic-init

sudo somagic-capture | mplayer -vf yadif,screenshot -demuxer rawvideo -rawvideo "pal:format=uyvy:fps=25" -aspect 4:3 -
```

###### NTSC format
```bash
#kill every process lost (could be usefull if the script is killed)
sudo killall -9 somagic-capture

# init the somagic driver
sudo somagic-init

# recording
sudo somagic-capture -n | mplayer -vf yadif,screenshot -demuxer rawvideo -rawvideo "ntsc:format=uyvy:fps=30000/1001" -aspect 4:3 -
```

###### SECAM format
```bash
#kill every process lost (could be usefull if the script is killed)
sudo killall -9 somagic-capture

# init the somagic driver
sudo somagic-init

# recording
sudo somagic-capture --secam | mplayer -vf yadif,screenshot -demuxer rawvideo -rawvideo "pal:format=uyvy:fps=25" -aspect 4:3 -
```

Record video and audio.
---------------------------------------

This section is a cookbook. Feel free to add yours solutions !

dependencies:
* libav-tools (avconv)
* moreutils   (buffer)

###### Secam format

``` bash
# kill every process lost (could be usefull if the script is restarted)
sudo killall -9 somagic-both
sudo killall -9 somagic-capture

# init the somagic driver
sudo somagic-init

# recording
rm -f  .video .audio .video_buffer .audio_buffer
mkfifo .video .audio .video_buffer .audio_buffer
sudo somagic-both --secam 1>.video 2>.audio & pid=$!

# buffer the data acquired (prevent frame lost)
buffer < .video > .video_buffer &
buffer < .audio > .audio_buffer &

sleep 1

avconv \
-f rawvideo -pix_fmt uyvy422 -r 25 -s:v 720x576 -i .video_buffer \
-f s16le -sample_rate 24000 -ac 2 -i .audio_buffer -strict experimental \
-vcodec mpeg4 -vtag xvid -qscale:v 7 \
-vf yadif -s:v 720x540 \
video.avi

# now, type ctrl-c to stop the encoding
# you can then kill somagic-both
```
