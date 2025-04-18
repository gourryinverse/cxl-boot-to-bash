.. ACPI documentation

ACPI Tables
###########

ACPI is the "Advanced Configuration and Power Interface", which is a standard
that defines how platforms and OS manage pwoer and configure computer hardware.
For the purpose of this theory of operation, when referring to "ACPI" we will
usually refer to "ACPI Tables" - which are the way a platform (BIOS/EFI)
communicates static configuration information to the operation system.

The Following ACPI tables contain *static* configuration and performance data about CXL devices.

.. toctree::
   :maxdepth: 1

   acpi/cedt.rst
   acpi/srat.rst
   acpi/hmat.rst
   acpi/slit.rst
   acpi/dsdt.rst

The SRAT table may also contain generic port/initiator content that is intended to describe the generic port, but not information about the rest of the path to the endpoint.

Linux uses these tables to configure kernel resources for statically configured (by BIOS/EFI) CXL devices, such as:

- NUMA nodes
- Memory Tiers
- NUMA Abstract Distances
- SystemRAM Memory Regions
- Weighted Interleave Node Weights

ACPI Debugging
**************

The :code:`acpidump` command can be used at runtime to dump the contents of the
tables that have been created. Running the command by itself displays the hex
contents of the ACPI to stdout. Not very helpful :_(

Running :code:`acpidump -b` will dump out the tables in individual .dat files,
which :code:`iasl -d` (disassemble) can use to convert into .dsl files, showing
human-readable tables.

acpidump
========

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
