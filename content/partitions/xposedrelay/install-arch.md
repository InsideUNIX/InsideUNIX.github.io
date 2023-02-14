# Install Arch with LVM, LUKS and systemd-boot (UEFI Only)
Today, I'll show you how to install **Arch Linux** with [systemd-boot](https://wiki.archlinux.org/title/systemd-boot) (a boot loader), [LUKS](https://en.wikipedia.org/wiki/Linux_Unified_Key_Setup) (Linux Unified Key Setup) and [LVM](https://es.wikipedia.org/wiki/Logical_Volume_Manager_(Linux)) (Logical Volume Manager)

## Before we continue, you will need:
- A *USB* with *4GB* or more of space.
- *WiFi* or *Ethernet* connection (*Ethernet* is prefered).
- A *SSD* or *HDD* where you can install **Arch Linux**.

With all of this, we can start!

## Preparation
Before start, you have to disable *Secure boot* so we can boot from our USB.

After this, we can proceed.
### Font
If the font is small or unreadable, you can change it:
```sh
setfont sun12x22
```
### System clock
To update the system clock, we have to do this:
```sh
timedatectl set-ntp true
```
> You can update your mirrors (if you need it) by modifying `/etc/pacman.d/mirrorlist`. This file will be copied yo your systemf once the installation is complete.

## Installation: part 1

### Partition
  - *512MB* or more for **ESP**
  - *75GB* to *100GB* for **/***
  - The *rest* of the drive use it for **/home**

### Formating
> Use `lsblk` in order to know your drives, I'll use a generic letter for each partition: "X, Y and Z".

- EFI
    ```sh
    mkfs.vfat /dev/sdaX
    ```

### Encryption

First, we have to modprobe for `dm-crypt`:
```sh
modprobe dm-crypt
```
Second, encrypt the disk:
```sh
cryptsetup luksFormat /dev/sdaY
```
Third, open the disk with the password set previously:
```sh
cryptsetup open --type luks /dev/sdaY cryptlvm
```
Fourth, we check if the LVM disk is there:
```sh
ls /dev/mapper/cryptlvm
```
Fifth, create a physical volume:
```sh
pvcreate /dev/mapper/cryptlvm
```
Sixth, create a volume group:
```sh
vgcreate volume /dev/mapper/cryptlvm
```
Seventh, create the logical partitions:
```sh
lvcreate -L75G volume -n root
lvcreate -l 100%FREE volume -n home
```
Eighth, format with a file system of your preference on logical partitions:
> In this example, I'll some of the comands from the 'partition' guide.
```sh
mkfs.btrfs /dev/volume/root
mkfs.xfs -m rmapbt=1 /dev/volume/home
```
Nineth, mount the following volumes and file systems:
  ```sh
## root
  mount /dev/volume/root /mnt
  mkdir /mnt/home
  mkdir /mnt/boot
## home  
  mount /dev/volume/home /mnt/home
## ESP
  mount /dev/sdaX /mnt/boot
  ```
## Installation: part 2
Install the essential packages:
> This time, I'll use `nano`, you can use `vim` if you want.
```sh
pacstrap /mnt base base-devel linux linux-firmware lvm2 nano vim xfsprogs exfatprogs btrfs-progs zstd
```
Generate the `fstab`:
```sh
genfstab -U /mnt >> /mnt/etc/fstab
```
Now, we `chroot`into system:
```sh
arch-chroot /mnt
```
Later, set the time locale:
> For this, I'll use my time locale.
```sh
ln -sf /usr/share/zoneinfo/America/Argentina/Mendoza /etc/localtime
```
Then, set clock:
```sh
hwclock --systohc
```
After that, uncomment the localization you need in `/etc/locale.gen` and run the following command:
> I'll use `es.AR.UTF-8` in my case.
![image](https://raw.githubusercontent.com/xposedrelay/kitty.conf/main/image.png)
```sh
locale-gen
```
Seventh, create locale config file:
```sh
locale > /etc/locale.conf
```
Eighth, set the language variable in previous file:
```sh
LANG=es.AR.UTF-8
```
Nineth, add a hostname in the following file:
```sh
/etc/hostname
```
Now, update `/etc/host` with your hostname:
> `navi` is my hostname, replace `navi` with your hostname:
```sh
127.0.1.1 `navi`.localdomain `navi`
```

Eleventh, we have to edit `/etc/mkinitcpio.conf`. Without this, you can´t boot into your system:
```sh
HOOKS=(base systemd btrfs keyboard autodetect sd-vconsole modconf block sd-encrypt lvm2 filesystems fsck)
```
Twelfth, generate the initramfs:
```sh
mkinitcpio -p linux
```
Thirteenth, install systemd-boot:
```sh
bootctl --path=/boot/ install
```
Fourteenth, we need to configure the bootloader. Edit `/boot/loader/loader.conf`:
> `default`: lets you choose your default entrie conf.
> `timeout`: as it says, timeout before selecting your default entrie.
> `editor`: makes your config unchangeable at boot. 
```sh
default arch
timeout 3
#console-mode keep
editor 0
```
Next, make an entry for your system in `/boot/loader/entries/arch.conf`
> This time, I'll choose `intel-ucode.img` because I use an Intel cpu.
> To know your `UUID`, use the tool `blkid`. (Example: `blkid /dev/sdaY`)
```sh
title Arch Linux
linux /vmlinuz-linux
initrd /initramfs-linux.img
initrd /intel-ucode.img
options rd.luks.name={UUID}=cryptlvm root=/dev/volume/root rd.luks.options=discard,timeout=30 quiet rw
```

## Complete
Before we finish, you'll like to install a few things:
```sh
pacman -Syu network-manager sudo
```
Later, set a password for your root user with:
```sh
passwd
```
Then, we'll add our user:
```sh
useradd -m -g users -G wheel -s /bin/bash <username>
passwd <username>
echo '<username> ALL=(ALL) ALL' > /etc/sudoers.d/<username>
```
Now, we can finish everything with:
```sh
## exit chroot
exit
## unmount everything
umount -R /mnt
## reboot
reboot
```
