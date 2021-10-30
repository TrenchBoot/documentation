===================================
Secure Launch Config and Interfaces
===================================

Configuration
=============

The settings to enable Secure Launch using Kconfig are under::

  "Processor type and features" --> "Secure Launch support"

A kernel with this option enabled can still be booted using other supported
methods. There are two Kconfig options for Secure Launch::

  "Secure Launch Alternate Authority usage"
  "Secure Launch Alternate Detail usage"

The help indicates their usage as alternate post launch PCRs to separate
measurements for more flexibility (both disabled by default).

To reduce the Trusted Computing Base (TCB) of the MLE [1]_, the build
configuration should be pared down as narrowly as one's use case allows.
The fewer drivers (less active hardware) and features reduces the attack
surface. E.g. in the extreme, the MLE could only have local disk access
and no other hardware support. Or only network access for remote attestation.

It is also desirable if possible to embed the initrd used with the MLE kernel
image to reduce complexity.

The following are a few important configuration necessities to always consider:

KASLR Configuration
-------------------

Secure Launch does not interoperate with KASLR. If possible, the MLE should be
built with KASLR disabled::

  "Processor type and features" -->
      "Build a relocatable kernel" -->
          "Randomize the address of the kernel image (KASLR) [ ]"

This unsets the Kconfig value CONFIG_RANDOMIZE_BASE.

If not possible, KASLR must be disabled on the kernel command line when doing
a Secure Launch as follows::

  nokaslr

IOMMU Configuration
-------------------

When doing a Secure Launch, the IOMMU should always be enabled and the drivers
loaded. However, IOMMU passthrough mode should never be used. This leaves the
MLE completely exposed to DMA after the PMR's [2]_ are disabled. The current default
mode is to use IOMMU in lazy translated mode but strict translated mode is the preferred
IOMMU mode and this should be selected in the build configuration::

  "Device Drivers" -->
      "IOMMU Hardware Support" -->
          "IOMMU default domain type" -->
              "(X) Translated - Strict"

It is recommended that no other command line options should be set to override
the default above.

Intel TXT Interface
===================

The primary interfaces between the various components in TXT are the TXT MMIO
registers and the TXT heap. The MMIO register banks are described in Appendix B
of the TXT MLE [1]_ Development Guide.

The TXT heap is described in Appendix C of the TXT MLE [1]_ Development
Guide. Most of the TXT heap is predefined in the specification. The heap is
initialized by firmware and the pre-launch environment and is subsequently used
by the SINIT ACM. One section, called the OS to MLE Data Table, is reserved for
software to define. This table is the Secure Launch binary interface between
the pre- and post-launch environments and is defined as follows::

        /*
         * Secure Launch defined MTRR saving structures
         */
        struct txt_mtrr_pair {
                u64 mtrr_physbase;
                u64 mtrr_physmask;
        } __packed;

        struct txt_mtrr_state {
                u64 default_mem_type;
                u64 mtrr_vcnt;
                struct txt_mtrr_pair mtrr_pair[TXT_OS_MLE_MAX_VARIABLE_MTRRS];
        } __packed;

        /*
         * Secure Launch defined OS/MLE TXT Heap table
         */
        struct txt_os_mle_data {
                u32 version;
                u32 boot_params_addr;
                u64 saved_misc_enable_msr;
                struct txt_mtrr_state saved_bsp_mtrrs;
                u32 ap_wake_block;
                u32 ap_wake_block_size;
                u64 evtlog_addr;
                u32 evtlog_size;
                u8 mle_scratch[64];
        } __packed;

Description of structure:

=====================  ========================================================================
Field                  Use
=====================  ========================================================================
version                Structure version, current value 1
boot_params_addr       Physical address of the zero page/kernel boot params
saved_misc_enable_msr  Original Misc Enable MSR (0x1a0) value stored by the pre-launch
                       environment. This value needs to be restored post launch - this is a
                       requirement.
saved_bsp_mtrrs        Original Fixed and Variable MTRR values stored by the pre-launch
                       environment. These values need to be restored post launch - this is a
                       requirement.
ap_wake_block          Pre-launch allocated memory block to wake up and park the APs post
                       launch until SMP support is ready. This block is validated by the MLE
                       before use.
