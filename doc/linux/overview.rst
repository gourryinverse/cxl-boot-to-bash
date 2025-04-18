.. Overview of linux kernel config

Overview
########

* Linux Boot Parameters

  * nosoftreserve

* Early Boot

  * Memory Map Creation

    * EFI Memory Map Is Consulted / Created

    * CXL Memory is set aside to be handled by the CXL driver

    * IO Resources are created for each piece

  * NUMA Node Creation

    * ACPI CEDT and SRAT table are used to create Nodes from Proximity domains (PXM)

  * Memory Tier Creation

    * A default tier is created with all nodes.

  * Contiguous Memory Allocation

    * Any requested CMA is allocated from Online nodes

  * Init Finishes, Drivers start probing

* ACPI and PCI Drivers

  * Detect CXL device, marking it for probe by CXL driver

* CXL Driver Operation

  * Base object creation (root, port, memdev)

    * CEDT CFMWS IO Resource creation

  * Decoder creation (root, switch, endpoint)

  * Logical device creation (region, endpoint)

  * Auto-regions surfaced as DAX region

* DAX Driver Operation

  * DAX kmem driver surfaces dax device

    * DAX kmem IO resource creation

  * DAX kmem surfaces memory region to Memory Hotplug to add to page allocator as "driver managed memory"

* Memory Hotplug

  * mhp component surfaces a memory region as multiple memory blocks to the page allocator

  * blocks are onlined into the requested zone (NORMAL or MOVABLE)

  * Memory is marked "Driver Managed" to avoid kexec from using it as region for kernel updates
