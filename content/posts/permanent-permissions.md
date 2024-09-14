+++
title="persistent permissions on /dev/dri"
date = "2024-09-10"
description = "make permission changes on /dev/dri permanent so docker containers can access it after reboot"
tags = [
    "debian"
]
+++

So in my latest [post](https://logprintf.com/setup-plex/emby/jellyfin-hardware-transcoding-via-intel-igpu/) regarding setting up your media server app (Pex/Emby/Jellyfin) so that it uses your iGPU utilizing QuickSync I mentioned that you needed to change permissions on `/dev/dri` so that your docker container can access it. In order to do that you do:

````bash
sudo chmod -R 777 /dev/dri
````

So, that works. But the problem with that is, when you restart your server the permissions are reset and you need to repeat the same command again.

What you can do to make the permission changes permanent is that, you can create a udev rule. That will ensure that the correct permissions are applied to the `/dev/dri` device at boot time. So here's how:

First we create a udev rule file:

````bash
sudo vim /etc/udev/rules.d/99-dri.rules
````

Then we add the following content to the file:

````bash
SUBSYSTEM=="drm", MODE="0777"
````

This rule sets the permissions of any device under the `drm` subsystem (like /dev/dri) to `0777`.

After saving the file, reload the udev rules:

````bash
sudo udevadm control --reload-rules
````

Reboot your system and your docker containers can access /dev/dri persistently after each reboot:

````bash
sudo reboot
````
