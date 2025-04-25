.. Overview of linux kernel config

Overview
########

* Early Boot

  * Linux Boot Parameters

    * nosoftreserve

  * Memory Map Creation

    * EFI Memory Map / E820 Consulted for Soft-Reserved

      * CXL Memory is set aside to be handled by the CXL driver

      * IO Resources are created for CFMWS entry

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

  * Devices are associated with each other

    * If auto-decoder (BIOS-programmed decoders), driver validates configurations and locks everything.

  * Regions surfaced as DAX region

    * DAX Region surfaced as DAX device

* DAX Driver Operation

  * DAX driver surfaces DAX region as one of two dax device modes

    * kmem - dax device is converted to hotplug memory

    * hmem - dax device is left to be accessed as a file.

  * DAX driver surfaces dax device to /dev/daxX.Y

    * DAX kmem IO resource creation

    * If hmem, journey ends here.

  * DAX kmem surfaces memory region to Memory Hotplug to add to page allocator as "driver managed memory"

* Memory Hotplug

  * mhp component surfaces a memory region as multiple memory blocks to the page allocator

    * blocks appear in /sys/bus/memory/devices and /sys/bus/memory/node/devices/nodeN/

  * blocks are onlined into the requested zone (NORMAL or MOVABLE)

    * Build option: CONFIG_MHP_DEFAULT_ONLINE_TYPE_*

    * Boot option: memhp_default_state

  * Memory is marked "Driver Managed" to avoid kexec from using it as region for kernel updates
