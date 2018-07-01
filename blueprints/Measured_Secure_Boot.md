Measured Secure Boot
====================

## Purpose

To build a Measured Secure Boot (MSB) implementation that uses a hardware based
Root of Trust for Measurement (RTM) from which to build a Root of Trust for
Verification (RTV).

## Background

To date the only supported Late Launch environment is the tboot project which
is maintained by Intel for their Intel TXT capability. While tboot provides a
complete capability, it is limited and has a number of deficiencies. While it
would be possible to build a Measured Secure Boot solution around tboot, it
will still be limited in comparison to approach provided by this blueprint.

Before detailing the approach and the improvements it provides, it is good to
understand how the tboot environment works. Before tboot can run, it must be
loaded into memory along with the target kernel it will launch into by either a
legacy multiboot compliant bootloader or by an UEFI boot manager. Tboot itself
is composed of two main parts, pre-launch and post-launch execution paths. Below
is a top-level execution flow for tboot from boot loader/manager to the trusted
kernel that will be given control.

```
                        |--          tboot             --|
boot ldr/mngr --load--> pre-launch --SENTER--> post-launch --launch--> trusted kernel
```

In terms of integrity policy, tboot policy can only be used to measure and
verify any multiboot modules loaded by the boot loader/manager. Typically tboot
policy is used in one of two manners, enforcing and non-enforcing. In both
cases the policy is used to control what tboot will measure and which PCRs
those measurements are stored. When an enforcing policy is in place, the
measurements are compared with values populated in the policy and will take
a selected action when the measurements do not match. With a non-enforcing
policy, measurement comparison is not done and it is left to the trusted
kernel to take action on the measurements. This means if advanced actions other
than measurement verification is desired, then the trusted kernel must be aware
and made to take the actions.

## Approach

The x86 Late Launch capability will be used to establish an RTM using a Dynamic
Root of Trust Measurement (DRTM) that will include the device owner's RSA
public key. The Late Launch will start a TrenchBoot Security Engine (TSE) that
is capable of simple measurement verification similar to tboot, but will be
able to do advance actions such as a KMIP attestation for device encryption
key.

MSB will consist of an enhancement to the GNU GRUB bootloader (grub), an
extended Linux kernel, and the uroot initramfs environment. The enhancement to
grub will be to add TPM 1.2/2.0 support along with relocators for AMD and Intel
Late Launch instructions. The Linux kernel will be extend to function as a
post-launch kernel that will run the TSE. Details for each of these are
documented in their own respective blueprints. Finally, below is the execution
flow for MSB for comparison with tboot's execution flow above.
```
grub --SENTER/SKINIT--> Linux/TSE --kexec--> trusted kernel
```
