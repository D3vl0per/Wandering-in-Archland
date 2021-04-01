# In progress.....
# Specs
- Boot mode: UEFI
- Bootloader: systemd boot
- Encyption: LUKS
- Filesystem: Btrfs (+ subvolume)
- Display manager: gdm
- Desktop: i3wm
- Purpose: Home useage

# 1. Partitioning
1. `gdisk /dev/sda`
2. `o` (Create a new empty GUID partition table (GPT))
3. `n` (Add a new partition)
  1. Partition number `1`
  2. First sector (default)
  3. Last sector `+512M`
  4. Hex code `EF00`
4. n (Add a new partition)
  1. Partition number `2`
  2. First sector (default)
  3. Last sector (press Enter to use remaining disk)
  4. Hex code `8300`
5. Save: `w`

# 2. Cryptsetup
1. `cryptsetup -c aes-xts-plain64 -y --use-random luksFormat /dev/sda2`
2. `cryptsetup open /dev/sda2 luks`

# 3. Filesystems things
1. `mkfs.vfat -F32 /dev/sda1`
2. `mkfs.btrfs /dev/mapper/luks`
3. `mount /dev/mapper/luks /mnt`
4. `btrfs subvolume create /mnt/@`
5. `btrfs subvolume create /mnt/@home`
6. `btrfs subvolume create /mnt/@var`
7. `umount /mnt`
8. If you use ssd:  
  `mount -o subvol=@,ssd,compress=lzo,noatime,nodiratime /dev/mapper/luks /mnt` 
  `mkdir /mnt/{boot,home,var}`  
  `mount -o subvol=@home,ssd,compress=lzo,noatime,nodiratime /dev/mapper/luks /mnt/home`  
  `mount -o subvol=@var,ssd,compress=lzo,noatime,nodiratime /dev/mapper/luks /mnt/var`  
   Or if not:  
  `mount -o subvol=@,compress=lzo,noatime,nodiratime /dev/mapper/luks /mnt`  
  `mkdir /mnt/{boot,home,var}`
  `mount -o subvol=@home,compress=lzo,noatime,nodiratime /dev/mapper/luks /mnt/home`   
  `mount -o subvol=@var,compress=lzo,noatime,nodiratime /dev/mapper/luks /mnt/var`   
9. `mount /dev/sda1 /mnt/boot`

# 4. Install base system
1. `pacstrap /mnt base base-devel linux linux-firmware btrfs-progs nmap curl wget intel-ucode nano vim sudo dhcpcd htop git zsh zsh-autosuggestions zsh-completions zsh-syntax-highlighting iwd gdm firefox vlc i3-wm i3status i3lock dmenu`
2. `genfstab -U /mnt >> /mnt/etc/fstab`
3. `arch-chroot /mnt /bin/zsh`
4. `echo <your-hostname> > /etc/hostname`
5. `ln -sf /usr/share/zoneinfo/<country>/<city-or-not> /etc/localtime`
6. `hwclock --systohc`
7. `echo LANG=en_US.UTF-8 > /etc/locale.conf`
8. `vim /etc/locale.gen` Remove comment from chosen line
9. `locale-gen`
10. `vim /etc/hosts`
    ```
    127.0.0.1   localhost  
    ::1     localhost  
    127.0.1.1   <your-hostname>.localdomain   <your-hostname>  
    ```
11. `vim /etc/mkinitcpio.conf`  
    ``` # Add encrypt and btrfs hooks before filesystems
    HOOKS=(base udev autodetect modconf block encrypt btrfs filesystems keyboard fsck) 
    ```
12. `mkinitcpio -p linux`
13. `bootctl --path=/boot install`
14. `vim /boot/loader/entries/arch.conf`
```
title Arch Linux
linux /vmlinuz-linux
initrd /intel-ucode.img
initrd /initramfs-linux.img
options cryptdevice=UUID=<blkid-sda1/2-whatever>:luks:allow-discards root=/dev/mapper/luks rootflags=subvol=@ rd.luks.options=discard rw mem_sleep_default=deep
```
15. `vim /boot/loader/loader.conf`
```
default arch
```
16. `systemctl enable dhcpcd`
17. `systemctl enable iwd`
18. `systemctl enable gdm`


