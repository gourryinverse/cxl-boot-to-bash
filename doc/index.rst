.. cxl-boot-to-bash documentation master file

CXL on Linux: Boot To Bash
##########################

This documentation is intended as a companion to systems engineers (Platform, BIOS, EFI, OS, etc)
trying to enable CXL devices on Linux.

CXL device configuration requires platform (Hardware, BIOS, EFI), OS (early boot, core kernel,
driver), and user policy decisions that all impact each other.  This doc breaks up these
configurations into five main areas:

.. toctree::
   :maxdepth: 1
   :caption: Overview

   self

.. toctree::
   :maxdepth: 2
   :caption: Device Reference

   devices/device-types.rst
   devices/uefi.rst


.. toctree::
   :maxdepth: 2
   :caption: Platform Configuration

   platform/bios-and-efi.rst
   platform/acpi.rst
   platform/example-configs.rst

.. toctree::
   :maxdepth: 3
   :caption: Linux Kernel Configuration:

   linux/overview.rst
   linux/early-boot.rst
   linux/cxl-driver.rst
   linux/dax-driver.rst
   linux/memory-hotplug.rst

.. toctree::
   :maxdepth: 2
   :caption: Memory Allocation

   allocation/dax.rst
   allocation/page-allocator.rst
   allocation/reclaim.rst
   allocation/hugepages.rst
   allocation/tiering.rst

.. toctree::
   :maxdepth: 2
   :caption: Use Cases

   use-case/memory-expansion.rst
   use-case/dynamic-capacity.rst
   use-case/virtual-machines.rst
   use-case/shared-memory.rst

Contributors
############

* Gregory Price
* Joshua Hahn
