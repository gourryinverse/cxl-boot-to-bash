.. sslbis reference

SSLBIS - Switch Scoped Latency and Bandwidth Informatoin Structure
==================================================================

The Switch Scoped Latency Bandwidth Information Structure contains information about the latency and bandwidth of a switch.

The table is used by Linux to compute the performance coordinates of a CXL path from the device to
the root port where a switch is part of the path.

Example ::

 Structure Type : 05 [SSLBIS]
       Reserved : 00
         Length : 20                           <- 32d, length of record, including SSLB entries
      Data Type : 00                           <- Latency
       Reserved : 000000
Entry Base Unit : 00000000000000001000         <- Matches Entry Base Unit in HMAT SSLBIS

                                               <- SSLB Entry 0
      Port X ID : 0100                         <- First port, 0100h represents an upstream port
      Port Y ID : 0000                         <- Second port, downstream port 0
        Latency : 0100                         <- Port latency
       Reserved : 0000
                                               <- SSLB Entry 1
      Port X ID : 0100
      Port Y ID : 0001
        Latency : 0100
       Reserved : 0000


 Structure Type : 05 [SSLBIS]
       Reserved : 00
         Length : 18                           <- 24d, length of record, including SSLB entry
      Data Type : 03                           <- Bandwidth
       Reserved : 000000
Entry Base Unit : 00000000000000001000         <- Matches Entry Base Unit in HMAT SSLBIS

                                               <- SSLB Entry 0
      Port X ID : 0100                         <- First port, 0100h represents an upstream port
      Port Y ID : FFFF                         <- Second port, FFFFh indicates any port
      Bandwidth : 1200                         <- Port bandwidth
       Reserved : 0000
