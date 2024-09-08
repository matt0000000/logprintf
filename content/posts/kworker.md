+++
title = "kworker high cpu usage fix"
date = "2024-09-01"
description = "process kworker using high amount of cpu, over 80%"
tags = [
]
+++

On my Debian 12 machine, I've been having a issue that, kworker take up 80%-100% of a CPU non-stop. After hours of searching through the web for a solution, what fixed it was on a post on askubuntu from 2018. So it seems like it's not a new thing. I'll link the relevant discussion below as well for reference.[^1]

[^1]: https://askubuntu.com/questions/1044872/ubuntu-16-04-kworker-using-high-cpu-constantly

So what you do is disabling dynamic USB power management in kernel.

In order to do that you have to add `usbcore.autosuspend=-1` to your kernel's boot command line:

```html
GRUB_CMDLINE_LINUX_DEFAULT="<existing stuff> usbcore.autosuspend=-1"</existing>
```

To do that permanently, open the grub configuration file

```html
sudo nano /etc/default/grub
```

and add the flag below:

```html
usbcore.autosuspend=-1
```

Next you have to update your grub via

```html
sudo update-grub
```

After reboot, it should be fixed.
