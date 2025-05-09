.. dsmas reference

DSMAS - Device Scoped Memory Affinity Structure
===============================================

The Device Scoped Memory Affinity Structure contains information such as DSMADHandle, the DPA Base, and DPA Length.

This table is used by Linux in conjunction with the Device Scoped Latency and Bandwidth Information Structure (DSLBIS) to determine the performance attributes of the CXL device itself.

Example ::

Structure Type : 00 [DSMAS]
      Reserved : 00
        Length : 0018              <- 24d, size of structure
   DSMADHandle : 01
         Flags : 00
      Reserved : 0000
      DPA Base : 0000000040000000  <- 1GiB base
    DPA Length : 0000000080000000  <- 2GiB size
