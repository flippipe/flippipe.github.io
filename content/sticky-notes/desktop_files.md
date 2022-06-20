---
title: "Desktop Files"
date: 2022-06-20T19:05:43+01:00
summary: "Desktop files to launch independent firefox instances with it's own profile"
description: "Desktop files to launch independent firefox instances with it's own profile"
tags: ["Linux", "Desktop Files", "CSP"]
draft: false
---

When working with cloud providers, security can not be a second thought.

To avoid contaminations between casual browsing and working in CSP consoles, I've
created for each CSP a profile in Firefox.

To be more handy to launch it, I've created three Desktop files, on for each, inside `~/.local/share/applications/`

![menu](/data/desktop%20files.png)

You can also search by the name of CSP.

#### gcp.desktop

``` ini
[Desktop Entry]
Type=Application
Name=GCP
Exec=firefox --no-remote --P GCP https://console.cloud.google.com --class GCP
Terminal=false
Encoding=UTF-8
Comment=Open GCP
X-MultipleArgs=false
Icon=/home/user/.local/share/icons/hicolor/128x128/apps/gcp.png
Categories=Application
StartupNotify=true
```


#### aws.desktop

``` ini
[Desktop Entry]
Type=Application
Name=AWS
Exec=firefox --no-remote --P AWS https://company.awsapps.com/start#/ --class AWS
Terminal=false
Encoding=UTF-8
Comment=Open AWS
X-MultipleArgs=false
Icon=/home/user/.local/share/icons/hicolor/128x128/apps/aws.png
Categories=Application
StartupNotify=true
```

#### azure.desktop

``` ini
[Desktop Entry]
Type=Application
Name=Azure
Exec=firefox --no-remote --P AZURE https://portal.azure.com/ --class Azure
Terminal=false
Encoding=UTF-8
Comment=Open Azure
X-MultipleArgs=false
Icon=/home/user/.local/share/icons/hicolor/128x128/apps/azure.png
Categories=Application
StartupNotify=true
```

#### icons

[Here](/data/icons.zip) is a zip with the three icons, to be placed into `~/.local/share/icons/hicolor/128x128/apps/`


#### Firefox Profiles

To create Firefox profiles, launch Firefix with

```
firefox --no-remote --ProfileManager
```