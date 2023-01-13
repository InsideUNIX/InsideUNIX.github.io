This is my metod to partition my storage. You could apply this guide to any distro as always as you had installed `btrfs-progs` & `xfsprogs` and corresponded modules enabled. This guide has been tested in Arch Linux, NixOs and Void Linux.

## Partitions
- 512mb to 1G for /boot/efi
- 75G to 100G /*
- Use the rest of the storage to /home

```
# Use 'cfdisk' tool in order to divide the storage device. I'd recommend
$ cfdisk /dev/sda
```

## Formating the filesystems

> Change "X, Y and Y" with your partition number, use `lsblk` in order to know what are them.

```
# EFI
mkfs.vfat /dev/sdaX

# System partitions
mkfs.xfs -m rmapbt=1 /dev/sdaY
mkfs.btrfs /dev/sdaZ
```

## Creating subvolumes
```
mount /dev/sdaZ /mnt
btrfs su cr /mnt/@root
btrfs su cr /mnt/@var
```

## Mounting top-level partitions
```
# Mount @root subvolume.
mount -o noatime,compress=zstd,space_cache=v2,subvol=@root /dev/sdaY /mnt

# Create systems folders. We will use /var & /home
mkdir -p /mnt/{home,boot,var}

# Mount @var subvolume
mount -o nodatacow,subvol=@var /dev/sdaZ /mnt/var
```

## Mounting home partition
```
# Just execute:
mount /dev/sdaY /mnt/home
```

## Mount EFI partition
```
# Create EFI folder
mkdir /mnt/boot/efi

# Mount the partition
mount /dev/sdaX /mnt/boot/efi
```

## Now What?
Depending on your GNU/Linux distribution, continue with your system installation, configuring the system from `chroot`, installing requiered packages. Read the corresponding wiki in order to continue.

**Other ways to partition**
- [ZFS by HBlanqueto](/content/partitions/hblanqueto-guidances/zfs.md)
