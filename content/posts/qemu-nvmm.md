---
title: 'Qemu Nvmm'
date: 2023-11-23T19:46:27-05:00
draft: true
tags: ["qemu", "nvmm"]
slug: "qemu-netbsd"
---

# Creating virtual containers with NetBSD

## Summary

I want to create a desktop installer for NetBSD. One that starts from scratch and installs and starts services, setups up user environments (i.e. shell, .xinitrc, etc). To help "test" the process I need a way to create a new "blank" system, run the install and test the results. 

## Requirements

* A script that creates a _blank_ NetBSD container.
* Containers need
  * CPU
  * Ram
  * Storage
  * Graphics
  * Networking
* Copy and rename vanilla image for testing

## Steps

1. Create an image
2. Choose NetBSD arch iso
3. Install and configure

Make sure you've reviewed this page. It will help you setup your system for Qemu with nvmm acceleration.
[Chapter 30. Using virtualization: QEMU and NVMM](https://www.netbsd.org/docs/guide/en/chap-virt.html)


