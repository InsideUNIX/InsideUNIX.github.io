## Logging in
User is `anon` and password is `voidlinux`.
The root user is `root` and has the same password.
This document implies you're using the `root` user, so no `sudo` is used until the main user is created and used.

## Setting keyboard layout
```sh
loadkeys es 

# This depends in your layout
```

## Connecting to the internet

```If you use Ethernet, do not make this```

```sh
cp /etc/wpa_supplicant/wpa_supplicant.conf /etc/wpa_supplicant/wpa_supplicant-<interface>.conf
wpa_passphrase <ssid> <passphrase> | tee -a /etc/wpa_supplicant/wpa_supplicant-<interface>.conf
sv restart dhcpcd
ip link set up <interface>
```
## Filesystems

**Formatting disk**

Run:
```sh
cfdisk /dev/sda
```

**Creating the filesystems**

I have Windows in my SSD so, change the sdaX number depending on your partitions numbers

```sh
mkfs.vfat /dev/sdaX
mkfs.xfs /dev/sdaX
mkfs.btrfs /dev/sdaX
```

**Mounting the `btrfs`partition and creating subvolumes**
```sh
mount /dev/sdaX /mnt
btrfs su cr /mnt/@root
btrfs su cr /mnt/@var
btrfs su cr /mnt/@logs
btrfs su cr /mnt/@tmp
btrfs su cr /mmt/@home
```

**Mounting top-level partitions**
```sh
mount -o noatime,compress=zstd,space_cache=v2,subvol=@root /dev/sdaX /mnt
mkdir -p /mnt/{home,boot,.snapshots,var,tmp,var/logs}
mount -o noatime,compress=zstd,space_cache=v2,subvol=@opt /dev/sdaX /homme
mount -o noatime,compress=zstd,space_cache=v2,subvol=@tmp /dev/sdaX /tmp
mount -o nodatacow,subvol=@var /dev/sdaX /mnt/var
mount -o nodatacow,subvol=@var /dev/sdaX /mnt/var/logs
```
**Mount & `/boot/`**
```sh
mount /dev/sdaX /mnt/boot
cp /etc/resolv.conf /mnt/etc/
```

## Installing the base system

**`glibc`**
```sh
xbps-install -Sy -R https://alpha.de.repo.voidlinux.org/current -r /mnt base-system btrfs-progs xfsprogs zstd grub-x86_64-efi nano
```
**`musl`**
```sh
export XBPS_ARCH=x86_64-musl
xbps-install -Sy -R https://alpha.de.repo.voidlinux.org/current/musl -r /mnt base-system btrfs-progs xfs-progs zstd grub-x86_64-efi nano
```

## Now, boot Arch Linux ISO

**fstab**

Before enter into chroot, make sure you have mounted correctly your system, then use the tool `genfstab`

```
genfstab -p /mnt >> /mnt/etc/fstab
cat /mnt/etc/fstab
```

**Chroot**
```
arch-chroot /mnt
```

**Hostname & other timezone stuff**

```
echo myhost > /etc/hostname
ln -sf /usr/share/zoneinfo/<timezone> /etc/localtime
export HARDWARECLOCK=localtime
```

Then edit the file `/etc/rc.conf` to contain the following:
```ini
HOSTNAME="<hostname>"

# Set RTC to UTC or localtime.
HARDWARECLOCK="UTC"

# Set timezone, availables timezones at /usr/share/zoneinfo.
TIMEZONE="America/Mexico_City"

# Keymap to load, see loadkeys(8).
KEYMAP="es"

# Console font to load, see setfont(8).
#FONT="lat9w-16"

# Console map to load, see setfont(8).
#FONT_MAP=

# Font unimap to load, see setfont(8).
#FONT_UNIMAP=

# Kernel modules to load, delimited by blanks.
#MODULES=""
```

**If you installed glibc, run the next command**
```
xbps-reconfigure -f glibc-locales
```
**Installing grub**

Note, if you want install `snapper` & `btrfs-grub` If you plan in the future to use snapshots in btrfs

```
grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id="Debian Boot Manager"
update-grub
```

Then run the following commands:
```
xbps-install -S void-repo-nonfree
xbps-install -Su
xbps-reconfigure -fa
passwd
exit
exit
umount -R /mnt
reboot
```

## Post-installation

**Creating the main user and adding them to sudoers**
Log in as root
```sh
useradd <username>
usermod -m -G wheel <username>
passwd <username>
visudo
```
After running `visudo`, uncomment the line that contains `%wheel`. Log out and log in with the newly created user.


**Connecting to the internet**
Run the proper `wpa_supplicant` steps to set Wi-Fi or stuff you need. Then, register and start the needed services:
```sh
sudo ln -s /etc/sv/dhcpcd /var/service/
sudo ln -s /etc/sv/wpa_supplicant /var/service/
```
By symlinking the services, runit will pick them up in the next five seconds and start them!

## Sources
- https://voidlinux.org/packages/ <- Search the package you need
- https://docs.voidlinux.org/config/network/networkmanager.html <- NetworkManager
- https://docs.voidlinux.org/xbps/repositories/index.html <- Repositories
- https://docs.voidlinux.org/config/date-time.html <- Date & Time
- https://wiki.voidlinux.org/Xorg <- Xorg
- https://docs.voidlinux.org/installation/guides/chroot.html <- Official Void Linux Guide
- https://gist.github.com/shizonic/be831a3ffc9a4cea820fdeaa1a955e73 <- Credits to shizonic who I based my config in