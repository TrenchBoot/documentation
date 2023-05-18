======================
Secure Launch Overview
======================

Overview
========

Prior to the start of the TrenchBoot project, the only active Open Source
project supporting dynamic launch was Intel's tboot project to support their
implementation of dynamic launch known as Intel Trusted eXecution Technology
(TXT). The approach taken by tboot was to provide an exokernel that could
handle the launch protocol implemented by Intel's special loader, the SINIT
Authenticated Code Module (ACM [2]_) and remained in memory to manage the SMX
CPU mode that a dynamic launch would put a system. While it is not precluded
from being used for doing a late launch, tboot's primary use case was to be
used as an early launch solution. As a result the TrenchBoot project started
the development of Secure Launch kernel feature to provide a more generalized
approach. The focus of the effort is twofold, the first is to make the Linux
kernel directly aware of the launch protocol used by Intel, AMD/Hygon, Arm, and
potentially OpenPOWER. The second is to make the Linux kernel be able to
initiate a dynamic launch. It is through this approach that the Secure Launch
kernel feature creates a basis for the Linux kernel to be used in a variety of
dynamic launch use cases.

.. note::
    A quick note on terminology. The larger open source project itself is
    called TrenchBoot, which is hosted on GitHub (links below). The kernel
    feature enabling the use of the x86 technology is referred to as "Secure
    Launch" within the kernel code.

Goals
=====

The first use case that the TrenchBoot project focused on was the ability for
the Linux kernel to be started by a dynamic launch, in particular as part of an
early launch sequence. In this case the dynamic launch will be initiated by a
boot loader with associated support added to it, for example the first targeted
boot loader in this case was GRUB2. An integral part of establishing a
measurement-based launch integrity involves measuring everything that is
intended to be executed (kernel image, initrd, etc) and everything that will
configure that kernel to execute (command line, boot params, etc). Then storing
those measurements in a protected manner. Both the Intel and AMD dynamic launch
implementations leverage the Trusted Platform Module (TPM) to store those
measurements. The TPM itself has been designed such that a dynamic launch
unlocks a specific set of Platform Configuration Registers (PCR) for holding
measurement taken during the dynamic launch.  These are referred to as the DRTM
PCRs, PCRs 17-22. Further details on this process can be found in the
documentation for the GETSEC instruction provided by Intel's TXT and the SKINIT
instruction provided by AMD's AMD-V. The documentation on these technologies
can be readily found online; see the `Resources`_ section below for references.

.. note::
    Currently only Intel TXT is supported in this first release of the Secure
    Launch feature. AMD/Hygon SKINIT and Arm support will be added in a
    subsequent release.

To enable the kernel to be launched by GETSEC a stub, the Secure Launch stub,
must be built into the setup section of the compressed kernel to handle the
specific state that the dynamic launch process leaves the BSP. Also the Secure
Launch stub must measure everything that is going to be used as early as
possible. This stub code and subsequent code must also deal with the specific
state that the dynamic launch leaves the APs as well.

Design Decisions
================

A number of design decisions were made during the development of the Secure
Launch feature. The two primary guiding decisions were:

 - Keeping the Secure Launch code as separate from the rest of the kernel
   as possible.
 - Modifying the existing boot path of the kernel as little as possible.

The following illustrate how the implementation followed these design
decisions:

 - All the entry point code necessary to properly configure the system post
   launch is found in st_stub.S in the compressed kernel image. This code
   validates the state of the system, restores necessary system operating
   configurations and properly handles post launch CPU states.
 - After the sl_stub.S is complete, it jumps directly to the unmodified
   startup_32 kernel entry point.
 - A single call is made to a function sl_main() prior to the main kernel
   decompression step. This code performs further validation and takes the
   needed DRTM measurements.
 - After the call to sl_main(), the main kernel is decompressed and boots as
   it normally would.
 - Final setup for the Secure Launch kernel is done in a separate Secure
   Launch module that is loaded via a late initcall. This code is responsible
   for extending the measurements taken earlier into the TPM DRTM PCRs and
   setting up the securityfs interface to allow access the TPM event log and
   public TXT registers.
 - On the reboot and kexec paths, calls are made to a function to finalize the
   state of the Secure Launch kernel.

The one place where Secure Launch code is mixed directly in with kernel code is
in the SMP boot code. This is due to the unique state that the dynamic launch
leaves the APs in. On Intel this involves using a method other than the
standard INIT-SIPI sequence.

