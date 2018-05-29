AMD Grub Late Launcher
======================

## Purpose

The intent of this project is to extend Grub with the ability to call the AMD
SKINIT instruction.

## Background

The AMD SKINIT instruction is a means to initiate a "late launch" that
establishes a Dynamic Root of Trust Measuremnt (DRTM). The instruction call
requires the system to be in a specific state as enumerated below,
 * SVM check, either the `EFER.SVME` bit is set to 1 or the feature flag `CPUID
   Fn8000_0001_ECX[SKINIT]` is set to 1
 * The CPU must be in protected mode
 * All microcode needs to be unloaded

## Approach

Grub will be extended with the following capabilities,
 * An SKINIT relocator that will,
  1. set protected mode
  2. enalbe apic
  3. verify no machine check in progress
  4. clear machine check regs
  5. skinit as final instruction
 * A late launch loader that will,
  1. load kernel starting at 0x100000, compatibility with a Linux Secure Loader
  2. verify SVM is supported
  3. disable all TPM localities
  4. evict microcode
  5. send INIT IPI to all APs
