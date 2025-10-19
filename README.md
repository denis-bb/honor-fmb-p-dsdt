# Honor Magicbook Pro 14 2025 (FMB-P) DSDT patch

This patch fix issue in DSDT table in BIOS version 1.13 (release date 05/08/2025)
It remove unnecesary bad NFC device record.

## How to use this

1. Download `dsdt.aml`, place it in `/boot`
2. Add `acpi /dsdt.aml` to the end of `/etc/grub.d/40_custom`
3. Execute `update-grub`
4. Reboot

## How to run installer in some distros

In some distros installers does not boot cause bugged DSDT, for example Debian or Ubuntu. It's stuck on root device search cause don't see any USB controllers and devices
So, make this:
* Place `dsdt.aml` to boot directory on installation disk
* During boot in grub (in ubuntu this is a place where you see "Try or nstall Ubuntu") press `e`, add line right before `linux ... bla, bla, bla` and wrote `acpi /dsdt.aml
* Press `Ctrl+X`

To boot in installed system after installation you must place `dsdt.aml` to /boot and add to kernel commandline in grub `acpi /dsdt.aml`.

## Decompile
To decompile DSDT please use `iasl` from `acpia` package.
```
acpidump -b
iasl -e ssdt*.dat -d dsdt.dat
```
In this step error aquired
```
Firmware Error (ACPI): Failure creating named object [\_SB.PC00.XHCI.RHUB.HS03._UPC], AE_ALREADY_EXISTS (20250404/dswload-495)
ACPI Error: AE_ALREADY_EXISTS, During name lookup/catalog (20250404/psobject-372)
Could not parse ACPI tables, AE_ALREADY_EXISTS
```
Just find and remove one of ssdt*.dat file that content this object. In my case this was ssdt23.dat. But ssdt number may be different from boot to boot.

After decompiling try to compile DSDT back
```
iasl -ve -tc dsdt.dsl
```
Fix all errors - just remove lines with error from dsdt.dsl

After successful compiling check ACPI tables execution
```
acpiexec ssdt*.dat dsdt.aml 
```
You must not see any errors.
