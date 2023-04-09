Introduction to Late Launch
===========================

This is an introduction of the "Late Launch" process on x86-based systems to
establish a Dynamic Root of Trust for Measurement (DRTM). Late Launch is
another name for a Dynamic Launch of a system for x86-based platforms. As such
it is good to understand the difference between a Static Launch and a Dynamic
Launch on x86 platforms.  For x86 the fixed location used for a Static Launch
is known as the reset vector which maps to SPI flash storage. A Dynamic Launch
is achieved with a light-weight processor bootstrap initiated through a CPU
instruction. For Intel the capability that provides this is called TXT and is
initiated with the GETSEC[SENTER], summarily SENTER, instruction. For AMD it is considered part
of AMD's secure virtualization (AMD-V) and is initiated with the SKINIT
instruction.

An important function of x86 Late Launch CPU instructions is that they
"measure" the execution code provided for the launch. This action of "measure"
is accomplished by taking a cryptographic hash using an algorithm supported by
the TCG's Trusted Platform Module (TPM) so that it may store the measurement
within one of the TPM's Platform Configuration Registers (PCR). This initial
measurement, referred to as the Core Root of Trust Measurement (CRTM), is the
trust anchor for the DRTM. 

While the Intel and AMD implementations of Late Launch both achieve a DRTM,
how their implementations arrive at a DRTM are significantly different. As a
result each will be addressed separately.

## Intel Trusted eXecution Technology (TXT)
 
