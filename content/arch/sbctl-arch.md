# Enable *Secure boot* for Arch Linux (UEFI Only)
This make our machine more secure, for our privacy.

## Preparation
- Know our motherboard, this will be helpful for entering *setup mode*
- Internet connection.
- Arch Linux installed.

## Enrollment
First of all, we have to install `sbctl`:
```sh
sudo pacman -S sbctl
```
Second, your motherboard needs to be in *setup mode* at this point, but with *secure boot* disabled.

Third, create the keys:
```sh
sudo sbctl create-keys
```
Fourth, enroll the keys:
> `-m`, `--microsoft` Enroll UEFI vendor certificates from Microsoft into the signature database. (From `man sbctl`)

> Note that some devices have hardware firmware that is signed and
validated when Secure Boot is enabled. Failing to validate this firmware
could brick devices. It's recommended to enroll your own keys with
Microsoft certificates. (From `man sbctl`)
```sh
sudo sbctl enroll-keys -m
```
Fifth, sign your *vmlinuz*:
```sh
sudo sign -s path/to/vmlinuz
```
Sixth, enroll the `.efi` of your boot loader:
```sh
sudo sign -s path/to/efi
```

## Finish
Now you can reboot, enable *secure boot* in your firmware options and enjoy.