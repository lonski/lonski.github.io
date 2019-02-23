---
layout: post
title: Unbootable RAID1 array
subtitle: Recovery guide
date: 2018-07-31
categories: [linux]
---
I failed to setup bootable software RAID1 array on my debian server. The array itself worked ok - I successfuly created working and synchronized two-disk array. I updated grub configuration, rebooted, and bah:

```
Error: unknown filesystem.
grub rescue>
``` 

Any attempt to `ls` or `insmod` ended up with `Error: unknown filesystem`. 

# Take step back

After some attempts to make it work, I decided that I want to take step back, remove raid and get back to single disk solution. I booted linux mint live cd. 

In order to do any raid-related actions `mdadm` is needed:

```sh
apt-get install mdadm
```

I was able to assemble and mount the array, and browse the files:

```sh
mdadm --assemble /dev/md0 /dev/sda1 /dev/sdb1
```

Where `sda1` and `sdb1` are the two raided partitions.

I could mount the array and check that it works, and contains all the files:

```sh
mkdir /mnt/raid
mount /dev/md0 /mnt/raid
cat /proc/mdstat
```

## Removing one drive from the array

However, what I need was remove one disk from the array, and restore it to old setup. To remove a disk from array:

```sh
mdadm /dev/md0 --fail /dev/sda1
mdadm /dev/md0 --remove /dev/sda1
```

Then I changed its partition type to `Linux`:
```sh 
fdisk /dev/sda
```
Then in fdisk shell hit `t` (for change type), `1` (for partition number), `83` (for `Linux` partition type) and in the end `w` for write the changes.

I tried to mount it..
```sh
mkdir /mnt/noraid
mount /dev/sda1 /mnt/noraid
```
But failed:
```sh
mount: unknown filesystem type 'linux_raid_member' 
```

Tried to explicity set type to `ext4`:
```sh
mount -t ext4 /dev/sda1 /mnt/noraid
```

But got:
```sh
mount: mount /dev/sda1 on /mnt/noraid failed: Structure needs cleaning
```

I run fsck to clean it up:
```sh
fsck.ext4 /dev/sda1
```

It found several errors, and warned me that if I will continue to fix it, then several data loss is possible. It wiped all the data, which was not a problem since there was still second disk in the array.

Afterwords I had clean, ext4 partition that could be mounted:
```sh
mount /dev/sda1 /mnt/noraid
```

## Copy system from raided disk to standard Linux ext4 partition

Next step was restoring the contents of `sda1`. I assembled and mounted the degradated array:
```sh
mdadm --assemble /dev/md0 /dev/sdb1
mkdir /mnt/raid
mount /dev/md0 /mnt/raid
```

And copied all the files to clean `sda1` disk:

```sh
cp -dpRx /mnt/raid/* /mnt/noraid
```

## Swap partition

I had also swap partition raided, partitions `/dev/sda2` and `/dev/sdb2`. I removed `sda2` from the array in the same way as `sda1`, changed its partition type to `Linux`, and make it swap:

```sh
mkswap /dev/sda2
```

## Install grub 

To install the boot loader I needed to chroot to the `sda1`:

```sh
mount -t proc /proc /mnt/noraid/proc
mount --rbind /sys /mnt/noraid/sys
mount --make-rslave /mnt/noraid/sys
mount --rbind /dev /mnt/noraid/dev
mount --make-rslave /mnt/noraid/dev
chroot /mnt/noraid /bin/bash
```

Then I reconfigured the grub:

```sh
dpkg-reconfigure grub-pc
```

And choosed `/dev/sda` to install the bootloader.

Also I had to edit `/etc/fstab` and change all `mdX` entires to point to `sdaX`, so it looked like this:
```sh
/dev/sda1   /               ext4    errors=remount-ro           0   1
/dev/sda2   none            swap    sw                          0   0
```

Then I rebooted. Grub showed up, I choosed system to run.. and black screen. The system did not start. Ok, so again, booted live cd, `chroot` into `sda1`. I thought that maybe initramfs needs update:

```sh
update-initramfs -u -k `uname -r`
```

Got warnings:
```sh
W: initramfs-tools configuration sets RESUME=UUID=xxxxxxxxxxxxxxxxxxxxx
W: but no matching swap device is available.
```

I had to remove the file with old swap uuid:

```sh
rm /etc/initramfs-tools/conf.d/resume
```

And update initramfs again. Rebooted, and it worked. Debian started without problems.
