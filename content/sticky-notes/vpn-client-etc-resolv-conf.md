---
title: "Vpn Clients and /etc/resolv.conf"
date: 2022-11-07T18:52:43Z
summary: "Some VPN clients still do not play nice with Network Manager, let's fix it"
description: "Some VPN clients still do not play nice with Network Manager, let's fix it"
tags: ["networkmanager", "pulsesecure", "resolvctl"]
---

If Ivanti Secure Access Client (former Pulse Secure Client) makes you mad by changing your `/etc/resolv.conf` you are not alone.

A script on [NetworkManager Dispatcher](https://networkmanager.dev/docs/api/latest/NetworkManager-dispatcher.html), with some [resolvectl](https://www.freedesktop.org/software/systemd/man/resolvectl.html) commands come handy.

Just create a script as `/etc/NetworkManager/dispatcher.d/99-fixDNS`, with something like:

``` bash
#!/bin/bash -e

[[ "${DEVICE_IFACE:-dummy}" = "tun0" ]] && [[ "${NM_DISPATCHER_ACTION:-dummy}" = "up" ]] && {
    resolvectl dns tun0 10.10.10.10
    resolvectl domain tun0 ~example.com ~example.org
} 
```

Much more settings can be added, but now you have an idea how to solve it.