+++
title = 'OS : Segmentation and virtual memory'
date = 2024-06-25T21:41:09+05:30
draft = false
description = "Some notes that I made a month or two ago"
image = "/images/stdImage.svg"
categories = ["General"]
+++

# Main Memory

several issues that are pertinent to managing memory: basic hardware, the binding of symbolic memory addresses to actual physical addresses, and the distinction between logical and physical addresses.

## 8.1

```python
# test python (sample from offlineimap)
 
class ExitNotifyThread(Thread):
    """This class is designed to alert a "monitor" to the fact that a thread has
    exited and to provide for the ability for it to find out why."""
    def run(self):
        global exitthreads, profiledir
        self.threadid = thread.get_ident()
        try:
            if not profiledir:          # normal case
                Thread.run(self)
            else:
                try:
                    import cProfile as profile
                except ImportError:
                    import profile
                prof = profile.Profile()
                try:
                    prof = prof.runctx("Thread.run(self)", globals(), locals())
                except SystemExit:
                    pass
                prof.dump_stats( \
                            profiledir + "/" + str(self.threadid) + "_" + \
                            self.getName() + ".prof")
        except:
            self.setExitCause('EXCEPTION')
            if sys:
                self.setExitException(sys.exc_info()[1])
                tb = traceback.format_exc()
                self.setExitStackTrace(tb)
        else:
            self.setExitCause('NORMAL')
        if not hasattr(self, 'exitmessage'):
            self.setExitMessage(None)
 
        if exitthreads:
            exitthreads.put(self, True)
 
    def setExitCause(self, cause):
        self.exitcause = cause
    def getExitCause(self):
        """Returns the cause of the exit, one of:
        'EXCEPTION' -- the thread aborted because of an exception
        'NORMAL' -- normal termination."""
        return self.exitcause
    def setExitException(self, exc):
        self.exitexception = exc
    def getExitException(self):
        """If getExitCause() is 'EXCEPTION', holds the value from
        sys.exc_info()[1] for this exception."""
        return self.exitexception
    def setExitStackTrace(self, st):
        self.exitstacktrace = st
    def getExitStackTrace(self):
        """If getExitCause() is 'EXCEPTION', returns a string representing
        the stack trace for this exception."""
        return self.exitstacktrace
    def setExitMessage(self, msg):
        """Sets the exit message to be fetched by a subsequent call to
        getExitMessage.  This message may be any object or type except
        None."""
        self.exitmessage = msg
    def getExitMessage(self):
        """For any exit cause, returns the message previously set by
        a call to setExitMessage(), or None if there was no such message
        set."""
        return self.exitmessage

```


### 8.1.1 Basic Hardware

1. Main memory and the registers built into the processor itself are the only general-purpose storage that the CPU can access directly.
2. There are machine instructions that take memory addresses as arguments, but none that take disk addresses.
3. Therefore, any instructions in execution, and any data being used by the instructions, must be in one of these direct-access storage devices.
4. Registers that are built into the CPU are generally accessible within one cycle of the CPU clock. Most CPUs can decode instructions and perform simple operations on register contents at the rate of one or more operations per clock tick. The same cannot be said of main memory, which is accessed via a transaction on the memory bus.
5. Completing a memory access may take many cycles of the CPU clock. In such cases, the processor normally needs to stall, since it does not have the data required to complete the instruction that it is executing.
6. The remedy is to add fast memory between the CPU and main memory, typically on the CPU chip for fast access. Such a cache was described in Section 1.8.3. To manage a cache built into the CPU, the hardware automatically speeds up memory access without any operating-system control.

We are concerned with 
1. Relative speed of accessing Physical Memory
2. Correct operation
3. Protect the OS from access by user processes
4. On *Multiuser systems*, protect one user processes from another user's processes. This protection must be provided by the hardware because the operating system doesn’t usually intervene between the CPU and its memory accesses (because of the resulting performance penalty).

	   One hardware method of implementing this is :
	   Each process has a separate memory space.
	   To separate memory spaces, we need the ability to
	   determine the the range of legal addresses, this is
	   done by using 2 registers, one base register and one
	   limit register. Base Register -> holds the smallest
	   legal physical memory addresses; 
	   Limit -> specefies the size of the range.

![[Pasted image 20240326124557.png|400x300]]


Any attempt by program executing in user mode to access operating-system memory or other users' memory results in a trap to the OS which treats the attempt as a fatal error.

![[Pasted image 20240326124845.png|400x300]]

1. Base, Limit Registers can only be loaded by OS which uses a special privileged instruction.
2. Privileged instructions can only be accessed in the kernel mode, Only OS executes in kernel mode, so only OS can load the base and limit registers.
3. The OS executing in kernel mode is given unrestricted access to both operating system memory and user's memory. This lets the OS to load users' programs into users' memory, to dump out those programs in case  of erros, to access and modify parameters of system calls, to perform I/O to and from user memory, and to provide many other services.

### 8.1.2 Address Binding

Process Execution : 
1. File to be executed is loaded into memory from the disk and placed within a process.
2. Depending on Memory Management the process may be moved between main memory and disk during its execution.
3. Processes on the disk that are waiting to be brought into the memory to be execution from the **input queue**.
4. The normal single-tasking procedure is to select one of the process in the input queue and to load that process into memory.





> Systems allow User Processes to reside in any part of the OS and not necessarily 0000.

 >Addresses in the  source program are generally symbolic(ex. Variable Count).
 >
 >Compiler typically binds these symbolic addresses to relocatable addresses (such as "14 bytes from the beginning of this module").
 >
 >Linkage editor or loader in turn binds the relocatable addresses to absolute addresses (such as 74014). Each binding is a mapping from one address space to another.

Classically, the binding of instructions and data to memory addresses can be done at any step along the way:

1. **Compile Time** : If location of process is known at compile time then *absolute code* can be generated. If sometime later the location of the process changes then the process needs to be recompiled. *MS-DOS .COM format programs are bound at compile time.*
2. **Load Time** : If location of the process is not known at the compile time then the compiler must generate a *relocatable code*. Final binding is delayed until load time. If starting address changes, we need to only reload the user code to incorporate for this changed value.
3. **Execution Time** :  If process can be moved during its execution from one memory segment to another, then binding must be delayed until run time. Special hardware must be available for this scheme to work. Most general purpose OSes uses this method.
![[Pasted image 20240326132619.png|400x300]]


### 8.1.3 Logical Versus Physical Address Space

Address generated by CPU is commonly referred to as a *logical address*, whereas an address seen by the memory unit that is loaded into the memory-address register of the memory is called *Physical Address*.

> Compile-Time and Load-Time address binding methods generate identical logical and physical addresses, Execution Time's logical and physical addresses are different.


> Logical address is referred to as a virtual address. (Only in this text)


Set of all logical addresses generated by a program is a *logical address space*.

The set of all physical addresses corresponding to these logical addresses is a *Physical Address Space*.  


The run-time mapping from virtual to physical addresses is done by a hardware device called the *memory-management unit(MMU)*.

Using a generalization of the base-register scheme described previously, the base register now is called a relocation register. 
Value in the relocation register is added to every address generated by a user process at the time the address is sent to memory.


### 8.1.4 Dynamic Loading

**Dynamic loading** : a routine is not loaded until it is called. All routines are kept on disk in a relocatable load format. The main program is loaded into memory and is executed. When a routine needs to call another routine, the calling routine ﬁrst checks to see whether the other routine has been loaded. If it has not, the relocatable linking loader is called to load the desired routine into memory and to update the program’s address tables to reﬂect this change. Then control is passed to the newly loaded routine.

Dynamic Loading does not require special support from the OS. Its the users responsibility to code in such a way to take advantage of such a method.
Operating systems may help the programmer, however, by providing library routines to implement dynamic loading.


### 8.1.5 Dynamic Linking and shared Libraries

Some OSes provide only *static linking* in which system libraries are treated like any other object module and are combined by the loader into the binary program image. 
Dynamic linking in contrast, is similar to dynamic loading. Here rather  than loading,  it is including references to repeated code/data.

This feature is usually used with system libraries, such as language subroutine libraries.

This referencing is done through something called as a *stub*.
**Stub** : Small piece of code that indicates how to locate the library if whether the needed routine is already in memory. 
The Stub replaces itself with the address of the routine and executes the routine. This results in no cost for dynamic linking. 

Dynamic linking results in shared libraries. 


## 8.2 Swapping
Enables the total physical address space of all process to exceed the real physical memory of the system.

![[Pasted image 20240326232848.png|400x300]]

### 8.2.1 Standard Swapping

Standard swapping involves moving processes between main memory and a backing store. Backing Store is generally a fast disk. 
Backing Store must be : 
1. large enough to accommodate copies of all memory images for all users.
2. It must provide direct access to these memory images. 

The system maintains a *ready queue* consisting of all the processes whose memory images are on the backing store or in the memory and are ready to run. 

When CPU Scheduler decides to execute a process, it calls a *dispatcher*, which checks to see if the next process in the queue is in the memory, if it isn't then there is no free space in the memory and the dispatcher swaps the desired process with an already present process in the main memory. It then reloads the registers and transfers the control to the selected process.
**Context-Switch time in such a swapping system is fairly high.**

