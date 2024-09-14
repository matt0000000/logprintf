+++
title = "how to set up static IP address on freebsd machine"
date = "2024-09-14"
description = "Learn how to configure a static IP address on your network to ensure consistent connectivity for your devices."
tags = [
    "freebsd"
]
+++

Setting up a static IP address can help ensure that a device keeps the same IP address over time, which is especially useful for servers or other network services. 

Below are the steps to configure a static IP address on a FreeBSD machine using the `/etc/rc.conf` file.


The network configuration in FreeBSD is controlled through the `/etc/rc.conf` file. Begin by opening this file with your preferred text editor. In this example, I will use `vim`:

```bash
vim /etc/rc.conf
```

Locate the line in the file that configures the Ethernet interface (usually `em0` or something similar). It will look something like this:

```bash
ifconfig_em0="DHCP"
```

This line tells FreeBSD to use DHCP for assigning an IP address. To assign a static IP, replace the line with the following:

```bash
ifconfig_em0="inet 192.168.1.61 netmask 255.255.255.0"
```

In this example:
- `192.168.1.61` is the static IP address you want to assign to your machine.
- `255.255.255.0` is the subnet mask.

Make sure to adjust the IP address and subnet mask according to your network configuration.

Next, we need to specify the default gateway (the router's IP address) that your machine will use to access devices outside its local network. Add or update the following line in `/etc/rc.conf`:

```bash
defaultrouter="192.168.1.1"
```

Replace `192.168.1.1` with the actual IP address of your network’s default gateway. Save and exit.

To apply the changes, you’ll need to restart the networking service on FreeBSD. This can be done by either rebooting the system or restarting the networking service with the following command:

```bash
service netif restart
```

You may also want to restart the routing service to ensure the default gateway is applied:

```bash
service routing restart
```

Once the system is back up and running, you can verify the static IP configuration using the `ifconfig` command:

```bash
ifconfig em0
```

Check that the `em0` interface is now using the static IP address (`192.168.1.1` in this example) and has the correct subnet mask.


