.. EFI & BIOS

EFI / BIOS
==========

The EFI (Extensible Firmware Interface) / BIOS (Basic Input Output System) is
the first thing that comes up when the system boots. More explicitly, it is what
the bootloader hands off to initiate the rest of startup. :code:`start_kernel`
is called in this step, which starts Linux.

Together, EFI and BIOS program the initial ACPI tables. Notably, they generate
the  CEDT (CXL Early Discovery Table) and SRAT (System Resource Affinity Table).
More detail on these tables can be found under Platform Configuration -> ACPI
Table Reference.

Finally, the EFI and BIOS also loads in the device drivers, which will be used
in later steps (device probe).

ACPI Debugging
--------------

The :code:`acpidump` command can be used at runtime to dump the contents of the
tables that have been created. Running the command by itself displays the hex
contents of the ACPI to stdout. Not very helpful :_(

Running :code:`acpidump -b` will dump out the tables in individual .dat files,
which :code:`iasl -d` (disassemble) can use to convert into .dsl files, showing
human-readable tables.

acpidump example
----------------

On a machine with CXL: :code:`acpidump -b && iasl -d cedt.dat` ::

        /*
         * Intel ACPI Component Architecture
         * AML/ASL+ Disassembler version 20210604 (64-bit version)
         * Copyright (c) 2000 - 2021 Intel Corporation
         *
         * Disassembly of cedt.dat, Fri Apr 11 07:47:31 2025
         *
         * ACPI Data Table [CEDT]
         *
         * Format: [HexOffset DecimalOffset ByteLength]  FieldName : FieldValue
         */

        [000h 0000   4]                    Signature : "CEDT"    [CXL Early Discovery Table]
        ...

EFI Configuration
-----------------
The :code:`uefisettings` command can be used to retrieve / set EFI settings.
Once changes are made, rebooting the machine incorporates the changes made into
the next boot.

One notable configuration here is the EFI Memory SP (Special Purpose) bit.
When this is enabled, it means that the driver is responsible for managing the
memory region. Otherwise, the memory is treated as "normal memory", and is
exposed to the page allocator.

uefisettings examples
---------------------

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

Handoffs
--------

* The bootloader starts the EFI / BIOS
* EFI / BIOS loads drivers (to be used during Device Probe)
* EFI / BIOS creates CEDT and SRAT tables
* Calls :code:`start_kernel` and beigns the Linux Early Boot process.
