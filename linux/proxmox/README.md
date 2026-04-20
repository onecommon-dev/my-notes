[Back](../README.md)

# proxmox-notes
This document holds all of my experience so far with Proxmox, with a focus on retro OSes. Windows 98, Windows XP, Windows Vista.

Proxmox version: 8.2

<details>
<summary>Previous machines</summary>

-----------------------
Previous Machine 1 - **Modern machine**
- CPU: i7 6700K
- iGPU: Intel HD 530
- Asus Maximus Hero VIII motherboard
- GPU1: RTX 3090
- GPU2: AMD R5 340X/R7 250
- GPU3: Geforce 6600
------------------------
</details>

Current Machine 1 - **Modern machine**
- CPU: Xeon E5 2679 V4
- iGPU: None
- Asus X99-E-10G WS motherboard
- GPU1: RTX 3090
- GPU2: GT 640
- GPU3: Geforce 6600
  
Current Machine 2 - **Time machine**
- CPU: i7 3770
- iGPU: Intel HD 4000
- Dell Optiplex 7010 MT motherboard
- GPU: GTX 750 Ti

## What is Proxmox
Proxmox is a Linux-based Type 1 Hypervisor. Some other Type 1 Hypervisors are Hyper-V (only when running on Windows Server), Red Hat Enterprise Virtualisation (RHEV) and vSphere (EXSi).

If you used VMWare, VirtualBox or Parallels before, you know what a Hypervisor is. Those are Type 2 Hypervisors. Type 2s are slow and inefficient, but they are convenient because you can run them on top of Windows, Mac or Linux, and easily share files between the host and the guest.

Type 1s on the other hand are a beast. They lack the convenience features, but they have extremely low overheads and very efficient. You can directly pass through physical hardware to the VMs, getting all the native performance, while keeping the benefits of virtual machines such as ease of backing up, portability and even compatibility. If running a XP VM in VMWare feels sluggish and laggy, running XP in a Proxmox VM feels like native, or even faster since new hardware is so much faster than old XP-supported hardware.

Yes, you can run XP and Vista, and even Windows 98 on unsupported motherboards and CPUs (*). I'm running XP and Vista on my modern machine via Proxmox, with full GPU acceleration. And I'm running regular full backups and uploading them straight to the cloud, **all while the VMs are running**. How cool is that?

(*) _Provided that the OS runs on a generation of x86 CPUs that KVM supports. KVM is the open-source virtualisation software that Proxmox uses under the hood. At the moment, KVM should support all the way back to DOS, even though Proxmox only lists Windows 2000 minimum._

Why Proxmox? Because it is Linux, free and open-source, and has a huge community.

There is a big catch. Type 1 hypervisors such as Proxmox have a very steep learning curve, it is not for the faint of heart. For instance, Proxmox doesn't come with a desktop environment, and you need another computer to use its web interface. Moreover, you'll need to learn (usually the hard way) which combinations of hardware work well and which don't. Also, while you don't have to be a Linux wizard, having fluency in it helps tremendously.

## Pre-requisites
### Hardware
#### Motherboard & CPU
For any of this to work well, your motherboard and CPU need to support hardware isolation which is called **IOMMU groupings**. I believe this technology is called VT-d in most motherboard BIOS. But just supporting and turning on VT-d is not enough. the motherboard's IOMMU groupings need to be separated enough. 

In layman's terms, this means the motherboard can allow some important hardware components (such as each PCI/PCIe slot) to be individually isolated and passed through to a Proxmox virtual machine. Without proper IOMMU groupings, hardware passthrough might be extremely painful or even impossible.

This is why if you want a machine to run Proxmox or any other type 1 hypervisor with hardware pass-through, you should build a machine around that requirement. I believe that server and workstation motherboards will more likely satisfy this requirement, consumer motherboards will likely be hit or miss.

For the CPU, it is not as important as long as it is x86, as it can be virtualised or emulated by KVM/Proxmox with decent speeds. Since retro OSes don't consume much CPU cycles, we should get decent speeds regardless. Not sure how this would work on an ARM based system as I have not tried.

