---
layout: post
title: Problems with NVIDIA driver in Debian
issue: 1
---

Today I installed nvidia-glx meta-package. Done so because Gnome Shell turns
screen into a kaleidoscope when started under an X.org which uses nouveau
driver.

Disclaimer: system is a Debian from unstable running on a Zotac MAG HD-ND1
hardware with NVIDIA ION graphics, nvidia-glx is version 304.38.


    $ glxinfo | grep rendering
    direct rendering: Yes

    $ glxgears
    Running synchronized to the vertical refresh.  The framerate should be
    approximately the same as the monitor refresh rate.
    300 frames in 5.0 seconds = 59.902 FPS

The most annoying problem I faced after switching to the nvidia driver is that
typed characters are displayed at a Skype chat's message box with a delay, it
looks like Skype is hung up for a few seconds and than displays all typed text
at once.
