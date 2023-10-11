---
title: "Raspberry Pi 4 Random Notes"
related: true
categories:
  - linux
  - raspberrypi
tags:
  - bluetooth
  - audio
  - ubuntu
  - raspbian
  - rpi
  - linux
toc: true  
---

This post has random experiments performed with Raspberrypi 

## Bluetooth Audio
* Currently Raspbian OS (bullseye) as of Oct/2023 has issues bluetooth audio profile switch from A2DP to HSP/HFP (Hands free)
* So, Ubuntu Mate 22.04.3 seems to solve the profile switch issue in Raspbian and quite stable with pulse audio server. 
* Even though Raspbian has quite some advantage, I think Raspbian (Bookworm) might solve this issue when released. 
* Make sure you connect the bluetooth headset using blueman or bluetoothctl
* After successful connection, To switch the bluetooth profile in Ubuntu Arm

```shell
# Make sure user is in appropriate groups
$ sudo groupadd pulseaudio (if not present)
$ sudo gpasswd -a bluetooth pulseaudio
# After Groupadd, need to logout/login

# List Bluetooth card
$ pacmd list-cards

# Get the Bluetooth Address and replace below FF
$ pacmd set-card-profile bluez_card.FF_FF_FF_FF_FF_FF handsfree_head_unit

# After switching profile, we can check the sources in pulseaudio,
$ pactl list short sources

# You should see source like bluez_source.FF_FF_FF_FF_FF_FF.handsfree_head_unit 
# To check bluetooth recording, we can use FFmpeg,
$ ffmpeg -f pulse -i bluez_source.FF_FF_FF_FF_FF_FF.handsfree_head_unit -ac 1 output.m4a

# After 5 secs, quit the FFmpeg and playback
$ ffplay output.m4a
```

## Bluetooth Key Press 
* To get the Bluetooth Key/Touch press, use the following,
  
```shell
$ sudo apt install libevdev-dev
$ pip install evdev
$ sudo gpasswd -a $USER audio video dialout

# After Groupadd, need to logout/login
$ python ~/.local/lib/python3.10/site-packages/evdev/evtest.py
# Select the listed Bluetooth device and check for the keycode

```

* Based on testing, irrespective of the Bluetooth profile, we get the same keycode KEY_PLAYCD(200), KEY_PAUSECD(201), KEY_NEXTSONG(163), KEY_PREVIOUSSONG (165). (Check linux/uapi.h for Keycode reference: https://github.com/torvalds/linux/blob/master/include/uapi/linux/input-event-codes.h)
