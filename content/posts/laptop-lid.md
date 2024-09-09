+++
title = "do nothing when laptop lid is closed"
date = "2024-09-05"
description = "how to set up debian 12 so that nothing happens when the laptop lid is closed"
tags = [
    "debian"
]
+++

So if you're like me and use one of your old laptops in your cluster as a headless server (mine is on Adguard duty) this could help you. You'd most likely want to keep your laptop lid/screen closed. If you do so and you want your Debian install to do nothing when the lid is closed all you have to do is:


edit `/etc/systemd/logind` 
```html
sudo nano /etc/systemd/logind
```

uncomment and set the following options: 

```html
HandleLidSwitch=ignore
HandleLidSwitchExternalPowe r=ignore
HandleLidSwitchDocked=ignore
LidSwitchIgnoreInhibited=no
```

reboot and you are set to go.

```html
sudo reboot
```