It is useful to know exactly how much memory a user process is using as we would only need to swap what is used, reducing the swap time.
For this method to be effective, the user must keep the system informed of any changes in memory requirements. Thus, a process with dynamic memory requirements will need to issue system calls(request memory() and release memory()) to inform the operating system of its changing memory needs.

Swapping is constrained by : 
1. Process needs to be idle to be swapped.
2. Processes with pending I/O operations cannot be swapped until the I/O operation is complete.
Normally, when a program is compiled, the compiler automatically constructs segments reﬂecting the input program, a C compiler might create separate segments for the following:

1. The code
2. Global variables
3. The heap, from which memory is allocated
4. The stacks used by each thread
5. The standard C library

Libraries that are linked in during compile time might be assigned separate segments. The loader would take all these segments and assign them segment numbers.

 Suppose we are to swap out process P1 and P2, The I/O operation might try to use the memory that belongs to process P2. 
   There are 2 main solutions to this problem : 
	  >never swap a process with pending I/O
	    execute I/O operations only into operating-system buffers


Transfers between operating-system buffers and process memory then occur only when the process is swapped in. Note that this double buffering itself adds overhead. We now need to copy the data again, from kernel memory to user memory, before the user process can access it.

Standard Swapping is not used in modern OSes.
1. It requires too much swapping time
2. It provides too little execution time to be a reasonable memory-management solution.
3. Modified versions of swapping are found in many systems including Linux, Unix, Windows. 


In one common variation, swapping is normally disabled but will start if the amount of free memory (unused memory available for the operating system or processes to use) falls below a threshold amount. Swapping is halted when the amount of free memory increases. Another variation involves swapping portions of processes—rather than entire processes—to decrease swap time.

### 8.2.2 Swapping on Mobile Systems 

Unlike PC and server operating systems, mobile OSes typically don't use swapping due to space constraints and limitations of flash memory. Instead, iOS and Android rely on app cooperation to free up memory and may terminate misbehaving apps. Mobile app developers need to be mindful of memory usage to avoid such situations.

Note : 
1. Flash Memory has limited number of writes that flash memory can tolerate.
2. Read Only Data(ex. Code) is freed up and later reloaded from flash memory is needed. 
3. Data that has been modified is never removed. 
4. Before terminating a process Android writes its application state to flash memory so that it can be quickly restarted. 
5. Both Android and Ios support paging, so they have memory management abilities. 


### 8.3 Contagious Memory Allocation 


The memory is usually divided into two different parts : 
1. A part for the operating system.
2. A part for the user processes.

We can place the OS in either *Low Memory* or *High Memory* 
The major factor that affects this decision is **Location Of The Interrupt Vector**. Since the interrupt vector is often in low memory, the programmers place the OS in low memory as well. 

In *contiguous memory allocation*, each process is contained in a single section of memory that is contiguous to the section containing the next process.

#### 8.3.1 Memory Protection 

We can prevent a process from accessing memory it does not own by combining two ideas previously discussed. 
i.e. With the use of (Relocation Register, Limit Register) <= MMU uses these to generate physical addresses which is sent to memory.

When CPU scheduler selects a process for execution, the dispatcher loads {Relocation Register, Limit Register}(different registers from the previous paragraph) with the correct values as a part of the context switch. 

Every address generated by a CPU is checked against these registers, we can protect both the OS and the users' programs and data from being modified by a running process.

The relocation-register scheme allows the operating system to dynamically adjust its size in memory. This is useful for loading and unloading infrequently used code (like device drivers) to free up memory space for other tasks.

Such code is called *transient* operating-system code.


#### 8.3.2 Memory Allocation 

**Multiple Partition Method** 
1. Memory is divided into several fixed-sized partitions.
2. Each Partition may contain exactly one process.
3. Degree of multi programming is bound by number of partitions. 
4. Originally used by IBM OS/360 OS (called MFT), it is no longer in use.


MVT(Multiprogramming with a Variable number of Tasks) (Generalization of Fixed Partition Scheme) Used Primarily in batch environments. Many of these ideas here are also applicable to a time-sharing environment in which pure segmentation is used for memory management.

In the variable-partition scheme, 
1. the operating system keeps a table indicating which parts of memory are available and which are occupied.
2. Initially when all memory is available, it is considered one large block of available memory, a *hole*. 
3. Memory contains a set of holes of various sizes.

As processes enter the system, they are put in to a input queue. The operating system takes into account the memory requirements of each process and the amount of available memory space in determining which processes are allocated memory. 


At any given time, then, we have a list of available block sizes and an
input queue. The operating system can order the input queue according to
a scheduling algorithm. Memory is allocated to processes until, ﬁnally, the
memory requirements of the next process cannot be satisﬁed—that is, no
available block of memory (or hole) is large enough to hold that process. The operating system can then wait until a large enough block is available, or it can skip down the input queue to see whether the smaller memory requirements of some other process can be met.

When a process arrives and needs memory, the system searches the set for a hole that is large enough for this process. If the hole is too large, it is split into two parts. One part is allocated to the arriving process; the other is returned to the set of holes.
If the new hole is adjacent to other holes, these adjacent holes
are merged to form one larger hole.

This procedure is a particular instance of the general dynamic storage-
allocation problem, which concerns how to satisfy a request of size n from a list of free holes. There are many solutions to this problem. The *ﬁrst-ﬁt*, *best-ﬁt*, and *worst-ﬁt* strategies are the ones most commonly used to select a free hole from the set of available holes.

* First Fit : Allocate the first hole that is big enough. Searching can start either at the beginning of the set of holes or at the location where the previous first fit search ended. We can stop searching as soon as we find a free hole that is large enough.
* Best Fit : Allocate the smallest hole that is big enough. We must search the entire list, unless the list is ordered by size. 
* Worst Fit : Allocate the largest hole. Again, we must search the entire list, unless it is sorted by size. This strategy produces the largest leftover hole, which may be more useful than the smaller leftover hole from a best-ﬁt approach.

Simulations have shown that both ﬁrst ﬁt and best ﬁt are better than worst
ﬁt in terms of decreasing time and storage utilization. Neither ﬁrst ﬁt nor best ﬁt is clearly better than the other in terms of storage utilization, but ﬁrst ﬁt is generally faster.


#### 8.3.3 Fragmentation

As process are loaded and removed from memory, the free memory is broken into smaller pieces. 
External fragmentation exists when there is enough total memory space to satisfy a request but the available spaces are not contiguous: storage is fragmented into a large number of small holes.

In the worst case scenario we could have a block of free/wasted memory between every two processes. If all these small pieces of memory were in one big free block instead we might be able to run several more processes. 

In some cases first fit affects the system more, in some other cases best fit affects the system more. 

Which end of the free block is allocated also matters. 

Statistical analysis of ﬁrst ﬁt, for instance, reveals that, even with some optimization, given N allocated blocks, another 0.5 N blocks will be lost to fragmentation.

That is, one-third of memory may be unusable! This property is known as the *50-percent rule*.

Memory fragmentation can be internal as well as external.

Consider a multiple-partition allocation scheme with a hole of 18,464 bytes. Suppose that the next process requests 18,462 bytes. If we allocate exactly the requested block, we are left with a hole of 2 bytes. The overhead to keep track of this hole will be substantially larger than the hole itself. The general approach to avoiding this problem is to break the physical memory into ﬁxed-sized blocks and allocate memory in units based on block size. With this approach, the memory allocated to a process may be slightly larger than the requested memory. The difference between these two numbers is internal fragmentation —unused memory that is internal to a partition.

One solution to the problem of external fragmentation is **compaction**.
Shuffle the memory blocks so all free memory is placed together in one large block.

Compaction is not always possible. 
* If relocation is static and is done at assembly or load time, compaction cannot be done. 
* It is possible only if relocation is dynamic and is done at execution time. 
* If addresses are relocated dynamically, relocation requires only moving the program and data and then changing the base register to reﬂect the new base address. 
* When compaction is possible, we must determine its cost. The simplest compaction algorithm is to move all processes toward one end of memory; all holes move in the other direction, producing one large hole of available memory. This scheme can be expensive.



Another Solution to the external-fragmentation  problem  is to permit the logical address space of the processes to be noncontagious thereby allowing a process to be allocated physical memory wherever such memory is available. 

Two complementary techniques can achieve this solution.
1. segmentation
2. paging

These techniques can be combined as well.


### 8.4 Segmentation

The user and the programmer's view of the memory isn't the actual physical memory.

If hardware can provide a memory mechanism that mapped programmer's view to the actual physical memory then 
1. System would have more freedom to manage memory
2. programmer will have a more natural programming environment

Segmentation provides such a mechanism.


#### 8.4.1 Basic Method

Most programmers would prefer to view memory as a collection of variable-sized segments, with no necessary ordering among the segments.

Segments vary in length, and the length of each is intrinsically deﬁned by its purpose in the program. Elements within a segment are identiﬁed by their offset from the beginning of the segment.

**Segmentation** is a memory-management scheme that supports this programmer view of memory. 
A logical address space is a collection of segments. Each segment has a name and a length. These addresses specify both the segment name and the offset within the segment. 


