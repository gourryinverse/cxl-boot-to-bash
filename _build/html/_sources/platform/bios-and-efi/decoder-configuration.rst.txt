.. decoder programming documentation

Decoder Programming Recommendations
===================================

If BIOS/EFI intends to program the decoders to be statically configured, there are a few things to consider to avoid major pitfalls that will prevent Linux compatibility.  These recommendations are not required "per the specification", but Linux makes no guarantees of support if otherwise.

Translation Point
-----------------
Per the specification, the only decoders which *translate* Host Physical Address to Device Physical Address are the **Endpoint Decoders**. All other decoders in the fabric are intended to route accesses without translating the addresses.

Due to some ambiguity in how Architecture, ACPI, PCI, and CXL specifications "hand off" responsibility between domains, some early adopting platforms attempted to do translation at the originating memory controller.  This is configuration requires an platform support extension to the driver and is not officially supported by Linux.

It is highly recommended *not* to do this; otherwise, uou are on your own to provide driver support.

Interleave support and Flexibility
----------------------------------
If providing cross-host-bridge interleave, a CFMWS entry in the CEDT must be presented with target host-bridges for the interleaved device sets (there may be multiple behind each host bridge).

If providing intra-host-bridge interleaving, only 1 CFMWS entry in the CEDT is required for that host bridge - if it covers the entire capacity of the devices behind the host bridge.

If intending to provide users flexibility in programming decoders beyond the root, you may want to provide multiple CFMWS entries in the CEDT intended for different purposes.  For example, you may want to consider adding

1) A CFMWS entry to cover all interleavable host bridges.
2) A CFMWS entry to cover all devices on a single host bridge.
3) A CFMWS entry to cover each device.

A platform may choose to add all of these, or change the mode based on a BIOS setting.  For each CFMWS entry, Linux expects descriptions of the described memory regions in the SRAT to determine the number of NUMA nodes it should reserve during early boot / init.

Memory Holes
------------
If your platform includes memory holes intersparsed between your CXL memory, it's recommended you utilize multiple decoders to cover these regions of memory, rather than try to program the decoders to accept the entire range and expect Linux to manage the overlap.

If your platform does not do this, you are on your own to provide driver support.

Multi-Media Devices
-------------------
Devices that have either: 

1) A multi-purpose media (i.e. persistent mem used as volatile), or
2) Multiple forms of memory

Require special restriction bits (specifically Volatile vs Persistent bits) set in the CFMWS entries in the CEDT.

A CFMWS may be set to allow either persistent or volatile, but for best flexibility platforms may wish to simply define multiple CFMWS entries to allow flexible configuration by a user at runtime.
