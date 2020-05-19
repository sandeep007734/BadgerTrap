In this section we will see the changes made by the BadgerTrap tool.

## Design

The [BadgerTrap] tool counts the total TLB miss of an application. This is little tricky to do. As we know from [the working of the page table](../page_table.md) that in-case of an TLB miss, if the page is present in the DRAM, i.e. if the page table contains a valid entry for the page the hardware will service the TLB miss and the software will not see this. This makes counting TLB miss in the software difficult, unless hardware does it for us.

[BadgerTrap] solved this by marking the page table entry invalid *after* servicing a page-fault, i.e. before returning to the hardware. We will see the exact working in the next few sections. As the entry in the page table is marked invalid, every TLB-miss will be serviced by the kernel, and hence can keep an accurate track of the TLB-misses. Please note that this causes a significant performance overhead (40X) and hence, is only used to profile the applications.

!!! note
    It might not be necessary to mark the page table invalid while servicing a page fault. This can be done during the periodic timer interrupt servicing. However, this might not given an accurate picture of the TLB misses.

## Header file

````c
// Process name limit
#define MAX_NAME_LEN	16
// Use the reserved 52nd bit
#define PTE_RESERVED_MASK	(_AT(pteval_t, 1) << 51)

// A 2D array to check if the current executing process is profiled or not.
char badger_trap_process[CONFIG_NR_CPUS][MAX_NAME_LEN];

// This checks the 2D array and returns TRUE if it is there otherwise FALSE.
int is_badger_trap_process(const char* proc_name);

// PTE updatation tools.
// Set the bit
inline pte_t pte_mkreserve(pte_t pte);
// Un-set the bit
inline pte_t pte_unreserve(pte_t pte);
// Check the bit
inline int is_pte_reserved(pte_t pte);

// PMD update methods, for huge page?
// Set the bit
inline pmd_t pmd_mkreserve(pmd_t pmd);
// Un-set the bit
inline pmd_t pmd_unreserve(pmd_t pmd);
// Check the bit
inline int is_pmd_reserved(pmd_t pmd);

// Initialize the 2D array 
void badger_trap_init(struct mm_struct *mm);

````


## Data structures

## System calls added

## Major changes to the kernel


[BadgerTrap]:  https://github.com/sonarsys/BadgerTrap