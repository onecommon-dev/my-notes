[Back to Proxmox](README.md)

# Mounting existing Proxmox disks to Storage
This is for when you need to reinstall Proxmox, maybe your Proxmox boot disk is broken, or the installation got botched somehow. You need to mount your existing backup & VM disks to continue using Proxmox as normal, here's how.

First, identify the path of the disks that you need to mount with the `lsblk` command. Use `lsblk -fs` to see the UUIDs of the disks. Take note of them, they will look something like `/dev/sda1` or `/dev/sdb2` or similar.

To temporarily mount them, use these commands
```
mkdir /mnt/oldbackups
mount /dev/sdb1 /mnt/oldbackups
```
This will temporarily mount the disk `/dev/sdb1` at the path `mnt/oldbackups`

If the disk is a backup disk, you can temporarily add that disk to Proxmox with
```
pvesm add dir oldbackups --path /mnt/oldbackups --content backup
```

To permanently mount them, we need to add an entry to `/etc/fstab`, using the disk's UUID:
```
echo 'UUID=d8871cd7-11b1-4f75-8cb6-254a6120 /mnt/oldbackups    ext4    defaults 0   2' >> /etc/fstab
```

If you want to transfer dumps from "local" to a new disk, they are stored in `/var/lib/vz/dump`
