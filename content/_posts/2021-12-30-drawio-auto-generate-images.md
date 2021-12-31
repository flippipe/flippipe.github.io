---
title: "Drawio - Auto Generate Images"
date: 2021-12-30T21:20:30Z
draft: true
---

In last weeks, I got a new work to do, and envolves creating diagrams and DrawIO was the lucky chosen.

As the diagrams will be used in documentation and exposed in GitHub, I need updated images.

# TLDR

Combining [snaps]() with [electron apps]() do not mix well.

[Final solution](#solution), use old tools like [incrond](https://github.com/ar-/incron) to captures the changes in the diagrams to auto generate the files.

# Story

## 💡

Some time ago, skimming [git](https://git-scm.com/book/en/v2) documentation I saw references for hooks and this time, was a good time to try it out. Each time I commit a new diagram, it will auto generate a new file.

drawio-desktop already [supports](https://j2r2b.github.io/2019/08/06/drawio-cli.html) command line interface to do it, it could not to be to difficult to do a simple script to do this task... and... oh god! how I was wrong.

## Hurdle ❶ - Snap Confinement and drawio-desktop

[Snap Confinement](https://snapcraft.io/blog/demystifying-snap-confinement) is good for the security of our computers, but, it expects users have their files in the default locations, and, almost all my files are not where it expect to be.

So I try to use the trick to remount my working directory inside `/mnt` to allow snap [removable-media interface](https://snapcraft.io/docs/removable-media-interface) to use it, but no luck.

    $ sudo snap connections drawio | grep removable-media
    $

### Solution

Modify the snap to add the `removable-media` interface

    $ snap download drawio
    $ unsquashfs -dest drawio_138 drawio_138.snap

Edit `drawio_138/meta/snap.yaml` and add `removable-media` into plugs

Resulting in something like:

    apps:
    drawio:
        command: command.sh
        plugs:
            (...)
            - opengl
            - removable-media
        environment:

Then, use the modified snap with [`snap try`](https://snapcraft.io/docs/snap-try#heading--snaptry)

    $ sudo snap try drawio_138

Snap, installed

    $ snap list 
    Name                             Version                     Rev    Tracking         Publisher          Notes
    core20                           20211129                    1270   latest/stable    canonical✓         base
    drawio                           16.0.2                      x1     -                -                  try

Last task, `mount` the directory in the new location, adapted from [Home directories outside of '/home'](https://snapcraft.io/docs/home-outside-home)

    # mount --bind  /work/WIP/ /mnt/WIP/


## Hurdle ❷ - Git Hooks

I'm new to git, so, choose the right [hook](https://git-scm.com/book/en/v2/Customizing-Git-Git-Hooks) to do the task, was not easy... some voices in internet, say to use `pre-commit`, but adding files at this stage is not linear.

My chosen solution was to do a `post-commit`, in which will generate the file and then add and commit the file, something like:

    /snap/bin/drawio --export 
    git add
    git commit -m "AutoCommit - Update DrawIO Images" 

I made the script work when I manually ran `git commit` on terminal... but when the hook ran inside [VSCodium](https://vscodium.com/)... it didn't work...

## Hurdle ❸ - Electron... Snaps...

From inside an Electron App (VSCodium) try to run another Electron App (drawio-desktop), was an hell, so I quit to make it work.

Just a quick note, if you see something like this in your system logs

    kernel: traps: drawio[767435] trap int3 ip:55fac191b086 sp:7ffe0bb228d0 error:0 in drawio[55fac0bcb000+6a10000]

Consider it as a core dump.

## Hurdle ❹ - No Git Hooks

Without git hooks, I need to find a solution to detect file changes, and inotify kernel feature is a great solution, but lack a good user land tools.

[inotifywait](https://github.com/inotify-tools/inotify-tools) paired with while, as many [solutions](https://unix.stackexchange.com/a/323919) out there, was not an option...

Just found to possible paths:

* [systemd.path](https://www.freedesktop.org/software/systemd/man/systemd.path.html#)
* [incron](https://github.com/ar-/incron)


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

# Solution

## Install and configure incron

This is the easiest part

    # apt install incron

Then, add your username to `/etc/incron.allow`

## Deploy user configurations

Edit incrontab 

    $ incrontab -e

and add your configuration

    /work/WIP/Clouds/*.drawio  IN_CLOSE_WRITE  /home/johndoe/bin/incron-drawio.sh $@

Then create your script `/home/johndoe/bin/incron-drawio.sh`

## My Script

```bash
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

# drawio is an electron app, inside a snap, and inside script 
# it do not work because gpu stuff
# you will in system log messages, messages like
# kernel: traps: drawio[678537] trap int3 ip:557dba461086 sp:7ffd451ee510 error:0 in drawio[557db9711000+6a10000]
# so, use a framebuffer without GPU, therefore it do not crash
export DISPLAY=':99.0'
Xvfb :99 -screen 0 1024x768x24 > /dev/null 2>&1 &
Xvfb_PID=$!

drawio='/snap/bin/drawio --export --width 1024 --border 10'
$drawio --output "${realFile%.*}".png "$realFile" | logger 

kill $Xvfb_PID

```