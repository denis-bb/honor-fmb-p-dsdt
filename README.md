# Honor Magicbook Pro 14 2025 (FMB-P) DSDT patch

This patch fix issue in DSDT table in BIOS version 1.13 (release date 05/08/2025)
It remove unnecesary bad NFC device record.

## Touchscreen (global version)

With the stock tables the touchscreen (FocalTech `FTSC1000`, `\_SB.PC00.I2C2.TPL1`)
never answers (`i2c_hid_acpi i2c-FTSC1000:00: nothing at this address: -121`).
Its power resource `PTPL` (power enable + reset GPIO) is defined in an SSDT under
`\_SB.PC00.I2C5`, where no touchscreen exists, so nothing powers the controller and
Linux turns the unused power resource off.

`dsdt.global.aml` gives `TPL1` a `_PR0`/`_PR3` pointing at `\_SB.PC00.I2C5.PTPL` and a
`_PS0` that waits for the controller to boot. The chinese version has the same `TPL1`
layout and probably needs the same change, but it is untested and not patched yet.

Once powered, the touchscreen also exposes a bogus keyboard interface
(`FTSC1000:00 2808:5662 UNKNOWN`) that sends phantom key presses (mic LED flicker,
stray hotkeys). Inhibit it with a udev rule, for example
`/etc/udev/rules.d/99-honor-touchscreen-keyboard.rules`:

```
SUBSYSTEM=="input", KERNEL=="input*", ATTR{name}=="FTSC1000:00 2808:5662 UNKNOWN", ATTR{inhibited}="1"
```

Apply it with `sudo udevadm control --reload-rules && sudo udevadm trigger --action=change --subsystem-match=input`, or reboot.

## Different version of BIOS for chinese and global version

There are at least two version of BIOS of this notebook was found. 
One notebook was brought directly in China and one was a global version. 
It's has same DSDT code and bugs but some ACPI tables was moved may be due to translations.

## How to use this

1. Download `dsdt.*.aml` for you version of notebook, rename in to `dsdt.aml`, place it in `/boot`
2. Add `acpi /dsdt.aml` to the end of `/etc/grub.d/40_custom`
3. Execute `update-grub`
4. Reboot

### With mkinitcpio instead of GRUB (Arch, CachyOS, ...)

1. Copy `dsdt.*.aml` for your version to `/etc/initcpio/acpi_override/dsdt.aml`
2. Add `acpi_override` to `HOOKS` in `/etc/mkinitcpio.conf`, before `autodetect`
3. Rebuild the initramfs: `mkinitcpio -P` (`limine-mkinitcpio` on Limine setups)
4. Reboot. `dmesg | grep 'Table Upgrade'` should show the DSDT override

## How to run installer in some distros

In some distros installers does not boot cause bugged DSDT, for example Debian or Ubuntu. It's stuck on root device search cause don't see any USB controllers and devices
So, make this:
* Place `dsdt.aml` to boot directory on installation disk
* During boot in grub (in ubuntu this is a place where you see "Try or nstall Ubuntu") press `e`, add line right before `linux ... bla, bla, bla` and wrote `acpi /dsdt.aml`
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

Decompile DSDT again but without errors
```
iasl -e ssdt{1..22}.dat ssdt{24..26}.dat -d dsdt.dat
```

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
