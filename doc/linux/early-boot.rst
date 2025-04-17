.. platform documentation

Linux Init (Early Boot)
=======================

During Linux Early Boot stage (functions in the kernel that have the __init
decorator), the system takes the resources created by EFI/BIOS (ACPI tables)
and turns them into resources that the kernel can consume.

Handoffs
--------

* Linux consumes ACPI tables to create software resources.

* Memory map is created from the EFI memory map

  * CXL memory is set aside to be handled by the CXL driver

* NUMA nodes and memory tiers are created from the PXM table

* Contiguous Memory Allocator allocates requested memory for each online numa node.

* Init finishes, Drivers start probing.


NUMA Nodes
----------

Linux takes the PXM (proximity domain) table in the SRAT to create NUMA
nodes in :code:`acpi_numa_init`. Typically, there is a 1:1 correspondance
between the proximity domains and NUMA nodes (exception: see :code:`fake_pxm`.
If there are CXL ranges in the CFMWS but not in SRAT, then a fake proximity
domain is created). As nodes come online, memory tiers are created by grouping
nodes with similar characteristics together.

NUMA nodes are *NOT* hot-pluggable.  All *POSSIBLE* NUMA nodes are identified
at `__init` time, more specifically during `mm_init`.  What this means is that
the CEDT and SRAT must contain sufficient `proximity domain` information for
linux to identify how many NUMA nodes are required (and what memory regions
should be associated with them).

SRAT is the only ACPI defined way of getting Proximity nodes. Linux chooses
to, at most, map those 1:1 with NUMA nodes.

CEDT adds a description of SPA ranges which Linux may wish to map to one or
more NUMA nodes

The relevant code exists in: linux/drivers/acpi/numa/srat.c

Basically, the heuristic is as follows

1. Add one NUMA node per Proximity Domain described in SRAT (one or more of: memory, cpu, generic initiator).
2. If SRAT describes some portion of a CFMWS SPA range - do not create a nodes for that CFMWS
3. If SRAT does not describe memory in a CFMWS SPA range - a node is created for that CFMWS

In the future, Linux may reject CFMWS not described by SRAT due to the ambiguity of proximity domain association.

Generally speaking, you will see one NUMA node per Host bridge, unless inter-host-bridge interleave is in use.  In environments with complex QoS concerns, or designed for runtime configuration flexibility, it may be reasonable to create more than one NUMA node per Host bridge.

EFI_MEMORY_SP
-------------

While the kernel parses the EFI memory map in :code:`do_add_efi_memmap` (x86),
it also checks for the EFI MEM SP bit that was set during the EFI/BIOS stage.
If the bit was set, then that memory range is set aside as :code:`SOFT_RESERVED`
i.e. defer management to the (CXL) driver. Otherwise, it will be onlined as
:code:`RAM` or :code:`RESERVED`, depending on whether the EFI MEM WB (writeback)
bit was set.

Memory Tiers Creation
---------------------
todo

Contiguous Memory Allocation
----------------------------
todo: explain limitations
