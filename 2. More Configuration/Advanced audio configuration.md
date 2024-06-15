# Advanced audio configuration

This page covers further audio configuration that the reader may wish to apply to their PipeWire setup.

## Audio routing

You can route any device or application's input and/or output to any other in PipeWire, you'll just need a graphical interface for it. There's two that I currently know of:

- `helvum` is very intuitive and simple to use. I personally like this one.

- `qpwgraph` is a little more convoluted (takes a lot after `qjackctl`), but it does allows you to save the graph state unlike `helvum`.

Either should work out of the box, but routing this way can be a little problematic; everytime you close and open an application, the graph connections will be undone. Sometimes closing and opening is not even necessary and the application will simply reset connections on its own.

Freezing the graph using `qpwgraph` fixes this but leads to a plethora of other problems, such as any new applications being unable to make connections anywhere.

If you want to apply effects to your audio output or microphone system-wide, or create separate audio streams for recording, it is better to create dedicated *sinks* and *sources* instead and make the connections with those.

### Creating a sink & source

Creating a sink (audio output) is pretty simple:

```Shell
pactl load-module module-null-sink media.class=Audio/Sink sink_name=audio-sink channel_map=front-left,front-right
```

This will create a new sink named `audio-sink`; you'll see it pop up inside the `helvum` or `qpwgraph` graph immediately. You can select this sink as the default output device system-wide, or select it manually inside applications to redirect audio there.

Creating a source (audio input) is not too dissimilar:

```Shell
pactl load-module module-null-sink media.class=Audio/Source/Virtual sink_name=audio-source channel_map=front-left,front-right
```

You'll see it pop up inside the graph just like the sink. You can similarly select it as the default recording device system-wide, or within desired applications.

By default, the sink won't send audio anywhere, and the source will be completely silent; you need to set up connections (and a chain of effects) on your own. You can do this easily using the graph; I personally just use a DAW when I need such effects to make it easier.

### Automating the process

Any sinks and sources you create using the above method will be destroyed as soon as you restart the session, so you'll have to create them again.

You can automate this process via your `~/.loginrc` file, if you followed [Running user scripts after login](../1.%20Installation/Guide.md#running-user-scripts-after-login).

Append the following:

```Bash
# Set up audio sinks & sources.
{
    # Give PipeWire a little bit of time to settle.
    sleep 2

    # Create sinks.
    pactl load-module module-null-sink media.class=Audio/Sink sink_name=audio-sink channel_map=front-left,front-right
    # Create sources.
    pactl load-module module-null-sink media.class=Audio/Source/Virtual sink_name=audio-source channel_map=front-left,front-right
} &>/dev/null &
```

Customize as desired.

## JACK support

Some audio plugins and applications support JACK, but not ALSA, PulseAudio or PipeWire. To be able to use them with PipeWire anyway, you can enable the JACK interface.


Install the relevant package:

```Shell
sudo xbps-install libjack-pipewire
```

You can run JACK applications under PipeWire by invoking them via `pw-jack`.

Alternatively, run the following commands to set up a system-wide override:

```Shell
echo "/usr/lib/pipewire-0.3/jack" | sudo tee -a /etc/ld.so.conf.d/pipewire-jack.conf
sudo ldconfig
```
