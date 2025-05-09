.. CDAT documentation

CDAT Tables
###########

The Coherent Device Attribute Table (CDAT) is created to provide a standard way to
expose the properties of a device such as an CXL accelerator or switch. The table
formatting is similar to ACPI tables. The tables are created to provide performance
calculation of a hot-plugged device where ACPI HMAT+SRAT tables are not able to
enumerate the performance by the BIOS ahead of time.

The following CDAT tables contain *static* configuration and performance data about CXL devices.

.. toctree::
   :maxdepth: 1

   cdat/dsmas.rst
   cdat/dslbis.rst
   cdat/sslbis.rst

The Linux CXL driver uses these tables in addition to attributes of the CXL links and 
Generic Target performance data provided by HMAT+SRAT to create the whole path performance
for a CXL device.
