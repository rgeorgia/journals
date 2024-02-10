---
title: "Kodiak Jails"
date: 2022-07-23T21:12:56-04:00
draft: true
categories: [notes]
tags: [kodiak, iocage, ]
---

## iocage setup

Created two jails with iocage

```bash
❯ sudo iocage list 
+-----+-----------+-------+--------------+---------------+
| JID |   NAME    | STATE |   RELEASE    |      IP4      |
+=====+===========+=======+==============+===============+
| 19  | kodiakdb  | up    | 13.1-RELEASE | 192.168.1.187 |
+-----+-----------+-------+--------------+---------------+
| 20  | kodiakweb | up    | 13.1-RELEASE | 192.168.1.188 |
+-----+-----------+-------+--------------+---------------+

```

### Setting up the network

Checking initial VNET

```bash
ronotes on  main [?] 
❯ sudo iocage get vnet kodiakdb
0

ronotes on  main [?] 
❯ sudo iocage get vnet kodiakweb
0
```

Set IPv4 address

```bash
ronotes on  main [?] 
❯ sudo iocage set ip4_addr="em1|192.168.1.187/24" kodiakdb
ip4_addr: none -> em1|192.168.1.187/24

ronotes on  main [?] 
❯ sudo iocage set ip4_addr="em1|192.168.1.188/24" kodiakweb
ip4_addr: none -> em1|192.168.1.188/24

ronotes on  main [?] 
❯ sudo iocage start kodiakdb
* Starting kodiakdb
  + Started OK
  + Using devfs_ruleset: 1000 (iocage generated default)
  + Using IP options: ip4.addr=em1|192.168.1.187/24 ip4.saddrsel=1 ip4=new ip6.saddrsel=1 ip6=new
  + Starting services OK
  + Executing poststart OK

ronotes on  main [?] 
❯ sudo iocage start kodiakweb
* Starting kodiakweb
  + Started OK
  + Using devfs_ruleset: 1001 (iocage generated default)
  + Using IP options: ip4.addr=em1|192.168.1.188/24 ip4.saddrsel=1 ip4=new ip6.saddrsel=1 ip6=new
  + Starting services OK
  + Executing poststart OK

```

### Building

iocage console kodiakdb
1. pkg update
2. install vim, git, fish, sudo, python310
3. install postgresql - postgresql15-client,postgresql15-contrib,postgresql15-docs,postgresql15-plpython,postgresql15-server
3. updated visudo
4. create users