---
title: "Configure New Drive Dfly"
date: 2021-12-21T15:52:40-05:00
draft: false
categories: [notes]
tags: [dfly, system, hardware]
---

# Add a new disk

#### References:
[Adding a Disk](https://www.dragonflybsd.org/docs/handbook/UnixBasics/#index22h3) from the Unix Basic section of the DragonflyBSD site.

```bash
sudo gpt destroy da8

rgeorgia@mini01 ~> sudo gpt show da8
       start        size  index  contents
           0           1      -  MBR
           1          62      -  Unused
          63  1953520065      0  MBR part 108 (active)
  1953520128        5040      -  Unused

rgeorgia@mini01 ~> sudo gpt create /dev/da8

rgeorgia@mini01 ~> sudo gpt show da8
       start        size  index  contents
           0           1      -  PMBR
           1           1      -  Pri GPT header
           2          32      -  Pri GPT table
          34  1953525101      -  Unused
  1953525135          32      -  Sec GPT table
  1953525167           1      -  Sec GPT header

rgeorgia@mini01 ~> sudo fdisk -I /dev/da8
******* Working on device /dev/da8 *******
Warning: ending cylinder wraps, using all 1's

rgeorgia@mini01 ~> sudo disklabel64 -w -r /dev/da8s1 auto


rgeorgia@mini01 ~> sudo disklabel64 -e /dev/da8s1
diskid: db35479c-0da4-11eb-b62e-41623103e5ba
label:
boot2 data base:      0x000000001000
partitions data base: 0x0000000f8200
partitions data stop: 0x00e8e0af8200
backup label:         0x00e8e0b37200
total size:           0x00e8e0b38200    # 953867.22 MB
alignment: 4096
display block size: 1024        # for partition display and edit only

16 partitions:
#          size     offset    fstype   fsuuid
# EXAMPLE
e:          *           *    HAMMER2


rgeorgia@mini01 ~ [1]> sudo newfs_hammer2 /dev/da8s1e
Volume /dev/da8s1e     size   0.91TB
---------------------------------------------
version:          1
total-size:         0.91TB (1000198897664 bytes)
boot-area-size:    64.00MB
aux-area-size:    256.00MB
topo-reserved:      3.64GB
free-space:         0.91TB
vol-fsid:         4b02424b-0da5-11eb-b62e-41623103e5ba
sup-clid:         4b02425f-0da5-11eb-b62e-41623103e5ba
sup-fsid:         4b02426c-0da5-11eb-b62e-41623103e5ba
PFS "LOCAL"
    clid 4d769bb1-0da5-11eb-b62e-41623103e5ba
    fsid 4d769bce-0da5-11eb-b62e-41623103e5ba
PFS "DATA"
    clid 4d769c78-0da5-11eb-b62e-41623103e5ba
    fsid 4d769c8c-0da5-11eb-b62e-41623103e5ba

rgeorgia@mini01 ~> sudo mount -t hammer2 /dev/da8s1e /mnt/data


rgeorgia@mini01 ~> mount
serno/50026B726B053990.s1d on / (hammer2, local)
devfs on /dev (devfs, nosymfollow, local)
/dev/serno/50026B726B053990.s1a on /boot (ufs, local)
/dev/serno/50026B726B053990.s1e@DATA on /build (hammer2, local)
/build/usr.obj on /usr/obj (null)
/build/var.crash on /var/crash (null)
/build/var.cache on /var/cache (null)
/build/var.spool on /var/spool (null)
/build/var.log on /var/log (null)
/build/var.tmp on /var/tmp (null)
tmpfs on /tmp (tmpfs, local)
procfs on /proc (procfs, local)
tmpfs on /var/run/shm (tmpfs, local)
tmpfs on /var/run/user/164 (tmpfs, mounted by lightdm, local)
tmpfs on /var/run/user/1001 (tmpfs, mounted by rgeorgia, local)
tmpfs on /var/run/user/164 (tmpfs, mounted by lightdm, local)
tmpfs on /var/run/user/1001 (tmpfs, mounted by rgeorgia, local)
tmpfs on /var/run/user/164 (tmpfs, mounted by lightdm, local)
tmpfs on /var/run/user/1001 (tmpfs, mounted by rgeorgia, local)
/dev/da8s1e@DATA on /mnt/data (hammer2, local)
```
