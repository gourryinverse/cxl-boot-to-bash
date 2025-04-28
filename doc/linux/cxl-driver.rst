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
The `CXL Root` is logical object created by the `cxl_acpi` driver during
:code:`cxl_acpi_probe` - if the :code:`ACPI0017` `Compute Express Link
Root Object` Device Class is found.

The Root contains links to:

* `Host Bridge Ports` defined by ACPI CEDT CHBS.

* `Root Decoders` defined by ACPI CEDT CFMWS.

::

  # ls /sys/bus/cxl/devices/root0
    decoder0.0          dport0  dport5    port2  subsystem
    decoders_committed  dport1  modalias  port3  uevent
    devtype             dport4  port1     port4  uport

  # cat /sys/bus/cxl/devices/root0/devtype
    cxl_port

  # cat port1/devtype
    cxl_port

  # cat decoder0.0/devtype
    cxl_decoder_root

The root is first `logical port` in the CXL fabric, as presented by the Linux
CXL driver.  The `CXL root` is a special type of `switch port`, in that it
only has downstream port connections.

Port
----
A `port` object is better described as a `switch port`.  It may represent a
host bridge to the root or an actual switch port on a switch. A `switch port`
contains one or more decoders used to route memory requests downstream ports,
which may be connected to another `switch port` or an `endpoint port`.

::

  # ls /sys/bus/cxl/devices/port1
    decoder1.0          dport0    driver     parent_dport  uport
    decoders_committed  dport113  endpoint5  subsystem
    devtype             dport2    modalias   uevent

  # cat devtype
    cxl_port

  # cat decoder1.0/devtype
    cxl_decoder_switch

  # cat endpoint5/devtype
    cxl_port

CXL `Host Bridges` in the fabric are probed during :code:`cxl_acpi_probe` at
the time the `CXL Root` is probed.  The allows for the immediate logical
connection to between between the root and host bridge.

* The root has a downstream port connection to a host bridge

* The host bridge has an upstream port connection to the root.

* The host bridge has one or more downstream port connections to switch
  or endpoint ports.

A `Host Bridge` is a special type of CXL `switch port`. It is explicitly
defined in the ACPI specification via `ACPI0016` ID.  `Host Bridge` ports
will be probed at `acpi_probe` time, while similar ports on an actual switch
will be probed later.  Otherwise, switch and host bridge ports look very
similar - the both contain switch decoders which route accesses between
upstream and downstream ports.

Endpoint
--------
An `endpoint` is a terminal port in the fabric.  This is a `logical device`,
and may be one of many `logical devices` presented by a memory device. It
is still considered a type of `port` in the fabric.

An `endpoint` contains `endpoint decoders` available for use and the
*Coherent Device Attribute Table* (CDAT) used to describe the capabilities
of the device. ::

  # ls /sys/bus/cxl/devices/endpoint5
    CDAT        decoders_committed  modalias      uevent
    decoder5.0  devtype             parent_dport  uport
    decoder5.1  driver              subsystem

  # cat /sys/bus/cxl/devices/endpoint5/devtype
    cxl_port

  # cat /sys/bus/cxl/devices/endpoint5/decoder5.0/devtype
    cxl_decoder_endpoint


Memory Device (memdev)
----------------------
A `memdev` is probed and added by the `cxl_pci` driver in :code:`cxl_pci_probe`
and is managed by the `cxl_mem` driver. It primarily provides the `IOCTL`
interface to a memory device, via :code:`/dev/cxl/memN`, and exposes various
device configuration data. ::

  # ls /sys/bus/cxl/devices/mem0
    dev       firmware_version    payload_max  security   uevent
    driver    label_storage_size  pmem         serial
    firmware  numa_node           ram          subsystem


Decoders
========
A decoder is short of a CXL Host-Managed Device Memory (HDM) Decoder. It is
a device that routes accesses through the CXL fabric to an endpoint, and at
the endpoint translates a `Host Physical` to `Device Physical` Addressing.

The CXL 3.1 specification heavily implies that only endpoint decoders should
engage in translation of `Host Physical Address` to `Device Physical Address`.

Specifically ::

  8.2.4.20 CXL HDM Decoder Capability Structure

  IMPLEMENTATION NOTE
  CXL Host Bridge and Upstream Switch Port Decode Flow

  IMPLEMENTATION NOTE
  Device Decode Logic

These notes imply that there are two logical types of decoders.

* Routing Decoder - a decoder which routes accesses but does not translate
  addresses from HPA to DPA.

* Translating Decoder - a decoder which translates accesses from HPA to DPA
  for an endpoint to service.

The CXL drivers distinguish 3 decoder types: root, switch, and endpoint.

.. note::

   Platform vendors be aware:

   Linux makes a strong assumption that endpoint decoders are the only decoder
   in the fabric that actively translates HPA to DPA.  Linux assumes routing
   decoders pass the HPA unchanged to the next decoder in the fabric.

   It is therefore assumed that any given decoder in the fabric will have an
   address range that is a subset of its upstream port decoder. Any deviation
   from this scheme is not supported by the core CXL driver and will require
   additional extensions.


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
