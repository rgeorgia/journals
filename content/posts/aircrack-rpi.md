---
title: 'Aircrack Rpi'
date: 2025-01-05T18:08:30-05:00
draft: true
categories: ["posts"]
tags: ["infosec", "hacking", "aircrack"]
slug: "hacking"
---

ref [aircrack-ng](https://www.aircrack-ng.org/doku.php?id=install_aircrack)

## Packages to Install

- gmake
- pkgconf
- pcre
- sqlite3
- gcc9 (or better)
- openssl 
- libtool 
- automake 
- autoconf 
- cmocka

Note: sqlite3 comes with base

```bash
Using built-in specs.
COLLECT_GCC=gcc
COLLECT_LTO_WRAPPER=/usr/libexec/lto-wrapper
Target: aarch64--netbsd
Configured with: /usr/src/tools/gcc/../../external/gpl3/gcc/dist/configure --target=aarch64--netbsd --enable-long-long --enable-threads --with-bugurl=http://www.NetBSD.org/support/send-pr.html --with-pkgversion='NetBSD nb2 20230710' --with-system-zlib --without-isl --enable-__cxa_atexit --enable-libstdcxx-time=rt --enable-libstdcxx-threads --with-diagnostics-color=auto-if-env --enable-fix-cortex-a53-835769 --enable-fix-cortex-a53-843419 --with-default-libstdcxx-abi=new --with-mpc-lib=/var/obj/mknative/evbarm-aarch64/usr/src/external/lgpl3/mpc/lib/libmpc --with-mpfr-lib=/var/obj/mknative/evbarm-aarch64/usr/src/external/lgpl3/mpfr/lib/libmpfr --with-gmp-lib=/var/obj/mknative/evbarm-aarch64/usr/src/external/lgpl3/gmp/lib/libgmp --with-mpc-include=/usr/src/external/lgpl3/mpc/dist/src --with-mpfr-include=/usr/src/external/lgpl3/mpfr/dist/src --with-gmp-include=/usr/src/external/lgpl3/gmp/lib/libgmp/arch/aarch64 --enable-tls --disable-multilib --disable-libstdcxx-pch --build=aarch64--netbsd --host=aarch64--netbsd --with-sysroot=/var/obj/mknative/evbarm-aarch64/usr/src/destdir.evbarm
Thread model: posix
Supported LTO compression algorithms: zlib
gcc version 10.5.0 (nb3 20231008)
```

## Optional stuff
* If you want SSID filtering with regular expression in airodump-ng (-essid-regex) pcre development package is required.
* If you want to use airolib-ng and '-r' option in aircrack-ng, SQLite development package >= 3.3.17 (3.6.X version or better is recommended)
* If you want to use Airpcap, the 'developer' directory from the CD is required. It can be downloaded here.
* For best performance on FreeBSD (50-70% more), install gcc5 via: pkg install gcc5 Then compile with: 
```bash
gmake CC=gcc5 CXX=g++5
```

* rfkill
* CMocka
* hwloc: strongly recommended, especially on high core count systems where it may give a serious performance boost


## Compiling and installing
**Notes**:

On OS X, BSD and Solaris, use '**gmake**' instead of 'make'.

In order to compile with clang instead of gcc, add 'CC=clang CXX=clang++' to the configure command.

### Current version

```
wget https://download.aircrack-ng.org/aircrack-ng-1.7.tar.gz
tar -zxvf aircrack-ng-1.7.tar.gz
cd aircrack-ng-1.7
autoreconf -i
./configure --with-experimental
make
make install
ldconfig
```

### Compiling on BSD

Commands are exactly the same as Linux but instead of make, use gmake (with CC=gcc5 CXX=g++5 or any more recent gcc version installed).

### Latest Git (development) Sources
Note: Compilation parameters can also be used with the sources from our git repository.

```bash
git clone https://github.com/aircrack-ng/aircrack-ng
cd aircrack-ng
autoreconf -i
./configure --with-experimental
make
make install
ldconfig
```

## Start Build and install

- ./autoget.sh
- ./configure
- gmake
- gmake install (sudo?)
