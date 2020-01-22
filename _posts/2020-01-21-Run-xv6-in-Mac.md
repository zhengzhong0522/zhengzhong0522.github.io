---
layout:     post
title:      Run Xv6 on Mac
subtitle:   
date:       2020-01-21
author:     Sudo
header-img: img/home-bg-o.jpg
catalog:    true
tags:
    - Linux
---

# Run Xv6 on virtualbox on Mac

> xv6 is a modern reimplementation of Sixth Edition Unix in ANSI C for multiprocessor x86 and RISC-V systems. It is used for pedagogical purposes in MIT's Operating Systems Engineering course as well as Georgia Tech's Design of Operating Systems Course, IIIT Hyderabad, IIIT Delhi and as well as many other institutions

### Prepare tools
- Xv6 &nbsp; [Xv6 source code](https://github.com/mit-pdos/xv6-public)
- Virtualbox [virtualbox mac version](https://www.virtualbox.org/wiki/Download_Old_Builds_6_0). I recommend that download 6.0.0 version.
- [Vagrant](https://www.vagrantup.com/)

### Potential problems
You might find that you fail to install virtualbox on Mac. Please check these [solutions](https://medium.com/@DMeechan/fixing-the-installation-failed-virtualbox-error-on-mac-high-sierra-7c421362b5b5).


### Get started
- Project setup
```
$ mkdir xv6
$ cd xv6
$ vagrant init hashicorp/bionic64
```
- Installing a Box
```
$ vagrant box add hashicorp/bionic64
```

- Using a Box  
Open *Vagrantfile* under sample directory, and configure our project in following ways:

```
Vagrant.configure("2") do |config|
  config.vm.box = "hashicorp/bionic64"
end
```

- Up And SSH  
```
$ vagrant up
$ vagrant ssh
```

- Done!
