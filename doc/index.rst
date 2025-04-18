.. cxl-boot-to-bash documentation master file

CXL on Linux: Boot To Bash
##########################

This documentation is intended as a companion to systems engineers (Platform, BIOS, EFI, OS, etc)
trying to enable CXL devices on Linux.

My only real goal is to help make the CXL ecosystem an ever-so-slightly more sane environment,
for my own sake. I hope this helps you as well.

Roadmap
*******

CXL device configuration requires platform (Hardware, BIOS, EFI), OS (early boot, core kernel,
driver), and user policy decisions that all impact each other.  This doc breaks up these
configurations into five main areas:

.. toctree::
   :maxdepth: 1
   :caption: Overview

   self

.. toctree::
   :maxdepth: 2
   :caption: Device Reference

   devices/index.rst

.. toctree::
   :maxdepth: 2
   :caption: Platform Configuration

   platform/index.rst

.. toctree::
   :maxdepth: 3
   :caption: Linux Kernel Configuration:

   linux/index.rst

.. toctree::
   :maxdepth: 2
   :caption: Memory Allocation

   allocation/index.rst

.. toctree::
   :maxdepth: 2
   :caption: Use Cases

   use-case/index.rst

Contributors
############

Authors
*******

* Gregory Price
* Joshua Hahn

Editors
*******
todo

Thanks
******
todo
