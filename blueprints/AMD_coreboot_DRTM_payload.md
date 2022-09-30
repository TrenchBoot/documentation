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
  - Added under `payloads` directory, perhaps this is not a proper place for it
  - Enabled through `Chipset/Launch DRTM payload before the real one` in config,
    available only for enabled platforms (depends on `CPU_AMD_PI`)
- Enabled SMMSTORE
  - 256KB of append only storage for UEFI authenticated variables
- IOMMU is always enabled if DRTM payload is used
  - Enforced on mainboard level due to specific implementation of runtime
    configuration options
- Loading and creating boot information tags for SKL
  - Code is built only for `CPU_AMD_PI` - CPU family 16h
  - Probably could be make to work on other AMD platforms with minimal changes

### SKL

- Support for simple payload - just base, size, entry point and argument:

```c
struct skl_tag_boot_simple_payload {
    struct skl_tag_hdr hdr;
    u32 base;
    u32 size;
    u32 entry;
    u32 arg;
} __packed;
```

### Tianocore

- SKINIT driver - discover if SKINIT was used, set GIF in that case
  - Performed early in DXE because other drivers (including SMMSTORE
    initialization) require enabled interrupts
- IOMMU driver - enable DMA access for devices that require and request it
  - Heavily used by drivers to access storage (USB, AHCI)
  - Uses `EDKII_IOMMU_PROTOCOL`
  - Many simplifications done, e.g. no differentiation between read and write
    permissions, no proper unmapping of device page tables, every invalidation
    uses `INVALIDATE_IOMMU_ALL` when smaller, targeted flushing would suffice

## TODOs

- (coreboot) information about DRTM TPM log location isn't passed to SKL
- (coreboot) SKL hash(es) aren't passed to SKL
- (SKL?) SMM is not measured
- (UEFI) APs are not started securely (i.e. without INIT) yet
- (UEFI) Tables exposed by coreboot (ACPI, SMBIOS) are not measured
- (UEFI) Polishing of IOMMU driver

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

Note that first boot takes few seconds to initialize SMMSTORE area, for release
builds this results in pause after `skl_main() is about to exit`, before any
output from UEFI appears. Do not be alarmed by this.

## Issues

- (UEFI) Support for AuthenticatedVariables is not discovered
- (UEFI) TPM driver returns error
