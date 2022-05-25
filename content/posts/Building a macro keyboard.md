---
title: "Building a Macro Keyboard"
date: 2020-05-23T22:15:43Z
lastmod: 2022-01-23T22:15:43Z
summary: "Building a macros keyboard from a conventional numeric keyboard"
description: "Building a macros keyboard from a conventional numeric keyboard"
tags: ["keyboard"]
draft: false
cover.image: "https://github.com/flippipe/macro-keyboard/blob/main/final%20product.png?raw=true"
cover.relative: false 
showtoc: true
---

# Introduction

In the old times, when we had a physical phone, it was easy to answer a call, just pick the phone.

Since the migration to software which substitue the phone, I want to recreate the workflow to answer a call: no need to find some button in some window just to answer a call.

[Using a numeric keypad](https://medium.com/@kenneth.skertchly/using-a-numeric-keypad-as-a-hotkey-pad-in-linux-6a487d624d75) it is possible to replicate the workflow... bind a key to answer the call, and more buttons, more actions we can automate.


{{< figure align="center" src="https://github.com/flippipe/macro-keyboard/blob/main/final%20product.png?raw=true" height="250" >}}

## Keyboard

First we need a dedicated keyboard for this function, and any keyboard should do.

Laying arround I just have full size keyboard, so I bought a new `MITSAI Q20` from a local shop.

I like it, because the keys are squared and low profile. Easy to customize with stickers.

## pluging keyboard the first time

In `systemd journal`, you will find information about the keyboard. As you can see, the brand name from my local shop do not show up anywhere. 

```
kernel: usb 1-1.4.3: new low-speed USB device number 15 using xhci_hcd
kernel: usb 1-1.4.3: New USB device found, idVendor=1a2c, idProduct=0e24, bcdDevice= 1.10
kernel: usb 1-1.4.3: New USB device strings: Mfr=1, Product=2, SerialNumber=0
kernel: usb 1-1.4.3: Product: USB Keyboard
kernel: usb 1-1.4.3: Manufacturer: SEM
kernel: input: SEM USB Keyboard as /devices/pci0000:00/0000:00:14.0/usb1/1-1/1-1.4/1-1.4.3/1-1.4.3:1.0/0003:1A2C:0E24.000B/input/input29
kernel: hid-generic 0003:1A2C:0E24.000B: input,hidraw5: USB HID v1.10 Keyboard [SEM USB Keyboard] on usb-0000:00:14.0-1.4.3/input0
```

# System configuration

## udev

[udev](https://www.freedesktop.org/software/systemd/man/udev.html) will allow us to do some setup and start actkbd service.

My `udev rule` file is:

```
ACTION=="add", \ # match the event, device and parent device
SUBSYSTEM=="input", \ 
ATTRS{name}=="SEM USB Keyboard", \
MODE="0666", \ # set mode to other read from the device, without root
SYMLINK+="macrokeyboard", \ # create a nicer symbolic link
TAG+="systemd", \ # this will used systemd to start actkbd
ENV{SYSTEMD_USER_WANTS}="actkbd.service"
```

To reload the configuration, we can use

``` terminal
udevadm control --reload-rules 
```

To check if the `udev rule` file is working

``` terminal
udevadm test /devices/pci0000:00/0000:00:14.0/usb1/1-1/1-1.4/1-1.4.3/1-1.4.3:1.0/0003:1A2C:0E24.000B/input/input29
```

To create an `udev rule` can be hit and miss. Doing the wrong filters, can lead to diferent devices be found, but is is possible to get all information

``` terminal
udevadm info --attribute-walk --path /devices/pci0000:00/0000:00:14.0/usb1/1-1/1-1.4/1-1.4.3/1-1.4.3:1.0/0003:1A2C:0E24.000B/input/input29
```

In case of problems, read [Writing udev rules](reactivated.net/writing_udev_rules.html) to learn how to write it.


## systemd

As configured in `udev rule`, this unit will start the service.

```
[Unit]
Description=Actkbd: Daemon for X-independent shortcuts
ConditionPathIsSymbolicLink=/dev/macrokeyboard

[Service]
Environment=DISPLAY=:0
ExecStart=actkbd --device /dev/macrokeyboard --daemon --pidfile /tmp/actkbd.pid
Type=forking
PIDFile=/tmp/actkbd.pid
Restart=on-failure
RestartSec=1
```

## X

Without instrcut XOrg to ignore this numeric keypad, when we press a key, it will send that key code the application.

So we need to place a file in `/usr/share/X11/xorg.conf.d/`, with:

```
Section "InputClass"
  Identifier "Ignore macro keyboard"
  MatchProduct "SEM USB Keyboard"
  Option "Ignore" "on"
EndSection
```

To test this, it is needed to restart X server...


## actkbd

[Warning](https://github.com/thkala/actkbd/issues/2#issuecomment-264657080), by project author:

> ...but here's the catch: actkbd (a) is single-threaded and (b) uses
> system() in the same thread to execute commands. Yes, I know, not
> nice; the whole program is basically a hack - it just worked well
> enough for me and a few friends back when I wrote it.

It can be an hack, but [actkbd](https://github.com/thkala/actkbd) have work without problems for me.

After installation we need to find the keycodes from our keyboard.

### KeyCodes

I've found two ways to do it...

using `actkbd`

```
$ actkbd --showexec --showkey --noexec --device /dev/macrokeyboard
Keys: 79
Keys: 80
Keys: 81
Keys: 96
```

or using `showkeys` 

``` 
$ sudo showkey 
press any key (program terminates 10s after last keypress)...
keycode  28 release
keycode  79 press
keycode  79 release
keycode  80 press
keycode  80 release
```

### configuration

The default location for the configuration file is `/etc/actkbd.conf`

I opted by send all keycodes to a single script...

``` 
    #1st line
    172:::/usr/bin/keyboardScript.sh 172 &
     15:::/usr/bin/keyboardScript.sh 15 &
    155:::/usr/bin/keyboardScript.sh 155 &
    140:::/usr/bin/keyboardScript.sh 140 &
```

Each line can execute any command, but I decided to have a script which will deal with all of it... it is easier to reuse code and maintain it. Also, I can change the script without to bother to restart actkbd to reload new actions.

### testing 

``` terminal
actkbd --showexec --showkey --device /dev/macrokeyboard
```

It should show somethig like

```
Keys: 76
Executing: /home/fcarvalho/bin/keyboardScript.sh 76 &
Keys: 69
Executing: /home/fcarvalho/bin/keyboardScript.sh 69 &
```


# The Script

My [script](https://github.com/flippipe/macro-keyboard/blob/main/keyboardScript.sh), is a giant `case`, something like:

``` bash
case $1 in
    79) 
        # xdotool key U1f600
        xdotool type 😀
        # slow https://github.com/jordansissel/xdotool/issues/10
    ;;
    71)
    mpc status | grep playing 2>&1 > /dev/null
    if [ $? -eq 0 ]; then
        # it is playing
        mpc --quiet stop
        dbusSend Pause
    else 
        mpc --quiet play
        dbusSend Play
    fi
    ;;
    76)
        # take a area screenshot
        flameshot gui &
    ;;
    (...)
```

and several tools were used

## xdotool

Without [xdotool](https://manpages.ubuntu.com/manpages/focal/man1/xdotool.1.html) it cloud not do the automations.

xdotool lets us programmatically simulate keyboard input and mouse.

As example, pressing a key to toggle video in `MS Teams`.

``` bash
82) # mute unmute video
    WID=$(xdotool search --onlyvisible --name "Microsoft Teams") ①
    if [ $WID ]; then
        xdotool windowfocus --sync $WID ②
        xdotool key --window $WID "ctrl+shift+o" ③
    fi
;;
```

where:

- ① Search for Teams window
- ② Put the window on focus
- ③ Send the Teams shortcut to Toggle video

## mpc

My music is streamed from other machine, where I've [mpd](https://www.musicpd.org/) running, so I use [mpc](https://www.musicpd.org/clients/mpc/) to control it remotelly.

## dbus
[dbus-send](https://manpages.ubuntu.com/manpages/focal/en/man1/dbus-send.1.html) + [MPRIS](https://specifications.freedesktop.org/mpris-spec/2.2/Player_Interface.html) allow to abstract which music player I'm using, as long as they support it.

*note* use [d-feet](https://wiki.gnome.org/Apps/DFeet) to explore dbus


# Final Result

Besides the software, the final touch was to print in photo papper some nice icons. Then with double face tape and a scissor, convert them to little stickers.

All the files are in [Building a macro keyboard](https://github.com/flippipe/macro-keyboard).

# Future

Most of this work was done with xdotool, on top X... which is being substituted by Wayland.

So, it is time to look into xdotool replacement options:

* https://github.com/ReimuNotMoe/ydotool
* https://github.com/atx/wtype

#### other resources

[Font Awesome](https://fontawesome.com/) or [Nerd Fonts](https://www.nerdfonts.com/)

[Scripting with udev](http://jasonwryan.com/blog/2014/01/20/udev/)

[Service unit configuration](https://www.freedesktop.org/software/systemd/man/systemd.service.html)