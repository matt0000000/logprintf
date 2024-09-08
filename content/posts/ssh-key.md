+++
title = "disable ssh password login"
date = "2024-09-01"
description = "process kworker using high amount of cpu, over 80%"
tags = [

]
+++

It is widely recommended to disable password authentication unless you have a specific reason not to. Instead you should use SSH keys.

To disable password authentication, look for the following line in your sshd_config file at `/etc/ssh/sshd_config`

```html
#PasswordAuthentication yes
```

replace it with a line that looks like this:

```html
PasswordAuthentication no
```

Now check for `/etc/ssh/sshd_config.d/50-cloud-init.conf`

If the file exists make the same change in that file as well. You may go ahead and delete the file altogether as well but some users report that it gets recreated.

Once you have saved the file and restarted your SSH server, password authentication should be disabled.
