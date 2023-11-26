---
title: "Smartd"
date: 2021-12-30T15:03:04-05:00
draft: true
categories: [notes]
tags: [desktop, system, bsd]
---

Verify that smartmontools have been installed

To check the status of drives, use the following:

```bash

	/usr/local/sbin/smartctl -a /dev/ad0	for first ATA/SATA drive
	/usr/local/sbin/smartctl -a /dev/da0	for first SCSI drive
	/usr/local/sbin/smartctl -a /dev/ada0	for first SATA drive
```

To include drive health information in your daily status reports,
add a line like the following to /etc/periodic.conf:

```bash
	daily_status_smart_devices="/dev/ad0 /dev/da0"
```

substituting the appropriate device names for your SMART-capable disks.

To enable drive monitoring, you can use /usr/local/sbin/smartd.
A sample configuration file has been installed as

```bash
/usr/local/etc/smartd.conf.sample
```

Copy this file to /usr/local/etc/smartd.conf and edit appropriately
