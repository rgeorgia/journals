---
title: "New Disk Notes - DFLYBSD"
date: 2021-12-30T15:13:03-05:00
draft: false
categories: [notes]
tags: [dfly, system, hardware]
---


Just notes on probing and finding disk drives attached to the system

- [Using dmesg](#dmesg)
- [Using sysctl](#sysctl)
- [Using camcontrol](#camcontrol)
- [Using gpt](#gpt)
- [Listing /dev](#list-dev)
- [Creating gpt partition](#create-gpt)
- [fdisk](#fdisk)
- [Labeling the disk](#disklabel64)
- [Creating new hammer2 fs](#newfs)
- [Mounting the disk](#mounting)
- [Setting up fstab](#fstab)
- [Using your disks serial number](#serial-number)

#### Using dmesg to find your disks {#dmesg}

You can use dmesg to find a whole

```bash
$ dmesg | grep -i disk
md0: Malloc disk
ahci0.1: Found DISK "ST2000DM006-2DM164 CC26" serial="Z4ZA96FR"
ahci0.2: Found DISK "ST2000DM006-2DM164 CC26" serial="Z4ZA96DM"
md0: Malloc disk

$ dmesg | egrep "ST2000"
ahci0.1: Found DISK "ST2000DM006-2DM164 CC26" serial="Z4ZA96FR"
ahci0.2: Found DISK "ST2000DM006-2DM164 CC26" serial="Z4ZA96DM"
da0: <SATA ST2000DM006-2DM1 CC26> Fixed Direct Access SCSI-4 device 
da1: <SATA ST2000DM006-2DM1 CC26> Fixed Direct Access SCSI-4 device 

$ dmesg | egrep "^da"
da0 at ahci0 bus 1 target 0 lun 0
da0: <SATA ST2000DM006-2DM1 CC26> Fixed Direct Access SCSI-4 device 
da0: Serial Number Z4ZA96FR
da0: 600.000MB/s transfers
da0: 1907729MB (3907029168 512 byte sectors: 255H 63S/T 243201C)
da1 at ahci0 bus 2 target 0 lun 0
da1: <SATA ST2000DM006-2DM1 CC26> Fixed Direct Access SCSI-4 device 
da1: Serial Number Z4ZA96DM
da1: 600.000MB/s transfers
da1: 1907729MB (3907029168 512 byte sectors: 255H 63S/T 243201C)
```

#### Using sysctl {#sysctl}


```bash
# sysctl kern.disks
kern.disks: da1 da0 vn3 vn2 vn1 vn0 md0
```

#### Using camcontrol {#camcontrol}

```bash
$ sudo camcontrol devlist 
<SATA ST2000DM006-2DM1 CC26>       at scbus1 target 0 lun 0 (sg0,pass0,da0)
<SATA ST2000DM006-2DM1 CC26>       at scbus2 target 0 lun 0 (sg1,pass1,da1)
```

#### gpt {#gpt}

Finding out which disk is the boot disk

```bash
# gpt show da0
     start        size  index  contents
         0           1      -  PMBR
         1           1      -  Pri GPT header
         2          32      -  Pri GPT table
        34        2014      -  Unused
      2048      262144      0  GPT part - EFI System
    264192  3906764800      1  GPT part - DragonFly Label64
3907028992         143      -  Unused
3907029135          32      -  Sec GPT table
3907029167           1      -  Sec GPT header

# gpt show da1
       start        size  index  contents
         0           1      -  PMBR
         1           1      -  Pri GPT header
         2          32      -  Pri GPT table
        34        2014      -  Unused
      2048  3907026944      0  GPT part - MS/Linux Basic Data
3907028992         143      -  Unused
3907029135          32      -  Sec GPT table
3907029167           1      -  Sec GPT header
```

##### Listing the /dev directory to pick out the slices {#listdev}

```bash
$ ll /dev | grep da
crw-r-----  1 root      operator   29, 0x1e110007 15-Dec-2020 13:29 da0
crw-r-----  1 root      operator   29, 0x1e100007 15-Dec-2020 13:29 da0s0
crw-r-----  1 root      operator   29, 0x1e120007 15-Dec-2020 13:29 da0s1
crw-r-----  1 root      operator   29, 0x00020000 15-Dec-2020 13:29 da0s1a
crw-r-----  1 root      operator   29, 0x00020001 15-Dec-2020 13:29 da0s1b
crw-r-----  1 root      operator   29, 0x00020003 15-Dec-2020 13:29 da0s1d
crw-r-----  1 root      operator   29, 0x00020004 15-Dec-2020 13:29 da0s1e
crw-r-----  1 root      operator   29, 0x1e11000f 15-Dec-2020 13:29 da1
crw-r-----  1 root      operator   29, 0x1e10000f 15-Dec-2020 13:29 da1s0
```

#### Creating a gpt parition {#create-gpt}

```bash
$ sudo gpt destroy da1
$ sudo gpt show da1
start        size  index  contents
    0           1      -  PMBR
    1  3907029167      -  Unused

$ sudo gpt create da1
$ sudo gpt show da1
       start        size  index  contents
           0           1      -  PMBR
           1           1      -  Pri GPT header
           2          32      -  Pri GPT table
          34  3907029101      -  Unused
  3907029135          32      -  Sec GPT table
  3907029167           1      -  Sec GPT header
```

Initialize the disk

**Note**: from the fdisk man page

```
-B      Reinitialize the boot code contained in sector 0 of the disk.
        Ignored if -f is given.
	
-I      Initialize the contents of sector 0 for one DragonFly slice
        covering the entire disk.
```

#### Using fdisk {#fdisk}


```bash
$ sudo fdisk -IB /dev/da1

******* Working on device /dev/da1 *******
Warning: ending cylinder wraps, using all 1's
```

#### disklabel64 {#disklabel64}

- creating a disk label

```bash
$ sudo disklabel64 -w -r /dev/da1s1 auto
```

- Editing the label.

By convention, dfly partitions are setup as follows. (ref [Disk layout](https://www.dragonflybsd.org/docs/handbook/environmentquickstart/#index3h2))

```
a - for /boot
b - for swap
d - for /, a HAMMER or HAMMER2 filesystem labeled ROOT
e - for /build, a HAMMER or HAMMER2 filesystem labeled (typically) DATA
```

```bash
$ sudo disklabel64 -e /dev/da1s1

diskid: cdf2ad26-4172-11eb-99ff-6905ca1b15f8
label:
boot2 data base:      0x000000001000
partitions data base: 0x0000000f8200
partitions data stop: 0x01d1c0df8200
backup label:         0x01d1c0e8f400
total size:           0x01d1c0e90400    # 1907726.56 MB
alignment: 4096
display block size: 1024        # for partition display and edit only

16 partitions:
#          size     offset    fstype   fsuuid
# EXAMPLE
e:          *           *    HAMMER2
```

#### Create a new hammer2 file systerm {#newfs}

```bash
$ sudo newfs_hammer2 /dev/da1s1e
Volume /dev/da1s1e     size   1.82TB
---------------------------------------------
version:          1
total-size:         1.82TB (2000389406720 bytes)
boot-area-size:    64.00MB
aux-area-size:    256.00MB
topo-reserved:      7.28GB
free-space:         1.81TB
vol-fsid:         46d208a2-4173-11eb-99ff-6905ca1b15f8
sup-clid:         46d208aa-4173-11eb-99ff-6905ca1b15f8
sup-fsid:         46d208b0-4173-11eb-99ff-6905ca1b15f8
PFS "LOCAL"
    clid 4853fa16-4173-11eb-99ff-6905ca1b15f8
    fsid 4853fa1d-4173-11eb-99ff-6905ca1b15f8
PFS "DATA"
    clid 4853fa64-4173-11eb-99ff-6905ca1b15f8
    fsid 4853fa6a-4173-11eb-99ff-6905ca1b15f8
```

#### Mount that baby! {#mounting}

```bash
$ sudo mkdir /usr/storage
$ sudo mount -t hammer2 /dev/da1s1e /usr/storage
```

```bash
$ mount

serno/Z4ZA96FR.s1d on / (hammer2, local)
devfs on /dev (devfs, nosymfollow, local)
/dev/serno/Z4ZA96FR.s1a on /boot (ufs, local)
/dev/serno/Z4ZA96FR.s1e@DATA on /build (hammer2, local)
/build/usr.obj on /usr/obj (null)
/build/var.crash on /var/crash (null)
/build/var.cache on /var/cache (null)
/build/var.spool on /var/spool (null)
/build/var.log on /var/log (null)
/build/var.tmp on /var/tmp (null)
tmpfs on /tmp (tmpfs, local)
procfs on /proc (procfs, local)
tmpfs on /var/run/shm (tmpfs, local)
/dev/da1s1e@DATA on /usr/storage (hammer2, local)
```

#### Setting up fstab {#fstab}

```bash
$ cat /etc/fstab
# Device		Mountpoint	FStype	Options		Dump	Pass#
/dev/serno/Z4ZA96FR.s1a		/boot		ufs	rw		2	2
/dev/serno/Z4ZA96FR.s1b		none		swap	sw		0	0
/dev/serno/Z4ZA96FR.s1d		/		hammer2	rw		1	1
/dev/serno/Z4ZA96FR.s1e		/build		hammer2	rw		2	2
<snip>
```

#### Using the disk serial number {#serial-number}
Note the serial number for the system disk.

```bash
$ ls /dev/serno/ | tr " " "\n"
Z4ZA96DM
Z4ZA96DM.s1
Z4ZA96DM.s1e
Z4ZA96FR
Z4ZA96FR.s0
Z4ZA96FR.s1
Z4ZA96FR.s1a
Z4ZA96FR.s1b
Z4ZA96FR.s1d
Z4ZA96FR.s1e
```

Add your disk using the serial number:

```bash
$ sudo vim /etc/fstab

# Device                Mountpoint      FStype  Options         Dump    Pass#
/dev/serno/Z4ZA96FR.s1a         /boot           ufs     rw              2       2
/dev/serno/Z4ZA96FR.s1b         none            swap    sw              0       0
/dev/serno/Z4ZA96FR.s1d         /               hammer2 rw              1       1
/dev/serno/Z4ZA96FR.s1e         /build          hammer2 rw              2       2
/dev/serno/Z4ZA96DM.s1e         /usr/storage    hammer2 rw              1       1
```
