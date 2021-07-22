UCX
===

This folder contains instructions for building UCX libraries on UK HPC systems.

Unified Communication X is an open-source, production-grade communication framework for data-centric
and high-performance applications. See the [OpenUCX home page](https://www.openucx.org/).

The UCX API is typically used for the Point-to-point Management Layer (PML) within OpenMPI's Modular Component Architecture (MCA).
For that reason, the UCX libraries are not used directly: the UCX library paths are required when building OpenMPI
(see the --with-ucx configure option).

History
-------

 Date | Person | System | Version | Notes
 ---- | ------ | ------ | ------- | -----
 2021-07-22 | Michael Bareford | Cirrus | 1.9.0 | GCC 8.2.0

Build Instructions
------------------

* [UCX 1.9.0 Cirrus Build Instructions (GCC 8)](build_ucx_1.9.0_cirrus_gcc8.md)

Notes
-----

