---
layout:     post
title:      Unix System-Level I/O
subtitle:   Buffer structure in Unix I/O System
date:       2019-12-06
author:     Hex
header-img: img/post-bg-linux.jpg
catalog:    true
tags:
    - Unix
---

> Let's get to know unix I/O design!

## Unix System-Level I/O

The "file" is widely used in linux system, and everything is regarded as"file". "Files" are a *general* OS *abstraction* for arbitrary data in linux system.   
<br> There are two general file type in unix:

- regular files
- special files

And there are two general classes of I/O devices:
- *block*: e.g., disk, memory
- *character*: e.g., network, mouse

*regular files* consist of ASCII or binary
data, stored on a block device.  
*special files* may represent directories, in-memory structures, sockets, or raw devices.


> How do we manage "file" in unix? Here comes the filesystem.

### OS's filesystem

#### Open File Description (OFD)
Each file has a unique **inode** ( aka. vnode ) data
structure in the filesystem, tracking:
- ownership & permissions
- size, type, and data location
- number of *links*

Also, each open file is tracked by the kernel using an *open file description* structure  

![OFD](https://tva1.sinaimg.cn/large/006tNbRwgy1g9nvxc8ztsj30hm06ygm1.jpg)

can have *multiple open file descriptions* referencing a *single* vnode(e.g., to track
separate read/write positions)

![multi OFD](https://tva1.sinaimg.cn/large/006tNbRwgy1g9nz1ddrkpj30ne0g2ta2.jpg)


for each process, the kernel maintains a table of pointers to its open file structures. In order to protect files, the kernel only returns *an index into the table*, and the index is called **file descriptor** (FD).   
after opening a file, **all** file operations are performed by file descriptor.

- read from FD 0 for standard input
- write to FD 1 for standard output
- write to FD 2 for standard error

![FD](https://tva1.sinaimg.cn/large/006tNbRwgy1g9nz711bbfj316i0mqada.jpg)


### Buffering

read/write are system calls used to read and write data. System call is very costly, so it's inefficient that you call read/write over and over again to read and write data from or to the same file.   
Example:

![Example](https://tva1.sinaimg.cn/large/006tNbRwgy1g9okf1q262j30y20omtbk.jpg)

one syscall per integer read = inefficient!

*But, there is a solution called **Buffering***

#### The steps of Buffering

1. read more bytes than we need into a separate *backing buffer*  

![step1](https://tva1.sinaimg.cn/large/006tNbRwgy1g9okjt89gzj30z60cgq3y.jpg)

2.
