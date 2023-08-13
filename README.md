# Arch Linux install guide by linkfrg for UEFI systems, zram, btrfs

# Connecting to the internet
### NOTE: If you have wired connection you can skip this stage
## Connecting to Wi-FI
```
iwctl
device list
```
```
station device scan            # where device is your wireless device name
station device get-networks
station device connect SSID    # There SSID is wifi network name, NOTE: If it have spaces put name in "", like "My home network"
```
### Now check connection
```
ping google.com
```

# Disk partitioning

## NOTE: replace /dev/sda by your disk
### Scheme 
| Partition | Mount point   | Size            | Type             |
| --------- | ------------- | --------------- | ---------------- |
| /dev/sda1 | /mnt/boot/efi | at least 100 mb | EFI system       |
| /dev/sda2 | /mnt          | Remainder       | Linux filesystem |

```
cfdisk /dev/sda
```

## Format the partitions
```
mkfs.btrfs -f /dev/sda2
```
### WARNING: IF YOU ALREADY HAVE EFI PARTITION, FORMATING IT WILL DESTROY OTHER SYSTEM BOOTLOADERS, IF YOU DOING DUALBOOT DONT FORMAT IT
```
mkfs.fat -F 32 /dev/sda1
```

## Mount the file systems
```
mount /dev/sda2 /mnt
```
```
btrfs su cr /mnt/@
btrfs su cr /mnt/@home
```

```
umount /mnt
mount -o defaults,noatime,compress=zstd,subvol=@ /dev/sda3 /mnt
```

```
mkdir /mnt/home
mount -o defaults,noatime,compress=zstd,subvol=@home /dev/sda3 /mnt/home
```

```
mkdir -p /mnt/boot/efi
mount /dev/sda1 /mnt/boot/efi
```

# Installation
```
pacstrap -K /mnt base base-devel linux linux-firmware linux-headers intel-ucode(for amd: amd-ucode) btrfs-progs man nano networkmanager grub efibootmgr
```

# Configure the system

## Fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

## Chroot
```
arch-chroot /mnt
```

## Time zone

### Tip: you can press double TAB after 'zoneinfo' to list all regions, same with cities
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
```
hwclock --systohc
```
## Localization
```
nano /etc/locale.gen
```
### uncomment ```en_US.UTF-8 UTF-8``` and other needed locales, for example ```ru_RU.UTF-8 UTF-8```

```
locale-gen
```
### Set the LANG variable to your locale
```
nano /etc/locale.conf
```
```
LANG=en_US.UTF-8
```
### if you use russian or any other cyrillic locale set font cyr-sun16 in tty
```
nano /etc/vconsole.conf
```
```
FONT=cyr-sun16
```

## Network configuration
```
nano /etc/hostname
```
```
archlinux
```
```
systemctl enable NetworkManager
```

## Users
### set password for root
```
passwd
```
### Create user and set password
```
useradd -m -G wheel -s /bin/bash username
passwd username
```
### Uncoment %wheel ALL=(ALL:ALL) ALL
```
EDITOR=nano visudo
```

## Install bootloader
```
grub-install
```
### Make configuration
```
grub-mkconfig -o /boot/grub/grub.cfg
```

# We complete the installation, unmount the partitions, reboot into a freshly installed system
```
exit
umount -R /mnt
reboot
```

# After installation
## enable zram
### Firstly, disable zswap
#### Add ```zswap.enabled=0``` in ```/etc/default/grub```
```
sudo nano /etc/default/grub
```

```
GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet zswap.enabled=0"
```

```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
### zram-generator
```
sudo pacman -S zram-generator
```

```
sudo nano /etc/systemd/zram-generator.conf
```

```
[zram0]
zram-size = ram
compression-algorithm = zstd
swap-priority = 100
fs-type = swap
```

```
reboot
```