Super UEFIinSecureBoot Disk
===========================

Super UEFIinSecureBoot Disk is a bootable image with GRUB2 bootloader designed to be used as a base for recovery USB flash drives.

**Key feature:** disk is fully functional with UEFI Secure Boot mode activated. It can launch any operating system or .efi file, even with untrusted, invalid or missing signature.

## Features:

 * GRUB2 Bootloader
 * 32-bit (ia32) / 64-bit (x86_64) UEFI (+ Secure Boot) support
 * BIOS / UEFI CSM support
 * Launch any operating system
 * Launch any .efi executable from GRUB2
 * Launch any .efi executable from another .efi application
 * Load any UEFI drivers

## Based on:

 * [Red Hat shim](https://github.com/rhboot/shim) (v13 for x86_64, v15 for i386), [from Fedora](https://apps.fedoraproject.org/packages/shim-signed), signed with Microsoft key, for initial boot
 * Modified [Linux Foundation PreLoader](https://git.kernel.org/pub/scm/linux/kernel/git/jejb/efitools.git) to install circumventing UEFI Security Policy
 * GRUB2 with security bypass patches to chainloader, linux/linuxefi and shim

## Description

Secure Boot is a feature of UEFI firmware which is designed to secure the boot process by preventing the loading of drivers or OS loaders that are not signed with an acceptable digital signature.

Most of modern computers come with Secure Boot enabled by default, which is a requirement for Windows 10 certification process. Although it could be disabled on all typical motherboards in UEFI setup menu, sometimes it's not easily possible e.g. due to UEFI setup password in a corporate laptop which the user don't know.

This disk, after being installed on a USB flash drive and booted from, effectively disables Secure Boot protection features and temporary allows to perform almost all actions with the PC as if Secure Boot is disabled. This could be useful for data recovery, OS re-installation, or just for booting from USB without thinking about additional steps.

## Installation

Download [image file from releases page](https://github.com/ValdikSS/Super-UEFIinSecureBoot-Disk/releases), write it to USB flash using one of the following programs:

* [Rosa ImageWriter](http://wiki.rosalab.ru/en/index.php/ROSA_ImageWriter) (for Windows and Linux)
* [Etcher](https://www.balena.io/etcher/) (for Windows, Linux and macOS)

WARNING: all your USB flash data will be deleted.

The image contains single FAT32 500MiB partition. Use [gparted](https://gparted.org/) or similar tool to resize it to get full USB drive space.

## Usage

First boot on a PC with Secure Boot will show Access Violation message box. Press OK and choose "Enroll cert from file" menu option. Select `ENROLL_THIS_KEY_IN_MOKMANAGER.cer` and confirm certificate enrolling.

Computers without Secure Boot will boot to GRUB without manual intervention.

## FAQ

* **Does this disk work in Secure Boot?**  
Yes, it does. It loads any unsigned or untrusted Linux kernel or .efi file or driver, after first-boot manual key enrolling using MokManager software. You don't need to disable Secure Boot to perform fist-boot key enrolling.

* **Does this disk work on UEFI-based computers without Secure Boot, or with Secure Boot disabled?**  
Yes, it would work like a stock GRUB2.

* **Does this disk work on older computers with BIOS?**  
Yes, it works just as any other GRUB2 bootloader.

* **Can this disk be used to bypass Secure Boot in UEFI bootkit/virus?**  
No, not really. *This* disk requires manual intervention of a physical user on first boot, which eliminates bootkit purpose to be stealth.

* **Can I replace GRUB with another EFI bootloader (rEFInd, syslinux, systemd-boot)?**  
Yes, replace `grubx64_real.efi`/`grubia32_real.efi` with your files. The bootloader does not require to be signed and should also start any .efi files thanks for Security Policy installed by `grubx64.efi`/`grubia32.efi` (PreLoader), just as GRUB2 included in disk.

## Technical information

UEFI boot process of this disk is performed in 3 stages.

`bootx64.efi (shim) → grubx64.efi (preloader) → grubx64_real.efi (grub2) → EFI file/OS`

**Stage 1**: motherboard loads shim. Shim is a special loader which just loads next executable, grubx64.efi (preloader) in our case. Shim is signed with Microsoft key, which allows it to be launched in Secure Boot mode on all stock PC motherboards.  
Shim contains embedded Fedora certificate (because it's extracted from Fedora repository). If Secure Boot is enabled, since grubx64.efi is not signed with embedded Fedora certificate, shim boots another executable, MokManager.efi, which is a special shim key management software. MokManager asks user to proceed with key or hash enrolling process.  
Newer versions of shim install hooks for UEFI LoadImage, StartImage, ExitBootServices and Exit functions to "Harden against non-participating bootloaders", which should be bypassed for this disk use-case. Fedora's shim does not install custom UEFI security policies, that's why it's not possible to load self-signed efi files from second stage bootloader, even if you add their hashes or certificates using MokManager.

**Stage 2**: preloader is a software similar to shim. It also performs executable validation and loads next efi file. Preloader included in this disk is a stripped down version which performs only one function: install allow-all UEFI security policy. This permits loading of arbitrary efi executables with LoadImage/StartImage UEFI functions even outside GRUB (for example, in UEFI Shell), and bypasses shim hardening.

**Stage 3**: GRUB2 is a well-known universal bootloader. It has been patched to load Linux kernel without additional vertification (linux/linuxefi commands), load .efi binaries into memory and jump into its entry point (chainloader command), and to mimic "participating bootloader" for shim.

## Additional information

Read my article on this topic: [Exploiting signed bootloaders to circumvent UEFI Secure Boot](https://habr.com/ru/post/446238/) (also [available in Russian](https://habr.com/ru/post/446072/))

## Notes

Super UEFIinSecureBoot Disk GRUB2 sets `suisbd=1` variable. It could be used to detect disk's patched GRUB2 in a `grub.conf` shared between multiple bootloaders.

Since version 3, GRUB uses stock UEFI .efi file loader, as there are some problems with internal loader implementation. To use internal loader, add `set efi_internal_loader=1` into GRUB configuration file. Both methods can load untrusted .efi files.
