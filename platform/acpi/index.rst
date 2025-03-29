.. ACPI documentation

ACPI Table Reference
====================

The Following ACPI tables contain *static* configuration and performance data about CXL devices.

- CEDT : CXL Early Detection Table
- SRAT : Static / System Resource Attribute Table
- HMAT : Heterogeneous Memory Attribute Table
- SLIT : System Locality Information Table

The SRAT table may also contain generic port/initiator content that is intended to describe the generic port, but not information about the rest of the path to the endpoint.

Linux uses these tables to configure kernel resources for statically configured (by BIOS/EFI) CXL devices, such as:

- NUMA nodes
- Memory Tiers
- NUMA Abstract Distances
- SystemRAM Memory Regions
- Weighted Interleave Node Weights

.. toctree::
   :maxdepth: 1
   :caption: Contents:

   cedt.rst
   srat.rst
   hmat.rst
   slit.rst
