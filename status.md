# General information
Started by creating a page with a video element showing a static file to test brrowser support.
Configured a ffserver that was streaming video recieved from wowza.
Used ffmpeg and m3u8_segmenter to create a m3u8 playlist file and stream files using nginx web server.

Bothe kinds of streaming could eventally configured to be served directly by wowza server.

# Static file serving
Only testested for html5 video element status on devices
Works well on all browsers using one video element with various format sources.
- low res devices need to go full screen (chat or other html elements not visible)


# ffserver file streaming
* mp4
  - no show for live streaming
  ```
  muxer does not support non seekable output Error writing output header
  ```
* webm
  - works on desktop browsers only
* ogg
  - no show for live streaming
  ```
  Feed stream has become desynchronized -- disconnecting
  ```

```
Port 8090                      # Port to bind the server to
BindAddress 0.0.0.0
MaxHTTPConnections 2000
MaxClients 1000
MaxBandwidth 10000             # Maximum bandwidth per client
                               # set this high enough to exceed stream bitrate
CustomLog -
NoDaemon                       # Remove this if you want FFserver to daemonize after start

<Feed feed1.ffm>               # This is the input feed where FFmpeg will send
   File ./feed1.ffm            # video stream.
   FileMaxSize 1M              # Maximum file size for buffering video
   ACL allow 127.0.0.1         # Allowed IPs
</Feed>

<Stream test.webm>              # Output stream URL definition
   Feed feed1.ffm              # Feed from which to receive video
   Format webm

   # Audio settings
   AudioCodec vorbis
   AudioBitRate 64             # Audio bitrate

   # Video settings
   VideoCodec libvpx
   VideoSize 720x576           # Video resolution
   VideoFrameRate 25           # Video FPS
   AVOptionVideo flags +global_header  # Parameters passed to encoder
                                       # (same as ffmpeg command-line parameters)
   AVOptionVideo cpu-used 0
   AVOptionVideo qmin 10
   AVOptionVideo qmax 42
   AVOptionVideo quality good
   AVOptionAudio flags +global_header
   PreRoll 15
   StartSendOnKey
   VideoBitRate 400            # Video bitrate
</Stream>
```

```
<video autoplay>
      <source type="video/webm" src="http://192.168.0.117:8090/test.webm"></source>
</video>
```


# HLS (HTTP Live Streaming - Apple)
```
ffmpeg -i rtmp://216.18.184.22:1935/UserEdge/Raychel4U/Raychel4U_Raychel4U \
    -acodec libmp3lame -ac 1 -vcodec libx264 -s 320x240  -level 30 -f mpegts - \
    | m3u8-segmenter -i - -d 3 -p video/raychel -m video/raychel.m3u8 -n 2 -u http://192.168.0.117/
```

```
 <video autoplay controls src="http://192.168.0.117/video/raychel.m3u8"></video>
 <video src="http://devimages.apple.com/iphone/samples/bipbop/bipbopall.m3u8" controls autoplay></video>
```

Works rather well on ios and desktop webkit but works randomly on andorid devices.

Could not get it to work on desktop browsers (have not tried on mac).

Page talking about staus of hls on andorid versions: 
http://blog.metacdn.com/live-streaming-to-android-devices

# Conclusion
At the time of this writing I could not find and acceptable solution for live streaming using HTML5 video element.
The closest to work is HLS but android is not consistent across versions/devices.
