# Page table

In this post, we understand the working of a page table in the Linux kernel. As we understand, the memory management is one of the most crucial component of the kernel management. Any change in this has to go through rigorous testing and validation.

Understanding the memory architecture will help us gain an insight into the kerne workings.


## Requirement

![large_drawing](https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/Virtual_address_space_and_physical_address_space_relationship.svg/880px-Virtual_address_space_and_physical_address_space_relationship.svg.png  )
**Virtual to physical address mapping. Source: Wikipedia**

There are multiple purposes of a page table:  

* Program isolation   
	The application does not have to worry about corrupting another application's memory space, willingly or un-willingly. The OS will ensure that any such access will result into a segmentation fault, otherwise know as `SEGSEV` fault.
* Security  
	By ensuring that an another processing cannot snoop into my memory, the OS guarantees that the the content of my memory is protected. Please note that the OS here is a trusted entity as it has access to memory areas of all the programs. 

!!! todo
		How does a program like CheatEngine works then?

* Increase working set size  
	Virtual memory also allows us to execute an application that requires more memory than currently present in the system. In this case, the OS will use the *swap* space to move-out and move-in memory pages as and when required. This is a very neat trick and is completely transparent to the applications. This will add some performance overhead due to swapping, which is typically stored in a slower storage medium, such as HDD or more recently SSD.

## Page table

![image](/images/4-level-page-table.png)
**Address translation for a 32 bit address in an X86 system using 4 level page table. Source: AMD 64 bit Programmers Manual 2 Chapter 5**

As can be seen the figure above, 

1. As can be seen in the figure above, the first 9 bits are used to offset into the *page map table*. The base of the table is obtained from the `CR3` register. This is a new addition to page table hierarchy since the page table moved from the 3-level to 4-level structure to support more memory in the 64-bit systems.
2. Each entry in the *page map table* points to a page directory pointer table, also know as *page global directory* or pgd. This was first level in the 3-level page table, hence, the term *global*. The next 9 bits are used to offset into the PGD.
3. Each PGD points to a page directory table also known as *page middle directory PMD*. Each entry in tha PMD points to a page table. The next 9 bits are used to offset into the page table.
4. The page table is the last level of in-direction. Each entry in tha page table points to a page. The next 9 bits are used to offset into the page table.
5. From the page table, we get the base address of a page. This is the granularity at which the page table operates. The last 12 bits are used to offset into the page table at byte-level address, at which the DRAM operates.

!!! note
	`CR3` is a control register that is used in virtual to physical address translation. Basically, during the context switch operation, the CR3 is populated with the address so as to locate the page table of the current process.

## TLB

Address translation is a costly process. Following the 4-level page table to resolve a virtual address to a physical address is called *page table walk*. This is done by the hardware so as to optimize the speed. Even then, in the worst case a single memory address may result into 4 extra memory accesses to fetch the PageMap, PGD, PMD and PTE. 
In some cases, it might be the case that the page has been swapped out to the disk due to high memory pressure. In that case, the page table will have the `invalid` flag set for that particular page. In this case, the hardware will generate a page-fault and the control is given to the OS. The OS will bring the page into the memory (may be by an eviction), set the page table entry as valid, and give the control to the hardware. 

To optimize this process, the hardware maintains a small cache of the recently translated addresses in a structure called the TLB or *translational lookaside buffer*. It leverage the spatial and the temporal locality of an application.

![large_drawing](https://upload.wikimedia.org/wikipedia/commons/thumb/b/be/Page_table_actions.svg/880px-Page_table_actions.svg.png)
**Translation look aside buffer working. Source: Wikipedia.**

!!! info
	*Spatial locality*: If you access a particular address, then you are likely to access the nearby addresses also.  
	*Temporal locality*: If you access an address, then it is likely that you will access the same address again in the near future.

## Huge page

As the size of the memory is increasing, the reach of the TLB is getting smaller. This causes a large TLB miss in a modern application with large memory footprint. To solve this issue, modern architecture supports pages of different sizes. X86 supports page size of 4KB, 2MB, and 1GB.

This comes from this:

1. Page offset: 12 bits i.e. $2^{12} = 4KB$
2. Page table offset 9 bits. Combine this with the previous 12 bits = $2^{21} = 2MB$.
3. PMD offset 9 bits. Combine this with the previous 21 bits = $2^{30} = 1GB$ 

Other architectures support pages of different sizes based on their design. Interested readers can read the [Haweye paper][hawkeye] to know more about the huge page.

[hawkeye]: https://www.cse.iitd.ernet.in/~sbansal/pubs/hawkeye.pdf