ap_wake_block_size     Size of the ap_wake_block. A minimum of 16384b (4x4K pages) is required.
evtlog_addr            Pre-launch allocated memory block for the TPM event log. The event
                       log is formatted both by the pre-launch environment and the SINIT
                       ACM. This block is validated by the MLE before use.
evtlog_size            Size of the evtlog_addr block.
mle_scratch            Scratch area used post-launch by the MLE kernel. Fields:
 
                        - SL_SCRATCH_AP_EBX area to share %ebx base pointer among CPUs
                        - SL_SCRATCH_AP_JMP_OFFSET offset to abs. ljmp fixup location for APs
=====================  ========================================================================

Error Codes
-----------

The TXT specification defines the layout for TXT 32 bit error code values.
The bit encodings indicate where the error originated (e.g. with the CPU,
in the SINIT ACM, in software). The error is written to a sticky TXT
register that persists across resets called TXT.ERRORCODE (see the TXT
MLE Development Guide). The errors defined by the Secure Launch feature are
those generated in the MLE software. They have the format::

  0xc0008XXX

The low 12 bits are free for defining the following Secure Launch specific
error codes.

======  ================
Name:   SL_ERROR_GENERIC
Value:  0xc0008001
======  ================

Description:

Generic catch all error. Currently unused.

======  =================
Name:   SL_ERROR_TPM_INIT
Value:  0xc0008002
======  =================

Description:

The Secure Launch code failed to get an access to the TPM hardware interface.
This is most likely to due to misconfigured hardware or kernel. Ensure the
TPM chip is enabled and the kernel TPM support is built in (it should not be
built as a module).

======  ==========================
Name:   SL_ERROR_TPM_INVALID_LOG20
Value:  0xc0008003
======  ==========================

Description:

The Secure Launch code failed to find a valid event log descriptor for TPM
version 2.0 or the event log descriptor is malformed. Usually this indicates
that incompatible versions of the pre-launch environment and the MLE kernel.
The pre-launch environment and the kernel share a structure in the TXT heap and
if this structure (the OS-MLE table) is mismatched, this error is often seen.
This TXT heap area is setup by the pre-launch environment so the issue may
originate there. It could be the sign of an attempted attack.

======  ===========================
Name:   SL_ERROR_TPM_LOGGING_FAILED
Value:  0xc0008004
======  ===========================

Description:

There was a failed attempt to write a TPM event to the event log early in the
Secure Launch process. This is likely the result of a malformed TPM event log
buffer. Formatting of the event log buffer information is done by the
pre-launch environment so the issue most likely originates there.

======  ============================
Name:   SL_ERROR_REGION_STRADDLE_4GB
Value:  0xc0008005
======  ============================

Description:

During early validation a buffer or region was found to straddle the 4GB
boundary. Because of the way TXT does DMA memory protection, this is an
unsafe configuration and is flagged as an error. This is most likely a
configuration issue in the pre-launch environment. It could also be the sign of
an attempted attack.

======  ===================
Name:   SL_ERROR_TPM_EXTEND
Value:  0xc0008006
======  ===================

Description:

There was a failed attempt to extend a TPM PCR in the Secure Launch platform
module. This is most likely to due to misconfigured hardware or kernel. Ensure
the TPM chip is enabled and the kernel TPM support is built in (it should not
be built as a module).

======  ======================
Name:   SL_ERROR_MTRR_INV_VCNT
Value:  0xc0008007
======  ======================

Description:

During early Secure Launch validation an invalid variable MTRR count was found.
The pre-launch environment passes a number of MSR values to the MLE to restore
including the MTRRs. The values are restored by the Secure Launch early entry
point code. After measuring the values supplied by the pre-launch environment,
a discrepancy was found validating the values. It could be the sign of an
attempted attack.

======  ==========================
Name:   SL_ERROR_MTRR_INV_DEF_TYPE
Value:  0xc0008008
======  ==========================

Description:

During early Secure Launch validation an invalid default MTRR type was found.
See SL_ERROR_MTRR_INV_VCNT for more details.

======  ======================
Name:   SL_ERROR_MTRR_INV_BASE
Value:  0xc0008009
======  ======================

Description:

