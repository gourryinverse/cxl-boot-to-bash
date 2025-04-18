.. cxl-boot-to-bash documentation master file

CXL on Linux: Boot To Bash
##########################

This documentation is intended as a companion to systems engineers (Platform, BIOS, EFI, OS, etc)
trying to enable CXL devices on Linux.

My only real goal is to help make the CXL ecosystem an ever-so-slightly more sane environment,
for my own sake. I hope this helps you as well.

Roadmap
*******

CXL device configuration requires platform (Hardware, BIOS, EFI), OS (early boot, core kernel,
driver), and user policy decisions that all impact each other.  This doc breaks up these
configurations into five main areas:

Device Reference
================

The type of CXL device (Memory, Accelerator, etc) dictates many configuration steps. This section
covers some basic background on device types and on-device resources used by the platform and OS
which impact configuration.

.. toctree::
   :maxdepth: 2

   devices/index.rst

Platform
========

Hardware, BIOS, and EFI are all responsible for configuring static information about devices (or
potential future devices) such that Linux can build the appropriate logical representations of these
devices. Much of what this section is concerned with is ACPI Table production and static memory map
configuration.

.. toctree::
   :maxdepth: 2

   platform/index.rst

Linux Kernel Configuration
==========================

Linux configuration is split into two major steps: Early-Boot and everything else.

During early boot, Linux sets up immutible resources (such as numa nodes), while later operations
include things like driver probe and memory hotplug.  Linux may read EFI and ACPI information
throughout this process to configure logical representations of the devices.

roadmap fixups:

* Linux Early Boot

  * NUMA Node Creation
  * EFI Memory Map

* Device Probe

  * ACPI
  * PCI
  * CXL
  * Dax

* Memory Hotplug

  * Memory Map
  * Memory Blocks

.. toctree::
   :maxdepth: 2

   linux/index.rst

Memory Allocation and Tiering
=============================

* Page allocator

  * Zones
  * Nodes

* Reclaim

  * Demotion
  * Zswap

* Tiering

  * Transparent Page Placement (TPP)
  * DAMON


Advanced Topics
===============

* Virtual Machine pass-through

* Logical Device Allocation

* Dynamic Capacity

* Shared Memory
  
  * FAMFS


Contributors
############

Authors
*******

* Gregory Price
* Joshua Hahn

Editors
*******
todo

Thanks
******
todo
