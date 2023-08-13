# Arch Linux install guide by linkfrg for UEFI systems

## First stage: connecting to the internet
#### NOTE: If you have wired connection you can skip this stage
### Connecting to Wi-FI
```
iwctl
device list

```
#### This command will show all your wireless device names
```
station device scan    # where device is your wireless device name
station device get-networks    # this will show all wifi networks
station device connect SSID    # There SSID is wifi network name, NOTE: If it have spaces put name in "", like "My home network"
```
### Now check connection
```
ping google.com
```

## Second stage: Disk partitioning

### NOTE: replace /dev/sdX by your disk
#### Scheme WITH swap partition
| Partition | Mount point          | Size            |
| --------- | -------------------- | --------------- |
| /dev/sdX1 | /boot/efi(in chroot) | at least 100 mb |
| /dev/sdX2 | [SWAP]               | ram size        |
| /dev/sdX3 | /mnt                 | Remainder       |


#### Scheme WITHOUT swap partition(You can use zram, after installation see #zram)
| Partition | Mount point          | Size            |
| --------- | -------------------- | --------------- |
| /dev/sdX1 | /boot/efi(in chroot) | at least 100 mb |
| /dev/sdX3 | /mnt                 | Remainder       |

```
cfdisk /dev/sdX
```

### Format the partitions
### Root partition
#### ext4
```
mkfs.ext4 /dev/sdX3
```
#### btrfs
```
mkfs.btrfs -f /dev/sdX3
```
### EFI partition
#### WARNING: IF YOU ALREADY HAVE EFI PARTITION, FORMATING IT WILL DESTROY BOOTLOADERS OTHER SYSTEM, IF YOU DOING DUALBOOT DONT FORMAT IT
```
mkfs.fat -F 32 /dev/sdX1
```
### Swap(If you created it)
```
mkswap /dev/sdX2
swapon /dev/sdX2
```

## Mount the file systems
### ext4
```
mount /dev/sdX3 /mnt
```
### btrfs
```
mount -o defaults,noatime,compress=zstd /dev/sdX3 /mnt
```

## Installation
```
pacstrap -K /mnt base base-devel linux linux-firmware linux-headers intel-ucode(for amd: amd-ucode) btrfs-progs(for btrfs) man nano network-manager 
```

## Configure the system

### Fstab
```
genfstab -U /mnt >> /mnt/etc/fstab
```

### Chroot
```
arch-chroot /mnt
```

### Time zone

#### Tip: you can press double TAB after 'zoneinfo' to list all regions, same with cities
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
```
hwclock --systohc
```
### Localization
```
nano /etc/locale.gen
```
#### uncomment ```en_US.UTF-8 UTF-8``` and other needed locales, for example ```ru_RU.UTF-8 UTF-8```

```
locale-gen
```
#### Set the LANG variable to your locale
```
nano /etc/locale.conf
LANG=en_US.UTF-8
```
#### if you use russian or any other cyrillic locale set font cyr-sun16 in tty
```
nano /etc/vconsole.conf
FONT=cyr-sun16
```

#### Network configuration
```
nano /etc/hostname
archlinux
```
```
systemctl enable NetworkManager
```

### Users
#### set password for root
```
passwd
```
#### Create user and set password
```
useradd -m -G wheel -s /bin/bash username
passwd username
```
#### set rights for wheel group
### Uncoment %wheel ALL=(ALL:ALL) ALL
```
EDITOR=nano visudo
```

### Install bootloader
#### mount efi partition
```
mkdir /boot/efi
mount /dev/sdX1 /boot/efi
```
```
pacman -S grub efibootmgr
```
```
grub-install
```

```
grub-mkconfig -o /boot/grub/grub.cfg
```

```
exit
umount -R /mnt
reboot
```