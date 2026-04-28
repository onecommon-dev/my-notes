[Back to Proxmox](README.md)

# System & Temperature Monitoring and optimisation in Proxmox

Since the motherboard is not directly exposed to the VMs, we can't monitor temperatures of the CPU and motherboard directly in the VM.
However, we can do this on the Proxmox host.

## Monitoring

### s-tui (preferred way)
My preferred way is to use a tool called [s-tui](https://github.com/amanusk/s-tui), which gives a nice graphical representation of the CPU speeds, temps and even power consumption, without the process view. 

It can be installed via apt
```
apt-get install s-tui
```
And we can run it with
```
s-tui
```
I sometimes struggle to remember the command name, so I create an alias in ~/.bashrc file
```
alias cpu='s-tui'
```
So I can run the tool with just `cpu`

Here's a preview of what it looks like

![s-tui preview](https://github.com/amanusk/s-tui/blob/master/ScreenShots/s-tui-1.0.gif?raw=true)

### lm-sensors
First, install the lm-sensors package
```
apt install lm-sensors
```

Now we can see the current readings with
```
sensors
```

To continually monitor the sensors, do
```
watch -n 4 sensors
```
The above command will show the readings and refresh every 4 seconds.

Here's an example output:
```
Every 4.0s: sensors                                                pve: Mon Sep  9 20:04:56 2024

coretemp-isa-0000
Adapter: ISA adapter
Package id 0:  +39.0°C  (high = +75.0°C, crit = +93.0°C)
Core 0:        +33.0°C  (high = +75.0°C, crit = +93.0°C)
Core 1:        +34.0°C  (high = +75.0°C, crit = +93.0°C)
Core 2:        +32.0°C  (high = +75.0°C, crit = +93.0°C)
Core 3:        +32.0°C  (high = +75.0°C, crit = +93.0°C)
Core 4:        +33.0°C  (high = +75.0°C, crit = +93.0°C)
Core 5:        +29.0°C  (high = +75.0°C, crit = +93.0°C)
Core 6:        +28.0°C  (high = +75.0°C, crit = +93.0°C)
Core 8:        +32.0°C  (high = +75.0°C, crit = +93.0°C)
Core 9:        +32.0°C  (high = +75.0°C, crit = +93.0°C)
Core 10:       +29.0°C  (high = +75.0°C, crit = +93.0°C)
Core 11:       +27.0°C  (high = +75.0°C, crit = +93.0°C)
Core 12:       +32.0°C  (high = +75.0°C, crit = +93.0°C)
Core 13:       +30.0°C  (high = +75.0°C, crit = +93.0°C)
Core 14:       +30.0°C  (high = +75.0°C, crit = +93.0°C)

nvme-pci-0100
Adapter: PCI adapter
Composite:    +27.9°C  (low  = -40.1°C, high = +83.8°C)
                       (crit = +87.8°C)
Sensor 1:     +43.9°C  (low  = -273.1°C, high = +65261.8°C)
Sensor 2:     +27.9°C  (low  = -273.1°C, high = +65261.8°C)
```

There is a way to edit the Proxmox dashboard's template and manually add html and javascript to show the values. However, it's not a method I recommend, due to the below reasons
- If you make a mistake when editing the template, such as a typo, you won't be able to access the dashboard
- Any future update that changes the template will revert your changes

##1# htop
Another way is to use good old `htop`.
```
apt install htop
```
And then simply run `htop` from terminal. Press F2 to go to htop settings and enable temperature and clock speed.

I haven't found a way to disable the process list that takes up 2/3 of the screen, but hey, at least we can monitor the temps and clock speed this way.

Here's an example output:
```
    0[1.1% 2200MHz 39°C]   4[0.4% 2204MHz 31°C]   7[|1.8% 2200MHz 28°C] 11[|0.7% 2203MHz 29°C]
    1[1.8% 2203MHz 33°C]   5[1.8% 2203MHz 32°C]   8[| 0.2% 2203MHz N/A] 12[|2.3% 2203MHz 25°C]
    2[4.5% 2203MHz 34°C]   6[2.0% 2200MHz 31°C]   9[|0.4% 2200MHz 30°C] 13[|1.6% 2203MHz 30°C]
    3[0.4% 2200MHz 32°C]                         10[|2.2% 2200MHz 32°C]
  Mem[|||||||||||||||||||||||||    33.5G/62.7G] Tasks: 46, 45 thr, 234 kthr; 1 running
  Swp[                                0K/4.00G] Load average: 0.41 0.64 0.77 
                                                Uptime: 00:18:32

  [Main] [I/O]
    PID USER       PRI  NI  VIRT▽  RES   SHR S  CPU% MEM%   TIME+  Command
      1 root        20   0  164M 12532  9076 S   0.0  0.0  0:01.22 /sbin/init
   2355 root        20   0 67.7G 32.8G 12160 S  35.9 51.1 15:15.60 ├─ /usr/bin/kvm -id 105 -name
   2356 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:00.16 │  ├─ /usr/bin/kvm -id 105 -n
   2357 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:04.77 │  ├─ /usr/bin/kvm -id 105 -n
   2418 root        20   0 67.7G 32.8G 12160 S   4.9 51.1  2:10.81 │  ├─ /usr/bin/kvm -id 105 -n
   2419 root        20   0 67.7G 32.8G 12160 S   5.2 51.1  2:07.07 │  ├─ /usr/bin/kvm -id 105 -n
   2420 root        20   0 67.7G 32.8G 12160 S   3.1 51.1  1:44.18 │  ├─ /usr/bin/kvm -id 105 -n
   2421 root        20   0 67.7G 32.8G 12160 S   5.6 51.1  2:19.32 │  ├─ /usr/bin/kvm -id 105 -n
   2422 root        20   0 67.7G 32.8G 12160 S   4.5 51.1  1:43.96 │  ├─ /usr/bin/kvm -id 105 -n
   2423 root        20   0 67.7G 32.8G 12160 S   3.1 51.1  1:27.96 │  ├─ /usr/bin/kvm -id 105 -n
   2424 root        20   0 67.7G 32.8G 12160 S   3.1 51.1  1:33.35 │  ├─ /usr/bin/kvm -id 105 -n
   2425 root        20   0 67.7G 32.8G 12160 S   7.0 51.1  1:53.32 │  ├─ /usr/bin/kvm -id 105 -n
   2448 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:00.00 │  ├─ /usr/bin/kvm -id 105 -n
   2546 root        20   0 67.7G 32.8G 12160 S   0.2 51.1  0:00.05 │  ├─ /usr/bin/kvm -id 105 -n
   2547 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:00.00 │  ├─ /usr/bin/kvm -id 105 -n
   2548 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:00.00 │  ├─ /usr/bin/kvm -id 105 -n
   2549 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:00.00 │  ├─ /usr/bin/kvm -id 105 -n
   2553 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:00.00 │  ├─ /usr/bin/kvm -id 105 -n
   2579 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:00.00 │  ├─ /usr/bin/kvm -id 105 -n
   2580 root        20   0 67.7G 32.8G 12160 S   0.0 51.1  0:00.04 │  ├─ /usr/bin/kvm -id 105 -n

```

## Optimisations

### Change CPU governer
CPU governors are part of the Linux kernel, they control the CPU frequency according to the needs of the applications. There are several profiles we can choose from, and since every CPU is slightly different, the available profiles might be different. For example, these profiles are available for my main PC:
```
conservative ondemand userspace powersave performance schedutil 
```
By default, `performance` is selected (at least for me), which means all cores will operate at their maximum allowed clock speeds all the time. As a result, my CPU pulls around 54W at idle.

By changing the governor to `ondemand`, the idle clock speeds are reduced from 3.2GHz to 1.4GHz, and the CPU pulls 34W. With this governor, when I put on a heavy load such as gaming, the frequency goes up to the max value, and I don't feel a performance lost. This seems like a win-win, I don't know why it's not the default.

To list available CPU governors for your system:
```
cat /sys/devices/system/cpu/cpu0/cpufreq/scaling_available_governors
```
To see what the current CPU governors being used are:
```
cat /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```
To change CPU governors for all CPUs and CPU cores (will not persist)
```
echo "ondemand" | tee /sys/devices/system/cpu/cpu*/cpufreq/scaling_governor
```
To change CPU governors permanently, append the following to te `GRUB_CMDLINE_LINUX_DEFAULT` line in grub
```
nano /etc/default/grub
```
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet intel_iommu=on iommu=pt cpufreq.default_governor=ondemand"
```
Then run `update-grub` to apply the change to the bootloader. The new governor will automatically activate upon reboot.

### Performance improvement (but hasn't worked)
Even though CPU-Z synthetic benchmark shows that the single thread and multi thread scores of the Windows 11 VM is within 5% of baremetal, the difference in games is massive. For example, in CS2, I get around 85 fps in-game with bots on baremetal but only 35 fps in the VM, and in Jedi Survivor, 110 fps baremetal vs 55 fps VM.
The below are the things I've tried so far that hasn't made a difference in performance.

#### Turn off mitigations
By default, Linux tries to mitigate vulnerabilities of the platform, such as mds and spectre. These mitigations slows down the system, so to get maximum performance, I'd need to disable them. If your system is so new that there are no vulnerabilities that needs mitigated, then you can skip this.

First, check the current status with 
```
lscpu
```
And note the lines in the Vulnerabilities section that says "Mitigated".

Modify grub with
```
nano /etc/default/grub
```
Then add "mitigations=off" to this line
```
GRUB_CMDLINE_LINUX_DEFAULT="quiet mitigations=off"
```
Rebuild grub with 
```
update-grub
```
After rebooting the whole system, use lscpu again to check if the system is now vulnerable. In my case, I see something like this:
```
Vulnerabilities:
  Gather data sampling:      Not affected
  Ghostwrite:                Not affected
  Indirect target selection: Not affected
  Itlb multihit:             KVM: Vulnerable
  L1tf:                      Mitigation; PTE Inversion; VMX vulnerable, SMT disabled
  Mds:                       Vulnerable; SMT disabled
  Meltdown:                  Vulnerable
  Mmio stale data:           Vulnerable
  Reg file data sampling:    Not affected
  Retbleed:                  Not affected
  Spec rstack overflow:      Not affected
  Spec store bypass:         Vulnerable
  Spectre v1:                Vulnerable: __user pointer sanitization and usercopy barriers only; no swapgs barriers
  Spectre v2:                Vulnerable; IBPB: disabled; STIBP: disabled; PBRSB-eIBRS: Not affected; BHI: Not affected
  Srbds:                     Not affected
  Tsx async abort:           Vulnerable
```
Which means it is working. Note that L1tf says "Mitigation" still, but the PTE Inversion mitigation has no performance impact according to this source: https://www.kernel.org/doc/html/latest/admin-guide/hw-vuln/l1tf.html

After disabling this, my Windows 11 VM's performance didn't change one bit, still as bad as before.

#### Pass L3 cache into the VM
When I view the CPU model using CPU-Z inside the VM running Windows 11, I see only 16MB of L3 cache, even though my CPU has 50MB. After some research, I found a way to make the VM more aware of this. It turns out that this is related to CPU topology, i.e. how the CPU is laid out internally. So the way to make this work is to add a few CPU arguments to QEMU.

Unfortunately, Proxmox doesn't have the UI to add this, so we'll have to do it manually.

First, grab the current set of QEMU arguments for the VM:
```
qm showcmd 105
```
where 105 is my VM's id. Substitute this number with your VM id.

Copy the output into notepad or similar and look for the -cpu arguments. Mine look like this:
```
-cpu 'host,hv_ipi,hv_relaxed,hv_reset,hv_runtime,hv_spinlocks=0x1fff,hv_stimer,hv_synic,hv_time,hv_vapic,hv_vendor_id=proxmox,hv_vpindex,kvm=off,+kvm_pv_eoi,+kvm_pv_unhalt'
```
Take this and add ```host-cache-info=on,topoext=on,hv_vendor_id=GenuineIntel``` to this list. I use GenuineIntel as the hv_vendor_id since my CPU is Intel, but I'm not sure if this makes any difference. Then add it to the args section of the vm .conf file like this:
```
nano /etc/pve/qemu-server/105.conf
```
```
args: -cpu 'host,host-cache-info=on,topoext=on,hv_ipi,hv_relaxed,hv_reset,hv_runtime,hv_spinlocks=0x1fff,hv_stimer,hv_synic,hv_time,hv_vapic,hv_vendor_id=proxmox,hv_vpindex,kvm=off,+kvm_pv_eoi,+kvm_pv_unhalt'
```
Reboot the VM and CPU-Z should now see 50MB of L3 cache.

This works, however from my testing, it made no performance impact.

#### CPU Pinning

By default, the Proxmox CPU scheduler will dynamically assign work to CPU cores as it sees fit. We can override this behaviour and pin the VM's vCPU processes to particular physical CPU cores, using Linux's tools.

It seems that the way to do this on Proxmox is using a hookscript.

First, check the CPU topology in Proxmox terminal with
```
lscpu -e
```
In my case, the output is:
```
CPU NODE SOCKET CORE L1d:L1i:L2:L3 ONLINE    MAXMHZ    MINMHZ       MHZ
  0    0      0    0 0:0:0:0          yes 3300.0000 1200.0000 1299.2390
  1    0      0    1 1:1:1:0          yes 3300.0000 1200.0000 1199.3400
  2    0      0    2 2:2:2:0          yes 3300.0000 1200.0000 1245.1010
  3    0      0    3 3:3:3:0          yes 3300.0000 1200.0000 2498.5979
  4    0      0    4 4:4:4:0          yes 3300.0000 1200.0000 1698.9690
  5    0      0    5 8:8:8:0          yes 3300.0000 1200.0000 1860.8810
  6    0      0    6 9:9:9:0          yes 3300.0000 1200.0000 1199.3409
  7    0      0    7 10:10:10:0       yes 3300.0000 1200.0000 1499.2620
  8    0      0    8 11:11:11:0       yes 3300.0000 1200.0000 1400.7240
  9    0      0    9 12:12:12:0       yes 3300.0000 1200.0000 1299.2260
 10    0      0   10 16:16:16:0       yes 3300.0000 1200.0000 1264.3220
 11    0      0   11 17:17:17:0       yes 3300.0000 1200.0000 1214.4041
 12    0      0   12 18:18:18:0       yes 3300.0000 1200.0000 2498.6289
 13    0      0   13 19:19:19:0       yes 3300.0000 1200.0000 1308.2870
 14    0      0   14 20:20:20:0       yes 3300.0000 1200.0000 1210.5470
 15    0      0   15 24:24:24:0       yes 3300.0000 1200.0000 1199.3600
 16    0      0   16 25:25:25:0       yes 3300.0000 1200.0000 1200.0000
 17    0      0   17 26:26:26:0       yes 3300.0000 1200.0000 2498.6270
 18    0      0   18 27:27:27:0       yes 3300.0000 1200.0000 1216.0460
 19    0      0   19 28:28:28:0       yes 3300.0000 1200.0000 1200.0000
```
This shows that I have 20 Cores, numbered from 0 to 19, which all live under the same Node and Socket. Moreover, they all have their own L1d, L1i and L2 caches. This is because I disabled Hyperthreading in my motherboard BIOS. If HT was enabled, there would be 40 "cores" in groups of 2, each group sharing L1 and L2 caches.
It is recommended to assign cores that share caches to the same VM to avoid random cache saturation due to spikes from another VM.

First, create a script in /var/lib/vz
```
cd /var/lib/vz
mkdir snippets
ln -s /var/lib/vz/snippets ~/snippets
cd ~/snippets
nano cpu-pinning-0-15.sh
```
and put the following content in:
```
#!/bin/bash

vmid="$1"
phase="$2"

cpuset="0-15"

if [[ "$phase" == "post-start" ]]; then
    main_pid="$(< /run/qemu-server/$vmid.pid)"
    taskset --cpu-list  --all-tasks --pid "$cpuset" "$main_pid"
fi
```
Make it executable
```
chmod +x cpu-pinning-0-15.sh
```
If this script runs, it will confine the VM vCPU processes to the cores we defined (0-15 in this example).

To make this script execute automatically, we add a Hookscript section to our VM's conf file:
```
nano /etc/pve/qemu-server/105.conf
```
And add
```
hookscript: local:snippets/cpu-pinning-0-15.sh
```
Save, reboot VM, and it should now be pinned to those CPU cores.

To make Proxmox's kernel processes avoid these CPU cores, we need to modify grub
```
nano /etc/default/grub
```
And add
```
GRUB_CMDLINE_LINUX="isolcpus=0-15"
```
Then rebuild grub with
```
update-grub
```
The isolcpus parameter will tell the Linux kernel to avoid the CPU cores specified.

However, after having done this, not only that I saw no performance gains, but the isolcpus parameter makes the whole system slow to a crawl. Same thing even if I change it to 0-9 to give 10 cores to the Linux kernel and 10 cores to the VM. I'm not sure why.

## Good resources:
- https://gist.github.com/tinwhisker/53d77c887535129021a1f58359930935
- https://itsfoss.com/monitor-cpu-gpu-temp-linux/
- https://docs.renderex.ae/posts/CPU-Pinning/
