AMD Landing Zone
================

## Purpose

The intent of this project is to implement the earliest code that is launched by
a DL Event on AMD platforms.

## Background

Contrary to the TXT solution, SKINIT is a single CPU instruction after which the
execution is passed to the user-provided block of code, called Secure Loader in
AMD documents. To some extent, this code can be treated as the AMD equivalent of
ACM, except that it doesn't have to be signed by the CPU vendor.

SKINIT extends PCR17 with the hash of SL, which size is specified in the SL
header (maximum of 64K - 1), protects SLB (Secure Launch Block, always 64K)
against DMA access from the devices, puts the CPU in defined execution state and
jumps to the entry point also specified in the header. At the entry point to the
SL the CPU works in flat 32-bit protected mode with paging disabled. Most
general purpose registers are cleared. Interrupts are disabled (held pending
until re-enabled), including NMI, SMI and INIT. Refer to [AMD APM Vol 2](../references/AMD64-Architecture-Programmers-Manual_Volume-2_Ch15.27.pdf)
for more details.

Landing Zone is an implementation of Secure Loader.

## Approach

This is a high-level overview of tasks performed by Landing Zone:
1. set GDT and segment registers (only CS and SS are valid after SKINIT)
2. enable Long Mode (optional)
3. initialize TPM interface
4. initialize TPM event log
   * log what was done by SKINIT
5. extend PCR18 with the hash of data passed by the bootloader to the LZ
   * log that event
6. obtain information about the kernel from the data provided by a bootloader,
   most notably:
   * kernel location
   * kernel size
   * entry point
7. extend PCR17 with the hash of the kernel
   * log that event
8. set CPU registers according to the kernel boot protocol
9. jump to the entry point of kernel

### Things that yet have to be implemented

#### DMA protection must be extended to cover any additional code and data

This includes mainly kernel and zero page. As of now, it is impossible to do
this securely.

[AMD APM Vol 2](../references/AMD64-Architecture-Programmers-Manual_Volume-2_Ch15.27.pdf)
states that it should be done with DEV (Device Exclusion Vector), but that is no
longer implemented on new CPUs, except for initial SLB protection (starting with
Family 17h, even that protection is handled differently).

DEV was surpassed by the [IOMMU](https://www.amd.com/system/files/TechDocs/48882_IOMMU_3.05_PUB.pdf).
It is much more capable than DEV, but it also requires more data and code to set
up. Those are not atomic operations, data must be first written to memory by the
CPU, then CPU sends information to the IOMMU that it can parse that data, and
only then the IOMMU actually reads that memory. A rogue device could potentially
modify that data in the meantime, so it all should happen in the protected part
of the memory.

The issue is that IOMMU is also treated as a device, so it cannot access the
memory protected by the DEV (or what's left of it). There are two possible
options, both of which result in window of opportunity for unauthorized change
of the memory:
* turn off the SLB protection before sending commands to the IOMMU
* build IOMMU structures outside of SLB

The second approach requires also that the space for those structures is
reserved and that LZ knows about its location.

## Build options

There are 3 configurable options, specified as command line parameters for
`make`. All of them are disabled by default, they can be enabled by running
`make <opt>=y`, where `<opt>` is one of the flags listed below. Note that
between builds with different sets of options you **must** run `make clean`.

* **LTO** - link time optimizations. Generally reduces the size of resulting
  binary and execution time, but the exact difference is hard to predict.
* **32** - do not jump to 64b mode. Saves 7*4 KB by not producing paging tables,
  and some additional space because of shorter GDT and less strict padding.
* **DEBUG** - enable debug output on serial port (0x3f8). Increases the size by
  about 1.3 KB.