During early Secure Launch validation an invalid variable MTRR base value was
found. See SL_ERROR_MTRR_INV_VCNT for more details.

======  ======================
Name:   SL_ERROR_MTRR_INV_MASK
Value:  0xc000800a
======  ======================

Description:

During early Secure Launch validation an invalid variable MTRR mask value was
found. See SL_ERROR_MTRR_INV_VCNT for more details.

======  ========================
Name:   SL_ERROR_MSR_INV_MISC_EN
Value:  0xc000800b
======  ========================

Description:

During early Secure Launch validation an invalid miscellaneous enable MSR value
was found. See SL_ERROR_MTRR_INV_VCNT for more details.

======  =========================
Name:   SL_ERROR_INV_AP_INTERRUPT
Value:  0xc000800c
======  =========================

Description:

The application processors (APs) wait to be woken up by the SMP initialization
code. The only interrupt that they expect is an NMI; all other interrupts
should be masked. If an AP gets some other interrupt other than an NMI it will
cause this error. This error is very unlikely to occur.

======  =========================
Name:   SL_ERROR_INTEGER_OVERFLOW
Value:  0xc000800d
======  =========================

Description:

A buffer base and size passed to the MLE caused an integer overflow when
added together. This is most likely a configuration issue in the pre-launch
environment. It could also be the sign of an attempted attack.

======  ==================
Name:   SL_ERROR_HEAP_WALK
Value:  0xc000800e
======  ==================

Description:

An error occurred in TXT heap walking code. The underlying issue is a failure to
early_memremap() portions of the heap, most likely due to a resource shortage.

======  =================
Name:   SL_ERROR_HEAP_MAP
Value:  0xc000800f
======  =================

Description:

This error is essentially the same as SL_ERROR_HEAP_WALK but occurred during the
actual early_memremap() operation.

======  =========================
Name:   SL_ERROR_REGION_ABOVE_4GB
Value:  0xc0008010
======  =========================

Description:

A memory region used by the MLE is above 4GB. In general this is not a problem
because memory > 4Gb can be protected from DMA. There are certain buffers that
should never be above 4Gb though and one of these caused the violation. This is
most likely a configuration issue in the pre-launch environment. It could also
be the sign of an attempted attack.

======  ==========================
Name:   SL_ERROR_HEAP_INVALID_DMAR
Value:  0xc0008011
======  ==========================

Description:

The backup copy of the ACPI DMAR table which is supposed to be located in the
TXT heap could not be found. This is due to a bug in the platform's ACM module
or in firmware.

======  =======================
Name:   SL_ERROR_HEAP_DMAR_SIZE
Value:  0xc0008012
======  =======================

Description:

The backup copy of the ACPI DMAR table in the TXT heap is to large to be stored
for later usage. This error is very unlikely to occur since the area reserved
for the copy is far larger than the DMAR should be.

======  ======================
Name:   SL_ERROR_HEAP_DMAR_MAP
Value:  0xc0008013
======  ======================

Description:

The backup copy of the ACPI DMAR table in the TXT heap could not be mapped. The
underlying issue is a failure to early_memremap() the DMAR table, most likely
due to a resource shortage.

======  ====================
Name:   SL_ERROR_HI_PMR_BASE
Value:  0xc0008014
======  ====================

Description:

On a system with more than 4G of RAM, the high PMR [2]_ base address should be set
to 4G. This error is due to that not being the case. This PMR value is set by
the pre-launch environment so the issue most likely originates there. It could also
be the sign of an attempted attack.

======  ====================
Name:   SL_ERROR_HI_PMR_SIZE
Value:  0xc0008015
======  ====================

Description:

On a system with more than 4G of RAM, the high PMR [2]_ size should be set to cover
all RAM > 4G. This error is due to that not being the case. This PMR value is
set by the pre-launch environment so the issue most likely originates there. It
could also be the sign of an attempted attack.

======  ====================
Name:   SL_ERROR_LO_PMR_BASE
Value:  0xc0008016
======  ====================

Description:

The low PMR [2]_ base should always be set to address zero. This error is due to
that not being the case. This PMR value is set by the pre-launch environment
so the issue most likely originates there. It could also be the sign of an attempted
attack.

