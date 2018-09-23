TrenchBoot Use Cases
====================

TrenchBoot is meant to be a universal framework to enable building integrity in
the launch process of systems. To relate to real world usage, it is good to
have a set of use cases that explain a subset of situations where TrenchBoot is
applicable and how it would work in those situations. Below are a series of use
cases that are actively being investigated and/or worked on.


## Crowd Sourcing Integrity

There is currently no known public authority available to verify BIOS/Firmware
PCR values. TrenchBoot would like to become such an authority but there is the
challenge of how to obtain all these values in a manner that provide assurance
to the authenticity of the values. Crowd sourcing provides the best means to
collect the largest and most diverse set of values. The challenge with crowd
sourcing the values is how to establish authenticity of the values. This
challenge can be overcome with a TrenchBoot based live CD that establishes an
attestation identity provisioned by a TrenchBoot Attestation Certificate
Authority (ACA).

## Network Attestation Launch

An individual or enterprise may not want to allow a system to boot on to their
network unless it is running a known configuration. When TrenchBoot is
installed onto a system it will work in conjunction with a TrenchBoot ACA
(public or private instance) that provides a key management service. TrenchBoot
will hold a potion of a Shamir Secret Sharing key with another portion held by
the key management service. For the system to boot it will attest to key
management service to obtain key fragment that will allow it to unlock system
disk.

## Travel Laptop 2FA Launch

Will traveling there are times when an individual looses positive control of
their device. During these times attackers can launch physical access attacks.
For this configuration TrenchBoot will "double chain wrap" the encryption key
for decrypting the system where each chain wrap correlates to an authentication
factor. Working internal to external, the system drive key is encrypted with
the first wrap key that is in turned encrypted with the second wrap key. The
first wrap key is stored on a removable token device, e.g. YubiKey, and the
second wrap key is sealed in a TPM NVRAM slot. For a system to boot it must
have launched with the correct firmware and the token must be present.
