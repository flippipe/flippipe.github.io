---
title: "Browser Redirection"
date: 2022-08-24T20:18:55+01:00
summary: "What if when I open an URL it goes to correct browser"
description: "What if when I open an URL it goes to correct browser"
tags: ["firefox", "Desktop Files"]
---

In a previous [note](/sticky-notes/desktop_files/) I've created several desktop files, one for each cloud provider I use, but...

When we use tools like `gcloud` or something like it, it open the default browser profile and not the one I've created to GCP, as example.

Coping and pasting URLs started to be boring... so, time to fix it.

First, we need to register a new application associated for schema http/https. With [Gnome](https://help.gnome.org/admin/system-admin-guide/stable/mime-types-application-user.html.en)'s help, I've got there.

So, a new application was born `~/.local/share/applications/superduper.desktop`.

``` ini
[Desktop Entry]
Version=1.0
NoDisplay=true
Name=Super Duper Browser
Comment=Browser chooser
Keywords=Internet;WWW;Browser;Web;Explorer
Exec=/home/user/bin/superduper.sh "%u"
Terminal=false
X-MultipleArgs=false
Type=Application
Icon=/home/user/Downloads/Robot-icon.png
Categories=GNOME;GTK;Network;WebBrowser;
MimeType=x-scheme-handler/http;x-scheme-handler/https;
StartupNotify=true
```

and do not forget to update `~/.config/mimeapps.list` as mentioned in Gnome Documentation. I added the new desktop file name in almost the same places where was firefox.desktop. This is a resume of the final [file](mimeapps.list)

And then, all magic happens inside the `superduper.sh` script (this is just an ideia)

``` sh
#!/bin/bash

if [[ $1 =~ client_id=99999999999 ]]; then
    firefox --P GCP --new-tab "$1"
else
    firefox --new-tab "$1"
fi
```

**note** I've to remove firefox option `--no-remote` from the other desktop files,
if present, does not allow to open new windows/tabs outside browser window.

Do not forget execution permission on `superduper.sh` script.

If all works fine, It should show up in `Default Application`

![menu](settings.png)