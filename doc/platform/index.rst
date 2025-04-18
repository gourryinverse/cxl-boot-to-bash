.. platform documentation

Platform Configuration
######################

Configuration Process
*********************

At a high level, this is what occurs during this phase of configuration.

* The bootloader starts the EFI / BIOS
* BIOS / EFI do early device probe to determine static configuration
* BIOS / EFI creates ACPI Tables that describe static config for the OS
* Calls :code:`start_kernel` and begins the Linux Early Boot process.

BIOS And EFI
************

.. toctree::
   :maxdepth: 2

   bios-and-efi/index.rst

ACPI
****

.. toctree::
   :maxdepth: 2

   acpi/index.rst

Example Platform Configurations
*******************************

.. toctree::
   :maxdepth: 2

   example-configurations/one-dev-per-hb.rst
   example-configurations/multi-dev-per-hb.rst
   example-configurations/hb-interleave.rst
   example-configurations/multi-media-dev.rst
