.. bios-and-efi documentation

BIOS/EFI Configuration
######################

Hardware, BIOS, and EFI are all responsible for configuring static information
about devices (or potential future devices) such that Linux can build the
appropriate logical representations of these devices.

Much of what this section is concerned with is ACPI Table production and static
memory map configuration.

At a high level, this is what occurs during this phase of configuration.

* The bootloader starts the BIOS/EFI.

* BIOS/EFI do early device probe to determine static configuration

* BIOS/EFI creates ACPI Tables that describe static config for the OS

* BIOS/EFI calls :code:`start_kernel` and begins the Linux Early Boot process.

BIOS/EFI are responsible for initializing ACPI tables. More detail on these
tables can be found under Platform Configuration -> ACPI Table Reference.

UEFI Settings
*************
If your platform supports it, the :code:`uefisettings` command can be used to
read/write EFI settings. Changes will be reflected on the next reboot. Kexec
is not a sufficient reboot.

One notable configuration here is the EFI_MEMORY_SP (Specific Purpose) bit.
When this is enabled, this bit tells linux to defer management of a memory
region to a driver (in this case, the CXL driver). Otherwise, the memory is
treated as "normal memory", and is exposed to the page allocator during
:code:`__init`.

uefisettings examples
=====================

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
*******************

Physical Address Region Alignment
=================================

As of Linux v6.14, the hotplug memory system requires memory regions to be
uniform in size and alignment.  While the CXL specification allows for memory
regions as small as 256MB, the supported memory block size and alignment is
architecture-defined.

A Linux memory blocks may be as small as 128MB and increase in powers of two.

* On ARM, the default block size and alignment is either 128MB or 256MB.

* On x86, the default block size is 256MB, and increases to 2GB as the
  capacity of the system increases up to 64GB.

For best support, platform vendors should place CXL memory at a 2GB aligned
base address, and regions should be 2GB aligned.  This also helps prevent the
memory hotplug subsystem from creating thousands of memory devices (one per
block).

Memory Holes
============

Holes in the memory map are tricky.  Consider a 4GB device located at base
address 0x100000000, but with the following following memory map ::

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

Two issues to consider: decoder programming and memory block alignment.

If your architecture requires 2GB uniform size and aligned memory blocks, the
only capacity Linux is capable of mapping (as of v6.14) would be the capacity
from `0x100000000-0x180000000`.  The remaining capacity will be stranded, as
they are not of 2GB aligned length.

Assuming your architecture and memory configuration allows 1GB memory blocks,
this memory map is supported and this should be presented as multiple CFMWS
in the CEDT that describe each side of the memory hole separately - along with
matching decoders.

Multiple decoders can be used to manage such a memory hole.


Decoder Programming
*******************

If BIOS/EFI intends to program the decoders to be statically configured,
there are a few things to consider to avoid major pitfalls that will
prevent Linux compatibility.  These recommendations are not required "per
the specification", but Linux makes no guarantees of support otherwise.


Translation Point
=================
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

Interleave and Configuration Flexibility
========================================
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

A platform may choose to add all of these, or change the mode based on a BIOS
setting.  For each CFMWS entry, Linux expects descriptions of the described
memory regions in the SRAT to determine the number of NUMA nodes it should
reserve during early boot / init.

Memory Holes
============
If your platform includes memory holes intersparsed between your CXL memory, it
is recommended to utilize multiple decoders to cover these regions of memory,
rather than try to program the decoders to accept the entire range and expect
Linux to manage the overlap.

Linux makes no guarantee of support for strange memory hole situations.

Multi-Media Devices
===================
Devices that have either: 

1) A multi-purpose media (i.e. persistent mem used as volatile), or
2) Multiple forms of memory

Require special restriction bits (specifically Volatile vs Persistent bits) set
in the CFMWS entries in the CEDT.

A CFMWS may be set to allow either persistent or volatile, but for best
flexibility platforms may wish to define multiple CFMWS entries to allow flexible
configuration by a user at runtime.
