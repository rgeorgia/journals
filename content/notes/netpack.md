---
title: "Setup Disk for NetBSD"
date: 2021-12-30T15:02:42-05:00
draft: true
categories: [notes]
tags: [netbsd, system, desktop, bsd]
---


## Current Drive Stats

```bash
rgeorgia@netpack ~> sudo *gpt* show wd0
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
rgeorgia@netpack ~> sudo dkctl wd0 listwedges
/dev/rwd0: 2 wedges:
dk0: aefb383d-5c26-4d11-bbaa-735447bcafcb, 1936911725 blocks at 2048, type: ffs
dk1: 5721838e-34ee-4255-bc34-6eb569e5fce8, 16610703 blocks at 1936914432, type: swap
```

## Adding the new drive

```bash
rgeorgia@netpack ~> sudo gpt add -a 512k -l Packages -t ffs wd1
/dev/rwd1: Partition 1 added: 49f48d5a-b10e-11dc-b99b-0019d1879648 1024 1953523712
rgeorgia@netpack ~> sudo gpt show wd1
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
rgeorgia@netpack ~> sudo dkctl wd1 listwedges
/dev/rwd1: 1 wedge:
dk2: Packages, 1953523712 blocks at 1024, type: ffs
```

### New File System

Now create a new file system for your "partition"

```bash
rgeorgia@netpack ~> sudo newfs -O2 dk2
/dev/rdk2: 953869.0MB (1953523712 sectors) block size 32768, fragment size 4096
        using 1285 cylinder groups of 742.31MB, 23754 blks, 46848 inodes.
super-block backups (for fsck_ffs -b #) at:
192, 1520448, 3040704, 4560960, 6081216, 7601472, 9121728, 10641984, 12162240, 13682496,
.................................................................................................

rgeorgia@netpack ~> sudo dkctl wd1 listwedges
/dev/rwd1: 1 wedge:
dk2: Packages, 1953523712 blocks at 1024, type: ffs
```

### ZFS

Now let's create a zfs pool.

```bash
rgeorgia@netpack ~> zpool create storage dk2
internal error: failed to initialize ZFS library
rgeorgia@netpack ~ [1]> sudo zpool create storage dk2
rgeorgia@netpack ~> zpool list
internal error: failed to initialize ZFS library
```

### What the... ?

Oh man! I forgot, you need to perform zfs stuff as root. Make sure you are in the visudo file.

```bash
rgeorgia@netpack ~ [1]> sudo -s
Welcome to fish, the friendly interactive shell
Type `help` for instructions on how to use fish
root@netpack /h/rgeorgia# zpool list
NAME      SIZE  ALLOC   FREE  EXPANDSZ   FRAG    CAP  DEDUP  HEALTH  ALTROOT
storage   928G   336K   928G         -     0%     0%  1.00x  ONLINE  -
root@netpack /h/rgeorgia# zpool status
  pool: storage
 state: ONLINE
  scan: none requested
config:

        NAME        STATE     READ WRITE CKSUM
        storage     ONLINE       0     0     0
          dk2       ONLINE       0     0     0

errors: No known data errors

root@netpack /h/rgeorgia# zfs list
NAME      USED  AVAIL  REFER  MOUNTPOINT
storage   244K   899G    88K  /storage
```