My previous modern machine has decent IOMMU groupings but not great. It has 3 long PCIe slots, 2 of which are separately grouped, which means I can pass through 2 GPUs to 2 different VMs. It has 3 PCIe x1 slots, but 2 out of 3 are disabled if one of the long PCIe slots mentioned above is occupied. This means I can have 1 main GPU for modern games, 1 retro GPU for XP/Vista, and 1 PCIe x1 slot for a sound card or whichever PCIe device (Note that my Sound Blaster XFi doesn't work on XP VMs for some strange reason).

I have upgraded to a workstation motherboard on the X99 platform, which was very expensive when it first came out but has since come down to a very reasonable price. It has 7 full-size PCIe x16 slots sharing 32 lanes to the CPU, and it supports PCIe bifurcation. Most importantly, it has superb IOMMU groupings, pretty much everything is in its own group. 

#### GPU
Next, the GPU needs to support the OS. This means we need a GPU from the era that has drivers for the specific OS we need. For example:
- Nvidia FX 5500 for Windows 98
- Nvidia GTX 750 Ti for Windows XP

The below section is outdated and no longer applies, but still included for completeness:
<details>
<summary>Outdated section</summary>

______________________________
For consumer Nvidia cards on Windows XP up to Windows 7, there is an issue with official drivers which disables itself when it detects that we are running in a VM. The affected drivers are roughly from versions 337.88, 340.52 and above.
To get pass this issue, we need to put this to our vm arguments `-cpu host,kvm=off`. I've verified that this works for driver version 337.88 and 340.52. For newer drivers, we apparently need to disable some kvm optimisations, which will tank performance so I'm not very interested.

This means if we want to use a consumer Nvidia GPU for XP up to 7 in Proxmox and want good performance, we should stick with driver version 340.52 maximum.

Professional Nvidia (i.e. Quadro) and AMD cards do not have this issue.

The latest Nvidia consumer drivers on Windows 10 and up added "support" for virtualisation, so it's not an issue for that use case.
______________________________
</details>

**Update: I later tested a GTX 750 Ti under XP 32-bit and Vista 64-bit with the latest drivers (368.81 for XP and 365.19 for Vista) and they both worked, without the `kvm=off` flag or having to do anything else.** It seems the issue described above does not exist anymore for the latest version of Proxmox.

What could have been the issue was that I was running the VMs with 
- The virtual GPU added (by having the VMWare compatible display option on) in addition to GTX 750 Ti
- Did not have the Primary GPU option selected for the GTX 750 Ti
- Had the wrong settings in /etc/modprobe.d/vfio.conf by blindly following guides. By having `disable-vga=1` at the end, the GTX 750 Ti do not output the boot sequence, therefore requiring the virtual GPU added to install and boot Windows.

Perhaps some or all of the above messed up the drivers somehow.

### Setting up Proxmox
#### Modern machine 
For storage, here is my setup:
- Boot drive: 256 GB Samsung 870 EVO SATA SSD running Proxmox
- VM drive: 2TB Western Digital Black NVMe PCIe SSD
- Backup drive: 2TB Seagate Baracuda 7200 rpm HDD

I plan to add another 2TB HDD in the future for a mirror ZFS setup (similar to RAID) for backups.

For GPU passthrough, do NOT follow the guides out there blindly. For example, what I found is that adding boot flags to the grub entry did nothing for me, and GPU passthrough works without them. My hypothesis is that the latest versions of Proxmox already have built-in fixes for most PCI passthrough issues, and the guides are outdated. In the end, what matters is whether your motherboard and CPU support hardware isolation which is called **IOMMU groupings**.

Here's what I have done:
- Enabling _VT-d / Intel Virtualisation Technology for Direct IO / IOMMU_ in the BIOS. They are different names for the same thing. 
- Add virtio modules in Proxmox (we may even not need to do this, I haven't tested)
```
nano /etc/modules
```
```
vfio
vfio_iommu_type1
vfio_pci
vfio_virqfd
```
- Blacklisting kernel modules in Proxmox (we may even not need to do this, I haven't tested)
```
nano /etc/modprobe.d/blacklist.conf
```
```
blacklist radeon
blacklist nouveau
blacklist nvidia
# iGPU
blacklist snd_hda_intel
blacklist snd_hda_codec_hdmi
blacklist i915
```

To find out which kernel modules are used for the hardware, run the below command and take note of the kernel modules
```
lspci -v
```
- If blacklisting the kernel modules don't work, we might need to blacklist the hardware ids directly.
```
nano /etc/modprobe.d/vfio.conf
```
```
options vfio-pci ids=1102:000b
```
The id above `1102:000b` is the hardware id of my Soundblaster XFi. This will force Proxmox to not use this soundcard. This is like blacklisting kernel modules, but for specific device ids.
To find out hardware ids, run
```
lspci -nn
```
- Set 2 options in kvm.conf, not sure what they do (we may even not need to do this, I haven't tested)
```
nano /etc/modprobe.d/kvm.conf
```
```
options kvm ignore_msrs=Y report_ignored_msrs=0
```

After changing any blacklisting settings in /etc/modprobe.d, don't forget to run the following command before rebooting:
```
update-initramfs -u -k all
```

Another thing we should do is set up our BIOS so that the IGPU is used to boot the computer and run Proxmox. That way all of our discreet GPUs will be able to see the VM's boot sequence and we won't get the above issue. Otherwise, we may get a black screen until the guest OS inside the VM loads the graphics driver.

Apparently, if the GPU is used to boot Proxmox, its vbios is touched somehow which prevents the card from showing VM boot screen.

#### Time machine
The above configuration even though allows GPU Passthrough to function on this machine, I could not get the GPU to output the boot sequence if the VM uses UEFI. If the VM uses SeaBIOS, the HD 6450 can output the boot sequence. The Geforce FX 5500 on the other hand will not output boot sequence for either type. However, I know the pass-through works because I could get Windows 11 to output an image on the FX 5500 when the drivers load.

Note that apparently, we shouldn't use Intel iGPU for booting Proxmox, because there is an issue with Intel iGPU that causes legacy GPUs to not work properly in Proxmox? This seems to make no difference in my setup.

#### ACS Override kernel patch
Note: For motherboards with bad IOMMU groupings, there is a workaround called ACS override which sacrifices security for more IOMMU grouping possibilities.
```
nano /etc/default/grub
```
Then add `pcie_acs_override=downstream` to `GRUB_CMDLINE_LINUX_DEFAULT` like below
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet pcie_acs_override=downstream"
```
This will enable a kernel patch which may make your IOMMU groupings more flexible, [at the cost of security](proxmox-acs-override-vulnerability.md).

## GPU vbios dumping
We can dump the GPU's vbios rom to supposedly preserve this ability to see the VM's boot sequence in case it breaks for whatever reason, but I have not confirmed whether this works. 
In all the different guides online, we'll only find these steps:
```
cd /sys/devices/[device-id-here]
echo 1 > rom
cat rom > /tmp/romfile
echo 0 > rom
```
However, if we do this while the IGPU is used for booting, we'll get this error `cat: rom: Input/output error`.
To dump this, here are the steps that worked for me:
```
find /sys/devices -name rom
```
This lists all the paths to a possible rom in all of our devices. Take note of the device that we want the rom from (we will need to know our device id for this, but it can be easily seen from Proxmox's PCI Passthrough menu). 
For example, this could be
```
/sys/devices/pci0000:16/0000:16:00.0/0000:17:00.0/0000:18:08.0/0000:19:00.0/0000:1a:10.0/0000:22:00.0/0000:23:00.0/0000:24:00.0/rom
```
Copy the whole thing.

Run this to somehow allow us to read the rom. I will not pretend that I understand what it does, but it works.
```
setpci -s [device id] COMMAND=2:2 (manually manipulate the memory enable bit with setpci)
# For example:
# setpci -s 0000:24:00.0 COMMAND=2:2
echo 1 > [paste the thing that we copied from above]
cat [paste the thing] > /tmp/romfile
echo 0 > [paste the thing]
setpci -s [device id] COMMAND=0:2
```

Once we have our romfile, for example, if we put it at `~/gpuroms/rtx3090.rom`, here's how to add it to the VM.
```
nano /etc/pve/qemu-server/[vmid].conf
```
Go to the GPU device, the line should start with `pci`. Add this to the comma-separated list
`romfile=../../../root/gpuroms/rtx3090.rom`

The 3 backlashes are needed because by default, Proxmox tries to find the rom in the default directory for roms, which is `/usr/share/kvm/`, even if we put in the full path (in this case it is `/root/gpuroms/rtx3090.rom`), so we need to get back to the root for the path to work.

To avoid the above, we can move our gpu roms there. I'd like to move them under `/usr/share/kvm/gpuroms`, then create a symlink to the home directory for easy access later.
```
ln -s /usr/share/kvm/gpuroms ~/gpuroms
```
With this, from now on, we can simply put `romfile=gpuroms/rtx3090.rom` in our vm conf.

**One big advantage of using the `romfile` parameter is that we can test updated bioses for our GPUs without having to flash it for real and risk permanently bricking it.**

I find this very helpful when I found for certain old GPUs, a newer bios works well for Windows 98 while an older version works well for XP and up.

## VirtIO drivers
https://pve.proxmox.com/wiki/Windows_VirtIO_Drivers

This driver is a bit like VMWare tools, but mainly for insanely fast networking and disk IO within the VMs. The drivers should work for Windows XP and up. Grab the latest bleeding edge on that link, it's stable enough to be used for our use case.

The Linux kernel supports VirtIO out of the box, so this separate driver ISO is only required for Windows.

## Cloud Backups
I have created a fork of TheRealAlexV's proxmox-vzbackup-rclone [here](https://github.com/hoangbv15/proxmox-vzbackup-rclone).
This script will do 2 things
- Compress everything under /etc, /var/lib/pve-cluster and /root into a tar in a ramdisk, and upload it using rclone
- Upload everything under your gzdump folder using rclone

For this to work, first we need to setup rclone. First install rclone
```
apt-get update
apt-get install rclone
apt-get install git
```
Then follow this guide [here](https://rclone.org/pcloud/) to set it up with pCloud or whichever cloud backup provider you use (which is compatible with rclone).
After that, simply git clone the script:
```
git clone https://github.com/hoangbv15/proxmox-vzbackup-rclone
cd proxmox-vzbackup-rclone
chmod +x vzbackup-rclone.sh
```
Edit the script to set the path to your dumps. It would be `/var/lib/vz/dump` by default if you just dump to the same disk as your Proxmox installation.
A full backup upload can be manually started with the command
```
/root/proxmox-vzbackup-rclone/vzbackup-rclone.sh full-backup
```
However keep in mind that the process will take a **very** long time, and the vnc connection might time out. If it does, the session will close and the upload will stop. So it's best to use a cron task for this.

To schedule this clone weekly, we could use vanilla cron, but that requires the computer to be on at a specific time. So we'll instead use `anacron`, which will allow us to run tasks on a non-absolute timing, best-effort basis.
To install `anacron`:
```
apt install anacron
```
Then run `nano /etc/anacrontab`, read the examples and add a line to schedule our backup cloud upload. In my case, I want to run the upload weekly, so I added
```
7       10      rclone-backup   /root/proxmox-vzbackup-rclone/vzbackup-rclone.sh full-backup
```
The 10 here means the job will be delayed by 10 minutes after booting.

## Mouse & Keyboard
We have 2 options.
- Passthrough the USB mouse & keyboard directly to the guest OS, and use a physical USB KVM to switch between OSes.
- Passthrough the mouse & keyboard events from the Proxmox host to the guest OSes using emulated PS/2. This method has the benefit of being able to software switch between the guest OSes without needing a USB KVM. The downside is that any software that relies on the USB connection won't work, for instance, Razer Synapse.

### Emulated
To add emulated PS/2 mouse & keyboard, which will use events from your USB mouse & keyboard connected to the host, find out the path of your USB devices by
```
cd /dev/input/by-id
ls
```
Then add it to the VM by editting its conf file and adding to args
```
args: -object input-linux,id=kbd1,evdev=/dev/input/by-id/YOURKEYBOARD,grab_all=on,repeat=on -object input-linux,id=mouse1,evdev=/dev/input/by-id/YOURMOUSE
```
For example, here are mine:
```
-object input-linux,id=kbd1,evdev=/dev/input/by-id/usb-Keychron_Keychron_K1-event-kbd,grab_all=on,repeat=on -object input-linux,id=mouse1,evdev=/dev/input/by-id/usb-Razer_Razer_Viper_8KHz-event-mouse
```
To switch between the guest OSes, press both the Ctrl keys on the keyboard. Note that the mouse needs to be after the keyboard in the entry above, for the switch to also switch the mouse. Otherwise, the mouse will stay in the first guest OS.

To see the raw events on the host, to confirm that you are using the correct event:
```
cat /dev/input/by-id/usb-Razer_Razer_Viper_8KHz-event-mouse | od -t x1 -w24
```
```
2e 16 e9 63 00 00 00 00 17 95 0a 00 00 00 00 00 02 00 00 00 fd ff ff ff
|          16 bytes long system time           |type |code |   value   |
```

## Sound
There are 2 approaches to sound
- **This only works with machine type i440, not q35** Creating an emulated sound card on the VM, such as AC97, which will pipe the audio to the Proxmox host to be output via a physical sound card that the host uses. This uses the Linux kernel's ALSA audio driver. The benefit of this method is that there is no need for PCI/USB passthrough. The downside is that we cannot have fancy features such as EAX, if we have a capable Soundblaster card. Also, the volume is much softer for some reason.
- Passing through a physical sound card to the guest VM. The upside is that it is possible to have special features such as EAX with a capable sound card. The downside is that it requires PCI/USB passthrough, which may or may not work. For instance, I could not get my Soundblaster XFi PCIe card to work under XP, despite that it works fine in a real XP machine.

### Emulated
First, there is no need to install ALSA as it is already baked in to the Linux kernel. We can list the sound cards and the card id number that are currently usable as sound output devices by the command
```
aplay -l
```
If there is only 1 device in the list and is the correct device that we want to use, there is no further work required for ALSA.

If the card we want does not show up, then the device is most likely blacklisted, either by kernel module  or device id blacklisting (see GPU passthrough section). For example, the below in `/etc/modprobe.d/blacklist.conf` will blacklist the kernel modules required for Intel chipset onboard sound
```
blacklist snd_hda_intel
blacklist snd_hda_codec_hdmi
blacklist i915
```
For some strange reason, if I don't blacklist i915, which is a kernel module for Intel graphics, my NVidia Geforce 6600 seems to be grabbed by Proxmox and stops working for passthrough. Strange.

If there are more than 1, we can select the one we want to output by forcing PCM and CTL to output to the card id number.
```
nano /etc/asound.conf
```
```
pcm.!default {
    type hw
    card 0
}

ctl.!default {
    type hw           
    card 0
}
```
Sometimes the card id can randomly change. To force a card id to follow a pattern, check out [this article](https://wiki.archlinux.org/title/Advanced_Linux_Sound_Architecture#Set_the_default_sound_card).
For archival sake I'll include the crucial info here. We can edit this file to specify which card id our audio devices will have
```
nano /etc/modprobe.d/alsa-base.conf
```
```
options snd_usb_audio index=0
options snd_hda_intel index=1
```
The above config will cause our USB DAC to have card id 0, which will be used by Alsa by default.

To test that your sound card is working properly with Alsa, use
```
speaker-test -t wav -c 2
```
Where 2 is the number of speakers you have.

To add an emulated sound card to our VM using ALSA as the output, we need to modify the VM's conf file and add this to the VM's args
```
-audio driver=alsa,model=ac97,id=audio0
```
A list of supported models by QEMU can be found [here](https://computernewb.com/wiki/QEMU/Devices/Sound_cards#QEMU_7_and_above:)

You may notice Soundblaster 16 and Adlib are in the list, but I have not managed to make it work for my Windows 98 VM, the guest won't see the sound card for some reason. AC97 works, but with low volume even with the slider maxed. I am temporarily getting around the issue by using a physical USB sound card with a volume boost function.

Note that for emulated sound cards that are not natively supported by the guest OS, you will need to install its driver. For instance, the emulated AC97 card is an Intel(r) 82801AA AC'97, which requires this [driver](disk-images/AC97-WDM-Driver-for-Windows-98.iso).

It is recommended to install the package `alsa-utils` in order to adjust the volume of the output device from proxmox
```
apt install alsa-utils
```
Run this to adjust volumes
```
alsamixer
```

Useful links:
- https://www.maketecheasier.com/alsa-utilities-manage-linux-audio-command-line/

### Passthrough
I have tried passthrough a Soundblaster XFi but that freezes the entire physical machine. It seems the reason is that the Proxmox host's ALSA driver gets a hold of the card.

The freezing stops happening if I blacklist it
```
nano /etc/modprobe.d/vfio.conf
```
```
options vfio-pci ids=1102:000b
```
The id above `1102:000b` is the hardware id of the soundcard. This will force Proxmox to not use this soundcard. This is like blacklisting drivers, but for specific device ids.

To find out hardware ids, run
```
lspci -nn
```

Now, the XP VM will recognise the card and the drivers will install successfully. However, there is no sound output. It seems there is an issue with the drivers of the XFi on XP. The sound card works fine on Vista with Windows default drivers. However, if I boot into XP with the drivers installed and the sound card attached, the card isn't released properly once the XP VM is shut down. If I boot into Vista again, it does not see the sound card, and only a reboot of the Proxmox host will fix it.

However, the same card and drivers work fine on a real XP machine, not to mention it works on Vista and up under Proxmox.

Perhaps getting this card to work under XP in Proxmox is a futile endeavour after all.

## Miscellaneous
- [VM Setup Guide](vm-setup-guide.md)
- [SSD Protection](ssd-protection-proxmox.md)
- [Mount existing disks from previous Proxmox installation](mount-existing-disks-to-storage.md)
- [Temperature monitoring and optimisations](temperature-monitoring-optimisation.md)
