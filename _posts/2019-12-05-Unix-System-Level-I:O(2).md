---
layout:     post
title:      Unix System-Level I/O (2)
subtitle:   
date:       2019-12-05
author:     Hex
header-img: img/post-bg-linux.jpg
catalog:    true
tags:
    - Unix
---

# Buffering in fread/fwrite
In last blog, we simply talked about how Input/Output designed in Linux system, and stream buffer will only be flushed in some certain situations. Link to [last blog](http://doitzhong.com/2019/12/04/Unix-System-Level-I-O/).

This time, I will discuss behaviors of buffering in actual code, especially in **fread/fwrite**.


We look the first example:    

![ex1](https://tva1.sinaimg.cn/large/006tNbRwgy1g9rcice1dhj30ye0j2q5r.jpg)

Each time call *read*, 10 bytes data are read into *buf*, then, *file descriptor* is advanced by 10 bytes. And buf is flushed immediately on terminal, since the lowest-syscall *read* does not implement as buffering.

> What if we use **fread** instead of read?  

![example2](https://tva1.sinaimg.cn/large/006tNbRwgy1g9rcueb19kj30yy0m2wht.jpg)

The result looks weird, but we figure it out! When **fread** is called, it reads size of backing buffer data from file into backing buffer, in this case, it is obviously to see the size of data in file is smaller than the size of backing buffer. So all data in fox.txt are read into backing buffer at line 4. Then, output first 10 bytes data which is **"The quick"**. Then the **cursor** in backing buffer is advanced by 10 bytes. In the child process, even the **fread** is called, but it will not read data from fox.txt, it will go to the backing buffer and check the position of "cursor". The interesting thing is that after child process write "brown fox" and terminates, the parent process also output the same result-"brown fox". The reason is: when child process is forked, it **copy** all data (**including backing buffer and the position of "cursor"**) from its parent process, and it reads 10 bytes from backing buffer and outputs; the point is that even the "cursor" is advanced by 10 bytes after child process reads 10 bytes, but the **backing buffer** and **"cursor"** are just copy of parent's. So it won't change any records in parent's. As a result, after the child process terminates, the "cursor" in the parent process is still right after "the quick ", and the output is supposed to be "brown fox".


> things gets even more confusing when we perform both input & output

## Perform both input & output
![ex3](https://tva1.sinaimg.cn/large/006tNbRwgy1g9rdj9pgq0j30yy0g0jtl.jpg)

This example is not tricky, everything is going well, and output is predictable.

> what if use fwrite other than write?

![ex4](https://tva1.sinaimg.cn/large/006tNbRwgy1g9rdxh0yl0j30yy0iumzz.jpg)

**fwrite** is a wrapper function of write. When you try to use fwrite to output data, data will be read into a backing buffer firstly. Data will only be flushed under some cases like call fflush explicitly, or process terminates normally(These cases were discussed in last blog). So here, there is no situation where the stream buffer is able to be flushed into fox.txt, data is remained in backing buffer to wait something happens. After ```write (1, buf, sizeof(buf)) ```, the "cursor" is advanced by 10 bytes and the process terminates. Then, data in backing buffer is flushed into fox.txt with new "cursor" which is right after "the quick".

> use both fread & fwrite?

![ex5](https://tva1.sinaimg.cn/large/006tNbRwgy1g9red9dqmsj30yy0iutbi.jpg)

What the hell???!🥶  
Well, that's what I was thing when I saw such a "ridiculous" result.
But, no worries! We are gonna figure it out why!

When **fread** gets called, the whole fox.txt is read into a backing buffer, and the "cursor" is advanced to the end of file(the "cursor" here is different with "cursor" in backing buffer. To be simple, there are two "cursors", one is in buffer to check the position of reading, and the other is in file stream). Then, when we try to call **fwrite**, the actual "cursor" in file stream is already in the end of the file. So the "green cat" will be added to the end of the file!


<br>

#### References:
Michael Saelee: Input/Output  <https://moss.cs.iit.edu/cs351/slides/slides-io.pdf>

Randal Bryant: Computer Systems: A Programmer's Perspective
