---
layout:     post
title:      Unix System-Level I/O (1)
subtitle:   
date:       2019-12-04
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

2. avoid system calls and process future
“reads” from that buffer  

![step2](https://tva1.sinaimg.cn/large/006tNbRwgy1g9okpj8682j30tc0i4jsp.jpg)


### fread/fwrite warped by buffering

To achieve the buffering, C library has already provided us with warped function like *fread*, *fwrite*, *printf* and so on.

Let's see how they work, although their performance might be a little weird for starter.

```C
int main() {
  printf("h");
  printf("e");
  printf("l");
  printf("l");
  printf("o");
  fork();
}
```
*fork* create a child process and the child process inherits data, files, signals and so on from its parent process. In this example, we can see there is nothing the child process can do after it is created, since there is no more instruction after *fork* a child process. So most people might guess the output should be ```hello```.  
But...
```
$ ./.a.out
hellohello
```
🤬  
Yes, it **is** the output. I was shocked when I first time saw this. And I think we should blame on "buffering" that it gives us the weird output. LOL

Here is a property of stream buffer:

- stream buffer can *absorb* multiple writes
before being flushed to underlying file

and flush happens on:
- buffer being filled
- (normal) process termination
- newline, in a line-buffered stream
- explicitly, with fflush

Among the above example, there is no newline(/n) after character each time we print it. Of course, the buffer is more larger than we think, so it is not filled. Also, no fflush called. Therefore, the stream would not be "print" on screen until the child process terminated and reaped.   
Each time, we call *printf*, the character is push into a stream buffer, before forking a child, the buffer contains "hello". As I mentioned before, the child process will inherit data, variable, file from parent process, so the buffer contains "hello" is brought to the child process.
There is no remaining to do in the child process, so it is terminated. Then the stream buffer in the child process is fflushed(printed). After child terminates, its parent terminates as well and buffer is fflushed. As a result, the output is gonna be **hellohello**.

#### Things get more confusing when we perform both Input/Output....

I will discuss it in my next blog.


<br>


#### References:
Michael Saelee: Input/Output  <https://moss.cs.iit.edu/cs351/slides/slides-io.pdf>

Randal Bryant: Computer Systems: A Programmer's Perspective
