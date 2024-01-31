# xv6-Thread
The xv6 operating system is an educational operating system designed for students in universities. 
This operating system is inspired by the Unix v6 operating system and is designed to be implemented on x86 systems.
Using the following commands, you can install this operating system on the Linux operating system:

`sudo apt update`

`sudo apt install qemu`

`git clone git://github.com/mit-pdos/xv6-public.git`

`make`

`make qemu`

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
# ulib.c
We need to define the user-space functions that we added to the `user.h` file in this file. 
To create a new thread, we define the `create_thread` function as follows:
```
int create_thread(void (*fnc)(void *), void *arg)
{
char *stack = sbrk(PGSIZE);   
 return clone(fnc,arg,stack);     
}
```
In the above-defined function, the `sbrk` function allows the program to extend the end of the memory block by a specified amount. In this function, the memory block used for the memory stack is increased by one page.
The join function is defined as follows:
```
int join_thread(void) 
     {
        return join(); 
        }
```
The lock definition function is defined as follows. In this function, the value of flag is set to zero.
```
void init_lock(struct t_lock *xlock)
 {
	xlock->flag = 0;
}
```
We define the lock acquisition function as follows. In this function, we define a spin lock that repeatedly tries to obtain the lock until it successfully acquires it.
```
void acquire_lock(struct t_lock *xlock)
 {
    while(xchg(&xlock->flag, 1) != 0);
}
```
The lock release function is also defined as follows, in which zero is exchanged with the value of the lock:
```
void release_lock(struct t_lock *xlock)
 {
	xchg(&xlock->flag, 0);
 }
```
# proc.c
We need to define the system calls that we added to the `user.h` file in this file.
The `clone` function is defined as follows:
```
int clone(void (*fcn)(void*), void *arg, void *stack) {
  int i;
  struct proc *thread;
  struct proc *curproc = myproc();
  if ((curproc->sz - (uint) stack) < PGSIZE || ((uint) stack % PGSIZE) != 0)
    return -1;
  if ((thread = allocproc()) == 0)
    return -1;
  thread->pgdir = curproc->pgdir;
  thread->sz = curproc->sz; 
  thread->parent = curproc; 
  *thread->tf = *curproc->tf;
  uint user_stack [2];
  user_stack[0] = 0xffffffff; 
  user_stack[1] = (uint) arg; 
  uint stack_top = (uint) stack + PGSIZE;
  stack_top -= 8;
  if (copyout(thread->pgdir, stack_top, user_stack, 8) < 0)
    return -1;
  thread->tf->ebp = (uint) stack_top;
  thread->tf->esp = (uint) stack_top;
  thread->tf->eip = (uint) fcn;
  thread->tf->eax = 0;
  for (i = 0; i < NOFILE; i++)
    if (curproc->ofile[i])
      thread->ofile[i] = filedup(curproc->ofile[i]);
  thread->cwd = idup(curproc->cwd);
  safestrcpy(thread->name, curproc->name, sizeof(curproc->name));
  acquire(&ptable.lock);
  thread->state = RUNNABLE;
  release(&ptable.lock);
  return thread->pid;
}
```
First, we need to check if the available address space in the current process is sufficient to create a new thread. The condition inside the loop checks if the remaining address space in the current process after the new stack location (which is provided as an argument to the `clone` function) is less than the size of one page, in which case creating a new thread will not be possible. The second condition checks whether the new stack page is properly aligned or not. Next, we need to check if `allocproc()` can allocate a new process or not. This function extracts a new process from the process pool in the xv6 operating system and prepares it for use in a new thread. If the pool is empty and a new process cannot be extracted, this function returns zero. In this condition, the `allocproc()` function is called first and its result is stored in the `thread` variable. Then, this condition checks whether `thread` has a value of zero or not. If it is zero, it means that the `allocproc()` function failed to allocate a new process and the `clone` function returns a negative one.
When a new thread is created, the address space of the current process must be transferred to the address space of the new thread. That is, the new thread will have exactly the same address space and physical memory pages. With each new thread definition, we must store the information related to the stack and other parameters required to create a thread in a suitable location so that we can use them in subsequent steps.
We define an array with two elements in which the first element is the start value of the stack and the second element is the value that comes in the argument of the `clone` function. To determine the memory range of the new process, we need to set the top address of the memory for the new process, which is obtained by adding the size of the page to the `stack` argument. However, the array we defined with two elements allocates 8 bytes (two 32-bit numbers) of memory for itself, so the top address of the stack must be increased by 8 bytes. But since the addresses grow in reverse, moving down the stack and creating the necessary space is done by reducing 8 bytes. These 8 bytes must be copied from `user_stack` to the top of the stack in memory.
Now we need to set the `esp` and `ebp` registers to the highest address of the stack. The two numbers stored at the top of the stack are placed in these registers to determine the base and top addresses of the stack.
`eip` is an important register in the xv6 operating system. When the processor executes instructions, `eip` automatically increases to the address of the next instruction to be executed. Therefore, `eip` indicates the current position in the program and shows which instruction is next. So we need to assign `fcn` to `eip` so that after executing the current function, the code will be directed to `fcn` and the execution of the `fcn` function will begin.
Now we need to set the return value of the function to zero to indicate that the child has been successfully created. This is done with the `eax` register. This register is used as the result register and the value in it is considered as the return value of the function.
All files that are open for the parent process must also be open for the child process. Therefore, we copy all of them to the child process by defining a loop and identifying the open files. Copying is done using the `filedup` function. Also, the parent process directory must be copied to the child process. This is done using the `idup` function. Finally, we need to assign the `RUNNABLE` state to the newly created child process so that the operating system selects it for execution and assigns it a time quantum.
The join function is defined as follows:
```
int join(void)
{
  struct proc *p;
  int haveThreads, pid;
  struct proc *curproc = myproc();
  
  acquire(&ptable.lock);
  for(;;) 
  {
      haveThreads = 0;
      for(p = ptable.proc; p < &ptable.proc[NPROC]; p++)
      {
      if((p->parent != curproc) || (p->parent->pgdir != curproc->pgdir) )
      continue;
       haveThreads = 1;
       if(p->state == ZOMBIE) 
       {  
      pid = p->pid;
      kfree(p->kstack);
      p->kstack = 0;
      p->pid = 0;
      p->parent = 0;
      p->name[0] = 0;
      p->killed = 0;
      p->state = UNUSED;
      release(&ptable.lock);

      return pid;
       }
    }
     if(!haveThreads || curproc->killed)
     {
       release(&ptable.lock);
       return -1;
     }
     sleep(curproc, &ptable.lock);
   }
}
```
The `join` function is used to wait for the termination of child processes of a parent process. In an infinite loop, we check the process table. The goal is to find the children of the current process. If a child process is not a child of the current process or its directory is different, we return to the beginning of the loop. Otherwise, we return its specifications. If no terminated child process exists in the process table or the process has already been killed, the loop is restarted until all child processes have terminated. Resources such as memory are released for terminated processes. The values related to the process are set to zero and its state is changed to `UNUSED`.

