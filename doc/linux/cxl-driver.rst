.. cxl driver documentation

CXL Driver Operation
####################

The devices described in this section are present in ::

  /sys/bus/cxl/devices/
  /dev/cxl/

The :code:`cxl-cli` library, maintained as part of the NDTCL project, may
be used to script interactions with these devices.

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
Here is an example from a single-socket system with 4 host bridges. Two host
bridges have a single memory device attached, and the devices are interleaved
into a single memory region. The memory region has been converted to dax. ::

  # ls /sys/bus/cxl/devices/
    dax_region0  decoder3.0  decoder6.0  mem0   port3
    decoder0.0   decoder4.0  decoder6.1  mem1   port4
    decoder1.0   decoder5.0  endpoint5   port1  region0
    decoder2.0   decoder5.1  endpoint6   port2  root0

We'll explore this configuration more in-depth in an example configuration.

Base Devices
============
Most devices in a CXL fabric are a `port` of some kind (because each
device mostly routes request from one device to the next, rather than
provide a manageable service).

root
----
todo: high level explanation, also port vs dport? can we have 2 roots?

::

  # ls /sys/bus/cxl/devoces/root0
    decoder0.0          dport0  dport5    port2  subsystem
    decoders_committed  dport1  modalias  port3  uevent
    devtype             dport4  port1     port4  uport

  # cat devtype
    cxl_port

The root contains references to all host bridges (ports) and CXL Fixed
Memory Windows (`root decoders`) described in the ACPI CEDT. ::

  # cat port1/devtype
    cxl_port

  # cat decoder0.0/devtype
    cxl_decoder_root

port
----
A `port` is better described as a `switch port`.  It may represent a host
bridge connection to the root, or an actual switch port on a switch. ::

  # ls /sys/bus/cxl/devices/port1
    decoder1.0          dport0    driver     parent_dport  uport
    decoders_committed  dport113  endpoint5  subsystem
    devtype             dport2    modalias   uevent

  # cat devtype
    cxl_port

A port contains decoders for routing to downstream ports, and may contain
endpoints if the next port is terminal. ::

  # cat decoder1.0/devtype
    cxl_decoder_switch

  # cat endpoint5/devtype
    cxl_port

endpoint
--------
An `endpoint` is a terminal port in the fabric.  This is a `logical device`,
and may be one of many `logical devices` presented by a memory device. It
is still considered a type of `port` in the fabric. ::

  # ls /sys/bus/cxl/devices/endpoint5

  # cat devtype
    cxl_port

An `endpoint` contains any `endpoint decoders` for this device. ::

  # cat decoder5.0/devtype
    cxl_decoder_endpoint


mem (memdev)
------------
todo: high level explanation, why no reference to endpoint?

::

  # ls /sys/bus/cxl/devices/mem0
    dev       firmware_version    payload_max  security   uevent
    driver    label_storage_size  pmem         serial
    firmware  numa_node           ram          subsystem


Decoders
========
todo: explain the difference between decoders and how they're discovered

root decoder
------------
todo

switch decoder
--------------
todo: what distinguishes a switch from a root decoder?

endpoint decoder
----------------
todo

Regions
=======

Memory Region
-------------
todo
::

  # ls /sys/bus/cxl/devices/region0/
    access0      devtype                 modalias  subsystem  uuid
    access1      driver                  mode      target0
    commit       interleave_granularity  resource  target1
    dax_region0  interleave_ways         size      uevent

DAX Region
----------
todo

::

  # ls /sys/bus/cxl/devices/dax_region0/
    dax0.0      devtype  modalias   uevent
    dax_region  driver   subsystem


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
todo: basic example of decoder programming steps.

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

Interleave
==========
todo

* CFMWS, CHBS, and Root Decoders

* Decoder-type behavior (root/switch route, endpoints translate)

* Unbalanced configurations are not supported by Linux.

Surfacing Memory as DAX
***********************
todo: explain any additional steps taken to hand off control to dax

Example Configurations
**********************
.. toctree::
   :maxdepth: 1

   example-configurations/single-device.rst
   example-configurations/hb-interleave.rst
   example-configurations/intra-hb-interleave.rst
   example-configurations/multi-interleave.rst
