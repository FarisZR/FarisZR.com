---
title: Set up scheduled Debian/Ubuntu unattended upgrades
categories:
  - guides
tags:
  - debian
  - ubuntu
  - apt
  - unattended-upgrades
date: 2024-01-31
slug: setup-scheduled-debian-ubuntu-unattended-upgrades
aliases:
  - /en/setup-scheduled-debian-ubuntu-unattended-upgrades/
image: thumbnail.jpg
draft: false
keywords:
  - debian
  - ubuntu
  - unattended upgrades
  - apt-get
---

When you run a server that relies on containers for almost everything, keeping the system up to date has almost zero chance of breaking anything.

So here is a quick guide to setting up automatic updates on a fixed schedule + automatic reboots for kernel updates (optional)

## installing unattended-upgrades
```bash
sudo apt update
sudo apt install unattended-upgrades
```

## Configuring Unattended-upgrades
Depending on what kind of updates you feel comfortable applying, you can customize the source repository as you like.

### allow security updates only

This is actually the default configuration for unattended-upgrades


/etc/apt/apt.conf.d/50unattended-upgrades
```conf
//      "origin=Debian,codename=${distro_codename}-updates";
//      "origin=Debian,codename=${distro_codename}-proposed-updates";
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
```

### allow all updates

/etc/apt/apt.conf.d/50unattended-upgrades

```conf
        "origin=Debian,codename=${distro_codename}-updates";
//      "origin=Debian,codename=${distro_codename}-proposed-updates";
        "origin=Debian,codename=${distro_codename},label=Debian";
        "origin=Debian,codename=${distro_codename},label=Debian-Security";
        "origin=Debian,codename=${distro_codename}-security,label=Debian-Security";
```

### Additional repositories

You can also automate the update for additional repositories such as Docker's official repositories.

First, go to `/var/lib/apt/lists/` and find the Docker repos filename.
In this file you should find something like
```
label: Docker CE
Origin: Docker
Suite: bookworm
```

For us, the most important details here are **origin** and **suite**, as suite is the archive that unattended-upgrades needs to specify which version to use.

To add the docker repository, create a new line under the Distro's repos in `/etc/apt/apt.conf.d/50unattended-upgrades' with the following format
`"<origin>:<archive>";`

so in our case it would be `"Docker:bookworm";`, but what if we don't want to update this every time the system is upgraded? then you use the `${distro_codename}` variable.

```conf
"Docker:${distro_codename}";
```

and now unattended upgrades should automatically update packages from the docker repos!

## Enable automatic reboot for kernel upgrades
If you can tolerate a little downtime, you can also automate system reboots for kernel upgrades

by changing the following settings
```conf
// Unattended-Upgrade::Automatic-Reboot "false
```
to 
```conf
Unattended-Upgrade::Automatic-Reboot "true";
```

And you can change the reboot time by changing the default `02:00' to any time you like (in the local server timezone).
**REMEMBER TO UNCOMMENT THE SETTING (remove the // and the spaces)**.
```conf
Unattended-Upgrade::Automatic-Reboot-Time "22:00";
```

## Setting up timers
We will set two timers, one to update apt's local repo cache and one to apply the updates.


### Apt update timer

```bash
sudo systemctl edit apt-daily.timer
```

then add the following in the space between systemd comments (assuming you want to update apt lists at 00:55 server time every day)

```conf
[Timer]
OnCalendar
OnCalendar=00:55
RandomizedDelaySec=0
```

and then you should restart the service and check the status of the timer, it should show the next trigger time

```bash
sudo systemctl restart apt-daily.timer
sudo systemctl status apt-daily.timer
```

### Apt upgrade timer

This is very similar to setting the update timer, we just need to make sure it's **after** the update job so we can apply the latest updates as they are released.

```bash
sudo systemctl edit apt-daily-upgrade.timer
```

Again, in the blank space, add the following timer settings (to apply updates at 01:00 server time every day)

```conf
[Timer]
OnCalendar
OnCalendar=01:00
RandomizedDelaySec=0
```

and then you should restart the service and check the status of the timer, it should show the next trigger time

```bash
sudo systemctl restart apt-daily-upgrade.timer
sudo systemctl status apt-daily-upgrade.timer
```

and now you have fully automated updates for your entire system!

## Sources
https://askubuntu.com/questions/87849/how-to-enable-silent-automatic-updates-for-any-repository

https://wiki.debian.org/StableProposedUpdates

https://linuxiac.com/how-to-set-up-automatic-updates-on-debian/