.. dslbis reference

DSLBIS - Device Scoped Latency and Bandwidth Information Structure
==================================================================

The Device Scoped Latency and Bandwidth Information Structure contains latency and bandwidth information based on DSMADHandle matching.

This table is used by Linux in conjunction with Device scoped Memory Affinity Structure to determine the performance attributes of a CXL device.

Example ::

  Structure Type : 01 [DSLBIS]
        Reserved : 00
          Length : 18                     <- 24d, size of structure
          Handle : 0001                   <- DSMAS handle or Initiator handle with no memory
           Flags : 00                     <- Matches flag field for HMAT SLLBIS
       Data Type : 00                     <- Latency
Entry Basee Unit : 0000000000001000       <- Matches Entry Base Unit field in HMAT SSLBIS
           Entry : 010000000000           <- First byte used here, CXL LTC
        Reserved : 0000

  Structure Type : 01 [DSLBIS]
        Reserved : 00
          Length : 18                     <- 24d, size of structure
          Handle : 0001                   <- DSMAS handle or Initiator handle with no memory
           Flags : 00                     <- Matches flag field for HMAT SLLBIS
       Data Type : 03                     <- Bandwidth
Entry Basee Unit : 0000000000001000       <- Matches Entry Base Unit field in HMAT SSLBIS
           Entry : 020000000000           <- First byte used here, CXL BW
        Reserved : 0000
