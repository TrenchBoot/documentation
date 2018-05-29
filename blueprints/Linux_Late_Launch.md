Linux Late Launch Kernel
========================

## Summary

The Linux kernel will be extended to make it function as the late launch kernel
for both AMD (Secure Loader) and Intel TXT (Measured Launch Environment). 

## Background

The late launch process provides a means to measure a target execution
environment and then jump into the environment as presribed by the CPU's late
launch protocol. This process provides a means by which to a establish a
hardware-based Root of Trust for Measurement (RTM). The implementation here
will provide a means to launch the Linux kernel as an RTM envrionment to enable
building trust in a platform's launch.

## Approach

A preliminary approach is as follows,
 * Create a late-launch landing zone (LZ) in 64k real code section of Linux
   kernel header
  * Sanitize any CPU state passed through late launch
  * Verify LZ signature with pubkey
  * Verify measurement from LZ signature against CRTM
  * (AMD) Create/Setup TPM Event log
  * Extend hash of pubkey into PCR 18, record in event log
   * This aligns to TXT's DA scheme
  * Parse Linux header to detect relevant details about kernel
  * DEV/DMAR protect remainder of Linux kernel
  * Extend hash of Linux kernel into PCR 17, record in event log
  * Prepare and then handoff to Linux kernel proper
   * Need to support Legacy/CSM and UEFI environments
 * Add a new start/launch path to Linux kernel
  * Secure rendezvous APUs
  * Process system state passed up from LZ
    * e.g. memory map, etc.
  * Setup Linux state for handover to mainline start up
  * Jump into mainline start up