For simplicity of implementation, segments are numbered and are referred to by a segment number, rather than by a segment name. Thus, a logical address consists of a two tuple:
*<segment-number, offset>*.


#### 8.4.2 Segmentation Hardware 

Programmer refers to objects in a program by a 2D address, physical address is 1 dimensional. 
An implementation must be designed to map 2D user defined address to 1D physical address.

This mapping is effected by a segment table. Each entry in the
segment table has a segment base and a segment limit. The segment base
contains the starting physical address where the segment resides in memory, and the segment limit speciﬁes the length of the segment.

![[Pasted image 20240327202653.png|400x300]]

Logical address has two parts, a segment number, s and an offeset into that segment, d. 

segment number is used as an index to the segment table. 
The offset d of the logical address must be between 0 and the segment limit if its not then we trap to the OS. 


![[Pasted image 20240327203553.png|400x300]]


### 8.5 Paging

Paging is another memory-management scheme that lets the address space be non contagious. Paging also *avoids external fragmentation* and the need for *compaction*.
It also solves *the problem of fitting memory chunks of varying sizes onto the backing store.* 

The problem arises because, when code fragments data residing in main memory need to be swapped out, space must be found on the backing store. The backing store has the same fragmentation problems discussed in connection with main memory, but access is much slower, so compaction is impossible.

Paging is used on many OSes even in smart phones.
Paging is implemented through cooperation between the OS and the computer hardware.

### 8.5.1 Basic Method

Physical memory is broken into fixed-sized blocks called *frames* and breaking the logical memory into the blocks of the same size called *pages*.

When a process is to be executed, its pages are loaded into any available memory frames from their source.

The backing store is divided into fixed-sized blocks that are the same size as the memory frames or clusters of multiple frames. 

This separates logical and physical address space.

so 64 bit logical addresses are possible despite the system having lesser than 2^64 bytes of physical memory.

![[Pasted image 20240327204511.png|400x300]]

Every address generated by the CPU is divided into two parts : 
1. page number(p)
2. page offset(d)

* Page number is used as an index.
* Page table contains the base address of each page in physical memory.
* This base address is combined with the page offset to deﬁne the physical memory address that is sent to the memory unit.
![[Pasted image 20240327234741.png|300x300]]


* Page Size(like the frame size) is defined by the hardware. The size of a page is a power of 2, varying between 512 bytes and 1GB per page, depending on the computer architecture. The selection of a power of 2 as a page size makes the translation of a logical address into a page number and page offset particularly easier. 
* If Size of logical address space is 2^m and page size is 2^n then the high order m-n bits of a logical address designate the page number and the n low order bits designate the offset.

Thus logical address is as follows : 
> Page number p = m-n, page offset d = n

Where p is an index into the page table and d is the displacement within the page.

Suppose, n = 2, m =4
we have page number m - n = 2; page offset n = 2.

![[Pasted image 20240329114446.png|300x300]]

1. For logical address 0 : page no = 0, offset  = 0, page no 0 is in frame 5.
2. So the physical mapping is given by {5(frame no) * 4(page size) + 0(offset)}


Paging itself is a form of *dynamic relocation*.

Using paging is similar to using a table of base (or relocation) registers,
one for each frame of memory.

When we use paging scheme : 
* There *is no* external fragmentation
* There *is* internal fragmentation

Frames are allocated as units.

Suppose we have a process of 72766 bytes, that will need 35 pages + 1086 bytes.
Page size is  2048 bytes so 2048-1086 = 962 bytes. This 962 bytes is left unused and is a memory gap when this is put back into the disk.
This is internal fragmentation.

In the worst case a process could need *n pages + 1 bytes*  which gets allocated n + 1 frames. So we have an internal fragmentation of almost an entire frame.

If process size is independent of page size, we expect internal fragmentation to average *one-half page per process*.

This suggests that small page sizes are desirable but having a smaller page size increases the number of pages which increases the overhead in the page table entry.
Disk I/O is more efficient when the amount of data being transferred is larger. 
Generally page sizes have grown larger over time as process, data sets, and main memory have become larger. 

Today, pages are typically between 4-8 KB in size.

Solaris uses page sizes of both 8KB and 4 MB depending on the data being stored by the pages.


Paging lets us use physical memory that is larger than what can be addressed by the CPU’s address pointer length.


>An important aspect of paging is the clear separation between the programmer’s view of memory and the actual physical memory

>The programmer views memory as one single space, containing only this one program. In fact, the user program is scattered throughout physical memory, which also holds other programs.

This mapping is hidden from the programmer and is controlled by the operating system. Notice that the user process by deﬁnition is unable to access memory it does not own. It has no way of addressing memory outside of its page table, and the table includes only those pages that the process owns.

**Frame Table** is the data structure used by the OS to keep a track of allocation details of physical memory.

The frame table has one entry for each physical page frame, indicating whether the latter is free or allocated and, if it is allocated, to which page of which process or processes.

The operating system maintains a copy of the page table for each process, just as it maintains a copy of the instruction counter and register contents. This copy is used to translate logical addresses to physical addresses whenever the operating system must map a logical address to a physical address manually. It is also used by the CPU dispatcher to deﬁne the hardware page table when a process is to be allocated the CPU. *Paging therefore increases the context-switch time.*

### 8.5.2 Hardware support 

Each OS has its own methods for storing page tables.

Some 
1. allocate a page table for each process.
2. A pointer to the page table is stored with the other register values(like the instruction counter) in the process control block.
3. When dispatcher is told to start a process, it must reset the user registers and define the correct hardware page table values from the stored user page table.

Other OSes provide one or at most a few page tables which decreases the overhead involved when the processes are context-switched.

Hardware implementation of the page table can be done in several ways, in the simplest case, 

* page table is implemented as a set of dedicated registers.
* These registers must be built with very high speed logic to make the paging address translation efficient.
* Efficiency is a major concern as every access to memory must go through the page map. 
* Instructions to modify/load page table registers are privileged so only the OS can change the memory map.
* DEC PDP 11 is an example of such an architecture. 
* The use of registers for page table is satisfactory if the page table is reasonably small( for example 256 entries) 

>Current computers allow page table to be very large so this is not feasible.
 > For these, the page table is kept in main memory and a page table base register (PTBR) points to the page table.
 > Changing the page tables requires changing only this one register so context switch time reduces by a lot.

* The problem with this is that, if we want to access location i we must first index into the page table using the value in the PTBR by an offset i which is stored in the memory. So this task requires a memory access. 
* This provides us with the frame number which is combined with the page offset to produce the actual address. 

*This scheme requires 2 memory accesses to access a byte(one for the page table entry and the other for the byte), so memory access is slowed by a factor of 2*
Swapping would be better than this.

The standard solution to this problem is to use a special, mall fast-lookup hardware cache called a translation look-aside buffer (TLB).

TLB is associative high speed memory.

* Each entry in TLB consists of two parts : a key(or tag) and a value. 
* When TLB is presented with an item, the item is compared with all the keys simultaneously. 
* If the item is found the corresponding field is returned. 
* TLB lookup in the modern hardware is a part of the instruction pipeline essentially adding no performance penalty.
* To be able to execute the search within a pipeline step, the TLB must be kept small.
* It is typically between 32 and 1024 entries in size.
* Some CPUs implement separate instruction and data address TLBs. That can double the number of TLB entries available because those lookups occur in different pipeline steps.
* Systems have evolved from no TLBs to multiple level of TLBs just like they have multiple levels of cache.
* TLB contains only a few of the page-table entries. When a logical address is generated by the CPU, its page number is presented to the TLB. If the page number is found its frame number  is immediately available and is used to access memory. 
* If page number is not in the TLB => *TLB Miss*, a memory reference to the page table must be made. Depending on the CPU, this may be done automatically in hardware or via an interrupt to the OS. 
* When the frame is found we can use it to access memory and the frame number and the page number is added to the TLB.
* If TLB is full then this is replaced with an already existing entry.
* Replacement policies range from *LRU*, *round-robin* to *random*.
* Some CPUs allow the OS to participate in LRU entry replacement, while others handle it themselves. 
* Some TLBs allow certain entries to be wired down meaning they cannot be removed from the TLB.
* TLB entries for kernel code are wired down.
* Some TLBs store address-space identifiers (ASIDs)  in each TLB entry.
* An ASID uniquely identifies each process and is used to provide address-space protection for that process. 
* When the TLB attempts to resolve virtual page numbers it ensures that the ASID for the currently running process matches the  ASID associated with the virtual page.  
* If the ASIDs do not match, the attempt is treated as a TLB miss. 
* ASID also allows the TLB to contain entries for several different processes simultaneously.
* If the TLB does not support separate ASIDs, then every time a new page table is selected (for instance, with each context switch), the TLB must be flushed to ensure that the next executing process does not use the wrong translated information. 
* Otherwise, the TLB could include the old entries that contain valid virtual addresses but have incorrect or invalid physical addresses left over from the previous process.


* **TLB HIT RATIO** : % of times that the page number of interest is found in the TLB.

![[Pasted image 20240401234822.png | 300x200]]

### 8.5.3 Protection

Protection bits associated with each frame help in memory protection in a paged environment.

Normally these bits are kept in the page table.

