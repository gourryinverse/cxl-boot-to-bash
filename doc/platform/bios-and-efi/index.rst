.. platform documentation


Configuration Process
#####################

The EFI (Extensible Firmware Interface) / BIOS (Basic Input Output System) is
the first thing that comes up when the system boots. More explicitly, it is what
the bootloader hands off to initiate the rest of startup. :code:`start_kernel`
is called in this step, which starts Linux.

Together, EFI and BIOS program the initial ACPI tables. Notably, they generate
the  CEDT (CXL Early Discovery Table) and SRAT (System Resource Affinity Table).
More detail on these tables can be found under Platform Configuration -> ACPI
Table Reference.

EFI Setting Configuration
#########################
The :code:`uefisettings` command can be used to retrieve / set EFI settings.
Once changes are made, rebooting the machine incorporates the changes made into
the next boot.

One notable configuration here is the EFI Memory SP (Special Purpose) bit.
When this is enabled, it means that the driver is responsible for managing the
memory region. Otherwise, the memory is treated as "normal memory", and is
exposed to the page allocator.

uefisettings examples
*********************

:code:`uefisettings identify` ::

        uefisettings identify

        bios_vendor: xxx
        bios_version: xxx
        bios_release: xxx
        bios_date: xxx
        product_name: xxx
        product_family: xxx
        product_version: xxx

:code:`uefisettings get "CXL Physical Addressing"` ::

        selector: xxx
        ...
        question: Question {
            name: "CXL Memory Attribute",
            answer: "Enabled",
            ...
        }


Physical Memory Map
###################

Physical Address Region Alignment
*********************************

As of Linux v6.14, the hotplug memory system requires memory regions to be uniform in size and alignment.  While the CXL specification allows for memory regions as small as 256MB, the supported memory block size and alignment is architecture-defined. Blocks may be as small as 128MB and increase uniformly as a power of two.

On ARM, the default block size and alignment is either 128MB or 256MB.

On x86, the default block size is 256MB, and increases to 2GB as the capacity of the system increases up to 64GB.

For best support, platform vendors should place CXL memory at a 2GB aligned base address, and regions should be 2GB aligned.

Memory Holes
************

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


Decoder Programming
###################

If BIOS/EFI intends to program the decoders to be statically configured,
there are a few things to consider to avoid major pitfalls that will
prevent Linux compatibility.  These recommendations are not required "per
the specification", but Linux makes no guarantees of support otherwise.


Translation Point
*****************
Per the specification, the only decoders which *translate* Host Physical
Address to Device Physical Address are the **Endpoint Decoders**. All other
decoders in the fabric are intended to route accesses without translating the
addresses.

Due to some ambiguity in how Architecture, ACPI, PCI, and CXL specifications
"hand off" responsibility between domains, some early adopting platforms
attempted to do translation at the originating memory controller or host
bridge.  This configuration requires a platform specific extension to the
driver and is not officially endorsed - despite being supported.

It is highly recommended *not* to do this; otherwise, you are on your own
to provide driver support for your platform.

Interleave support and Flexibility
**********************************
If providing cross-host-bridge interleave, a CFMWS entry in the CEDT must be
presented with target host-bridges for the interleaved device sets (there may
be multiple behind each host bridge).

If providing intra-host-bridge interleaving, only 1 CFMWS entry in the CEDT is
required for that host bridge - if it covers the entire capacity of the devices
behind the host bridge.

If intending to provide users flexibility in programming decoders beyond the
root, you may want to provide multiple CFMWS entries in the CEDT intended for
different purposes.  For example, you may want to consider adding

1) A CFMWS entry to cover all interleavable host bridges.
2) A CFMWS entry to cover all devices on a single host bridge.
3) A CFMWS entry to cover each device.

A platform may choose to add all of these, or change the mode based on a BIOS setting.  For each CFMWS entry, Linux expects descriptions of the described memory regions in the SRAT to determine the number of NUMA nodes it should reserve during early boot / init.

Memory Holes
************
If your platform includes memory holes intersparsed between your CXL memory, it
is recommended to utilize multiple decoders to cover these regions of memory,
rather than try to program the decoders to accept the entire range and expect
Linux to manage the overlap.

Linux makes no guarantee of support for strange memory hole situations.

Multi-Media Devices
*******************
Devices that have either: 

1) A multi-purpose media (i.e. persistent mem used as volatile), or
2) Multiple forms of memory

Require special restriction bits (specifically Volatile vs Persistent bits) set
in the CFMWS entries in the CEDT.

A CFMWS may be set to allow either persistent or volatile, but for best flexibility
platforms may wish to define multiple CFMWS entries to allow flexible configuration
by a user at runtime.
