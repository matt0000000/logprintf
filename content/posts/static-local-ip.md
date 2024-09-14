+++
title = "how to set up static IP address"
date = "2024-09-14"
description = "Learn how to configure a static IP address on your network to ensure consistent connectivity for your devices."
tags = [
    "debian"
]
+++


Setting up a static IP address can help ensure that a device keeps the same IP address over time, which is especially useful for servers or other network services. 

Below are the steps to configure a static IP address on a Linux machine using the `/etc/network/interfaces` file.

First, open the terminal application on your local machine or log into your remote server using SSH. If you are working on a remote machine, use the following command to establish an SSH connection:

```bash
ssh user@remote-server-ip
```

Replace user with your actual username and remote-server-ip with the IP address of your remote machine.

Before making any changes, it’s a good practice to back up your existing network configuration file. Run the following command to create a backup of the /etc/network/interfaces file:

```bash
sudo cp /etc/network/interfaces /root/interfaces.bak
```

This backup allows you to restore the configuration in case something goes wrong.

Now, open the `/etc/network/interfaces` file in your preferred text editor to make changes. I use vim:

```bash
sudo vim /etc/network/interfaces
```

Find the section of the file that corresponds to your Ethernet interface. In this example, we’ll configure the `enp2s0` interface with the following details:

```bash

IP Address: 192.168.1.158
Subnet Mask: 255.255.255.0
Default Gateway: 192.168.1.1
DNS Nameservers: 192.168.1.1, 8.8.8.8, 1.1.1.1
```

Add or modify the lines in the file as follows:

```bash
auto enp2s0
iface enp2s0 inet static
    address 192.168.1.158
    netmask 255.255.255.0
    gateway 192.168.1.1
    dns-nameservers 192.168.1.1 8.8.8.8 1.1.1.1
```

Make sure to replace `enp2s0` with the correct name of your Ethernet interface, which you can find using the `ip a` command.

To apply the changes, restart the networking service with the following command:

```bash
sudo systemctl restart networking.service
```

Make sure service restarted without any errors with the following command:

```bash
sudo systemctl status networking.service
```

You can verify that the static IP has been successfully configured by running:

```bash
ip a
```

Check that the IP address for the `enp2s0` interface matches the static IP you set (`192.168.1.158` in this example).