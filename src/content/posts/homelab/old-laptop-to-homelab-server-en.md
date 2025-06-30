---
title: Using an Old Laptop as a Homelab Server
published: 2025-06-29
description: 'A sleeping ThinkPad becomes a 24/7 server in another world...?'
image: ''
tags: [homelab, ubuntu-server]
category: 'homelab'
draft: false 
lang: 'en'
---

> A sleeping ThinkPad becomes a 24/7 server in another world...?

# Background

![](images/2025-06-29-14-09-46.png)

I have an old ThinkPad laptop. When I bought it, it was quite expensive, but as time passed, I ended up only using my MacBook and completely stopped using the ThinkPad.
I've always had a dream about building a homelab, and now that I have some free time, I plan to use this ThinkPad as a homelab server.
I'm not sure what services I should host, but I figure if I set it up first, I'll find some way to use it, right?

# Operating System

Currently, this ThinkPad is running Windows OS.
No matter how I think about it, Linux seems like the obvious choice for a homelab server over Windows.

The main reasons I think Linux is better are:
1. The inexplicable comfort that open-source projects provide
2. Efficient computing resource utilization
    - I want to minimize unnecessary resource usage to save on electricity bills as much as possible
3. High stability
    - Having been traumatized by Windows forced updates, I wanted to avoid situations where the server goes into forced updates
4. Developer experience
    - Personally, I'm much more familiar with Linux CLI environment than Windows CLI. Remote access and service deployment also felt much more convenient in a Linux environment

For these reasons, I looked into Linux distributions and ultimately chose **Ubuntu Server** as it has the best accessibility and the most reference materials.

:::note
**Concerns about Windows Product Activation**

My ThinkPad came with Windows pre-installed. So when I was worried about deleting Windows to install Linux, I was concerned about how to handle product activation if I wanted to go back later.
After researching, I found that most modern laptops have a high probability of having the Windows product key embedded in the motherboard.
Therefore, if I reinstall Windows later, it will automatically complete product activation, so there's no need to worry about this aspect.
:::

