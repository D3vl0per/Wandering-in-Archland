# Specs
- Boot mode: Legacy
- Bootloader: GRUB
- Encyption: LUKS
- Filesystem: Btrfs (+ subvolume)
- Purpose: Server userage (but you can create user and leave openssh install)


# 1. Partitioning
- /dev/sda
  - /dev/sda1 /boot
    - dos partiton table!
    - I left 1 GB space for stuff
  - /dev/sda2 /
    - User leftover space

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
1. `pacstrap /mnt base base-devel linux linux-firmware btrfs-progs zsh grub nmap curl wget intel-ucode nano vim sudo dhcpcd htop grub openssh git zsh-autosuggestions zsh-completions zsh-syntax-highlighting`
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
12. `vim /etc/default/grub`
    ```
    GRUB_CMDLINE_LINUX_DEFAULT="cryptdevice=/dev/sda2:luks root=/dev/mapper/luks rootflags=subvol=@ rd.luks.options=discard rw mem_sleep_default=deep"
    GRUB_PRELOAD_MODULES="part_msdos luks cryptodisk btrfs"
    GRUB_ENABLE_CRYPTODISK=y
    ```
13. `mkinitcpio -p linux`
14. `grub-install /dev/sda`
15. `grub-mkconfig -o /boot/grub/grub.cfg`
16. (optional) `sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"`
17. `systemctl enable dhcpcd`
18. `systemctl enable sshd`
19. `vim /etc/ssh/sshd`
    ```Remove comment from this line
    PermitRootLogin yes
    ```
20. `passwd`
21. Leave chroot
22. `umount -R /mnt`
23. `cryptsetup close luks`
24. `reboot`
25. `Pray to God for boot...`
