# PoC: coreboot with payload started through DRTM (AMD)

This proof of concept shows how TrenchBoot can be used to start coreboot
payload. One of use cases may be to root chain of trust in hardware on platforms
that don't support SRTM, or require proprietary tools and/or documentation that
isn't publicly available.

PC Engines apu2 platform is used in this PoC. Software stack consists of:

- coreboot,
- SKL,
- Tianocore edk2 payload.

## Modifications done to support this PoC

### coreboot

- Build system now includes SKL
- Enabled SMMSTORE
- IOMMU is always enabled if DRTM payload is used
- Loading and creating boot information tags for SKL

### SKL

- Support for simple payload - just base, size, entry point and argument

### Tianocore

- SKINIT driver - discover if SKINIT was used, set GIF in that case
- IOMMU driver (WIP) - enable DMA access for devices that require and request it
(USB, AHCI)

## TODOs

- (coreboot) information about DRTM TPM log location isn't passed to SKL
- (coreboot) SKL hash(es) aren't passed to SKL
- (SKL?) SMM is not measured
- (UEFI) APs are not started securely (i.e. without INIT) yet
- (UEFI) IOMMU driver is not finished, as a result neither booting from media
nor UEFI Shell are able to boot
- (UEFI) Tables exposed by coreboot (ACPI, SMBIOS) are not measured

## Building

Clone coreboot repository, checkout appropriate branch and clone submodules:

```shell
$ git clone https://github.com/pcengines/coreboot.git
$ cd coreboot
$ git checkout drtm_payload
$ git submodule update --init --checkout
```

Build using `coreboot-sdk` container:

```shell
$ docker run --rm -it -v $PWD:/home/coreboot/coreboot \
  -w /home/coreboot/coreboot -e USER_ID=$USER_ID -e GROUP_ID=$GROUP_ID \
  coreboot/coreboot-sdk:0ad5fbd48d /bin/bash
$ cp configs/config.pcengines_apu2_drtm_payload .config
$ make olddefconfig
$ make
```

This should produce `build/coreboot.rom` that is to be flashed into apu2. Make
sure you have a way of recovering through external flashing.

## Issues

- (UEFI) Support for AuthenticatedVariables is not discovered
- (UEFI) TPM driver returns error
