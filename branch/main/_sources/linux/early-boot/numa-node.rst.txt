.. numa node creation

NUMA Node Creation
==================

NUMA nodes are *NOT* hot-pluggable.  All *POSSIBLE* NUMA nodes are identified at `__init` time, more specifically during `mm_init`.  What this means is that the CEDT and SRAT must contain sufficient `proximity domain` information for linux to identify how many NUMA nodes are required (and what memory regions should be associated with them).

SRAT is the only ACPI defined way of getting Proximity nodes. Linux chooses to at most map those 1:1 with NUMA nodes.

CEDT adds a description of SPA ranges which Linux may wish to map to one or more NUMA nodes

The relevant code exists in: linux/drivers/acpi/numa/srat.c

Basically, the heuristic is as follows

1. Add one NUMA node per Proximity Domain described in SRAT (one or more of: memory, cpu, generic initiator).
2. If SRAT describes some portion of a CFMWS SPA range - do not create a nodes for that CFMWS
3. If SRAT does not describe memory in a CFMWS SPA range - a node is created for that CFMWS

In the future, Linux may reject CFMWS not described by SRAT due to the ambiguity of proximity domain association.

Generally speaking, you will see one NUMA node per Host bridge, unless inter-host-bridge interleave is in use.  In environments with complex QoS concerns, or designed for runtime configuration flexibility, it may be reasonable to create more than one NUMA node per Host bridge.
