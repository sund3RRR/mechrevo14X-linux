# Mechrevo 14X Linux
This repository provides several fixes and tweaks to run Linux on MECHREVO 14X with Ryzen 7 8845HS.

## Table of Contents
1. [Fixes](#fixes)
   - [Fix artifacts/glitching/flickering on eDP display](#1-fix-artifactsglitchingflickering-on-edp-display)
   - [Fix immediate wakeup on sleep](#2-fix-immediate-wakeup-on-sleep)
   - [Fix keyboard issues after suspend/resume](#3-fix-keyboard-issues-after-suspendresume)
   - [Fix RJ-45 not working](#4-fix-rj-45-not-working)
2. [Additional TUXEDO Drivers and TUXEDO Control Center Installation](#additional-tuxedo-drivers-and-tuxedo-control-center-installation)
   - [tuxedo-drivers](#tuxedo-drivers)
   - [TUXEDO Control Center](#tuxedo-control-center)
3. [Cool Projects](#cool-projects)
4. [Development](#development)
    - [Get DSDT](#get-dsdt)
    - [Load DSDT](#load-dsdt)
    - [Get Manufacturer](#get-manufacturer)
    - [Decode WMI Events](#decode-wmi-events)
    - [Reverse-engineering](#reverse-engineering)

---

## Fixes

### 1. Fix artifacts/glitching/flickering on eDP Display
The issue is that the `amdgpu` driver has a bug that causes Panel Self Refresh (PSR) to malfunction. This problem only occurs on embedded displays with a resolution of 2880x1800 and Phoenix/Hawk Point graphics.

Related issue: [https://gitlab.freedesktop.org/drm/amd/-/issues/3388](https://gitlab.freedesktop.org/drm/amd/-/issues/3388)

**Solution**: Add `amdgpu.dcdebugmask=0x10` to the kernel parameters.

#### GRUB2
Edit:
#### ***`/etc/default/grub`***
```
GRUB_CMDLINE_LINUX_DEFAULT="... amdgpu.dcdebugmask=0x10"
```
Update:
```bash
update-grub2
```
#### systemd-boot
Edit:
#### ***`/boot/loader/entries/config.conf`***
```
options ... amdgpu.dcdebugmask=0x10
```
Update:
```bash
bootctl update
```
### 2. Fix Immediate Wakeup on Sleep
The issue likely occurs because the Embedded Controller (EC) is not functioning properly due to an ACPI/BIOS bug.

**Solution**: Add `acpi.ec_no_wakeup=1` to the kernel parameters.
#### GRUB2
Edit:
#### ***`/etc/default/grub`***
```
GRUB_CMDLINE_LINUX_DEFAULT="... acpi.ec_no_wakeup=1"
```
Update:
```bash
update-grub2
```
#### systemd-boot
Edit:
#### ***`/boot/loader/entries/config.conf`***
```
options ... acpi.ec_no_wakeup=1
```
Update:
```bash
bootctl update
```
### 3. Fix keyboard issues after suspend/resume
Add `i8042_nomux` to your kernel parameters.
#### GRUB2
Edit:
#### ***`/etc/default/grub`***
```
GRUB_CMDLINE_LINUX_DEFAULT="... i8042.nomux"
```
Update:
```bash
update-grub2
```
#### systemd-boot
Edit:
#### ***`/boot/loader/entries/config.conf`***
```
options ... i8042.nomux
```
Update:
```bash
bootctl update
```
### 4. Fix RJ-45 not working
Install `yt6801` driver. You can download the `.deb` from the TUXEDO [repo](https://deb.tuxedocomputers.com/ubuntu/pool/main/t/tuxedo-yt6801/)

## Additional TUXEDO drivers and TUXEDO Control Center installation
Since the MECHREVO 14X is based on the Tong Fang GX4HRXL board, similar to the TUXEDO InfinityBook Pro 14 Gen9, we can use TUXEDO drivers to get TUXEDO Control Center working.
### tuxedo-drivers

This should work for any .deb-based distro with any kernel (it will probably also work on Fedora/Arch, etc.; you just need to deal with dependencies).
1. Install the dependencies (for .deb based distros):
```bash
apt update && apt install -y git make gcc14 dkms linux-headers-$(uname -r)
```
2. Download `tuxedo-drivers`:
```bash
git clone https://gitlab.com/tuxedocomputers/development/packages/tuxedo-drivers.git
```
3. Apply patch to add MECHREVO as a vendor:
```bash
curl -L https://github.com/sund3RRR/Mechrevo14X-linux/raw/master/patches/add_mechrevo_vendor.patch | git apply
```
4. Install drivers:
```bash
make dkmsinstall
```
5. Reboot your system
### TUXEDO Control Center
Just install the latest `.deb` from the TUXEDO [repo](https://deb.tuxedocomputers.com/ubuntu/pool/main/t/tuxedo-control-center/). For Fedora/Arch, you might need to convert the package format to match your system package manager or build it from source.

Now you can adjust CPU clock, fan speed, disable CPU cores/SMT (just disable half the cores), and toggle Fn mode.

## Cool projects
- [mechrevo-wujie14-kmod](https://github.com/xuwd1/mechrevo-wujie14-kmod)

## Development
### Get DSDT
```bash
cat /sys/firmware/acpi/tables/DSDT > dsdt.dat
iasl -d dsdt.dat
```

### Load DSDT
```bash
iasl -tc dsdt.dsl
cp dsdt.aml /boot/acpi_override
```

### Get manufacturer
```bash
dmidecode -s system-manufacturer
```

### Decode WMI events
```bash
fwts wmi - > wmi.txt
```

### Reverse-engineering
https://gist.github.com/w568w/b2fc5f9d1f4dff13efe751abec27b396
