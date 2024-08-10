---
title: "Setup Disk for NetBSD"
date: 2021-12-30T15:02:42-05:00
draft: true
categories: [notes]
tags: [netbsd, system, desktop, bsd]
---


## Current Drive Stats

To find out which drives have been discovered use the sysctl tool.


```bash
$ sysctl hw.disknames               
hw.disknames = wd0 dk0 dk1 dk2 wd1 wd2
```

## Show some drive stats with gpt, not gptchat

```bash
$ sudo *gpt* show wd0
       start        size  index  contents
           0           1         PMBR
           1           1         Pri GPT header
           2          32         Pri GPT table
          34        2014         Unused
        2048  1936911725      1  GPT part - NetBSD FFSv1/FFSv2
  1936913773         659         Unused
  1936914432    16610703      2  GPT part - NetBSD swap
  1953525135          32         Sec GPT table
  1953525167           1         Sec GPT header
```

## List the wedges

```bash
$ sudo dkctl wd0 listwedges
/dev/rwd0: 2 wedges:
dk0: aefb383d-5c26-4d11-bbaa-735447bcafcb, 1936911725 blocks at 2048, type: ffs
dk1: 5721838e-34ee-4255-bc34-6eb569e5fce8, 16610703 blocks at 1936914432, type: swap
```

## Adding the new drive

The two drives I have are from a previous system. Their data looks like this. Both disk look the same, but only one is depicted.

```bash
sudo gpt show wd1
GPT not found, displaying data from MBR.

  start        size  index  contents
  0  1953525135         Unused
  1953525135          32         Sec GPT table
  1953525167           1         Sec GPT header

```
I really don't care what was on the disk, so I destroy the gpt partion on both disks.

```bash

sudo gpt destroy wd1
$ sudo gpt destroy wd2
$ sudo gpt show wd1  
GPT not found, displaying data from MBR.

       start        size  index  contents
           0  1953525168         Unused
```

## Create a new gpt partition

This is done with the `sudo gpt create` command.

```bash

$ sudo gpt create wd1

$ sudo gpt show wd1

       start        size  index  contents
           0           1         PMBR
           1           1         Pri GPT header
           2          32         Pri GPT table
          34  1953525101         Unused
  1953525135          32         Sec GPT table
  1953525167           1         Sec GPT header
```


```bash
$ sudo gpt add -a 512k -l Packages -t ffs wd1
/dev/rwd1: Partition 1 added: 49f48d5a-b10e-11dc-b99b-0019d1879648 1024 1953523712
$ sudo gpt show wd1
       start        size  index  contents
           0           1         PMBR
           1           1         Pri GPT header
           2          32         Pri GPT table
          34         990         Unused
        1024  1953523712      1  GPT part - NetBSD FFSv1/FFSv2
  1953524736         399         Unused
  1953525135          32         Sec GPT table
  1953525167           1         Sec GPT header
```

Now list out the wedges to verify you will mount the correct portion of your new disk.

```bash
$ sudo dkctl wd1 listwedges
/dev/rwd1: 1 wedge:
dk2: Packages, 1953523712 blocks at 1024, type: ffs
```

### New File System

Now create a new file system for your "partition"

```bash
$ sudo newfs -O2 dk2
/dev/rdk2: 953869.0MB (1953523712 sectors) block size 32768, fragment size 4096
        using 1285 cylinder groups of 742.31MB, 23754 blks, 46848 inodes.
super-block backups (for fsck_ffs -b #) at:
192, 1520448, 3040704, 4560960, 6081216, 7601472, 9121728, 10641984, 12162240, 13682496,
.................................................................................................

$ sudo dkctl wd1 listwedges
/dev/rwd1: 1 wedge:
dk2: Packages, 1953523712 blocks at 1024, type: ffs
```

### ZFS

Now let's create a zfs pool.

```bash
$ sudo zpool create storage mirror dk3 dk4 
internal error: failed to initialize ZFS library
$ sudo zpool create storage dk3
$ zpool list
internal error: failed to initialize ZFS library
```

### What the... ?

Oh man! I forgot, you need to perform zfs stuff as root. Make sure you are in the visudo file.

```bash
$ sudo -s
Welcome to fish, the friendly interactive shell
Type `help` for instructions on how to use fish
# zpool list
NAME      SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
storage   928G   336K   928G         -     0%     0%  1.00x  ONLINE  -

sudo zpool status                                                Fri Aug  9 20:29:24 2024
  pool: storage
 state: ONLINE
  scan: none requested
config:

	NAME        STATE     READ WRITE CKSUM
	storage     ONLINE       0     0     0
	  mirror-0  ONLINE       0     0     0
	    dk3     ONLINE       0     0     0
	    dk4     ONLINE       0     0     0

errors: No known data errors

# zfs list
NAME      USED  AVAIL  REFER  MOUNTPOINT
storage   244K   899G    88K  /storage
```
