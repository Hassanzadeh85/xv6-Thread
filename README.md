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



