.. Linux Early Boot

Linux Early Boot
================

During Linux Early Boot stage (functions in the kernel that have the __init
decorator), the system takes the resources created by EFI/BIOS (ACPI tables)
and turns them into resources that the kernel can consume.

NUMA Nodes
----------

Linux takes the PXM (proximity domain) table in the SRAT to create NUMA
nodes in :code:`acpi_numa_init`. Typically, there is a 1:1 correspondance
between the proximity domains and NUMA nodes (exception: see :code:`fake_pxm`.
If there are CXL ranges in the CFMWS but not in SRAT, then a fake proximity
domain is created). As nodes come online, memory tiers are created by grouping
nodes with similar characteristics together.

EFI MEM SP
----------

While the kernel parses the EFI memory map in :code:`do_add_efi_memmap` (x86),
it also checks for the EFI MEM SP bit that was set during the EFI/BIOS stage.
If the bit was set, then that memory range is set aside as :code:`SOFT_RESERVED`
i.e. defer management to the (CXL) driver. Otherwise, it will be onlined as
:code:`RAM` or :code:`RESERVED`, depending on whether the EFI MEM WB (writeback)
bit was set.

Handoffs
--------

* Linux consumes ACPI tables to create software resources.

* NUMA nodes and memory tiers are created from the PXM table

* Memory map is created from the EFI memory map

  * CXL memory is set aside to be handled by the CXL driver