======  ====================
Name:   SL_ERROR_LO_PMR_MLE
Value:  0xc0008017
======  ====================

Description:

This error indicates the MLE image is not covered by the low PMR [2]_ range. The
PMR values are set by the pre-launch environment so the issue most likely originates
there. It could also be the sign of an attempted attack.

======  =======================
Name:   SL_ERROR_INITRD_TOO_BIG
Value:  0xc0008018
======  =======================

Description:

The external initrd provided is larger than 4Gb. This is not a valid
configuration for a Secure Launch due to managing DMA protection.

======  =========================
Name:   SL_ERROR_HEAP_ZERO_OFFSET
Value:  0xc0008019
======  =========================

Description:

During a TXT heap walk an invalid/zero next table offset value was found. This
indicates the TXT heap is malformed. The TXT heap is initialized by the
pre-launch environment so the issue most likely originates there. It could also
be a sign of an attempted attack. In addition, ACM is also responsible for
manipulating parts of the TXT heap so the issue could be due to a bug in the
platform's ACM module.

======  =============================
Name:   SL_ERROR_WAKE_BLOCK_TOO_SMALL
Value:  0xc000801a
======  =============================

Description:

The AP wake block buffer passed to the MLE via the OS-MLE TXT heap table is not
large enough. This value is set by the pre-launch environment so the issue most
likely originates there. It also could be the sign of an attempted attack.

======  ===========================
Name:   SL_ERROR_MLE_BUFFER_OVERLAP
Value:  0xc000801b
======  ===========================

Description:

One of the buffers passed to the MLE via the OS-MLE TXT heap table overlaps
with the MLE image in memory. This value is set by the pre-launch environment
so the issue most likely originates there. It could also be the sign of an attempted
attack.

======  ==========================
Name:   SL_ERROR_BUFFER_BEYOND_PMR
Value:  0xc000801c
======  ==========================

Description:

One of the buffers passed to the MLE via the OS-MLE TXT heap table is not
protected by a PMR. This value is set by the pre-launch environment so the
issue most likey  originates there. It could also be the sign of an attempted
attack.

======  =============================
Name:   SL_ERROR_OS_SINIT_BAD_VERSION
Value:  0xc000801d
======  =============================

Description:

The version of the OS-SINIT TXT heap table is bad. It must be 6 or greater.
This value is set by the pre-launch environment so the issue most likely
originates there. It could also be the sign of an attempted attack. It is also
possible though very unlikely that the platform is so old that the ACM being
used requires an unsupported version.

======  =====================
Name:   SL_ERROR_EVENTLOG_MAP
Value:  0xc000801e
======  =====================

Description:

An error occurred in the Secure Launch module while mapping the TPM event log.
The underlying issue is memremap() failure, most likely due to a resource
shortage.

======  ========================
Name:   SL_ERROR_TPM_NUMBER_ALGS
Value:  0xc000801f
======  ========================

Description:

The TPM 2.0 event log reports an unsupported number of hashing algorithms.
Secure launch currently only supports a maximum of two: SHA1 and SHA256.

======  ===========================
Name:   SL_ERROR_TPM_UNKNOWN_DIGEST
Value:  0xc0008020
======  ===========================

Description:

The TPM 2.0 event log reports an unsupported hashing algorithm. Secure launch
currently only supports two algorithms: SHA1 and SHA256.

======  ==========================
Name:   SL_ERROR_TPM_INVALID_EVENT
Value:  0xc0008021
======  ==========================

Description:

An invalid/malformed event was found in the TPM event log while reading it.
Since only trusted entities are supposed to be writing the event log, this
would indicate either a bug or a possible attack.

.. [1]
    MLE: Measured Launch Environment is the binary runtime that is measured and                                                                      
    then run by the TXT SINIT ACM. The TXT MLE Development Guide describes the                                                                       
    requirements for the MLE in detail.

.. [2]
    PMR: Intel VTd has a feature in the IOMMU called Protected Memory Registers.                                                                     
    There are two of these registers and they allow all DMA to be blocked
    to large areas of memory. The low PMR can cover all memory below 4Gb on 2Mb                                                                      
    boundaries. The high PMR can cover all RAM on the system, again on 2Mb
    boundaries. This feature is used during a Secure Launch by TXT.

