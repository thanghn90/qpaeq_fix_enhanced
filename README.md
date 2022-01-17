# Preface
There are many other superior alternatives to qpaeq (pulseaudio-equalizer default GUI, https://github.com/pulseaudio/pulseaudio/blob/master/src/utils/qpaeq), such as pulseeffects(https://salsa.debian.org/debian/pulseeffects) and pulseaudio-equalizer-ladspa(https://github.com/pulseaudio-equalizer-ladspa/equalizer). However, I have stuttering issue with pulseeffects on my GalliumOS-installed Acer CB3-431 chromebook that I did not when using qpaeq, and I do like the default 10-band equalizer of qpaeq where the frequencies are spread out logarithmically, unlike the ladspa one that has 15 non-logarithmic bands by default. Pulseeffects are too sophisticated for my need and are a lot more resource-hungry. qpaeq is the "just enough" GUI, but it does NOT show the db Gain value (not to mention that the gain values, or coefficients, are not following logarithmic scale). Because I know python AND Qt AND a bit of digital signal processing maths, I set out to modify the default qpaeq GUI to make more sense out of it. I know it may be too late to help save qpaeq from deprecation (and oblivion?), but still, it's an effort to create something that works for such a low-end hardware like a 5-year-old chromebook. Plus, as far as I understand correctly, pulseeffects are not available for ARM systems yet, due to its dependency on fortran (?). When your system have only the default pulseaudio-equalizer choice in your apt repositories, I hope you will find my work useful.

# Fixes and Enhancement
- Added a dB-gain label to each slider
- Increase default slider height to make the sliders longer
- Change the slider ranges to be between -30dB and +30dB

# Instruction
Very simple: just download the qpaeq from this repositories (under the correct version directory) and replace it under your /usr/bin/ using sudo. You may need to change file permission to executable afterward:

<code>sudo chmod 755 /usr/bin/qpaeq</code>

Also, if you didn't know already, pulseaudio-equalizer requires (1) modification to /etc/pulse/default.pa to run, and (2) you need to change the output sink of your audio player to pulseaudio-equalizer's.
1. Type:

sudo nano /etc/pulse/default.pa

to edit default.pa with nano text editor under sudo, and add the following lines to the end of it:

<code>load-module module-equalizer-sink</code>

<code>load-module module-dbus-protocol</code>

Then save it (Ctrl-O) and exit (Ctrl-X).

Reboot to take effect (yes, not just invoke <code>pulseaudio --kill</code> and <code>pulseaudio --start</code>, but a full reboot).

2. Once rebooted, run qpaeq to make some change to the equalizer and save it as a test preset, and then start playing some sound/music via an audio player. If the changes to equalizer does not make any effect, chances are your audio player still point to the built-in output audio "sink". You need to change it to the equalized "sink". Click on the PulseAudio System Tray --> Playback Streams --> <your_audio_player> --> Select "FFT based equalizer on..." instead of the default "Build-in..."



I believe a full reboot in step 1 above should keep you away from this headache, as I do have to make this change when I only kill and start pulseaudio process. Rest assured, I don't have to change the output sink of every audio player, so I think a reboot should fix this.

Currently, I can only support qpaeq version of Qt4, pulseaudio-equalizer 1:11.1-1ubuntu7.11 and version of Qt5 on the official pulseaudio github repo (https://github.com/pulseaudio/pulseaudio/blob/master/src/utils/qpaeq). If both doesn't work for you, send me your qpaeq and I'll make the needed changes.
  
# Caveats
For some reason, all equalizers (qpaeq, pulseeffects, and ladspa) yield the same stuttering issue for me when I change the volume of individual app. It's a good practice to change the volume of the app while keeping the master volume at maximum, since reducing the source's volume will help prevent clipping from happening in case the equalizer amplifies the signal too much. I guess it's an inherent issue with pulseaudio equalizer framework in general. A workaround is to change the preamp value of qpaeq, but that means everytime you want to change the volume, you have to open qpaeq. PEACE audio equalizer in Windows does not have this issue, though.

# How I did it
First, I added another set of labels underneath each slider of the equalizer to show me the gain value.
I also stretched the slider to make them longer by changing the slider's minimum height.
As I read the code, I realized the coefficients (i.e. gain values) are not logarithmic (i.e. not measured in dB, but rather just a pure real floating-point value to be multiplied with the signal's value - yes, you can call it linear coefficients). Thus, I further modify the code where the coefficients are computed to turn them into dB.
The equation I used to convert logarithmic coefficients (dbGains) into linear coefficients is:

<code>linear_coefficient = 10^(dbGain/20)</code>

(mathematically, it should be /40, but my test shows that /20 makes it much closer to the result of pulseeffects and Peace audio equalizer in Windows. Beside, pulseaudio-equalizer is a fft-based filter, so the equation should be different from biquad filters)

In case you want to modify the code by yourself (maybe because no versions of qpaeq in this repository work with your Linux distro), simply search for the lines of codes containing the word <code>Thang</code> in any of the qpaeq versions in this repository and then start modify your qpaeq accordingly.