One bit can define a page to be read-write or read-only. Every reference to memory goes through the page table to find the correct frame number.

Writing to a read only page causes a hardware trap to the OS.

One additional bit is generally attached to each entry in the page table: a
valid –invalid bit. When this bit is set to VALID, the associated page is in the process's logical address space and is thus a legal (or valid) page. 

When the bit is set to invalid, the page is not in the process's logical address space. 
Illegal addresses are trapped using valid-invalid bit.
The OS sets this bit for each page to allow or disallow access to the page.

Some systems provide hardware, in the form of a page-table
length register (PTLR), to indicate the size of the page table. This value is
checked against every logical address to verify that the address is in the valid range for the process. Failure of this test causes an error trap to the operating system.



### 8.5.4 Shared Pages 


An advantage of paging is the possibility of sharing common code. 
This consideration is particularly important in time-sharing environment. Consider a system that supports 40 users each of whom executes a text editor. If the text editor consists of 150KB of code and 50KB of data space, we need 8000 KB to support 40 users. 

*if the code is reentrant code (or pure code), however it can be shared.*
Each process has its own data page. 
Reentrant code is non-self-modifying code: it never changes during execution. Thus, two or more process can execute the same code at the same time.

Each process has its own copy of registers and data storage to hold the data for the process's execution The data for two different processes will, of course, be different.

Only one copy of the editor needs to be kept in the physical memory. 
Each user's page table maps onto the same physical copy of the editor. 
Data pages are mapped onto different frames. 

Thus to support 40 users, we need only one copy of the editor (150 kb), plus 40 copies of the 50KB of data space per user. 

Total space required  is now 2150kb instead of 8000kb

Other heavily used programs can also be shared : 

* Compilers
* Window Systems
* Run-time libraries
* Database system
* etc
The read-only nature of shared code should not be left to the correctness of the code; the OS should enforce this property.

Some OSes implement shared memory using shared pages.


## Structure of the Page Table

Structuring of the page table : 
1. Hierarchical  paging
2. hashed page tables
3. inverted page tables

### 8.6.1 Hierarchical paging

Most modern computers support huge logical address spaces 2^32 to 2^64.
In such situations the page table itself becomes really large. If page size = 2^12(4KB) then page table contains about 1 Million entries 2^32/2^12.

Assuming that each entry consists of 4 bytes each process may need upto 4MB of physical address space for the page table along. 
Clearly we could not want to allocate the page table contiguous  in main memory.

Simple solution for this is to divide the page table into smaller pieces. We can accomplish this division in many ways.

**Two Level Paging**
In this the page table itself is also paged. 
Consider the above system itself. A logical Address is divided into a page number consisting of 20 bits and a  page offset consisting of 12 bits.

Since we page the page table, the page number is further divided into a 10-bit page number and a 10-bit offset. This a logical address is as follows: 

![[Pasted image 20240402181653.png || 300 x 50]]
 

Where p1 is an index  into the outer page table and p2 is the displacement within the page of the inner page table. 

This scheme is also known as a forward-mapped page table as the address translation works from the outer page table inward.

> VAX minicomputer from DEC(DIGITAL EQUIPMENT CORPORATION).

VAX supported 2 level paging. The VAX is a 32 bit machine with a page size of 512 bytes. The logical address space of a process is divided into four equal sections, each of which consists of 2^30 bytes. 

First 2 high-order bits of the logical address designate the appropriate section. 
The next 21 bits represent the logical page number of that section, and the final 9 bits represent an offset in the desired page.

Partitioning the page table this way, the OS can leave partitions unused until a process needs them.

This greatly decreases the amount of memory needed to store virtual data structures. 

To further reduce main-memory use, the vax pages the user-process page tables. 


For a 64 bit logical address space, a two-level paging scheme is no longer appropriate. 
The outer page can further be paged in 64 bit logical address spaces. 
![[Pasted image 20240402183302.png  || 300x50]]
The outer page table is still 234 bytes (16 GB) in size.
The next step would be a four-level paging scheme, where the second-level
outer page table itself is also paged, and so forth. The 64-bit UltraSPARC would require seven levels of paging—a prohibitive number of memory accesses—to translate each logical address. You can see from this example why, for 64-bit architectures, hierarchical page tables are generally considered inappropriate.

### 5.6.2 Hashed Page Tables

Common approach for handling address spaces larger than 32 bits is to use a hashed page table, with the hash value being the virtual page number. 
Each entry in the hash table contains a linked list of elements that hash to the same location to handle collisions. 
Each element consists of three fields : 
1. the virtual page number
2. the value of the mapped page frame
3. a pointer to the next element in the linked list

The algorithm works as follows : 

* The Virtual page number in the virtual address is hashed into the hash table. 
* The virtual page number is compared with field 1 in the first element in the linked list. 
* If there is a match, the corresponding page frame (ﬁeld 2) is used to form the desired physical address. If there is no match, subsequent entries in the linked list are searched for a matching virtual page number.

*Clustered Page Tables* => similar to hashed page tables except that each entry in the hash table refers to several pages (such as 16) rather than a single page. Therefore a single page-table entry can store the mappings for multiple physical-page frames. 
Clustered page tables are particularly useful for sparse address spaces, where memory references are noncontinuous and scattered throughout the address space. 


### 8.6.3 Inverted Page Tables

