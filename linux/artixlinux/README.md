[Back](../README.md)

# Artix Linux

Contains all of my knowledge of using Artix.

## Things to note when installing

Pick btrfs for root (a.k.a. the boot disk). It will give us snapshots and rollback functionality, extremely important. The Artix installer is smart enough to set up correct volumes for btrfs to work properly, so it is as straight forward as it can get.

## OpenRC

Since we're not on `systemd`, we'll need to configure some of the services ourselves. Generally, this involves creating a file in `/etc/init.d/`, such as `/etc/init.d/grub-btrfsd`, giving it something like
```
#!/sbin/openrc-run
# Copyright 2021 Pascal Jaeger
# Distributed under the terms of the GNU General Public License v3

name="grub-btrfs daemon"
command="/usr/bin/grub-btrfsd"
command_args="$optional_args ${snapshots}"
pidfile="/run/{RC_SVCNAME}.pid"
command_background=true

depend() {
   use localmount
}
```

Making it start at boot, then manually start it now
```
sudo chmod +x /etc/init.d/grub-btrfsd
sudo rc-update add grub-btrfsd default
sudo rc-service grub-btrfsd start
```

## Post-Install initial set-up

[This is a good article](https://www.linux.org/threads/things-to-do-after-installing-artix-linux.39252/) that talks about things to do post-install, but we don't need to read the whole thing, just these parts:

### Updating Artix Linux

System and packages update
```
sudo pacman -Syu
```
Reboot if needed to make sure we're not broken already.

To automate this process next time, we can create a script that will automatically create a timeshift snapshot, update all pacman and AUR packages as well as flatpaks too
```
sudo timeshift --create --comments "Pre-update backup" --scripted   
sudo yay
sudo yay -Yc
sudo yay -Scc
sudo flatpak update
```
Obviously this will require `timeshift`, `yay` and `flatpak` to already be installed.

### Installing needed packages for getting your OS up and running

Install base-devel (a group of packages needed in Arch Linux) and git (needed to clone packages from https://aur.archlinux.org/)
```
sudo pacman -S base-devel git
```

Enable Arch Linux repositories

Resource:
https://wiki.artixlinux.org/Main/Repositories

Install artix-archlinux-support
```
sudo pacman -S artix-archlinux-support
```

Edit /etc/pacman.conf
```
sudo nano /etc/pacman.conf
```

Edit the file placing the following after [galaxy] and it's repositorie:
```
[extra]
Include = /etc/pacman.d/mirrorlist-arch
```

Upgrade packages
```
sudo pacman -Syu 
```

At this point, assuming that everything went smoothly, we should be good to start setting Artix up to our needs. If anything broke at this point, continuing might not be the best idea, and perhaps we need to switch to another distro.

## Setting up

### yay

I'm not sure if `yay` is in Artix's official pacman repos or not, but since we installed `base-devel` and `git`, we can always install `yay` this way:
```
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

### Btrfs

Install `timeshift` and [grub-btrfs](https://github.com/Antynea/grub-btrfs)
```
sudo pacman -S timeshift
```
then
```
sudo pacman -S grub-btrfs
```

It might be safer to install them one at a time. `timeshift` will give us a GUI to create and delete snapshots, while `grub-btrfs` will create grub boot entries so we can boot directly into these snapshots.

Note that Artix uses OpenRC, so `timeshift`'s scheduled snapshots might not work, but we don't need it anyway.

We need to setup `grub-btrfsd` with OpenRC which is a daemon that helps `grub-btrfs` auto-detect `timeshift`'s snapshots as they are created/deleted.

Just need to create this file `/etc/init.d/grub-btrfsd`
```
sudo nano /etc/init.d/grub-btrfsd
```
And put this in the file
```
#!/sbin/openrc-run
# Copyright 2021 Pascal Jaeger
# Distributed under the terms of the GNU General Public License v3

name="grub-btrfs daemon"
command="/usr/bin/grub-btrfsd"
command_args="$optional_args ${snapshots}"
pidfile="/run/{RC_SVCNAME}.pid"
command_background=true

depend() {
   use localmount
}
```
Make the service runs on boot, then optionally start it immediately if we don't want to wait for a reboot
```
sudo chmod +x /etc/init.d/grub-btrfsd
sudo rc-update add grub-btrfsd default
sudo rc-service grub-btrfsd start
```
We should run the below commands manually once initially to setup `grub` with any snapshots that we created with `timeshift` prior to installing `grub-btrfs`
```
sudo /etc/grub.d/41_snapshots-btrfs
grub-mkconfig -o /boot/grub/grub.cfg
```

### Installing Steam

Since Steam still depends on `multilib` for 32 bit games at the point of writing this article, we'll need to enable it in our sources. [This Arch wiki article](https://wiki.archlinux.org/title/Steam) talks about it. Once this dependency is removed from Steam, we can remove `multilib`.

In `/etc/pacman.conf`, uncomment or add
```
[multilib]
Include = /etc/pacman.d/mirrorlist
```

Then install `steam`
```
sudo pacman -S steam
```

### Installing input-remapper

I use [input-remapper](https://github.com/sezanzeb/input-remapper) to perform simple macros with mouse and keyboard. It can be installed using `yay`
```
sudo yay -S input-remapper-git
```

It comes with a `systemd` service to auto-start the macros at boot, and it obviously won't work with OpenRC. Without the daemon, the `input-remapper-control` terminal command will not work. 

So to make it work in OpenRC
```
sudo nano /etc/init.d/input-remapper-openrc
```
```
#!/sbin/openrc-run
# Copyright 1999-2020 Gentoo Authors
# Distributed under the terms of the GNU General Public License v2

description="Daemon for key-mapper-service"
start() {
    ebegin "key-mapper service"
    start-stop-daemon --background --start --exec /usr/bin/key-mapper-service \
    --make-pidfile --pidfile /run/key-mapper.pid
    eend $?
}
#notice the bash like escape to use newlines
depend() {
	need dbus
}
```
```
sudo chmod +x /etc/init.d/input-remapper-openrc
sudo rc-update add input-remapper-openrc default
sudo rc-service input-remapper-openrc start
```

### Installing LACT

I use [LACT](https://github.com/ilya-zlobintsev/LACT) to set power limit for my AMD Instinct MI50. It can be installed with
```
sudo pacman -S lact
```

It comes with a `systemd` service, which is required for it to work properly. We'll need to create an OpenRC service for it manually.
```
sudo nano /etc/init.d/lact-daemon-openrc 
```
```
#!/sbin/openrc-run

# This service allows running LACT with Artix Linux OpenRC.
# Place it under /etc/init.d and make it executable.
# Run
# # rc-update add lact-daemon-openrc 
# # rc-service lact-daemon-openrc start

supervisor=supervise-daemon
command="lact"
command_args="daemon"

depend() {
    need localmount

    after bootmisc consolefont modules netmount
    after ypbind autofs openvpn gpm lircmd
    after quota keymaps
    before alsasound
    want logind

    provide lactd lact-daemon
}
```

Then make it start on boot and manually start it
```
sudo rc-update add lact-daemon-openrc default
sudo rc-service lact-daemon-openrc start
```

Note that there is also a flatpak for LACT, which won't work properly with this, since the service expects `lact` to be in the path. It might work if you modify the command in the service file to start the flatpak version, but I haven't tried.

### Installing flatpak

I like to use flatpaks for things that can be flatpaks, such as my browser. This way, my system is as stable as possible.
```
sudo pacman -S flatpak
```

Then install [Warehouse](https://flathub.org/en/apps/io.github.flattool.Warehouse) to manage flatpaks with a GUI. [Bazaar](https://flathub.org/en/apps/io.github.kolunmi.Bazaar) has better UX, but at this point [it requires systemd](https://github.com/bazaar-org/bazaar/issues/1298).
```
flatpak install flathub io.github.flattool.Warehouse -y
```

Here are some other flatpaks that I use:
```
flatpak install flathub com.brave.Browser -y
flatpak install flathub net.lutris.Lutris -y
flatpak install flathub chat.delta.desktop -y
flatpak install flathub com.github.tchx84.Flatseal -y
flatpak install flathub it.mijorus.gearlever -y
flatpak install flathub com.heroicgameslauncher.hgl -y
flatpak install flathub com.vysp3r.ProtonPlus -y
flatpak install flathub com.github.Matoking.protontricks -y
flatpak install flathub org.qbittorrent.qBittorrent -y
```

### Installing Vistathemeplasma

Note: This works as of KDE Plasma 6.6.4

I love Vista, and this one is a godsent. 

This theme needs to be cloned from git and compiled. First, follow the tutorial on [the repo itself](https://gitgud.io/aeroshell/vtp/vistathemeplasma/-/blob/Plasma/6.6/INSTALL.md?ref_type=heads), to install all the packages you need for compiling the theme.

Once ready, clone the repo and run the install.sh script with sudo.
```
git clone https://gitgud.io/aeroshell/vtp/vistathemeplasma.git
cd vistathemeplasma
chmod +x install.sh
sudo ./install.sh
```
or use https://gitgud.io/aeroshell/atp/aerothemeplasma.git for Windows 7 theme.
```
git clone https://gitgud.io/aeroshell/atp/aerothemeplasma.git
cd vistathemeplasma
chmod +x install.sh
sudo ./install.sh
```

After installing, reboot, then at the sddm login, select the VistaThemePlasma (Wayland) or (X11) session. (Mostly) everything will be auto-configured.

We still need to download Segoe UI and TerminalVector fonts.
Grab segoeui.tff from [here](https://github.com/mrbvrz/segoe-ui-linux/blob/master/font/segoeui.ttf) and optionally TerminalVector from [here](https://www.yohng.com/software/terminalvector.html). The TerminalVector font is to make the terminal look like Windows command prompt. It does complete the look, but with reduced readability.

Put them both in `~/.local/share/fonts`, then refresh the font cache
```
sudo fc-cache -f -v
```

Now the fonts should be added. Open Fonts settings in KDE and select Segoe UI for everything. Use size 8 for "Small", and size 9 for everything else. Leave the monospace one alone.
```
General - size 9
Small - size 8
Toolbar - size 9
Menu - size 9
Window Title - size 9
```
Leave Anti aliasing enabled, set sub-pixel rendering to RGB and Hinting to Slight.

For the TerminalVector font, open a Konsole window, right click, create new profile, change the font and colour scheme from there.

[A Youtube video](https://www.youtube.com/watch?v=S6l1peUm_c0) recommends adding an environment variable to make the font rendering a bit closer to Vista:

```
sudo nano /etc/environment
```
Add at the end
```
# For vista theme
QML_DISABLE_DISTANCEFIELD=1
```
I'm not sure if this does anything, but it doesn't hurt to have it there.

Finally, we need to install gadgets
```
git clone https://gitgud.io/catpswin56/win-gadgets.git
cd win-gadgets
chmod +x install.sh
sudo ./install.sh
```

After installing gadgets, we should be able to add them to the desktop using KDE's Add or Manage Widgets menu (by right-clicking the taskbar)

## Fixing common issues

### Wrong fs type, bad option, bad superblock

Occasionally, Dolphin might show this error when trying to mount a media device. Most of the time, this is caused by a dirty flag being unset when the machine was forced shutdown. It happened to me after I needed to force shutdown due to an issue with my GPU (not an issue with Artix itself).

To fix, first identify the device's name with `lsblk`. Or use Dolphin as it tells you the device name when hovering the mouse over the drive's icon.

Then, if the drive is NTFS, run this command to unset the dirty flag
```
ntfsfix -d /dev/sdd1
```
For exfat, run this command to scan for errors and unset the dirty flag
```
sudo fsck.exfat /dev/sdd1
```
