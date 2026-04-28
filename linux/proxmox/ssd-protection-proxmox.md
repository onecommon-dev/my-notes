[Back to Proxmox](README.md)

# SSD Protection in Proxmox

> [!WARNING]  
> These are info that I gathered. I have tested them, but not extensively. Treat them as anecdotal experiences. After doing all of these, I managed to bring the writes from several hundred MBs to about 50 KB per hour.

Proxmox by default writes frequent data to the drive that you installed it in. There's a high chance you used an SSD for this. So here are some relevant info on how to limit frequent writing to the drive.

## Monitor SSD writes
This can be done by installing **iotop**
```
apt install iotop
```
This will show us all the current processes that cause disk reads and writes.
From this, I have discovered 4 more things that are causing writes to the disk: **kvm**, **jbd2**, **pmxcfs** and **rrdcached**.

For **kvm**, I believe the writes that **iotop** is showing are to in-memory pipes and sockets rather than actual disk. I couldn't find any extra clues, and the amount reported by iotop is way more than what's being written to disk, which can be determined with smartctl. 

**jbd2** is a kernel process in charge of journaling. This basically means it's not the culprit causing the IO, but another process is.

For the other processes, see their own sections.

To see SMART status of our disk, do
```
smartctl -a /dev/sdc
```
where ```sdc``` is the disk you want to monitor. Pressing tab after typing ```smartctl -a /dev/``` should display a list of all disks.

