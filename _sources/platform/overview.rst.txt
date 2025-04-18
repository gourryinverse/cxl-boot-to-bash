.. platform config overview

Overview
########

Hardware, BIOS, and EFI are all responsible for configuring static information about devices (or
potential future devices) such that Linux can build the appropriate logical representations of these
devices. Much of what this section is concerned with is ACPI Table production and static memory map
configuration.

At a high level, this is what occurs during this phase of configuration.

* The bootloader starts the EFI / BIOS

* BIOS / EFI do early device probe to determine static configuration

* BIOS / EFI creates ACPI Tables that describe static config for the OS

* Calls :code:`start_kernel` and begins the Linux Early Boot process.
