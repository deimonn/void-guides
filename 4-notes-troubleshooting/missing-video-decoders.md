# Missing video decoders

Ever played a video and realized there's no audio even though there should be? Or perhaps you use my `clapper` package and you get an error saying "X" decoder is missing.

There are various libraries and plugins that video players can use that are not automatically installed. For the case of `clapper` and other GStreamer-based players, I recommend you install all of the following:

```Shell
sudo xbps-install gst-plugins-bad1 gst-plugins-base1 gst-plugins-good1 gst-plugins-ugly1 gst-libav gstreamer-vaapi
```

These are all available open-source plugins for GStreamer, plus `libav` and video acceleration.
