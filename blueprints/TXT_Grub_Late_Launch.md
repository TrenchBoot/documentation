TXT Grub Late Launcher
======================

## Purpose

The intent of this project is to extend Grub with the ability to call the Intel
SENTER instruction.

## Background

The Intel SENTER instruction is a means to initiate a "late launch" that
establishes a Dynamic Root of Trust Measuremnt (DRTM). The instruction call
requires the system to be in a specific state as enumerated below,

## Approach

Grub will be extended with the following capabilities,
 * An SENTER relocator that will,
  1. set protected mode
  2. enable cache
  3. enable native FPU error reporting
  4. ensure not in virtual-8086 mode
  5. verify no machine check in progress
  6. parse GETSEC[PARAMETERS]
  7. clear machine check regs
  5. senter as final instruction
 * Extend the late launch loader,
  1. Determine CPU type and select SKINIT or SENTER path
  1. load ACM and verify it matches platform
  2. verify SENTER is supported
  3. build pagetable for MLE
