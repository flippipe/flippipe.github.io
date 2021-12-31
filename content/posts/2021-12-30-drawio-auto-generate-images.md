---
title: "draw.io - Auto Generate Images"
date: 2021-12-30T21:20:30Z
draft: false
summary: "I got a new assignment to do and envolves creating diagrams with draw.io.
As the diagrams will be used in documentation and exposed in GitHub, I need updated images automatically."
description: "I got a new assignment to do and envolves creating diagrams with draw.io.
As the diagrams will be used in documentation and exposed in GitHub, I need updated images automatically."
tags: ["git", "draw.io"]
---

# TLDR

Application in [snaps](https://en.wikipedia.org/wiki/Snap_(package_manager)) running script to call other [electron](https://www.electronjs.org/docs/latest) snapped application do not mix well.

[Final solution](#solution-1), was use old tools like [incrond](https://github.com/ar-/incron) to captures the changes in the diagrams to auto generate the files.

# Story

## 💡

My first thought was to create a GitHub Action to generate these files, but [actions-drawio](https://github.com/marketplace/actions/actions-drawio) is based in code it is [archived](https://github.com/languitar/drawio-batch), because drawio-desktop already [supports](https://j2r2b.github.io/2019/08/06/drawio-cli.html) command line interface to export to image formats.

Some time ago, skimming [git](https://git-scm.com/book/en/v2) documentation I saw references for hooks and this time, was a good time to try it out. Each time I commit a new diagram, it will auto generate a new image.

It could not to be to difficult to do a simple script to do this task... and... oh god! how wrong I was.

## Hurdle ① - Snap Confinement and drawio-desktop

[Snap Confinement](https://snapcraft.io/blog/demystifying-snap-confinement) is good for the security of our computers, but, it expects users have their files in the default locations, and, almost all my files are not where it expect to be.

So I try to use the trick to remount my working directory inside `/mnt` to allow snap [removable-media interface](https://snapcraft.io/docs/removable-media-interface) to use it, but no luck.

    $ sudo snap connections drawio | grep removable-media
    $

### Solution

Modify the snap to add the `removable-media` interface

``` shell
$ snap download drawio
$ unsquashfs -dest drawio_138 drawio_138.snap
```

Edit `drawio_138/meta/snap.yaml` and add `removable-media` into plugs

Resulting in something like:

``` yaml
apps:
drawio:
    command: command.sh
    plugs:
        (...)
        - opengl
        - removable-media
    environment:
```
Then, use the modified snap with [`snap try`](https://snapcraft.io/docs/snap-try#heading--snaptry)


``` shell
$ sudo snap try drawio_138
```

Snap, installed

``` shell
$ snap list 
Name                             Version                     Rev    Tracking         Publisher          Notes
core20                           20211129                    1270   latest/stable    canonical✓         base
drawio                           16.0.2                      x1     -                -                  try
```

Last task, `mount` the directory in the new location, adapted from [Home directories outside of '/home'](https://snapcraft.io/docs/home-outside-home)

    # mount --bind  /work/WIP/ /mnt/WIP/


## Hurdle ② - Git Hooks

I'm new to git, so, choose the right [hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) to do the task, was not easy... some voices in internet, say to use `pre-commit`, but adding files at this stage is not linear.

My chosen solution was to do a `post-commit`, in which will generate the file and then add and commit the file, something like:

```shell
/snap/bin/drawio --export 
git add
git commit -m "AutoCommit - Update DrawIO Images" 
```

I made the script work when I manually ran `git commit` on terminal... but when the hook ran inside [VSCodium](https://vscodium.com/)... it didn't work...

## Hurdle ③ - Electron... Snaps...

From inside an Electron App (VSCodium) try to run another Electron App (drawio-desktop), was an hell, so I quit to make it work.

Just a quick note, if you see something like this in your system logs

``` log
kernel: traps: drawio[767435] trap int3 ip:55fac191b086 sp:7ffe0bb228d0 error:0 in drawio[55fac0bcb000+6a10000]
```

Consider it as a core dump.

## Hurdle ④ - No Git Hooks

Without git hooks, I need to find a solution to detect file changes, and inotify kernel feature is a great solution, but lack a good user land tools.

[inotifywait](https://github.com/inotify-tools/inotify-tools) paired with while, as many [solutions](https://unix.stackexchange.com/a/323919) out there, was not an option...

Just found to possible paths:

* [incron](https://github.com/ar-/incron)
* [systemd.path](https://www.freedesktop.org/software/systemd/man/systemd.path.html#)


### incron

* 👍 Already use it on other project and it simple to use

* 👎🏼 mask special symbol do not work: [dotdirs](https://github.com/ar-/incron/issues/57), [loopable](https://github.com/ar-/incron/issues/77) and [recursive](https://github.com/ar-/incron/issues/78),
* 👎🏼 Security [problems](https://github.com/ar-/incron/issues/35)
* 👎🏼 Project [stalled](https://github.com/ar-/incron/issues/68)

### systemd.path

- 👍 I would to use `StartLimitIntervalSec` and `StartLimitBurst`, because drawio-desktop, after each change, save the file
- 👎🏼 [How to tell which PathChanged?](https://unix.stackexchange.com/questions/600642/systemd-path-how-to-tell-which-pathchanged)

Do not have an option to have the file name of the modified file, kill the possibility to use `systemd.path`, I do not know how many files and directories I will have in the end, and manage all manually will kill the agility.

So in the end, I just have the option of `incron` 😒

## Hurdle ⑤ - Electron is not meant to run on headless

Electron apps are made to be used in a graphical environment, not to be used in a script. 

But fortunately, someone use CICD process to test their electron app, and already add that [problem](https://webuild.envato.com/blog/running-headless-javascript-testing-with-electron-on-any-ci-server/)

Meanwhile, while the [feature](https://github.com/electron/electron/issues/228) request in electron is not done, we have an option to create a [Xvfb](https://www.x.org/releases/X11R7.6/doc/man/man1/Xvfb.1.xhtml#heading6) 

# Solution

## Install and configure incron

This is the easiest part

``` shell
$ sudo apt install incron
```

Then, add your username to `/etc/incron.allow`

## Deploy user configurations

Edit incrontab 

``` shell
$ incrontab -e
```

and add your configuration

``` txt
/work/WIP/Clouds/*.drawio  IN_CLOSE_WRITE  /home/johndoe/bin/incron-drawio.sh $@
```

Then create your script `/home/johndoe/bin/incron-drawio.sh`

## My Script

``` bash
#!/bin/bash
set -Eeuo pipefail

# because drawio runs as a snap, it can not access the real location of the file
# to be changed as needed
realFile=$( readlink -f "$1" | sed "s#/work#/mnt#" )

lastChange=$(stat --format="%Y" "${realFile%.*}".png)
now=$(date +"%s")
delta=$(( now - lastChange ))

# drawio saves file each time it move an item... ignore changes below 30 seconds
[ $delta -lt 30 ] && exit

echo "incron generating image from $1" | logger 

# drawio is an electron app, and expect to have a display server
# so, use a framebuffer without GPU
export DISPLAY=':99.0'
Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
Xvfb_PID=$!

drawio='/snap/bin/drawio --export --width 1024 --border 10'
$drawio --output "${realFile%.*}".png "$realFile" | logger 

kill $Xvfb_PID

```


# 🍬

Some other resources how help me in this journey

* [Git Hooks](https://githooks.com/)
* [incron](https://wiki.archlinux.org/title/Incron)
* [incrontab - tables for driving inotify cron (incron)](https://man.archlinux.org/man/incrontab.5)