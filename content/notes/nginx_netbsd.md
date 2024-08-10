---
title: "Nginx NetBSD"
date: 2022-09-24T16:15:40-04:00
draft: true
categories: [notes]
tags: [netbsd, nginx]
---

## Getting nginx on NetBSD up and running... kind of

Make the directory where all your http happiness will live, and make the owner nginx

```bash
% sudo mkdir /var/www/ronverbs.dev/
% sudo chown -R nginx:nginx /var/www
```

Now create your directories and update the ownership

```bash
var/www/ronverbs.dev % ll
total 48
drwxr-xr-x  6 nginx  nginx  512 Sep 24 12:13 ./
drwxr-xr-x  3 nginx  nginx  512 Sep 18 09:37 ../
drwxr-xr-x  2 nginx  nginx  512 Sep 24 12:13 biblenotes/
drwxr-xr-x  2 nginx  nginx  512 Sep 22 15:54 html/
drwxr-xr-x  2 nginx  nginx  512 Sep 24 12:12 prayerlist/
drwxr-xr-x  2 nginx  nginx  512 Sep 24 12:12 ronotes/
```

Now I go to my hugo projects, `workspace/hugo_projects/prayerlist` and enter the command `hugo`. This builds a new _public_ directory. Now copy the files in the new public directory to /var/www/ronverbs.dev/prayerlist.

```bash
sudo cp -r public/* /var/www/ronverbs.dev/prayerlist/.
```

Now, make sure nginx=YES in /etc/rc.conf
Start the service.

```bash
sudo nginx -t
sudo service nginx start

```
