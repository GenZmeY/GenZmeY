# About VFR support:
I can get all information about each frame like this:  
`ffprobe -v quiet -show_entries packet -select_streams <stream_index> <videofile>`

And here is a way to take something specific:  
`ffprobe -v quiet -show_entries packet=pts_time,duration_time -select_streams <stream_index> <videofile>`

It is possible to concatenate all frames from variable times using ffmpeg concat.
Need to make a file with the following content:  
```
ffconcat version 1.0
file './frames_upscaled/000001.png'
duration 0.042000
file './frames_upscaled/000002.png'
duration 0.042000
file './frames_upscaled/000003.png'
duration 0.042000
...
```

I should note that in the case of VFR there is more than just frame time. There are two parameters: pts and dts  
* pts is the presentation time stamp, that is, how the frames should be displayed.  
* dts is a decoding time stamp, that is, in what order the frames should be decoded.  
And apparently the frames are stored in dts order, which complicates things for me.  

Then merge video like this:  
```
ffmpeg \
	-hide_banner \
	-f "concat" \
	-safe 0 \
	-i "$FrameDurationList" \
	-vsync vfr \
	-r "42" \
	-vcodec "$VideoCodec" \
	-preset "$Preset"  \
	-pix_fmt "$PixelFormat" \
	$(auto_bitrate) \
	$(auto_x265params) \
	"$VideoUpscaled"
```

This is the closest thing I could do (this option gives VFR video at the output, all other attempts continued to create CFR) but this is still a wrong option, since if you decompose it into frames again, you can see that their duration does not match the original video.
In addition, the -r parameter is specified here. In this case, it sets the maximum FPS (not average). I don't understand why it is needed, because all the information about the frame time is already exists in the concat file (!), but if you do not specify it, ffmpeg sets it to 25fps and this is definitely not what I need
