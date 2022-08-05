+++
title = "Linux Process Lifecycle"
date = "2022-07-28T17:49:41+05:30"
author = "Nitin"
authorTwitter = "" #do not include @
cover = ""
tags = ["linux", "OS"]
keywords = ["", ""]
description = ""
showFullContent = false
readingTime = true
hideComments = false
+++

These are notes taken while learning Linux internals. My main source is [The Linux Programming Interface](https://man7.org/tlpi/).

## Overview

The four major system calls involved in the lifecycle of a process are the following.
1. **fork** - Create a replica of parent process
2. **exec** - Replace the program with intended new program
3. **wait** - Monitor the status of child process
4. **exit** - Terminate the process

There are 2 other system calls which might be used as well

- **system** - Allows executing of arbitrary shell commands  
- **clone** - Used in Linux for creating threads

{{< image src="/img/process_life.png" alt="Hello Friend" position="center" style="border-radius: 8px;" >}}

### 1. Fork

This system call creates a child process which is a replica of parent process. The data, heap and stack memory segments are copied to the child. The text segment of memory is made read-only shared between parent and child. The file descriptors of parent are duplicated in child. The stdio buffers of parent are also inherited by child.  
After `fork` depending on system it can be either parent or child that is scheduled for CPU time next. 

 
`child_pid = fork()` 


In modern Linux, the virtual memory of parent does not at once get copied to child. This is because in many cases the child calls `exec` just after creation which clears the memory. So instead copying memory kernel uses copy-on-write (COW) method. The kernel shares the memory between parent and child and on receiving any call to modify the memory, creates a copy of virtual memory to the calling process.  

There are pros and cons for scheduling parent or child at once after fork. Child getting scheduled is helpful if it calls `exec` replacing the memory. In this case parents' memory stays intact and does not require re-allocation. The advantage of running parent process is that the memory pages required by the process are probably already loaded and it enables faster execution. 
In Linux, the parent process is generally scheduled to run after fork. This kernel setting can changed using sysconfig or /proc filesystem.  


The file descriptors are copied to child and so child shares the file description table. The file_offset which points the location of file last access set by `read`, `write`, `lseek` system calls. The file_offset being shared prevents parent and child from over-writing each other. But it might still intermingle the writes. To avoid intermingling of writes `wait` system call can be used, so that parent process is blocked till child is running. 

#### vfork

`vfork` is variant of fork system call that is more efficient if the child is supposed to `exec` after fork. This does not copy parent's memory and blocks the parent until the child calls `exec`.  

### 2. Exec

There isn't system call named exec, instead `execve` and list of exec variants are used. This causes the memory of the child inherited parent to be cleared and the new program passed as argument to exec function to be loaded. 






