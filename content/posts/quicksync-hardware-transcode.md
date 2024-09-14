+++
title = "setup plex/emby/jellyfin hardware transcoding via intel igpu"
date = "2024-09-09"
description = "Setup your Plex/Emby/Jellyfin so that it uses Intel iGPU to hardware transcode utilizing Intel QuickSync"
tags = [
    "debian",
    "docker",
    "emby",
    "plex",
    "jellyfin",
    "quicksync"
]
+++

So I've been having issues with enabling hardware transcode in my Emby Container on my Debian machine with i5-8500. These are the steps I had to take to enable hardware transcoding utilizing QuickSync.

First I had to add non-free to the debian sources in order to install Intel non-free drivers.

````bash
sudo nano /etc/apt/sources.list
````

I added `contrib` and `non-free` to the end of the relevant lines so my sources.list file looks like below:


````bash
#deb cdrom:[Debian GNU/Linux 12.6.0 _Bookworm_ - Official amd64 NETINST with firmware 20240629-10:18]/ bookworm contrib main non-free-firmware

deb http://deb.debian.org/debian/ bookworm main non-free-firmware contrib non-free
deb-src http://deb.debian.org/debian/ bookworm main non-free-firmware contrib non-free

deb http://security.debian.org/debian-security bookworm-security main non-free-firmware contrib non-free
deb-src http://security.debian.org/debian-security bookworm-security main non-free-firmware contrib non-free

# bookworm-updates, to get updates before a point release is made;
# see https://www.debian.org/doc/manuals/debian-reference/ch02.en.html#_updates_and_backports
deb http://deb.debian.org/debian/ bookworm-updates main non-free-firmware contrib non-free
deb-src http://deb.debian.org/debian/ bookworm-updates main non-free-firmware contrib non-free

# This system was installed using small removable media
# (e.g. netinst, live or single CD). The matching "deb cdrom"
# entries were disabled at the end of the installation process.
# For information about how to configure apt package sources,
# see the sources.list(5) manual.
````

Then we go ahead and update our package list:

````bash
sudo apt update
````

Now we install the Intel non-free drivers:

````bash
sudo apt install intel-media-va-driver-non-free
````

Now we make sure everything is in order with our Intel drivers:

```bash
ls /dev/dri
```

You should see something like this:

```bash
by-path/   card0   renderD128
```

If not try:

```bash
modprobe i915
```

And checking it out again via:

```bash
ls /dev/dri
```

Now you should see something like this:

```bash
by-path/   card0   renderD128
```

Now we need to make sure our Emby/Plex/Jellyfin docker container has access to the Intel iGPU.

First we need to set the permissions:

```bash
sudo chmod -R 777 /dev/dri
```

Now we edit our docker compose file so the `/dev/dri:/devdri` mapping is correct. My compose file looks like as below:

```bash
services:
  emby:
    image: emby/embyserver:latest
    container_name: emby
    devices:
      - /dev/dri:/dev/dri  
    environment:
      - UID=1000 # Replace with your user ID
      - GID=1000 # Replace with your group ID
    volumes:
      - ./config:/config # Replace with the path to your Emby config directory
      - /mnt/merged:/mnt/media # Replace with the path to your media files
    ports:
      - 8097:8096
      - 8920:8920 # Optional for HTTPS
    restart: unless-stopped
```

Now you should be able to setup your Emby/Plex/Jellyfin so that it can utilize your Intel iGPU to hardware transcode via QuickSync.