On some SSDs, this command will directly give the total number of TBs or GBs written. On some other, it will give the number of Total LBAs Written. To convert this to MBs, we need the a Sector Size, usually 512 KB, and is in the result of the same command. Take these values to an online calculator such as [this one](https://dannyda.com/2020/09/06/how-to-convert-smart-attribute-241-total_lbas_written-to-mb-gb-tb-pb/).

Another tool is **iostat**, which can be installed with 
```
apt install sysstat
```
This tool shows the IO rate of all devices, but I found it to be not very accurate. Either that or I haven't found a way to read the stats properly.

One more tool is **fatrace**
```
apt install fatrace
```
This can monitor which files are having disk operations. We can filter to only see Write operations with
```
fatrace -f W
```
There is another tool called **csysdig** that some people like, which is a part of the package **sysdig**. However I won't be using this, as installing it on Proxmox requires installing a bazillion other packages and more than 600 MB of disk space.

## Use zram for swap
https://pve.proxmox.com/wiki/Zram

To turn off physical SSD swap, comment out the other swap entry that is not the `zram` one in `/etc/fstab`

## Limit the number of days that logs are stored 
Check out the comments in this topic
https://forum.proxmox.com/threads/log2ram-getting-full-frequently-and-proxmox-becomes-unusable.98647/

Edit the files in `/etc/logrotate.d` and change the `rotate` value to 3.
For example:
```
~# cat /etc/logrotate.d/pve-firewall
/var/log/pve-firewall.log {
    rotate 7
    daily
    missingok
    notifempty
    delaycompress
    compress
    sharedscripts
    create 640 root adm
    postrotate
        invoke-rc.d pvefw-logger restart 2>/dev/null >/dev/null || true
    endscript
}
```

## Proxmox logging to RAM
We can use **log2ram** to avoid writing logs to disk: https://github.com/azlux/log2ram

Note that **log2ram** will still commit the changes in the ram disk to the ssd upon system shutdown or once a day. The daily timer can be disabled.
```
systemctl disable log2ram-daily.timer
```
Ideally, we should disable logging altogether if we don't need logging at all, by disabling **rsyslog** and **journald** services.
```
systemctl disable rsyslog journald
```
https://forum.proxmox.com/threads/is-it-possible-to-disable-all-logs-local-completly.52067/

In case we want **journald** running, better configure it to not forward logs to rsyslog with
```
nano /etc/systemd/journald.conf
```
```
Storage=volatile
ForwardToSyslog=no
```

## Turn off High Availability services and Corosync
These services are there to enable more resilience in cases where multiple host machines are used in a cluster. For our usage, we only use a single-node Proxmox install (ie, you're not clustering / using high availability). You can enable these services later on if you decide to use clustering (not personally tested). These services do a lot of constant writes.

```
systemctl disable --now pve-ha-crm.service
systemctl disable --now pve-ha-lrm.service
systemctl disable --now pvesr.timer
systemctl disable --now corosync.service
```

## rrdcached
This seems to relate to **pvestatd**, a service that deals with status and graphs.

https://pve.proxmox.com/wiki/Service_daemons

```
systemctl disable rrdcached.service
```
Turning this off will cause the graphs on the Web UI to stop working, which is not a big deal. We could also do
```
systemctl disable pvestatd
```
But this will make the Web UI lose vital information, such as VM names, status, CPU usage, memory usage, disk usage, etc. Plus, it doesn't cause any less IO. So we can leave this on.

If we must have **rrdcached** running, here are some things that can reduce its disk flushing frequency:
- Enabled ```WRITE_TIMEOUT=3600``` and add ```FLUSH_TIMEOUT=7200``` in ```/etc/defaults/rrdcached``` config file to reduce disk IOPS. 
- Disabled ```JOURNAL_PATH=/var/lib/rrdcached/journal/``` in ```/etc/defaults/rrdcached``` config file to reduce disk IOPS.
- Added ```${FLUSH_TIMEOUT:+-f ${FLUSH_TIMEOUT}} \``` to ```/etc/init.d/rrdcached```

## pmxcfs
This is a service that takes care of Proxmox VM configs. On disk, the configs are stored in a SQL database, and at runtime, this is copied to RAM, which the service keeps in sync with the disk version (using **corosync**, which we disabled in a step above). However, this syncing happens every several seconds. Proxmox does this so that if you create a "cluster", the configs are synced to all machines in the cluster. This feature is meaningless to us since we don't use it.

I believe this service is part of the High Availability services that we shut down in a step above. For some reason, this one is still running despite of that, I don't know why.

It seems to be possible to stop the service with:
```
systemctl stop pve-cluster.service
```
However, doing so will cause the web GUI to stop working entirely. It is possible that we also can't change VM configs, since the service is no longer syncing them with the database. So, it is not advisable to disable the service. We could disable it and remember to turn it on every time we want to change VM configs, but that can be inconvenient.

At the moment, the only known way to mitigate this is to host the folders that hold the database in RAM. One way is to use something called **pmxcfs-ram**, which does something similar to **log2ram**.

https://forum.proxmox.com/threads/ssd-wearout-and-rrdcache-pmxcfs-commit-interval.124638/

https://github.com/isasmendiagus/pmxcfs-ram/tree/main

Another way is to use **folder2ram** which is a more generic way to host a high IO throughput folder in RAM and keep a synced copy in the disk. 

https://github.com/bobafetthotmail/folder2ram

Also, **log2ram** can be configured to do the same thing too! 

https://github.com/Jahfry/Miscellaneous/blob/main/proxmoxVE/03.ProxmoxTweaks.md#03ciib-moving-other-stuff-to-ram

The benefit of using **folder2ram** is the ability to customise the size for each folder. With **log2ram** it is possible to specify only 1 size for everything.

Here are the folders we want to host in RAM:
- ```/var/log```
- ```/var/lib/pve-cluster```
- ```/var/lib/pve-manager``` (not sure if we need this to add this one, need to do more research)
- ```/var/lib/rrdcached```
- ```/var/spool```

Let's configure **log2ram** by editing `/etc/log2ram.conf`
- Change `SIZE=40M` to `SIZE=400M`. 10x the original size is still only 0.3% of 128GB, to minimize risk of overflow. Put what you feel comfortable with on your system
- Change `PATH_DISK="/var/log"` to
```
PATH_DISK="/var/log;/var/lib/pve-cluster;/var/lib/rrdcached;/var/lib/pve-manager;/var/spool"
```

## smartmontools / smartd

For some reason, this preinstalled package has a service that continually writes logs to disk, but it doesn't show up in **iotop**. The service **smartd**, part of **smartmontools**, is in charge of updating SMART statuses of disks on the Web UI. Since we can always use the command line tool, we can disable the service.
```
systemctl disable smartd
```

## Enable noatime

By default, Linux (as well as MacOS) stores the access time for every file the OS reads, which results in a lot of writes. This is required for a small number of applications that we do not need. So we can disable it by editing the mount entry in `/etc/fstab`:
```
/dev/pve/root / ext4 errors=remount-ro,noatime 0 1
```
Note that I have `ext4` because that's the format I chose for the Proxmox drive, yours might differ depending on what you chose.

## Disable apt daily service and timer

I believe these services and timers are inherited from Debian, where there is a desktop environment and the user wants to see automated updates. I have observed that these writes to disk every time shortly after booting, presumably for preemptive apt cache refreshing. For Proxmox, I always manually refresh and update using the web GUI, so this is unneeded.
To disable them all:
```
systemctl disable apt-daily.service
systemctl disable apt-daily.timer
systemctl disable apt-daily-upgrade.timer
systemctl disable apt-daily-upgrade.service
```