A final note is that originally the extending of the PCRs was completed in the
Secure Launch stub when the measurements were taken. An alternative solution
had to be implemented due to the TPM maintainers objecting to the PCR
extensions being done with a minimal interface to the TPM that was an
independent implementation of the mainline kernel driver. Since the mainline
driver relies heavily on kernel interfaces not available in the compressed
kernel, it was not possible to reuse the mainline TPM driver. This resulted in
the decision to move the extension operations to the Secure Launch module in
the mainline kernel where the TPM driver would be available.

Basic Boot Flow
===============

Pre-launch: *Phase where the environment is prepared and configured to initiate the
secure launch in the GRUB bootloader.*

 - Prepare the CPU and the TPM for the launch.
 - Load the kernel, initrd and ACM [2]_ into memory.
 - Setup the TXT heap and page tables describing the MLE [1]_ per the
   specification.
 - Initiate the secure launch with the GETSET[SENTER] instruction.

Post-launch: *Phase where control is passed from the ACM to the MLE and the secure
kernel begins execution.*

 - Entry from the dynamic launch jumps to the SL stub.
 - SL stub fixes up the world on the BSP.
 - For TXT, SL stub wakes the APs, fixes up their worlds.
 - For TXT, APs are left halted waiting for an NMI to wake them.
 - SL stub jumps to startup_32.
 - SL main locates the TPM event log and writes the measurements of
   configuration and module information into it.
 - Kernel boot proceeds normally from this point.
 - During early setup, slaunch_setup() runs to finish some validation
   and setup tasks.
 - The SMP bring up code is modified to wake the waiting APs. APs vector
   to rmpiggy and start up normally from that point.
 - SL platform module is registered as a late initcall module. It reads
   the TPM event log and extends the measurements taken into the TPM PCRs.
 - SL platform module initializes the securityfs interface to allow
   access to the TPM event log and TXT public registers.
 - Kernel boot finishes booting normally
 - SEXIT support to leave SMX mode is present on the kexec path and
   the various reboot paths (poweroff, reset, halt).

PCR Usage
=========

The TCG DRTM architecture there are three PCRs defined for usage, PCR.Details
(PCR17), PCR.Authorities (PCR18), and PCR.DLME_Authority (PCR19). For a deeper
understanding of Detail and Authorities it is recommended to review the TCG
DRTM architecture.

Primarily the Authorities is expected to be in the form of a cryptographic
signature of a component in the DRTM chain. A challenge for Linux kernel is
that it may or may not have an authoritative signature associated with it and
Secure Launch intends to support a maximum number of configurations. To support
the Details/Authority scheme Secure Launch is built with the concept that
the runtime configuration of a kernel is the "authority" under which the user
executed the kernel. By default the authority for the kernel is extended into
PCR.Authorities with a Kconfig option to have it extended into PCR.DLME_Authority.

An extension Secure Launch introduces is the PCR.DLME_Detail (PCR20) PCR.
Enabling the usage of this PCR is set through Kconfig and results in any DRTM
components measured by the kernel, e.g. external initrd image, to be extended
into the PCR. When combined with Secure Launch's user authority being stored in
PCR.DLME_Authority allows the ability to seal/attest to different variations of
platform details/authorities with user details/authorities. An example of this
was presented in the FOSDEM - 2021 talk "Secure Upgrades with DRTM".

Resources
=========

The TrenchBoot project including documentation:

https://github.com/trenchboot

Trusted Computing Group's D-RTM Architecture:

https://trustedcomputinggroup.org/wp-content/uploads/TCG_D-RTM_Architecture_v1-0_Published_06172013.pdf

TXT documentation in the Intel TXT MLE Development Guide:

https://www.intel.com/content/dam/www/public/us/en/documents/guides/intel-txt-software-development-guide.pdf

TXT instructions documentation in the Intel SDM Instruction Set volume:

https://software.intel.com/en-us/articles/intel-sdm

AMD SKINIT documentation in the System Programming manual:

https://www.amd.com/system/files/TechDocs/24593.pdf

GRUB pre-launch support patchset (WIP):

https://lists.gnu.org/archive/html/grub-devel/2020-05/msg00011.html

FOSDEM 2021: Secure Upgrades with DRTM

https://archive.fosdem.org/2021/schedule/event/firmware_suwd/

.. [1]
    MLE: Measured Launch Environment is the binary runtime that is measured and                                                                      
    then run by the TXT SINIT ACM. The TXT MLE Development Guide describes the                                                                       
    requirements for the MLE in detail.

.. [2]
    ACM: Intel's Authenticated Code Module. This is the 32b bit binary blob that                                                                     
    is run securely by the GETSEC[SENTER] during a measured launch. It is described                                                                  
    in the Intel documentation on TXT and versions for various chipsets are
    signed and distributed by Intel.
