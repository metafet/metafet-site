---
title: Enhancing experience on nvidia dGPU laptops.
published: 2025-09-09
description: 'Guide to reducing stutters and slugginess on nvidia hybrid graphics laptops when docked.'
image: ''
tags: [Guide]
category: 'Guide'
draft: false
lang: ''
---

This will reduce battery life as it utilizes reverse-prime

- You have nvidia dGpu + iGpu
- You dont have a mux switch
- You use external monitors

Hdmi port on your laptop is most likely to be wired directly to dGpu.

## Prerequsites
- Open kernel modules doesnt support disabling gsp firmware. Use proprietary one.
- KDE Plasma plays better with nvidia drivers, kde plasma will be used for this guide.



## Change default renderer
Since hdmi port is wired to dGpu, it has to copy frames from iGpu. This causes bottleneck and results in stuttering. To solve that we will set nvidia as primary gpu.
Get hardware information:
```bash frame=none
$ ls -l /dev/dri/by-path/
total 0
lrwxrwxrwx 1 root root  8 Eyl  8 17:34 pci-0000:01:00.0-card -> ../card1
lrwxrwxrwx 1 root root 13 Eyl  8 17:34 pci-0000:01:00.0-render -> ../renderD128
lrwxrwxrwx 1 root root  8 Eyl  8 17:34 pci-0000:05:00.0-card -> ../card2
lrwxrwxrwx 1 root root 13 Eyl  8 17:34 pci-0000:05:00.0-render -> ../renderD129
```

Add this line to /etc/environment:
```bash frame=none
KWIN_DRM_DEVICES="/dev/dri/by-path/pci-0000\:01\:00.0-card:/dev/dri/by-path/pci-0000\:00\:05.0-card"
```

## Block nvidia from entering low power state

First we have to get suported clock speeds:
```bash frame=none
#It'll spurt out bunch of numbers, find the highest graphics value.
nvidia-smi -q -d SUPPORTED_CLOCKS
```

Then we can set min and max clock speeds.
```bash frame=none
sudo nvidia-smi -lgc 1000,2100
```

Enable persistance mode to keep clock rates.
```bash frame=none
sudo nvidia-smi -pm 1
```

## Disable gsp firmware
This isnt necessary since nvidia fixed the bug with gsp firmware.
Add the following line to kernel parameters:
```bash frame=none
nvidia.NVreg_EnableGpuFirmware=0
```

:::warning
GSP Firmware **cannot** be disabled on 40 series and newer cards.
:::

## Confirmation
Reboot and run these lines:

```bash frame=none
# If output shows version, gsp firmware isnt disabled
$ nvidia-smi -q | grep GSP
    GSP Firmware Version                  : N/A

#If you see kde processes nvidia is primary
$ nvidia-smi
```