I decided on the latest LTS version of Ubuntu Server, `24.04.2 LTS`, and was downloading it when I realized I didn't have a USB drive.
Usually, USB drives are rolling around everywhere, but why can't I find one now of all times...
So I connected a camera microSD to an SD card reader and used [Balena Etcher](https://etcher.balena.io/) to create a USB boot drive.
The process of burning the image was very well documented [here](https://ubuntu.com/tutorials/create-a-usb-stick-on-macos).

After preparing the boot USB, I installed Ubuntu Server by referring to the [Ubuntu Server Installation Guide](https://ubuntu.com/tutorials/install-ubuntu-server#1-overview) and the [ThinkPad Official Linux Installation Guide](https://download.lenovo.com/pccbbs/mobiles_pdf/tp_p1_gen2_ubuntu_18.04_lts_installation_v1.0.pdf_). 
It was much simpler than I expected. I wish there were more documents written this thoroughly and clearly.

And then I returned to my Mac and

## Post-Ubuntu Server Installation Setup

The first problem I encountered after installing Ubuntu Server was that the console text was too small.
It was so small that I couldn't read anything at all.
The reason the text was so small was that my ThinkPad has a 4K monitor, and I suspected that the default font in the console environment was pixel-based fixed size, making it appear very small.

So I adjusted the font size using the `console-setup` tool as follows:

1. `sudo dpkg-reconfigure console-setup`
2. Keep encoding as UTF-8
3. Keep character set as "Guess optimal character set"
4. Changed to Terminus font, which I personally like
5. Changed font size to the largest `16x32`

Now it's finally at a readable size.

After that, I got the IP address from the Ubuntu server using `ip a`, and when I tried to connect via SSH from my Mac using the format below, there was an infinite loading issue...
```bash frame="none"
ssh <USER>@<SERVER IP>
```
So I tried `ping <SERVER IP>` and confirmed it was successful.
I was certain this was a server firewall issue, so I checked the firewall on the server using the command below.
```bash frame="none"
sudo ufw status
```
I confirmed there were no SSH rules, so I enabled the firewall and opened port 22 to allow SSH access.
```bash frame="none"
sudo ufw enable && sudo ufw allow ssh
```
And when I tried SSH connection from my Mac, it worked without any issues!
```bash frame="none"
ssh junsu@<SERVER IP>
The authenticity of host '<SERVER IP> (<SERVER IP>)' can't be established.
ED25519 key fingerprint is SHA256:F+2M6uBdFrj9gRdsX/vSSRngAYGcehtsyDCe4koXC2o.
This key is not known by any other names.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added '<SERVER IP>' (ED25519) to the list of known hosts.
junsu@<SERVER IP>'s password: 
Welcome to Ubuntu 24.04.2 LTS (GNU/Linux 6.8.0-62-generic x64)

 * Documentation:  https://help.ubuntu.com
 * Management:     https://landscape.canonical.com
 * Support:        https://ubuntu.com/pro

 System information as of Mon Jun 30 01:25:28 AM UTC 2025

  System load:  0.16              Temperature:                36.0 C
  Usage of /:   6.6% of 97.87GB   Processes:                  181
  Memory usage: 2%                Users logged in:            1
  Swap usage:   0%                IPv4 address for wlp0s20f3: <SERVER IP>


Expanded Security Maintenance for Applications is not enabled.

57 updates can be applied immediately.
To see these additional updates run: apt list --upgradable

Enable ESM Apps to receive additional future security updates.
See https://ubuntu.com/esm or run: sudo pro status
```
Now, since this is a laptop server, let's configure it so that it doesn't go into sleep mode when the laptop lid is closed.

Open the configuration file:
```bash frame="none"
# Open configuration file
sudo vim /etc/systemd/logind.conf
```
Change the following settings as shown below:
```bash frame="none"
HandleLidSwitch=ignore
HandleLidSwitchExternalPower=ignore
HandleLidSwitchDocked=ignore
```
Let's look at each one:
- `HandleLidSwitch=ignore`: Ignore closing the laptop when it's running on battery power
- `HandleLidSwitchExternalPower=ignore`: Ignore closing the laptop when it's connected to power
- `HandleLidSwitchDocked=ignore`: Ignore closing the laptop when it's connected to a docking station (I don't have one, but...)

Now restart the login service to apply the new settings:
```bash frame="none"
sudo systemctl restart systemd-logind.service
```

Additionally, we need to add a setting to automatically turn off the screen after a certain time if there's no keyboard input on the laptop.

Open the following file:
```bash frame="none"
sudo vim /etc/default/grub
```

Find `GRUB_CMDLINE_LINUX_DEFAULT` and add the following setting:
```bash frame="none"
GRUB_CMDLINE_LINUX_DEFAULT="consoleblank=180"
```
- Set to turn off the screen after 180 seconds (3 minutes) of no keyboard input

After saving the file, apply the GRUB settings to the system and reboot to activate the automatic screen-off mode:
```bash frame="none"
sudo update-grub
```
After that, I confirmed that the screen turns off automatically and that remote access is possible with the laptop lid closed.

# Conclusion
I've given this dusty ThinkPad a job by installing Ubuntu Server on it.
Starting from the dilemma of choosing an operating system, I overcame unexpected obstacles like small text and firewall issues, and now it has the appearance of a real server that operates 24/7 even when the lid is closed or the screen is off. Now, every time I connect via SSH from my MacBook terminal, I feel an inexplicable sense of security thinking about the ThinkPad quietly doing its job in the corner.

Of course, it's still just an empty shell. The real value of this server depends on what services I deploy and how I utilize it going forward.

The next goal is to give this server practical utility through **'Network Configuration and Service Deployment'**. Beyond being a server that only circles within the internal network, the ultimate goal is to create an environment where I can safely access my services from outside. In this process, I plan to explore the following technologies:

- Using container technology to manage multiple services easily and cleanly
- Reverse Proxy: Cleanly connecting multiple web services through domain addresses
- DDNS and SSL certificates: Maintaining a domain in a dynamic home IP environment and building a secure HTTPS communication environment

I'll keep working on it steadily. Thank you.
