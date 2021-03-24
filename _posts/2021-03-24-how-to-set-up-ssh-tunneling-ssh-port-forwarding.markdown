---
layout: post
title:  "How to set up SSH tunneling (SSH port forwarding)"
---

### SSH tunnel basics

**Problem**: PC1 can’t connect to PC2 via ssh, because PC2 doesn’t allow incoming ssh connections (outgoing connections allowed).

**Solution**: make PC2 to open ssh port on PC1, then PC1 can use this port to ssh to PC2.

Assume that PC1 is behind a router. In this case you need to set up port forwarding on the router, so router’s WAN port “65003” (can be any port) forwards to PC1’s port “22”. Port forwarding setup on the router is out of scope for this guide.

A user on PC1 is needed. You can use the one that already exists, or create a new one - dedicated user, which is safer.

To set up ssh tunnel from PC2 to PC1 use:
```
$ ssh -R <port_on_PC1>:127.0.0.1:22 <username>@<router_IP> -p 65003
```

An example of how it may look:
```
$ ssh -R 9999:127.0.0.1:22 guest@176.43.95.30 -p 65003
```

If there is no router, but direct access to PC1 via ssh:
```
$ ssh -R 9999:127.0.0.1:22 guest@176.43.95.30
```

After the connection is established, ssh from PC1 to PC2:
```
$ ssh <username>@localhost -p <port_on_PC1>
```

Example:
```
$ ssh guest@localhost -p 9999
```

### SSH Key-Based Authentication

This is needed to prevent ssh requesting password every time the connection establishes.

If you don’t already have ssh key you need to generate one on a client:
```
$ ssh-keygen
```

Agree to the defaults, so it is stored into your home directory without a “passphrase”.

Copy the public key to the computer you want to ssh to without password:
```
$ ssh-copy-id -i ~/.ssh/id_rsa.pub <username>@<remote_host>
```

Now ssh key-based authentication should work.

You can read more about it [here](https://www.digitalocean.com/community/tutorials/how-to-configure-ssh-key-based-authentication-on-a-linux-server).

### Tunnel script

Create `/opt/connect_to_home.sh` script with this content:
```
#!/bin/bash
while :
do
    ssh -o ServerAliveInterval=60 -o ExitOnForwardFailure=yes -o IdentityFile=/home/kp/.ssh/id_rsa -N -R 9999:localhost:22 guest@176.43.95.30 -p 65003
    sleep 60
done
```

Adjust used path, name, address in the script above, so they are correct in your case.

Make it executable:
```
$ sudo chmod +x /opt/connect_to_home.sh
```

### Tunnel autostart

Create `ssh_tunel_to_home.service` file with this content:
```
[Unit]
Description=ssh tunnel to home

[Service]
Type=simple
ExecStart=/opt/connect_to_home.sh

[Install]
WantedBy=multi-user.target
```

Put it in `/etc/systemd/system/` directory.

Enable this systemd unit, so it autostarts:
```
$ sudo systemctl enable /etc/systemd/system/ssh_tunel_to_home.service
```