For TXT, Intel set about a holistic approach<sup>\[[1](#1)\]</sup> that
introduced the Safer Mode Extensions (SMX) instruction set. As a result TXT
provides for advanced security capabilities such as measuring System Management
Mode (SMM) when an SMI Transfer Monitor (STM) is in place. The TXT process is
built around the SINIT Authenticated Code Module (ACM) and a Measured Launch
Environment (MLE). The ACM is a binary provided by Intel and the MLE is a
software solution typically provided by the OS provider. Details about the ACM
and the MLE are explained in the "Intel TXT Measured Launch Environment
Developer's Guide". <sup>\[[2](#2)\]</sup>

### SENTER Procedure

From the moment when the SENTER instruction is invoked until execution control
is handed to the MLE, a series of computations are completed by the CPU and
then by the ACM to generate integrity assertions in the form of measurements
about the platform environment as well as the MLE that will be given control.
Details about the CPU's role in the launch can be found in the Intel Software
Developer's Manual (SDM) under Vol. 2D 6.2.3 para 3 and 6.3 GETSEC[SENTER].
<sup>\[[3](#3)\]</sup> The primary role for the CPU is to establish an
environment that minimizes the ability of external tampering and taking the
CRTM used for the DRTM. Below is an outline of the internal steps that the CPU
takes when the SENTER instruction is initial invoked,

1. Inhibit processor response to external events: INIT, A20M, NMI, and SMI.
2. Establish and check the location and size of the ACM to be executed by the ILP.
3. Check for the existence of an Intel® TXT-capable chipset.
4. Verify the current power management configuration is acceptable.
5. Broadcast a message to enable protection of memory and I/O from the activities of other processor agents.
6. Load the designated ACM into the authenticated code execution area.
7. Isolate the content of the authenticated code execution area from further state modification by external agents.
8. Authenticate the ACM.
9. Updated the TPM with the ACM's hash.
10. Initialize processor state based on the ACM header information.
11. Unlock the Intel® TXT-capable chipset private configuration register space and TPM locality 3 space.
12. Begin execution in the ACM at the defined entry point.

### TXT ACM 

The ACM is the portion that is responsible for implementing the advanced
security capabilities provided by TXT. To achieve this, the CPU provides a
highly privileged execution mode that is capable of inspecting System
Management Mode (SMM) memory (SMRAM), this is needed to allow the measurement
of the STI. As a result it is highly important that only authorized code is allowed
to execute in this mode. This is handled in step 8., the authentication of
the ACM. The details of this process can be found in section A.1.2 of the MLE
Developers Guide. <sup>\[[2](#2)\]</sup> The ACM is entrusted with a series of responsibilities,
ones of particular note are IOMMU protecting the MLE,
measuring the MLE, and the enforcement of the Launch Control Policy (LCP), 
 
#### Launch Control Policy

One of the capabilities provided by the ACM is the LCP Engine. The LCP is a
rarely used mechanism to enforce that a known environment is being used. Within
the ACM is an LCP engine that will look for a LCP in a designated TPM NVRAM
address. The LCP allows for defining expected values for TPM PCRs and/or the
expected hash value of the MLE. If a policy check fails, then the LCP Policy
Engine will trigger the ACM to error exit from the SENTER instruction.

### Measured Launch Environment

The final component in the Intel TXT process is the MLE, a software component
that is responsible for the secure setup and execution of the target runtime.
As the ACM conducted a transitive trust extension to the MLE, the MLE should
similar conduct a transitive trust extension to bring the target runtime within
the SMX trust boundary by protecting the runtime's memory from tampering,
measuring the runtime, and optionally enforcing policy to ensure only
authorized runtimes are allowed to execute.

## AMD Secure Startup

The AMD approach is a simpler one that allows more control over code executed
by the SKINIT instruction, to include environment setup and measurements, but
unlike Intel's ACM, execution is limited to the same accesses as
superprivileged mode. This means it is not possible to obtain a measurement of
SMRAM at the time of the late launch. Therefore the trust boundary of SKINIT
still bound to the SRTM. To use the Secure Startup capability, a Secure Loader
(SL) image must be loaded and passed to SKINIT. Details about building an SL
and calling the SKINIT instruction can be found in the AMD ARM64 Architecture
Programmer's Manual, Volume 2. <sup>\[[2](#4)\]</sup>

### SKINIT Procedure

When the SKINIT instruction is executed, the base address for the SL is passed
in the EAX register. The CPU will then execute the following sequence,

1. Reinitialize the processor state similar to INIT signal
2. Enter 32bit protected mode with paging disabled
3. Clear all general purpose registers except,
    * EAX: start address of SL
    * EDX: CPU model, family, and stepping
4. Secures registers,
    * Most MSRs retain their values (except those which might compromise SVM protections)
    * EFER MSR is cleared
    * setting DPD, R_INIT and DIS_A20M flags in the VM_CR register unconditionally to 1
4. Page align EAX and use as the start address for 64KBytes of DEV protection
5. Securely initializes AP(s)
    * clears Global Interrupt Flag (GIF)
    * setting DPD, R_INIT and DIS_A20M flags in the VM_CR register
6. Transmit SL image to TPM for hash, any failure will trigger SKINIT failure
7. Clear the GIF on the BSP which disables all interrupts, including NMI, SMI, and INIT
8. Update ESP to point at SL stack (SLB base + 65536)
9. Add SL entry offset to SL base and jump to that address

### Secure Loader

In the AMD world, the initial code executed by the SKINIT instruction is the
Secure Loader (SL). The SL is an Owner provided code base that is responsible
for securely initializing the system and handover to a Security Kernel (SK).
The SL must meet two primary conditions,

1. SL image's first two words contain the entry point offset and image size
2. SL image and stack must be less than 64KBytes
3. SL image must be loaded page aligned

Upon execution, the SL is responsible for protecting the remainder of the
execution environment through measurement and memory protection of the SK to which it
will be handing over control.

### Security Kernel

For AMD Secure Startup the last component is the SK. The SK can be an
intermediate kernel or a target runtime kernel. The situation that drives the
need for an intermediate kernel is for solutions that need to do more complex
security verification and/or hand-off to target runtime kernel that cannot be
implemented in less than 64Kbytes.

# Foot Notes
###### 1.
Grawrock, D. (2006). The Intel safer computing initiative. Hillsboro, Or.:
Intel Press.<br>
https://books.google.com/books?id=WmGjSgAACAAJ

###### 2.
Intel® Trusted Execution Technology (Intel® TXT)
Software Development Guide:
Measured Launched Environment Developer’s Guide

https://www.intel.com/content/dam/www/public/us/en/documents/guides/intel-txt-software-development-guide.pdf

###### 3.
Intel® 64 and IA-32 Architectures
Software Developer’s Manual,
Combined Volumes:
1, 2A, 2B, 2C, 2D, 3A, 3B, 3C, 3D and 4

https://software.intel.com/sites/default/files/managed/39/c5/325462-sdm-vol-1-2abcd-3abcd.pdf

###### 4.
AMD64 Architecture
Programmer’s Manual
Volume 2:
System Programming

https://www.amd.com/system/files/TechDocs/24593.pdf