Usually each process has an associated page table. 
Each page table has one entry for each page that the process is using (or one slot for each virtual address, regardless of the later's validity). 
This table representation is a natural one, since processes reference pages through the pages' virtual addresses. 
The OS must then translate this reference into a physical memory address.
Since the table is sorted by virtual address, the OS is able to calculate where in the table the associated physical address entry is located and to use that value directly. 
One of the drawbacks of this method is that *each page table may consists of millions of entries*. 
This table representation is a natural one, since processes reference pages through the pages' virtual addresses. 
The OS must then translate this reference into a physical memory address.
The Since the table is sorsorted by virtual address, the OS is able to calculate where in the table the associated physical addressaddress entry is located and to use that vale value directly. 
One of the drawbacks of the this method is that *each page table may consists of millions of entries*. 

To solve this problem we can use an inverted page table. 

* An inverted page table has one entry for each real page  (or frame) of memory.
*  Each entry consists of the virtual address of the page stored in that real memory location, with information about the process that owns the page.
* Thus, only one page table is in the system and it has only one entry for each page of physical memory.
* Inverted page tables often require than an address-space identifier be stored in each entry of the page table, since the table usually contains several different address spaces mapping physical memory.
* Storing the address-space identiﬁer ensures that a logical page for a particular process is mapped to the corresponding physical page frame. Examples of systems using inverted page tables include the 64-bit UltraSPARC and PowerPC.

![[Pasted image 20240402201429.png || 400x300]]


IBM => One of the first companies to use inverted page tables, eg IBM RT.
Starting with IBM System 38 and continuing through the RS /6000 and the current IBM Power CPUs. For the IBM RT, each virtual address in the system consists of a triple : 

<process-id, page-number, offset>

Each inverted page-table entry is a pair <process-id, page-number> where the process-id assumes the role of the address-space identiﬁer. When a memory reference occurs, part of the virtual address, consisting of <process-id, page-number>, is presented to the memory subsystem. The inverted page table is then searched for a match. If a match is found—say, at entry i—then the physical address <i, offset> is generated. If no match is found, then an illegal address access has been attempted.

This scheme decreases the amount of memory needed to store each page table, it increases the time needed to search the table when a page reference occurs. 

The whole table might need to be searched before a match is found. This Search would take far too long.

To alleviate this problem, we use a hash table, to limit the search to one or at most a few page table entries. 
One virtual memory reference requires at least two real memory reads -- one for the hash-table entry and one for the page table.

Systems that use inverted page tables have difficulty implementing shared memory. Shared memory is usually implemented as multiple virtual addresses(one for each process sharing the memory) that are mapped to one physical address.
This standard method cannot be used with inverted page tables; 
because there is only one virtual page entry for every physical page, one physical page cannot have two(or more) shared virtual addresses. 

A simple technique for addressing this issue is to allow the page table to contain contain only one mapping of a virtual address to the shared physical address. This means that references to virtual addresses that are not mapped result in page faults.

### 8.6.4 Oracle SPARC Solaris

* There are two hash tables—one for the kernel and one for all user processes.
* Each maps memory addresses from virtual to physical memory.
* Each Hash-table entry represents a contiguous area of mapped virtual memory, which is more efficient than having a separate hash-table entry for each page. 
* Each entry has a base address and a span indicating the number of pages the entry represents.
* CPU implements a TLB that holds translation table entries (TTEs) for fast hardware lookups.
* A cache of these TTEs reside in a translation storage buffer(TSB), which includes an entry per recently accessed page.
* If no translation is found in TLB then the hardware walks through the in memory TSB looking for the TTE that corresponds to the virtual addresses that caused the lookup.
* This TLB walk functionality is found on many modern CPUs.
* If no match is found in the TSB, the kernel is interrupted to search the hash table. 
* The kernel then creates a TTE from the appropriate hash table and stores it in the TSB for automatic loading into the TLB by the CPU memory-management unit.
## 8.7 Example Intel 32 and 64-bit Architecture


Both 8086 and 8088 were based on segmented architecture.
IA-32 supported both paging and segmentation.

### 8.7.1 IA-32 Architecture
Memory management in IA-32 systems is divided into two components : 
1. Segmentation
2. Paging

* CPU generates logical addresses which are given to the segmentation unit
* The segmentation unit produces a linear address for each logical address.
* Linear address is then given to the paging unit, which in turn generates the physical address in main memory. 
* Segmentation and paging units form the equivalent of the memory-management unit

#### IA-32 Segmentation

The IA-32 architecture allows a segment to be as large as 4 gigs. 
Max number of segments per process is 16 k. 

The logical address space of a process is divided into two partitions : 
1. upto 8k segments that are private to that process.
2. upto 8k segments that are shared among all the processes. 
Information about the first partition is kept in the local descriptor table (LDT) 
Information about the second partition is kept in the global descriptor table(GDT) 

The logical address is a pair where the selector is a 16 bit number :
![[Pasted image 20240403003709.png || 100x50]]

S = segment number
G = GDT or LDT
p = protection

The offset is a 32-bit number specifying the location of the byte within the segment in question.

The machine has 6 segment registers, allowing 6 segments to be addressed at one time by a process.

It also has 8-byte microprogram registers to hold the corresponding descriptors from either the LDT or GDT.

This cache lets the Pentium avoid having to read the descriptor from memory for every memory reference.
![[Pasted image 20240403004144.png]]

Linear address of the IA-32 is 32 bits long and is formed as follows : 

Segment register points to the appropriate entry in the LDT or GDT. 
The base and limit information about the segment in question is used to generate a *linear address*. 

* First the limit is used to check the address validity. 
* If the address isn't valid then a memory fault is generated, resulting in a trap to the OS. 
* If it is valid then the value of the offset is added to the value of the base, resulting in a 32-bit linear address. 

#### 8.7.1.2 IA-32 Paging

The IA-32 architecture allows a page size of either 4kb or 4mb. For 4kb pages, IA-32 uses a two-level paging scheme in which the division of the 32 bit linear address is as follows : 
![[Pasted image 20240403004920.png | | 200x50]]

The 10 high-order bits reference an entry in the outermost page table, 
which IA-32 terms the page directory. 
CR3 register points to the page directory for the current process.

The page directory entry points to the inner page table that is indexed by the contents of the innermost 10 bits in the linear address. Finally the low order bits 0-11 refer to the offset in the 4kb page pointed to in the page table.

One entry in the page directory is the Page_Size flag, which if set indicates that the size of the page frame is 4 mb and not the standard 4kb.

If this flag is set, the page directory points directly to the 4 mb page frame, 
bypassing the inner table; 

and the 22 low order bits in the linear address refer to the offset in the 4mb page frame.

IA -32 Page tables can be swapped to disk to improve efficiency. 
In this case an invalid bit is used in the page directory entry to indicate whether the table to which the entry is pointing is in memory or on disk.

When the table is on disk, the OS can use the other 31 bits to specify the disk location of the table.
The table can be then brought into memory on demand.

When devs discovered the 4 gig memory limitations,
Intel adopted *page address extension PAE* which allows 32 bit processors to access a physical space larger than 4 gigs. 
PAE => paging went from 2 level to 3 level. Where top 2 bits refer to a *page directory pointer table*.
![[Pasted image 20240403101353.png || 300 x 150]]


PAE increase the page directory and page table entries from 32 to 64 bits in size, which allowed the base address of the age tables and the page frames to extend from 20 to 24 bits. Combined with the 12 bit offset, adding PAE support to IA-32 increased the address space to 36 bit, which supports upto 64 gigs of physical memory.

OS support is required to required to use PAE.
linux and intel mac os x support PAE. 
32 bit versions of windows desktop OSes still provide support for only 4 gigs of physical memory, even if PAE is enabled. 


### 8.7.2 

AMD's x86-64 architecture is based on extending the existing IA-32 instruction set.
In practice far fewer bits than 64 are used for address space representation in current designs. 

x86-64 architecture currently provides a 48 bit virtual address with support for page sizes of 4 KB, 2 MB, or 1 GB using 4 levels of paging hierarchy.
Since the addressing scheme uses PAE, Virtual addresses are 48 bits but support 52 bit physical addresses. 




# Virtual Memory

Virtual memory is a technique that allows the execution of processes
that are not completely in memory.
Advantages are as follows : 
* Programs can be larger than physical memory.
* Virtual Memory abstracts main memory into an  extremely large, uniform array of storage, separating logical memory as viewed  by the user from the physical memory.
* This technique frees the programmer from the concerns of memory-storage limitations.
* Virtual memory allows users to share files easily and to implement shared memory. 
* It provides an efficient mechanism for process creation. 

Virtual memory may substantially decrease performance if used carelessly.

## 9.1 Background

Instructions being executed must be in physical memory.
First approach to meet this requirement is to place the entire logical address space in physical memory.
Dynamic loading helps ease this restriction.
This limits the size of a program to the size of physical memory.

In many cases the entire program is not needed.
Even in the cases where the entire program is needed, it may not all be needed at the same time.

The ability to execute a program that is only partially in memory would confer many benefits : 

1. A program would no longer be constrained by the amount of physical that is available.  Users would be able to write programs for an extremely large virtual address space, simplifying the programming task.
2. If each user program takes less memory then more programs can run at the same time. (i) Increase in CPU utilization (ii) throughput increase. BUT no increase in (i) response time or (ii) turnaround time.
3. Less I/O would be needed to load or swap user programs into memory, so each user program would run faster.

Running programs that are not entirely in memory would benefit both 
* System 
* User

Virtual Memory involves the separation of memory as perceived by users from physical memory.


*Virtual address space* of a process refers to the logical view of how a process is stored in memory. 

* The heap is allowed to grow upward in memory as it is used for dynamic memory allocation.
* Stack is grown downward in memory through successive function calls. 
* The large blank space ( or hole )  between the head and the stack is part of the virtual address space but will require actual physical pages only if the heap or stack grows.
The virtual address spaces that include holes are known as *sparse* address spaces.

Using a parse address space is beneficial because the holes can be filled as the stack or the heap segments grow or if we wish to dynamically link libraries during program exection.

Virtual memory allows the files and memory to be shared by two or more processes through page sharing.

This has the following benefits : 
* System libraries can be shared by several people through mapping of the shared object into a virtual address space, the actual pages where the libraries reside in physical memory are shared by all the processes. 
  Typically a library is mapped read-only into the space of each process that is linked with it.
* Processes can also share memory. Virtual memory allows one process to create a region of memory that it can share with another process. Processes sharing this region consider it a part of their own virtual address space.
* Pages can be shared during process creation with the fork() system call.


## 9.2 Demand Paging


Load pages only as they are needed. This strategy is called demand paging, commonly used in virtual memory systems.

A demand-paging system is similar to a paging system with swapping where processes reside in the secondary memory. We swap the entire process into the memory when we wanna use it. We use a *lazy swapper*.
A lazy swapper never swaps a page into memory unless that page will be needed.

In the context of demand-paging system, the use of the term "swapper" is technically incorrect. A swapper manipulates the entire process, whereas a pager is concerned with the individual pages of  a process.  

### 9.2.1 Basic Concepts

 process is to be swapped in, the pager guesses which pages will be used  before the process is swapped out again.
 With this scheme we need some form of hardware support to distinguish between the pages  that are in memory and the pages that are on the disk. 

The valid-invalid bit scheme can be used here, however when this bit is set to "valid", the associated page is both legal and in memory. If the bit is set to "invalid", the page either is not valid or is valid but is currently on the disk. 

The page table entry for a page that is not currently in memory is marked invalid or it contains the addreess of the page on the disk. 

The process executes and accesses pages that are *memory resident,*  execution proceeds normally. 


> What happens if the process tries to access a page that was not brought into the memory ? 

Access to a page marked invalid cases a page fault. The paging hardware in translating the address through the page table, will notice that the invalid bit is set, causing a trap to the OS.

**Procedure For Handling The Page Fault : **

1. We check an internal table(usually kept with the process control block) for this process to determine whether the reference was a valid or an invalid memory access. 
2. If the reference was invalid, we terminate the process. If it was valid but we have not yet brought in that page, we now page it in.
3. We find a free frame  (by taking one from the free-frame list)
4. we schedule a disk operation to read the desired page into the newly allocated frame. 
5. when the disk read is compete, we modify the internal table kept with the process and the page table to indicate that the page is now in memory.
6. We restart the instruction that was interrupted by the trap. The process can now access the page as though it had always been in memory.


We can also start executing in a process with no pages in memory. When the OS sets the instruction pointer to the first  instruction of the process, which is on a non-memory resident page, the process immediately faults for the page. This continues until it can execute with no more faults, This scheme is called **Pure Demand Paging**.

Theoretically, some programs could access several new pages of memory with each instruction execution (one page for the instruction and many for data),  possibly causing multiple page faults per instruction this slow down the system considerably but analysis of running processes shows that this behavior is extremely unlikely. 
Programs tend to have locality of reference which results in reasonable performance from demand paging.


* Page Table : marks an entry invalid through a valid-invalid bit or special protection bits


Crucial requirement for demand paging is the ability to restart any instruction after a page fault.

We save the state :
* registers
* condition code
* Instruction counter
of the interrupted process when the page fault occurs we must be able to restart the process in exactly the same place and state, except that the desired page is now in memory and is accessible and is accessible.

A page fault may occur at any memory reference.
If page fault occurs on *the instruction fetch*, we can restart by fetching the instruction again
If page fault occurs on *fetching an operand*, we must fetch and decode the instruction again and then fetch the operand.

> A worst-case example, consider a three address instruction such as an ADD the content of A to B, placing the result in C. These are the steps to execute this instruction : 

1. Fetch and decode the instruction *ADD*
2. Fetch A
3. Fetch B
4. Add A and B
5. Store the sum in C

when fault happens at storing in C, we have to get the desired page, correct the page table and restart the instruction meaning 1..5

The major difficulty arises when one instruction may modify several different locations. 

Demand paging cannot be added to any system.  


### 9.2.2 Performance of Demand Paging

For most computers memory access time (ma) ranges from 10 to 200 nanoseconds.
As long as we have *no* page faults the *effective access time* is equal to the the memory access time.

If a page fault occurs we must first read the relevant page from the disk and then access the desired word.

Let p be the probability of a page fault, we would expect p to be close to zero, The effective access time is then, 

		effective access time = (1 - p) x ma + p x page fault time

To compute the EAT we must know how much time is needed to service a page fault.

Sequence when a page fault occurs : 

1. Trap to the OS
2. Save the user registers and process state
3. Determine that the interrupt was a page fault
4. check that the page reference was legal and determine the location of the page on the disk
5. Issue a read from the disk to a free frame : 
	   a. Wait in a queue for this device until the read request is serviced 
	   b. Wait for the device seek and/or latency time
	   c. Begin the transfer of the page to a free frame
6. While waiting allocate, allocate the CPU to some other user(CPU scheduling optional)
7. Receive an interrupt from the disk I/O subsystem (I/O completed) 
8. Save the registers and process state for the other user (If step 6 is executed) 
9. Determine that the interrupt was from the disk.
10. Correct the page table and other tables to show that the desired page is now in memory.
11. Wait for the CPU to be allocated to this process again.
12. Restore the user registers, process state, and new page table, and then resume the interrupted instruction

We are faced with three major components of page fault service time  at any given case : 

1. Service the page fault interrupt 
2. Read in the page
3. Restart the process

The first and the third tasks can be reduced with careful coding, to several hundred instructions.
These tasks may take from 1 to 100 ms each.
The page switch time will probably be close to 8 ms.
A typical hard drive disk has an avg latency of 3 ms, a seek of 5 ms, and a transfer time of 0.05 ms inc HW and SW time.

If a queue process is waiting for the device, we have to add device-queuing time as we have to wait for the paging device to be free to service our request, increasing even more the time to swap.

With an avg page-fault service time of 8ms and a memory access time of 200ns the effective access time in ns is : 

> Effective Access time = (1 - p) x (200) + p(8 ms)
   \                                               = (1 - p)  x 200 + p x 8,000,000
> \                                            = 200 + 7999800 x p

The effective access time is directly proportional to the *page-fault rate*. 
If one access out of 1000 causes a page fault, the effective access time is 8.2 microseconds. The computer will be slowed down by a factor of 40 because of demand paging! if we want performance degradation to be less than 10% then we need to keep the probability of page faults at p < 0.0000025

That is fewer than one memory access out of 399,990 to page-fault.

*Disk I/O to swap space is generally faster than that to the file system*, it is a faster file system because swap space is allocated in much larger blocks, and file lookups and indirect allocation methods are not used. 

The system can therefore gain better paging throughput by copying an entire file image into the swap space at process startup and then performing demand paging from the swap space. Another option is to demand pages from the file system initially but to write the pages to swap space as they are replaced. 
This will ensure that only the needed pages are read from the file system but that all subsequent paging is done from swap space. 


Some systems attempt to limit the amount of swap space used through demand paging of binary files. Demand pages for such  files are brought directly from the file system. However when page replacement is called for these frames can simply be overwritten (because they are never modified) and the pages can be read in from the file system again if needed. Using this approach the file system itself serves as the backing store. 

However the swap space must still be used for pages not associated with a file (known as anonymous memory);
These pages include the stack and the heap for a process. This method appears to be a good compromise and is used in several systems, including Solaris and BSD UNIX.

## 9.3 Copy on Write

This technique works by allowing the parent and child process initially to share the same pages. These shared pages are marked as copy-on-write pages, meaning that if either process writes to a shared page, a copy of the shared page is created.
Copy on write is a common technique used by several OSes including : 
1. Win XP
2. Linux
3. Solaris

It is important to know the location of the free page that gets allocated when a page marked copy-on-write is duplicated. Many OSes provide a pool of free pages for such requests. 
These free pages are typically allocated when the stack or heap for a process must expand or when there are copy-on-write  pages to be managed. 
OSes typically allocate these pages using a technique known as *zero-fill-on-demand*. 

Zero-fill-on-demand pages have been zeroed out before being allocated thus erasing the previous contents. 

Several versions of UNIX (including solaris and linux) provide a variation of the fork() system call - vfork() (for virtual memory fork) -that operates differently from the fork() with copy-on-write . 
With vfork() the parent process is suspended, and the child process uses the address space of the parent. 

Because vfork() does not use copy-on-write if the child process changes any pages of the parent's address space, the altered pages will be visible to the parent once it resumes. Thus it must be used with caution to ensure that the child process does not modify the address space of the parent. vfork() is intended to be used when the child process calls exec() immediately after creation. Because no copying of pages takes place, vfork() is an extremely efficient method of process creation and is sometimes used to implement UNIX command-line shell interfaces.


## 9.4 Page Replacement


If we increase our degree of multi-programming, we are over-allocating memory.
Suppose we run 6 processes, each of which is 10 pages in size but actually only uses 5 pages, we have a higher CPU utilization and throughput with ten frames to spare(when we take total as 40). It is possible however that each of these processes for a particular data set, may suddenly try to use all ten of its pages, resulting in a need for sixty frames when only forty are available. 

System memory is not only used for holding program pages. Buffers for I/O also consume a considerable amount of memory. This use can increase the strain on memory placement algorithms. 

Deciding how much memory to allocate for I/O and for user processes is a big challenge.
Some systems allocate a fixed % for each. Others let the two compete.

Memory over-allocation : 

* While a user process is executing, a page fault occurs, indicating that the process needs a page that is not currently in physical memory.
* The OS determines the location of the desired page on the disk.
* However when the OS checks the free-frames list, it finds that there are no free frames available in the physical memory. All memory is currently in use by other process or system data.

The OS has several options at this point. It could terminate the user process. However this option isn't the best choice as terminating the process violates the transparency in demand paging and is generally not the best choice.

The OS could instead swap out a process freeing all its frames and reducing the level of multi-programming. This option is a good one in certain circumstances. 

The most common solution here is *Page Replacement*

### 9.4.1 Basic Page Replacement


If no frame is free then we find one that isn't currently in use and free it.
We free a frame by : 
1. writing its contents to a swap space
2. and then changing the page table to indicate that the page is no longer in memory

We now use the freed frame to hold the page for which the process faulted. 
We modify the page fault routine to include page replacement.

If no frames are free then two page transfers are required.
We can reduce this overhead by using a modify bit or a dirty bit. This technique also applies to read-only pages(ex pages of binary code)

This scheme greatly reduces the time required to service a page fault, since it reduces I/O time by one half if the page has not been modified. 

We must solve two major problems to implement demand paging : 

1. We must develop a frame allocation algorithm
2. We must develop a page replacement algorithm

We evaluate an algorithm by running it on a particular string of memory references and computing the number of page-faults.

The string of memory references is called a reference string. We can generate the reference strings artificially(by using a random-number generator) or we can trace a given system and record the address of each memory reference. 

To reduce the amount of data when tracing a real system, two facts are exploited:
1. Only the page number needs to be considered, not the entire address, since the page size is fixed by the hardware or system.
2. If there are multiple consecutive references to the same page, only the first reference needs to be recorded, as subsequent references will not cause a page fault.

![[Pasted image 20240409160802.png || 300 x 200]]

### 9.4.2 FIFO Page Replacement


Simplest page-replacement algorithm is a first-in, first-out (FIFO) algorithm. 
A FIFO replacement algorithm associates with each page the time when that page was brought into memory. 
When a page must be replaced, the oldest page is chosen.
We can create a FIFO Queue to hold all pages in memory.  We replace the page at the *head* of the queue. When a page is brought into memory we insert it at the tail of the queue.
![[Pasted image 20240409162203.png]]

**Belady's Anomaly** : For some page-replacement algorithms the page fault may increase as the number of allocated frames increases. 

### Optimal Page Replacement 

It is the algorithm that has the lowest page-fault rate and will never suffer from Belady's anomaly. Such an algorithm and does exist and has been called OPT or MIN. It is simply this : 

*Replace the page that will not be used for the longest period of time.*

Use of this page-replacement algorithm guarantees that the lowest possible page-fault rate for a fixed number of frames. 

![[Pasted image 20240410233456.png || 300x200]]


It works by scanning reference string for the next least used frame and that gets replaced. 

This is very difficult to implement because it requires the future knowledge of the reference string. 
As a result optimal algorithm is used mainly for comparison studies.


### 9.4.4 LRU Page Replacement

If we use the recent past as an approximation for near future then we can replace the page that has not been used for the longest period of time. 

LRU replacement associates with each page the time of that page's last use. When a page must be replaced, LRU chooses the page that has not been used for the longest period of time. 

If we let S^R be the reverse of a reference string S, then the page-fault rate for the OPT algorithm on S is the same as the page-fault rate for the OPT algorithm on S^R,
Similarly the page fault rate for LRU algorithm on S is the same as the page-fault rate for LRU algorithm on S^R.

The LRU policy is often used as a page-replacement algorithm and is considered to be good. The major problem is how to implement LRU replacement. An LRU page-replacement algorithm may require substantial hardware assistance. The problem is to determine an order for the frames defined by the time of last use. 

There are two implementations that are feasible : 
1. *Counters* : We associate with each page-table entry a time of use field and add to the CPU a logical clock or counter.  The clock is incremented for every memory reference. 
   Whenever a reference to a page is made, the contents of the clock register are copied to the time of use field in the page table entry for that page. In this way we always have the time of the last reference to each page. We replace the page with the smallest time value. This scheme requires a search of the page table to find the LRU page and a write to memory for each memory access. The times must also be maintained when the page tables are changed (due to CPU scheduling). Overflow of the clock must be considered. 
2. *Stack* : Keep a stack of page numbers. Whenever a page is referenced, it is removed from the stack and put on the top. In this way, the most recently used page is always at the top of the stack and the least recently used page is always at the bottom. Because entries must be removed from the middle of the stack. it is best to implement this approach using a doubly linked list with a head pointer and a tail pointer. Removing a page and putting it on the top of the stack then requires changing 6 pointers at worst. Each update is a little more expensive, but there is no search for a replacement. The tail pointer points to the bottom of the stack, which is the LRU page.  This approach is particularly appropriate for software or microcode implementations of LRU replacement.
LRU replacement doesn't suffer from Belady's anomaly. Both belong to a class of page-replacement algorithms called stack algorithms, that can never exhibit Belady's anomaly. A stack algorithm is an algorithm for which it can be shown that the set of pages in memory for n frames is always a subset of the set of pages that would be in memory with n + 1 frames. 

Neither implementation of LRU is conceivable without the hardware assistance beyond the standard TLB registers. The updating of the clock fields or stack must be done for every memory reference. 

If we were to use an interrupt for every reference to allow software to update such data structures, it would slow every memory reference down by a factor of atleast ten.

### LRU Approximation Page Replacement

Very few computers provide sufficient hardware support for true LRU page replacement.

Many systems provide some help however in the form of a reference bit. The reference bit for a page is set by the hardware whenever that page is referenced (either a read or a write to any byte in the page). 
Reference bits are associated with each entry in the page table.
We do not know the order of use through this reference bit.

#### Additional-Reference Bits algorithm

We can maintain a history of the reference bits using a byte of data in which bits are regularly shifted and the low ordering bit is discarded. These 8-bit shift registers contain the history of page use for the last eight time periods. A page with a history register value of 11000100 has been used more recently than one with a value of 01110111.

If we interpret these 8 bit bytes as unsigned integers then the page with the lowest number is the LRU Page, and it can be replaced. The numbers are not guaranteed to be unique. We can either replace all the pages with the smallest value or use the FIFO method to choose among them.

The no of bits of history included in the shift register can be varied, of course and is selected (depending on the hardware available) to make the updating as fast as possible. In the extreme case, the number can be reduced to zero, leaving the reference bit itself. 
This algorithm is called the second-chance page-replacement algorithm.

#### Second-chance algorithm

The basic algorithm of second-chance replacement is a FIFO replacement algorithm. When a page has been selected however, we inspect its reference bit. If the value is 0, then we proceed to replace this page; but if the value is 1 then we give this page a second chance and move to the next selected FIFO page. When a page gets a second chance, its reference bit gets cleared and its arrival time is reset to the current time. 

Clearing the reference bit ensures that the algorithm can accurately track the page's usage after it has been given a second chance, allowing it to make a more informed decision about whether to replace the page or give it another chance the next time it is considered for replacement.

A page that is given  a second chance will not be replaced until all other pages have been replaced. In addition if a page is used often enough to keep its reference bit set, it will never  be replaced. 
 
One way to implement the second chance algorithm(sometimes referred to as the clock algorithm) is a circular queue. A pointer ( that is a hand on the clock) indicates which page is to be replaced next. When a frame is needed the pointer, advances until it finds a page with a 0 reference bit. As it advances it clears the reference bits. Once a victim page is found the page is replaced and the new page is inserted in the circular queue in that position. 

Notice that in the worst case scenario when all the bits are set, the pointer cycles through the whole queue giving each page a second chance. It clears all the reference bits before selecting the next page for replacement. Second-chance replacement degenerates into FIFO replacement if all the bits are set.

#### 9.4.5.3 Enhanced Second-Chance algorithm 

We can enhance the second chance algorithm by considering the reference bit and and the modify bit as an ordered pair. With these two bits we have the following classes : 

1. (0, 0) neither recently used nor modified -> best page to replace
2. (0,1) not recently used but modified -> not as good as before
3. (1, 0) recently used and clean -> probably will be used again
4. (1, 1) Recently used and modified -> probably will be used again soon and the page will be need to be written out to disk before it can be replaced.

Each page is one of those four classes. When page replacement is called for we use the same scheme as in the clock algorithm; but instead of examining whether the page to which we are pointing has the reference bit set to 1, we examine the class to which that page belongs. We replace the first page encountered in the lowest nonempty class. Notice that we  may have to scan the circular queue several times before we find a page to be replaced. 


### 9.4.6 Counting-Based Page replacement

We can keep a counter of the number of references that have been made to each page and develop the following : 

: 

1. Least Frequently Used : This algorithm requires that the page with the smallest count be replaced. Reason : actively used page should have large reference count. This runs into a problem that a process that was heavily used initially stays in the memory even though it is no longer needed. One solution is to shift the counts right by 1 bit at regular intervals, forming an exponentially decaying average usage count.
2. Most Frequently Used :  based on the argument that the page that was used smallest count was just brought in to be used.


### 9.4.7 Page-Buffering Algorithms

Systems often keep a pool of free frames, when a page fault occurs a victim frame is chosen as before, however the desired page is read into a free frame before  the victim is written out, this ensures the process to restart asap.

An expansion of this idea is to maintain a list of modified pages, whenever the paging device is idle, a modified page is selected and written to the disk. Its modify bit is then reset. 
This scheme increases the probability that a page will be clean when it is selected for replacement and will not need to be written out.

Another modification is to keep a pool of free frames but to remember which page was in each frame. Since the frame contents are not modified when a free frame is written to the disk, the old page can be reused directly from the free-frame pool if it is needed before that frame is reused. No I/O is needed in this case. 
When a page fault occurs, we first check whether the desired page is in the free frame pool. If it is not we must select a free frame and read into it.

This technique is used in the VAX/VMS system along with a FIFO replacement algorithm. When the FIFO replacement algorithm mistakenly replaces a page that is still in active use, that page quickly retrieved from the free-frame pool and no I/O is necessary. The free frame buffer provides protection against the relatively poor but simple FIFO replacement algorithm. This method was necessary because the older versions of VAX did not implement the reference bit correctly. 

Some versions of unix system use this method in conjunction with the second chance algorithm. It can be a useful augmentation to any page-replacement algorithm, to reduce the penalty incurred if the wrong victim page is selected. 

### 9.4.8 Applications and Page Replacement

In some cases applications accessing data through the OS's virtual memory perform worse than if the OS provided no buffering at all. A typical example is a database. 

Another example is data warehouse computers. Here MFU is better than LRU.

Because of these problems Some OSes give special programs the ability to use a disk partition as a large sequential array of logical blocks, without any file-system data structures.  This array is sometimes called the *raw disk* and the I/O to this array is called raw I/O, Raw I/O bypasses all file system services like 

1. I/O demand paging
2. file locking
3. prefetching
4. space allocation
5. file names
6. directories

Most applications perform better when they use the *regular* file-system services. 

## 9.5 Allocation of Frames 

Basic Strategy : The user process is allocated any free frame.

### 9.5.1 Minimum Number of Frames

We can allocate more than the total number of frames only if there is page sharing.

Our one reason for allocating atleast a minimum number of frames involves performance. 
The minimum number of frames is defined by the computer architecture.
PDP-11 includes more than one word for the move instruction.
In addition each of its two operands may be indirect references, for a total of 6 frames.
IBM 370 :- 6(8 at max) frames for mvc instruction. 

Worst case scenario occurs in a computer architectures that allow multiple level of indirection. chaining of references can happen and the entire virtual memory might get touched. So in the worst case scenario the entire virtual memory must be in the physical memory. To prevent this we have to limit the depth of indirection. 

Example to limit an instruction to at most 16 levels, when the first indirection occurs a counter is set to 16; the counter is then decremented for each successive indirection for this instruction. If the counter is decremented to 0, a trap occurs (excessive indirection). 
This limitation reduces the maximum number of memory references per instruction to 17, requiring the same number of frames.

Whereas the minimum number of frames per processes is defined by the architecture, the maximum number is defined by the amount of available physical memory.

### 9.5.2 Allocation Algorithms 

*Equal Allocation* : give everyone equal amount of frames, left over frames can be used as a free frame buffer pool.

*Proportional Allocation* : We allocate available memory to each process according to size. Let the size of the virtual memory for process Pi be Si and define 

\                                              S = Summation Si
If the total number of available frames is m, we allocate Ai frames to process Pi, where Ai is approximately 
\                                             Ai = Si/S x m

Of course we must adjust each Ai to be an integer that is greater than the minimum number of frames required by the instruction set, with a sum not exceeding m.

With proportional allocation we would split the 62 frames between two processes, on of 10 pages and another of 57 frames respectively since 

10/137 x 62 ~ 4 and 127/137 x 62 ~ 57

In both equal and proportional allocation the allocation may vary according to the minimum multi-programming level. 

If the multi-programming level is increased, each process will lose some frames to provide the memory needed for a new process. 

With either of these allocation types, a high priority process is treated the same way as a low priority process. 


### Global vs Local Allocation 

With multiple processes competing for frames, we can classify page-replacement algorithms into two broad categories : 

1. Global Replacement : Allows a process to select a replacement frame from the set of all frames, even if that frame is currently  allocated to some other process. i.e one process can take a frame from another process.
2. Local Replacement : requires that each process select from only its own set of allocated frames.

One problem with global replacement algorithm is that a process cannot control its own page-fault rate. 
The set of pages in memory for a process depends not only on the paging behavior of that process but also on the paging behavior of other processes. <= this was only for Global.

Local replacement might hinder a process, however by not making available to it other, less used pages of memory. 

*Thus global replacement generally results in better system throughput and is therefore more commonly used method.*

### 9.5.4 Non Uniform Memory Access

In systems with multiple CPU's some CPUs can access some sections of the main memory faster than the others. 
Systems in which memory access times vary significantly are known collectively as *non-uniform memory access (NUMA)* systems, and without exception, they are slower than systems in which memory and cpus are located on the same motherboard. 

Managing which page frames are stored at which locations can significantly affecct performance in NUMA systems. 

Algorithmic  changes on NUMA : 

1. Having the scheduler track the last CPU on which each process ran.
2. If the scheduler tries to schedule each process onto its previous CPU, and the memory-management system tries to allocate frames for the process close to the CPU on which it is being scheduled, then improved cache hits and decreased memory access times will result.

Solaris solves the problem of of multiple threads running at different system boards by creating *lgorups* (for latency groups) in the kernel. Each lgroup fathers together close CPUs and memory. In fact, there is a hierarchy of lgroups gathers together close CPUs and memory. In fact, there is a hierarchy of lgroups based on the amount of latency between the groups. 
Solaris tries to schedule all threads of a process and allocate all memory of a process within an lgroup. If that is not possible it picks nearby lgroups for the rest of the resources needed. This practice minimizes overall memory latency and maximizes CPU cache hit rates.


## 9.6 Thrashing 

If the number of frames allocated to a low-priority process falls below the minimum number required by the computer architecture, we must suspend that process's execution.
We should then page out its remaining pages, freezing all its allocated frames. This provision introduces a swap-in swap-out level of intermediate CPU scheduling. 

Any process that does not have "enough" frames, If the process does not have the number of frames it needs to support pages in active use, it will quickly page fault.

as it will replace a page which is in use and it will page fault again, this leads to constant page faults and replacements . This high paging activity is called *thrashing*.

A process is thrashing if its spending more time paging than executing.


### 9.6.1 Cause of Thrashing

The OS monitors CPU utilization, if it is low then it increases the degree of multi programming, by adding more processes. A global page replacement algorithm is used, a currently faulting process this takes away frames from other processes and causes the other processes to fault as well, this empties the ready queue and decreases the cpu utilization, this makes the OS increase the level of Multi-programming, which further increases page faults and decreases CPU utilization. This causes thrashing to occur eventually.

So 
1. Thrashing occurs
2. System Throughput reduces
3. page fault rate increases
4. effective memory access time increases
5. no work gets done because processes spend all their time in paging.

![[Pasted image 20240411221941.png || 300x200]]

We can limit the effects of thrashing by using a local replacement algorithm or priority replacement algorithm.

With local replacement if one process starts thrashing, it cannot steal frames from another process and caase the latter to thrash as well, the problem isn't entirely solved as if processes are thrashing, they will be in the queue for the paging device most of the time. 

The avg service time for page fault will increase because of the longer average queue for the paging device. Thus the effective access time will increase even for a process that is not thrashing.

To prevent thrashing we must provide a process with as many frames as it needs. 
The working set strategy starts by looking at how many frames a process is actually using. This approach defines the locality model of  process execution.

The locality model states that as a process executes it moves from locality to locality. A locality is a set of pages that are actively used together, A program is generally composed which may overlap.

When a function is called it defines a new locality. In this locality, memory references are made to the instructions of the function call, its local variables, and a subset of the global variables. 

When we exit the function the process leaves this locality. We may return to this locality later.

Thus we see that localities are defines by the program structure and its data structures. 
If we do not allocate enough frames to accommodate the size of the current locality, the process will thrash, since it cannot keep in memory all the pages that it is actively using.

### 9.6.2 Working-Set Model

Working set model is based on the assumption of locality. This model uses a parameter, Δ, to define the working set window. The idea is to examine the most recent Δ page references. 
The set of pages in the most recent Δ is the working set. If a page is in active use it will be in the working set. If it is no longer being used, it will drop from the working set Δ time units after its last reference. Thus, the working set is an approximation of program's locality.

The accuracy of the working set depends on the selection of Δ, if Δ is too small it will not encompass the entire locality; if it is too large then it may overlap with several localities. *If Δ is infinite, the working set is the set of pages touched during the process exeuction.* 

The most important property of the working set is its size. If we compute the working-set size, WSSi for each process in the system we can then consider that 

D = summation WSSi
Where D is the total demand for frames. Each process is actively using the pages in its working set. Thus, process i needs WSSi frames. 

If the total demand is greater than the available frames (D > m) thrashing will occur, because some processes will not have enough frames.

Once Δ has been selected use of the working set model is simple, the OS monitors the working set of each process and allocates to that working set enough frames to provide it with its working-set size. If there are enough frames then another process can be initiated.  
If the sum increases, exceeding the number of available frames, the OS selects a process to suspend, the process's pages are written out(swapped), and its frames are reallocated to other processes. The suspended process can be restarted later.

Working Set Strategy prevents thrashing while keeping degree of multiprogramming has high as possible.
It optimizes the CPU utilization. 

Main difficulty in implementation is to keep track of the working set.

We can approximate the working set model with a fixed interval timer interrupt and a reference bit. Pages with atleast one bit on will be considered a part of the working set.

This arrangement is not entirely accurate, because we cannot tell where within an interval of 5000, a reference has occurred. We can reduce the uncertainty by increasing the number of history bits and the frequency of interrupts <= Cost to service these will also be higher.

### 9.6.3 Page Fault Frequency

A strategy that uses page-fault frequency takes a more direct and better approach to tackle thrashing. 

Thrashing has a high page fault rate, thus we wanna control the page fault rate. When it is too high we know that the process needs more frames, conversely if the page fault rate is too low, then the process may have too many frames. We can establish upper and lower bounds on the desired page-fault rate. If the actual page-fault rate exceeds the upper limit, we allocate the process another frame. If the page-fault rate falls below the lower limit, we remove a frame from the process. Thus we can directly measure and control the page-fault rate to prevent thrashing. 

Similar to the working set strategy we use the same principles when we may have to swap a process out.

### 9.6.4 Conclusion
The current best practice in implementing a computer facility is to include enough physical memory, whenever possible, to avoid thrashing and swapping. From smartphones through mainframes, providing enough memory to keep all working sets in memory concurrently except under extreme conditions, gives the best user experience.



## 9.10 OS Examples 

Windows implements virtual memory using demand paging with clustering. Clustering handles page faults by bringing in faulting pages and some more pages that follow the faulting page. 

When a process process is first created it is assigned a working set minimum and maximum. 
The virtual memory manager maintains a list of free page frames. Associated with this list is a threshold value that is used to indicate whether sufficient free memory is available or not. If a page fault occurs for a process that is below its working-set maximum, the virtual memory manager allocates a page from this list of free pages. If a process that is at its working-set maximum incurs a page fault, it must select a page for replacement using a local LRU page-replacement policy.

When the amount of free memory falls below the threshold, the virtual memory manager uses a tactic known as automatic working-set trimming to restore the value above the threshold. Automatic working-set trimming works by evaluating the number of pages allocated to processes. If a process has been allocated more pages than its working-set minimum, the virtual memory manager removes pages until the process reaches its working-set minimum. A process that is at its working-set minimum may be allocated pages from the free-page-frame list once sufﬁcient free memory is available. Windows performs working-set trimming on both user mode and system processes.
