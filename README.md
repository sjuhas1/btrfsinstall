# btrfsinstall
```
sgdisk --zap-all /dev/nvme0n1
sgdisk --clear \
  --new=2:0:+1G --typecode=2:ef00 --change-name=2:EFI \
  --new=1:0:+1G --typecode=1:8300 --change-name=1:boot \
  --new=3:0:+16G --typecode=3:8200 --change-name=3:swap \
  --new=4:0:+0 --typecode=4:8300 --change-name=4:system \
  /dev/nvme0n1

sgdisk --print /dev/nvme0n1

cryptsetup luksFormat --type luks1 /dev/nvme0n1p1
cryptsetup luksFormat /dev/nvme0n1p3
cryptsetup luksFormat /dev/nvme0n1p4

cryptsetup open /dev/nvme0n1p1 boot
cryptsetup open /dev/nvme0n1p3 swap
cryptsetup open /dev/nvme0n1p4 system


mkfs.fat -F32 -n EFI /dev/nvme0n1p2


mkswap -L swap /dev/mapper/swap
swapon -L swap

mkfs.btrfs --force --label boot /dev/mapper/boot
mkfs.btrfs --force --label system /dev/mapper/system
mount -t btrfs LABEL=system /mnt
btrfs subvolume create /mnt/root
btrfs subvolume create /mnt/home
btrfs subvolume create /mnt/snapshots
umount -R /mnt
mount -t btrfs LABEL=boot /mnt
btrfs subvolume create /mnt/boot
umount -R /mnt


mount -t btrfs -o subvol=root,defaults,x-mount.mkdir,compress=lzo,ssd,noatime /dev/mapper/system /mnt
mount -t btrfs -o subvol=home,defaults,x-mount.mkdir,compress=lzo,ssd,noatime /dev/mapper/system /mnt/home
mount -t btrfs -o subvol=boot,defaults,x-mount.mkdir,compress=lzo,ssd,noatime /dev/mapper/boot /mnt/boot
mount -o defaults,x-mount.mkdir LABEL=EFI /mnt/boot/efi

sed -i 's/main restricted/main restricted universe multiverse/g' /etc/apt/sources.list
apt-get update && apt-get install vim debootstrap arch-install-scripts

debootstrap --arch amd64 eoan /mnt http://hu.archive.ubuntu.com/ubuntu

dd if=/dev/random of=/boot/secretkey bs=1 count=4096
chmod 0400 /boot/secretkey 

arch-chroot /mnt
passwd
apt-get update && apt-get install locales console-setup tasksel cryptsetup
dpkg-reconfigure locales
dpkg-reconfigure tzdata
dpkg-reconfigure console-setup 

cryptsetup luksAddKey /dev/nvme0n1p3 /boot/secretkey
cryptsetup luksAddKey /dev/nvme0n1p4 /boot/secretkey

hostnamectl set-hostname nb-sjuhasz

apt-get install linux-image-generic grub-efi-amd64 efibootmgr cryptsetup btrfs-progs sudo vim bash
adduser [your username]
usermod -aG sudo [your username]


echo -e "CRYPTSETUP=y\n" >> /etc/cryptsetup-initramfs/conf-hook
echo -e "RESUME=none\n" >> /etc/initramfs-tools/conf.d/noresume.conf

update-initramfs -k all -u

GRUB_ENABLE_CRYPTODISK=y
GRUB_CMDLINE_LINUX_DEFAULT="net.ifnames=0 biosdevname=0 noresume"
GRUB_CMDLINE_LINUX="quiet"

update-initramfs -k all -u

```
