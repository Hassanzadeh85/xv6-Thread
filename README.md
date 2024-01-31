# xv6-Thread
The xv6 operating system is an educational operating system designed for students in universities. 
This operating system is inspired by the Unix v6 operating system and is designed to be implemented on x86 systems.
Using the following commands, you can install this operating system on the Linux operating system:

sudo apt update

sudo apt install qemu

git clone git://github.com/mit-pdos/xv6-public.git

make

make qemu

The purpose of this project is to add support for creating threads in the xv6 operating system.
To achieve this, we need to define two system calls: `clone` and `join`. 
The `clone` function creates a new process or thread using an existing process or thread. 
The `join` function is used to prevent concurrency between threads. 
In addition to the system calls, we also need to add several user-space functions to the operating system.
These functions include: a function to create a new thread (`create_thread`), a function to prevent thread concurrency (`join_thread`), 
and three functions to define a spin lock (`init_lock`, `acquire_lock`, and `release_lock`). 
To define a new system call and user-space functions in the xv6 operating system, 
we need to modify some of the existing files in the operating system. 
The following is a list of the files that need to be modified, along with a description of the changes that need to be made to each file:
# syscall.h
In this file, a number has been assigned to each of the various types of system calls used in the operating system. 
There are 21 types of system calls in the xv6 operating system. At the end of this file, we add two system calls: 
`SYS_clone` with number 22 and `SYS_join` with number 23.
# syscall.c
By adding the two system calls defined above to this file, we inform the compiler of their existence.
# usys.S
We need to add the system call wrappers for the aforementioned system calls to this file. 
This file contains the system call wrappers that create the system calls from user space.
# defs.h
One of the header files in the xv6 operating system contains a list of functions that can be called in the kernel.
System calls should also be added to this file. Two system calls, `clone` and `join`, are added to this file.
The `clone` function is defined as `clone(void(*fcn)(void*), void *arg, void *stack)`. 
This function receives the required data from `arg` using the `fcn` function and creates a new object using the memory specified by `stack`.
Then, the `fcn` function is called on this object to perform the desired operation. 
The purpose of defining the `join` function is to wait for the connected thread to finish before the main thread continues its work.
This function prevents concurrency. The main thread waits for the created thread to release resources so that it can continue its work.
# user.h
One of the header files in the xv6 operating system contains a list of functions that can be called in the kernel. 
System calls should also be added to this file.







