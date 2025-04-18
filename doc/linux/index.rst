.. linux kernel config documentation

Linux configuration is split into two major steps: Early-Boot and everything else.

During early boot, Linux sets up immutible resources (such as numa nodes), while later operations
include things like driver probe and memory hotplug.  Linux may read EFI and ACPI information
throughout this process to configure logical representations of the devices.

.. toctree::
   :maxdepth: 2
   :caption: Contents:

   early-boot.rst
   driver-operation/index.rst
   memory-hotplug/index.rst
