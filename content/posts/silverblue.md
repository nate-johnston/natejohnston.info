---
title: "Fedora Silverblue part 1"
date: 2022-11-09T02:27:01+01:00
draft: false
categories: ["How I Do Things"]
---

### Introduction

This is likely to be the first in a series of notes on my experience with Fedora
Silverblue.  There are a number of things that I have had to piece together from
multiple sources, so I thought it would be helpful to write them down in one
place.

### Get Started

So you have just installed Fedora Silverblue!  I trust that you read and grokked
the general idea that Silverblue is [an immutable operating
system](https://blog.christophersmart.com/2020/04/11/fedora-silverblue-is-an-amazing-immutable-desktop/),
right?  While it does provide a number of advantages, it means you may want to
decide at the outset what things you want to install on the base operating
system, as opposed to running in toolboxes.

Things that I have found are best installed on the base OS:

- Applications that need to survey system-wide resources
    - gkrellm
    - bpytop
- VPN control
    - wireguard-tools
- Gnome plugins
    - gnome-shell-extension-caffeine
    - gnome-shell-extension-dash-to-dock
- A few basic debugging tools
    - tcpdump
    - sysstat
    - setroubleshoot-server
- Conveniences
    - zsh
    - vim
    - tmux
    - pv

I would install all of these (or whatever fits the bill for you in each of these
categories) before proceeding, because with rpm-ostree you have to reboot to
complete the install.

### Other Important Additions

Definitely add flathub to your flatpak setup - there are so many important apps
in there that are key to usability as a desktop.  Follow the [basic
instructions](https://flatpak.org/setup/Fedora).

```
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```

### Toolbox

[Toolbox](https://docs.fedoraproject.org/en-US/fedora-silverblue/toolbox/) is an
incredible utility to give you the flexibility of standard Fedora combined with
the power of containers.  Getting started with toolbox requires just two
commands: `toolbox create` creates a toolbox, and then `toolbox enter` enters
it.  Both of those take an optional argument specifying the name of the toolbox,
without it the toolbox created is just named "toolbox".  Each toolbox is a
podman container running based on a fedora image by default.

I highly suggest that you use named toolboxes for each application you need.
First, if you have to install a bunch of dependencies for an app you know you
won't conflict with other apps.  If you screw something up, you haven't messed
up the environment you depend on for a bunch of other stuff.  And each app is
completely sandboxed in it's container.


```
toolbox create audio
toolbox run -c audio sudo dnf install -y pavucontrol
toolbox run -c audio pavucontrol
```

If you alias pavucontrol to the last command above then from the CLI you have
pavucontrol install and you don't have to know that it's in a toolbox.  You can
also [add a desktop
file](https://developer-old.gnome.org/integration-guide/stable/desktop-files.html.en)
to cause the app to show up in your gnome applications menu.

### Toolbox and Cron

Sometimes you may want to use toolbox to run something that needs to run
periodically, not started by our input.  One example of this is mbsync, a
program to synchronize email from an IMAP server to a local filesystem so you
can run a kickass text-mode MUA like mutt on it.  

In order to do that you need to use the standard [systemd pattern of a timer
running a
oneshot](https://www.linux.com/topic/desktop/setting-timer-systemd-linux/).
Since this part takes place outside a toolbox container, we can use it to start
and control toolbox containers.  First, create a service file that runs the
command you need as a oneshot, named with the .service extension in the
`~/.config/systemd/user` directory.  Second, create a timer file in the same
directory with the .timer extension.  I have included examples of an mbsync
version of the files below.  Once you have done this, you need to enable and
start the timer like so:

```
systemctl --user enable mbsync.timer
systemctl --user start mbsync.timer
```

Note that you are using systemd here strictly to run things under your account,
in the mutable space of your user environment.  The systemd files are in your
~/.config directory and systemd is fully capable of managing daemons in your
userspace.

#### mbsync.service
```
[Unit]
Description=Mailbox synchronization service

[Service]
Type=oneshot
ExecStart=-/usr/bin/toolbox run -c mbsync /usr/bin/mbsync -Va -c /var/home/nate/.config/mbsync/mbsync.rc
```

#### mbsync.timer
```
[Unit]
Description=Mailbox synchronization timer

[Timer]
OnBootSec=2m
OnUnitActiveSec=5m
Unit=mbsync.service

[Install]
WantedBy=timers.target
```

### Toolbox and VPNs

Let's say you are a big user of WireGuard tunnels and you want to expose a web
server that only serves on your WireGuard IP.  You'll find that the web server
crashes because the IP isn't available until WireGoard initializes.  So you need
to add a dependency on it like the "After" line here:

#### thttpd.service
```
[Unit]
Description=thttpd
After=system-wg\x2dquick.slice
     
[Service]
ExecStart=/usr/bin/toolbox run -c thttpd /usr/sbin/thttpd -C /etc/thttpd.conf
     
[Install]
WantedBy=multi-user.target
```
