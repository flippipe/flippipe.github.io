---
title: "Past without CTRL+V"
date: 2022-06-02T11:17:09+01:00
summary: "Past to windows without CTRL+V"
description: "Past to windows without CTRL+V"
tags: ["X", "notes"]
draft: false
---

Today I need to type a long string into a VNC window, without the almost ubiquitous functionality of copy and past to window.

This just works under **XServer** and not **Wayland** because the use of `xdotool`.

``` bash
STRING="yum-config-manager --add-repo=https://rpms.famillecollet.com/enterprise/remi-release-7.rpm"
WID=$(xdotool search --name "centos7.0 on QEMU/KVM User session" )

xdotool windowactivate --sync "${WID}"  
xdotool windowfocus --sync "${WID}"  
sleep 0.5
xdotool getwindowfocus windowfocus --sync type --window "${WID}" "${STRING}"
sleep 0.5
xdotool key --window "${1}" KP_Enter 
``` 


Maybe, on day I will change this to [ydotool](https://github.com/ReimuNotMoe/ydotool) to work also under Wayland.