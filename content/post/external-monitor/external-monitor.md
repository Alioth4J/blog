---
title: Getting Stuck When Connecting an External Monitor
description: 
date: 2025-09-29
slug: external-monitor
image: 
categories:
    - Linux
---

## Environment
OS: Fedora Silverblue 42  
GPU Driver: nouveau  
External Monitor Connecting Method: HDMI  

## Reproducing the Bug
When a Linux with GUI is running, connect an external monitor using HDMI, my computer will be stuck.  

## Debug
journalctl:  
```bash
Sep 26 00:30:08 aliothfedora gnome-shell[2530]: libinput error: event11 - <machine-info> Touchpad: kernel bug: Touch jump detected and discarded.
Sep 26 00:30:08 aliothfedora gnome-shell[2530]: See https://wayland.freedesktop.org/libinput/doc/1.29.1/touchpad-jumping-cursors.html for details
Sep 26 00:30:32 aliothfedora systemd[1]: rpm-ostreed.service: Deactivated successfully.
Sep 26 00:30:32 aliothfedora audit[1]: SERVICE_STOP pid=1 uid=0 auid=<auid> ses=<ses> subj=system_u:system_r:init_t:s0 msg='unit=rpm-ostreed comm="systemd" exe="/usr/lib/systemd/systemd">
Sep 26 00:31:23 aliothfedora systemd-logind[1380]: Power key pressed short.
Sep 26 00:31:56 aliothfedora systemd-logind[1380]: Power key pressed short.
```

Then goto the [url](https://wayland.freedesktop.org/libinput/doc/latest/touchpad-jumping-cursors.html) it hints. It shows "Touchpad jumping cursor bugs". However, it is just a warning.  

There may be something more with Firmware or Kernel Driver.  

// TODO  

## Temp Solution
Connect the external monitor before booting.  

