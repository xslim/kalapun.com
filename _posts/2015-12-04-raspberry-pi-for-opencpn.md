---
published: false
title: Raspberry Pi for OpenCPN
---


## Installing the image

## First steps
Run `sudo raspi-config` and expand filesystem

## VNC

``` sh
sudo apt-get update
sudo apt-get install tightvncserver
vncserver :1
mkdir -p /home/pi/.config/autostart
vi /home/pi/.config/autostart/tightvnc.desktop
```

``` ini
[Desktop Entry]
Type=Application
Name=TightVNC
Exec=vncserver :1
StartupNotify=false
```

## Arduino

```
sudo apt-get install arduino arduino-mk
sudo arduino
```

## Small screen

```
sudo apt-get install matchbox-keyboard
```



