.. platform documentation

BIOS and EFI Configuration
##########################

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

EFI Configuration
*****************
The :code:`uefisettings` command can be used to retrieve / set EFI settings.
Once changes are made, rebooting the machine incorporates the changes made into
the next boot.

One notable configuration here is the EFI Memory SP (Special Purpose) bit.
When this is enabled, it means that the driver is responsible for managing the
memory region. Otherwise, the memory is treated as "normal memory", and is
exposed to the page allocator.

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

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   memory-map.rst
   decoder-configuration.rst
   example-tables/index.rst
