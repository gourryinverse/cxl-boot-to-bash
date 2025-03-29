.. physical memory map documentation

Physical Memory Map
===================

Physical Address Region Alignment
---------------------------------

As of Linux v6.14, the hotplug memory system requires memory regions to be uniform in size and alignment.  While the CXL specification allows for memory regions as small as 256MB, the supported memory block size and alignment is architecture-defined. Blocks may be as small as 128MB and increase uniformly as a power of two.

On ARM, the default block size and alignment is either 128MB or 256MB.

On x86, the default block size is 256MB, and increases to 2GB as the capacity of the system increases up to 64GB.

For best support, platform vendors should place CXL memory at a 2GB aligned base address, and regions should be 2GB aligned.


Memory Holes
------------

Holes in the memory map are tricky.  Consider a 4GB device set at base 0x100000000, set up in the following memory map ::

  ---------------------
  |    0x100000000    |
  |                   |
  |        CXL        |
  |                   |
  |    0x1BFFFFFFF    |
  ---------------------
  |    0x1C0000000    |
  |    MEMORY HOLE    |
  |    0x1FFFFFFFF    |
  ---------------------
  |    0x200000000    |
  |                   |
  |     CXL CONT.     |
  |                   |
  |    0x23FFFFFFF    |
  ---------------------

There are two issues to consider: decoder programming and memory block alignment.

If your architecture requires 2GB uniform size and aligned memory blocks, the only capacity Linux is capable of mapping (as of v6.14) would be the capacity from `0x100000000-0x180000000`.  The remaining capacity will be stranded, as they are not of 2GB aligned length.

Assuming your architecture and memory configuration allows 1GB memory blocks, this memory map is supported and this should be presented as multiple CFMWS in the CEDT that describe each side of the memory hole separately - along with matching decoders.

Each endpoint intended to be mapped into this layout should plan to use multiple decoders as well.
