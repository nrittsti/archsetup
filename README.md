# Arch Linux Setup

## Basesystem

```bash
# Keyboard configuration
loadkeys de-latin1-nodeadkeys
localectl set-x11-keymap de pc105 nodeadkeys
# Partition the disks:
# Create GPT patition table with option g
# Create new parttitons with option n
# For ESP partion, use 100M with type 1 (EFI)
lsblk
fdisk /dev/nvme0n1
# Format partitions
mkfs.vfat -F 32 -n ESP /dev/nvme0n1p1
mkfs.btrfs -f -L LINUX /dev/nvme0n1p2
mkfs.btrfs -f -L HOME /dev/nvme0n1p3
# optional activate swap
swapon /dev/sda2
# mount dirs
mount /dev/nvme0n1p2 /mnt
mkdir -p /mnt/home
mount /dev/nvme0n1p3 /mnt/home
# create subvolumes
cd /mnt
btrfs sub create @
btrfs sub create @pkg
btrfs sub create @log
btrfs sub create @snapshots
cd /mnt/home
btrfs sub create @home
umount /mnt/home
umount /mnt
# mount all
mount -o  noatime,compress=lzo,ssd,commit=120,subvol=@ /dev/nvme0n1p2 /mnt
mount -o  noatime,compress=lzo,ssd,commit=120,subvol=@home /dev/nvme0n1p3 /mnt/home
mkdir -p /mnt/boot/esp
mkdir -p /mnt/var/cache/pacman/pkg
mkdir -p /mnt/var/log
mkdir -p /mnt/btrfs
mkdir -p /mnt/.snapshots
mount -o  noatime,compress=lzo,ssd,commit=120,subvolid=5 /dev/nvme0n1p2 /mnt/btrfs
mount -o  noatime,compress=lzo,ssd,commit=120,subvol=@snapshots /dev/nvme0n1p2 /mnt/.snapshots
mount -o  noatime,compress=lzo,ssd,commit=120,subvol=@pkg /dev/nvme0n1p2 /mnt/var/cache/pacman/pkg
mount -o  noatime,compress=lzo,ssd,commit=120,subvol=@log /dev/nvme0n1p2 /mnt/var/log
mount  /dev/nvme0n1p1 /mnt/boot/esp
# Update Pacman Mirrors
reflector --verbose --country 'Germany' -l 10 -p https --sort rate --save /etc/pacman.d/mirrorlist
# install base system
pacstrap /mnt btrfs-progs base base-devel linux linux-lts linux-firmware nano bash-completion intel-ucode amd-ucode networkmanager man-db man-pages texinfo fish ansible
# /etc/fstab generieren
genfstab -Lp /mnt > /mnt/etc/fstab
# chroot
arch-chroot /mnt
```

## Configuration with Ansible

```bash
ansible-playbook cfgarch.yaml --extra-vars "@cfgarch_vars.yaml"
```