.. cxl driver documentation

CXL Driver Operation
####################

The devices described in this section are present in ::

  /sys/bus/cxl/devices/

They are intended for interaction via the :code:`cxl-cli` library, but
may be interacted with directly.  This cxl is maintained as part of the
larger ndctl project at: https://github.com/pmem/ndctl

.. toctree::
   :maxdepth: 1

   cxl-cli.rst

Drivers
*******
todo: list of drivers and probe responsibilities

* cxl_core
* cxl_port
* cxl_acpi
* cxl_pmem
* cxl_mem
* cxl_pci

Driver Devices
**************

Base Devices
============
Root, Port, Memory Device (memdev)

Decoders
========
Root, Switch, Endpoint

Memory and DAX Regions
======================
todo

Mailbox Interfaces
==================
A mailbox command interface for each device is exposed in ::

  /dev/cxl/mem0
  /dev/cxl/mem1

These mailboxes may receive any specification-defined command. Raw commands
(custom commands) can only be sent to these interfaces if the build config
:code:`CXL_MEM_RAW_COMMANDS` is set.  This is considered a debug and/or
development interface, not an officially supported mechanism for creation
of vendor-specific commands (see the `fwctl` subsystem for that).

Decoder Programming
*******************

Runtime Programming
===================
todo

Auto Decoders
=============
Auto Decoders are decoders programmed by BIOS/EFI at boot time, and are
almost always locked (cannot be changed).  This is done by a platform
which may have a static configuration - or certain quirks which may prevent
dynamic runtime changes to the decoders (such as requiring additional 
controller programming within the CPU complex outside the scope of CXL).

Auto Decoders are probed automatically as long as the devices and memory
regions they are associated with probe without issue.  When probing Auto
Decoders, the driver's primary responsibility is to ensure the fabric is
sane - as-if validating runtime programmed regions and decoders.

If Linux cannot validate auto-decoder configuration, the memory will not
be surfaced as a DAX device - and therefore not be exposed to the page
allocator - effectively stranding it.


Example Configurations
**********************
todo

Single Device Region
====================
todo

Intra-Host-Bridge Interleave
============================
todo

Inter-Host-Bridge Interleave
============================
todo

Multi-level Interleave
======================
todo

Surfacing Memory as DAX
***********************
todo
