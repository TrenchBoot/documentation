# Quick Start Guide

A quick start guide to getting a Linux system running with the latest Secure
Launch bits from TrenchBoot. Note that this is a bare bones document meant to help
someone get up and running with Secure Launch. It does not contain detailed
descriptions of all the technologies and terminology involved in doing a
Secure Launch. The repository this document resides in as well as the
Linux Secure Launch documentation submitted with the Linux patch sets
(under Documentation/security/launch-integrity/) contain a plethora of other
resources and information that can be used to understand the Secure Launch
technology more broadly.

For topics not addressed by this document, please contact TrenchBoot developers
via the community site:

 - [Community](https://trenchboot.org/community)

## Platforms

The current patchset (version 9) only supports Intel TXT. AMD SKINIT support
is in the works and coming soon.

An Intel system (desktop, server, laptop) needs to be a vPro SKU in order to
have TXT support available. Generally speaking, vPro systems will advertise this
with a sticker somewhere on the unit. Intel TXT support usually needs to be
enabled in the firmware setup program. It depends on both the TPM and VTd being
enabled. The details on how to do this are system specific. To see if the CPU
supports TXT, run the following (SMX (Safe Mode Extensions) indicates the CPU
does support TXT):

`# grep smx /proc/cpuinfo`

Also note, the TrenchBoot project has a hardware test matrix though only the
Intel systems are relevant at present:

 - [Test Matrix](https://trenchboot.org/documentation/test_matrix/)

## Linux

TrenchBoot is an active open-source project for system launch integrity, from
which the Secure Launch feature is being upstreamed to the Linux kernel.

The following repository and branch have the latest release of the Secure
Launch feature. This is a vanilla Linux kernel based off a torvalds/master branch
snapshot at the time time patch set was assembled. The patches could be
applied to different distros of Linux, probably requiring some rebasing:

 - [Latest Linux Patch Set](https://github.com/TrenchBoot/linux/tree/linux-sl-master-5-16-24-v9)

The Secure Launch feature is enabled through a Kconfig setting and can
be found here using e.g. `make menuconfig`:

`"Processor type and features" -> "[ ] Secure Launch support"`

The Linux Secure Launch in-tree documentation mentioned in the first section
contains other instructions on properly configuring a Secure Launch kernel.

## GRUB

Each recent release of the Linux patches is accompanied by a GRUB branch
in TrenchBoot that works with the specified version. The branch for version
9 can be found here:

 - [GRUB for Version 9](https://github.com/TrenchBoot/grub/tree/grub-sl-2.12-v9)

This version of GRUB is based off of upstream GRUB 2.12 with the patches to
support the Secure Launch feature. The folloing is a basic set of instructions
for bulding a standalone version of UEFI GRUB on this branch:

```
$ cd <grub-branch-checkout-location>
$ ./bootstrap
$ mkdir build
$ cd build
$ ../configure --with-platform=efi --target=x86_64
$ make
$ ./grub-mkimage -O x86_64-efi -o grubx64.efi -p /EFI/redhat -d grub-core all_video boot btrfs cat chain configfile echo efifwsetup efinet ext2 fat font gfxmenu gfxterm gzio halt hfsplus iso9660 jpeg loadenv loopback lvm mdraid09 mdraid1x minicmd normal part_apple part_msdos part_gpt password_pbkdf2 png reboot regexp search search_fs_uuid search_fs_file search_label serial sleep syslinuxcfg test tftp video xfs backtrace http linux usb usbserial_common usbserial_pl2303 usbserial_ftdi usbserial_usbdebug keylayouts at_keyboard multiboot2
```

The final command will produce the UEFI GRUB image `grubx64.efi` needed.

## Configuration

There is a new GRUB command that instructs GRUB to initiate a Secure Launch called
`slaunch`. This is an example of a GRUB menuentry that would be used to do a Secure
Launch of the Linux kernel:

```
menuentry 'Linux with Secure Launch 6.8.0-rc3-master-v8' --unrestricted {
        load_video
        insmod gzio
        insmod part_gpt
        insmod xfs
        if [ x$feature_platform_search_hint = xy ]; then
                search --no-floppy --fs-uuid --set=root bba24662-776e-4396-9b1e-9ee5606d79b8
        else
                search --no-floppy --fs-uuid --set=root bba24662-776e-4396-9b1e-9ee5606d79b8
        fi
        slaunch
        linux /vmlinuz-6.8.0-rc3-master-v8 root=/dev/mapper/root ro crashkernel=auto resume=/dev/mapper/swap rd.lvm.lv=my/root rd.lvm.lv=my/swap rhgb console=ttyS0,115200n8 console=tty0 LANG=en_US.UTF-8
        initrd /initrd-6.8.0-rc3-master-v8.img
        slaunch_module /txt-sinit-for-given-platform
}
```

Note this example contains the optional `slaunch_module` command that tells GRUB to load an
external SINIT ACM for this configuration. In general, server platforms contain an existing
SINIT ACM in the firmware and this line is not needed. For client platforms, an external one
is required to be supplied. The SINIT ACM for a given platform can be acquired from Intel:

 - [Intel SINIT ACM information](https://www.intel.com/content/www/us/en/developer/articles/tool/intel-trusted-execution-technology.html)

## Validation

There are a number of ways to validate that a successful Secure Launch was done. Using serial
logging or `dmesg`, search for the string "TXT" after booting:

```
[root@my-system ~]# dmesg | grep TXT
[    0.000094] slaunch: Intel TXT setup complete
[    2.617782] slaunch: TXT AP startup vector address updated
```

That indicates a successful Secure Launch boot. Another way is to display the Secure Launch
TPM event log. This can be done as follows after booting (note only the tail end of the log
is shown here for brevity, the rest is snippped):

```
[root@my-system ~]# cat /sys/kernel/security/slaunch/eventlog | hexdump -C
...
[snip]
...
00000490  a3 e2 de 6b fb 1f 79 ef  c9 5e de bf ef bf 92 fb  |...k..y..^......|
000004a0  fc b2 89 ea 64 c1 d7 d2  99 fb 49 e6 12 00 00 00  |....d.....I.....|
000004b0  4d 65 61 73 75 72 65 64  20 53 4c 52 20 54 61 62  |Measured SLR Tab|
000004c0  6c 65 12 00 00 00 02 05  00 00 01 00 00 00 0b 00  |le..............|
000004d0  cd 64 bf e1 70 96 4c ce  53 2f 2f 7a 85 85 fe f0  |.d..p.L.S//z....|
000004e0  05 22 40 f6 62 18 bf 94  2a 2f 3d 14 b1 25 60 31  |."@.b...*/=..%`1|
000004f0  18 00 00 00 4d 65 61 73  75 72 65 64 20 62 6f 6f  |....Measured boo|
00000500  74 20 70 61 72 61 6d 65  74 65 72 73 11 00 00 00  |t parameters....|
00000510  02 05 00 00 01 00 00 00  0b 00 18 7d 80 8f 2c ca  |...........}..,.|
00000520  03 bf a7 54 ff 1d 16 6d  49 51 25 f6 bc ec 46 dc  |...T...mIQ%...F.|
00000530  23 a7 39 a8 db 96 28 8e  d4 1d 16 00 00 00 4d 65  |#.9...(.......Me|
00000540  61 73 75 72 65 64 20 4b  65 72 6e 65 6c 20 69 6e  |asured Kernel in|
00000550  69 74 72 64 12 00 00 00  02 05 00 00 01 00 00 00  |itrd............|
00000560  0b 00 11 02 09 6f c6 1d  78 11 87 1a 93 49 10 2f  |.....o..x....I./|
00000570  14 69 dd 45 b8 c3 03 e7  e6 80 6e 21 9b 87 47 90  |.i.E......n!..G.|
00000580  d6 27 1c 00 00 00 4d 65  61 73 75 72 65 64 20 4b  |.'....Measured K|
00000590  65 72 6e 65 6c 20 63 6f  6d 6d 61 6e 64 20 6c 69  |ernel command li|
000005a0  6e 65 12 00 00 00 02 05  00 00 01 00 00 00 0b 00  |ne..............|
000005b0  b2 29 3f 3c da 25 4a 78  61 be 76 91 3e 06 f9 5d  |.)?<.%Jxa.v.>..]|
000005c0  7d 6b 0d 75 6b 30 74 0c  26 b2 76 96 1e 60 19 a5  |}k.uk0t.&.v..`..|
000005d0  18 00 00 00 4d 65 61 73  75 72 65 64 20 55 45 46  |....Measured UEF|
000005e0  49 20 6d 65 6d 6f 72 79  20 6d 61 70 11 00 00 00  |I memory map....|
000005f0  04 05 00 00 01 00 00 00  0b 00 00 00 00 00 00 00  |................|
00000600  00 00 00 00 00 00 00 00  00 00 00 00 00 00 00 00  |................|
*
00008000
```
The final measurements starting with the description "Measured..." are put in the
log by the Secure Launch kernel code after successfully running. During a poweroff,
restart or a kexec of another kernel, the following log lines will show TXT being
properly disabled and SMX mode being exited.:

```
[  696.907094] slaunch: TXT clear secrets bit and unlock memory complete.
[  696.914827] slaunch: TXT SEXIT complete.
```

## Troubleshooting

Covering all the possible reasons why a Secure Launch might fail is beyond the scope
of this document. If problems are encountered, the first thing to check is the firmware
setting on the system:

 - Is the TPM enabled?
 - Are VTx and VTd enabled?
 - Is TXT enabled?

The next thing to check depending on the failure mode is the serial log or `dmesg` for
any errors traced out that might indicate what went wrong.

TXT also provides a sticky error register that will contain any previous error coming
from TXT or the Secure Launch kernel. If there was an error, the TXT error register can
be read using a GRUB command from within a GRUB shell on reboot. Drop into a GRUB shell
by typing `c` at the GRUB menu and run:

```
grub> slaunch
grub> slaunch_state
Secure launcher: Intel TXT
  TXT.STS: 0x0000000000004092
    SENTER.DONE.STS:        0
    SEXIT.DONE.STS:         1
    MEM-CONFIGLOCK.STS:     0
    PRIVATEOPEN.STS:        1
    TXT.LOCALITY1.OPEN.STS: 0
    TXT.LOCALITY2.OPEN.STS: 0
  TXT.ESTS: 0x00
    TXT_RESET.STS: 0
  TXT.E2STS: 0x0000000000000000
    SECRETS.STS: 0
  TXT.ERRORCODE: 0x00000000
  TXT.DIDVID: 0x00000001b0078086
    VID:    0x8086
    DID:    0xb007
    RID:    0x0001
    ID-EXT: 0x0000
  TXT.VER.FSBIF: 0xffffffff
  TXT.VER.QPIIF: 0x9d003000
    DEBUG.FUSE: 1
  TXT.SINIT.BASE: 0x77ec0000
```

This shows a number of the TXT register values including `TXT.ERRORCODE` which is the
one of interest. In this case the error is `0x0000000` meaning there was no previous
error. An error of the form `0xc0008XXX` is coming from the Secure Launch kernel code.
The errors are listed in the main header file:

 - [SL Error Codes](https://github.com/TrenchBoot/linux/blob/linux-sl-master-5-16-24-v9/include/linux/slaunch.h#L114)

Errors coming from other sources like the CPU or the SINIT ACM have different forms.
Consult the TXT documentation from Intel to determine what the error means.
