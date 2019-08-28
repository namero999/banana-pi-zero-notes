These notes apply to this custom image built by avafinger https://github.com/avafinger/bananapi-zero-ubuntu-base-minimal/releases/tag/v15

## Keyboard

By default, the keyboard layout is brazilian. To change layout, edit `/etc/default/keyboard`

```
$ sudo localectl set-keymap us
```

<br/>

## Wireless

Configure your network by editing `/etc/network/interfaces`

```
wpa-ssid your-network-id
wpa-psk your-network-key
```

##### Speed problems?
If you experience painfully slow network speed, your board might have decided to power-throttle the network device.
Disable this behaviour by adding the following line

`wireless-mode managed`

<br/>

## Expand root partition

The image is trimmed down to fit in 8GB, but if you flash it onto a bigger SD, you may want to expand your partition
to make use of the unused space.

This is the situation after first boot (only 7.1GB used)

```
ubuntu@bpi-m2z:~$ df -h

Filesystem      Size  Used Avail Use% Mounted on
/dev/mmcblk0p2  7.1G  722M  6.0G  11% /
devtmpfs        119M     0  119M   0% /dev
tmpfs           248M     0  248M   0% /dev/shm
tmpfs           248M  7.7M  240M   4% /run
tmpfs           5.0M  4.0K  5.0M   1% /run/lock
tmpfs           248M     0  248M   0% /sys/fs/cgroup
/dev/mmcblk0p1   93M  6.5M   80M   8% /boot
```

We'll use fdisk to resize the partition

`$ sudo fdisk /dev/mmcblk0`

* List partitions with `p` and save the start sector of root partition

  ```
  ubuntu@bpi-m2z:~$ sudo fdisk /dev/mmcblk0

  Welcome to fdisk (util-linux 2.27.1).
  Changes will remain in memory only, until you decide to write them.
  Be careful before using the write command.

  Command (m for help): p
  Disk /dev/mmcblk0: 29.7 GiB, 31914983424 bytes, 62333952 sectors
  Units: sectors of 1 * 512 = 512 bytes
  Sector size (logical/physical): 512 bytes / 512 bytes
  I/O size (minimum/optimal): 512 bytes / 512 bytes
  Disklabel type: dos
  Disk identifier: 0x9ebb22cd

  Device         Boot  Start      End  Sectors  Size Id Type
  /dev/mmcblk0p1       49152   253951   204800  100M 83 Linux
  /dev/mmcblk0p2      253952 15605759 15351808  7.3G 83 Linux
  ```

  In this case, the number we are interested in, is `253952`

* Delete current partition: `d` to delete, and when prompted, choose `2`

  ```
  Command (m for help): d
  Partition number (1,2, default 2): 2

  Partition 2 has been deleted.
  ```

* Create a new partition with `n`

  ```
  Command (m for help): n
  Partition type
     p   primary (1 primary, 0 extended, 3 free)
     e   extended (container for logical partitions)
  Select (default p): p
  Partition number (2-4, default 2): 2
  First sector (2048-62333951, default 2048): 253952
  Last sector, +sectors or +size{K,M,G,T,P} (253952-62333951, default 62333951):

  Created a new partition 2 of type 'Linux' and of size 29.6 GiB.
  ```

* Write changes with `w`

  ```
  Command (m for help): w
  The partition table has been altered.
  Calling ioctl() to re-read partition table.
  Re-reading the partition table failed.: Device or resource busy

  The kernel still uses the old table. The new table will be used at the next reboot or after you run partprobe(8) or kpartx(8).
  ```

* Reboot the machine

  ```
  ubuntu@bpi-m2z:~$ sudo resize2fs /dev/mmcblk0p2

  resize2fs 1.42.13 (17-May-2015)
  Filesystem at /dev/mmcblk0p2 is mounted on /; on-line resizing required
  old_desc_blocks = 1, new_desc_blocks = 2
  The filesystem on /dev/mmcblk0p2 is now 7760000 (4k) blocks long.

  ubuntu@bpi-m2z:~$ df -h

  Filesystem      Size  Used Avail Use% Mounted on
  /dev/mmcblk0p2   30G  726M   28G   3% /
  devtmpfs        119M     0  119M   0% /dev
  tmpfs           248M     0  248M   0% /dev/shm
  tmpfs           248M  4.6M  244M   2% /run
  tmpfs           5.0M  4.0K  5.0M   1% /run/lock
  tmpfs           248M     0  248M   0% /sys/fs/cgroup
  /dev/mmcblk0p1   93M  6.5M   80M   8% /boot
  ```

<br/>

## Disable MOTD

Make logins slightly faster by disabling MOTD

`$ touch ~/.hushlogin`

<br/>