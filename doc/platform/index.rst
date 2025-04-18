.. platform documentation

Platform Configuration
######################

.. toctree::
   :maxdepth: 1
   :caption: Contents:

   acpi/index.rst
   bios-and-efi/index.rst

Handoffs
********

* The bootloader starts the EFI / BIOS
* EFI / BIOS loads drivers (to be used during Device Probe)
* EFI / BIOS creates CEDT and SRAT tables
* Calls :code:`start_kernel` and beigns the Linux Early Boot process.

