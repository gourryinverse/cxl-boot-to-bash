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

For this section we'll explore the devices present in this configuration, but
we'll explore more configurations in-depth in example configurations below.

Base Devices
============
Most devices in a CXL fabric are a `port` of some kind (because each
device mostly routes request from one device to the next, rather than
provide a manageable service).

Root
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

Port
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

Endpoint
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


Memory Device (memdev)
----------------------
todo: high level explanation, why no reference to endpoint?

::

  # ls /sys/bus/cxl/devices/mem0
    dev       firmware_version    payload_max  security   uevent
    driver    label_storage_size  pmem         serial
    firmware  numa_node           ram          subsystem


Decoders
========
todo: explain the difference between decoders and how they're discovered

Root Decoder
------------
todo:

* created by CEDT CFMWS
* target list filled by CFMWS target fields
* targets validated against CHBS and DSDT.
* how are memory ranges validated?

::

  # ls /sys/bus/cxl/devices/decoder0.0/
    cap_pmem           devtype                 region0
    cap_ram            interleave_granularity  size
    cap_type2          interleave_ways         start
    cap_type3          locked                  subsystem
    create_ram_region  modalias                target_list
    delete_region      qos_class               uevent

Switch Decoder
--------------
todo:

* what distinguishes it from a root decoder?
* how is it created?
* how does it get targets?
* how are targets validated?
* how are memory ranges validated?

::

  # ls /sys/bus/cxl/devices/decoder0.0/
    devtype                 locked    size       target_list
    interleave_granularity  modalias  start      target_type
    interleave_ways         region    subsystem  uevent


Endpoint Decoder
----------------
todo:

* how is it created?
* how are memory ranges validated?

::

  # ls /sys/bus/cxl/devices]# ls decoder5.0
    devtype                 locked    start
    dpa_resource            modalias  subsystem
    dpa_size                mode      target_type
    interleave_granularity  region    uevent
    interleave_ways         size


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
