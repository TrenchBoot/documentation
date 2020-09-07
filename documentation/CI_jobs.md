# Jobs description

## landing-zone

### build_debug_disabled

Job builds landing-zone with disabled debug option.

### build_debug_enabled

Job builds landing-zone with enabled debug option.

### build_nixpgs_debug_disabled

Job triggers building landing-zone NixOS package with disabled debug option.

### build_nixpkgs_debug_enabled

Job triggers building landing-zone NixOS package with enabled debug option.

---

## grub-tb

### build_and_install

Job builds TrenchBoot version of the GRUB. Then installs the GRUB in the docker
and checks if required modules are available.

### build_nixpkg

Job triggers building the GRUB package for the NixOS.

---

## linux-tb

### build

Job builds linux kernel with the secure launch driver.

### build_nixpkg

Job triggers building the NixOS package of the TrenchBoot linux kernel.

---

## nixos-trenchboot-configs

### build_lz_debug_disabled_local

Job builds NixOS landing-zone package with disabled debug option.

### build_lz_debug_enabled_local

Job builds NixOS landing-zone package with enabled debug option.

### build_grub

Job builds NixOS package of the TrenchBoot GRUB.

### build_linux

Job builds NixOS package of the TrenchBoot linux kernel.

---

## meta-trenchboot

### build

Job builds legacy Yocto TrenchBoot for the PC Engines apu2.

### build_uefi

Job builds UEFI Yocto TrenchBoot image for generic x86-64 hardware.

### test_boot_apu2

Job installs the legacy Yocto TrenchBoot image on the the PC Engines apu2.
Then boot the platform without and with DRTM. In the end the job verifies,
if PCR values correspond to manually extended.

### test_boot_asrock

Job installs the UEFI Yocto TrenchBoot image on the the ASRock-R1000V.
Then boot the platform without and with DRTM. In the end the job verifies,
if PCR values correspond to manually extended.

### test_boot_supermicro

Job installs the UEFI Yocto TrenchBoot image on the Supermicro M11SDV-8CT-LN4F.
Then boot the platform without and with DRTM. In the end the job verifies,
if PCR values correspond to manually extended.

---

## Test infrastructure

* [PC Engines apu2](https://www.pcengines.ch/apu2.htm)
* [ASRock-R1000V](https://www.asrockind.com/en-gb/4X4%20BOX-R1000V)
* [Supermicro M11SDV-8CT-LN4F](https://www.supermicro.com/en/products/motherboard/M11SDV-8CT-LN4F)
