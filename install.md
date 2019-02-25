# Install Arch EFI (2019-02-18)
## Prerequesites
* UEFI
* More than 2GB RAM
* Secure boot disabled
## Goals
* Hibernation
* Seperate root and home partitions

## Install Process
### Keyboard layout
> loadkeys sv-latin1

### Test if UEFI
> efivar -l
> should return several rows with text, if not it's booted to legacy

### Test network
> ping -c 4 google.com  
> some packages should be received, if not see [this](https://wiki.archlinux.org/index.php/Network_configuration)

### Update system clock
> timedatectl set-ntp true

### Create partitions (this will delete the partitions)
 1. List disks
    > fdisk -l  
    > note down the disk you want to partition, eg. /dev/sda
 2. Partition the disk
    > * cfdisk /dev/sda  
    > * (if new disk) choose gpt, else verify that disk label is gpt, if not google wipefs
    > * create following partitions  
    ---

    > Size | Type
    > ------------ | -------------
    > 512M | EFI System
    > (1.5*RAM)G | Linux swap
    > 50G | Linux filesystem
    > (rest)G | Linux filesystem
    ---
    > * write changes to the disk and quit
### Format partitions (this will format the partitions)
1. Format boot
    > mkfs.fat -F32 /dev/sda1
2. Format swap
    > mkswap /dev/sda2  
    > swapon /dev/sda2
3. Format /
    > mkfs.ext4 /dev/sda3
4. Format home
    > mkfs.ext4 /dev/sda4
### Download my Arch mirrorlist
1. Update mirrors
    > pacman -Sy
2. Install git
    > pacman -S git
3. Clone my repository
    > git clone https://github.com/MichalKir/myarch.git
4. Backup old mirror list
    > mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.old
5. Move my mirror list to pacman mirrorlist
    > mv myarch/mirrorlist /etc/pacman.d/mirrorlist
6. Update mirrors
    > pacman -Sy
### Mount the system
1. Mount / partition
    > mount /dev/sda3 /mnt
2. Create boot and home folders
    > mkdir /mnt/boot
    > mkdir /mnt/home
3. Mount boot
    > mount /dev/sda1 /mnt/boot
4. Mount home
    > mount /dev/sda4 /mnt/home
### Install the system
> pacstrap /mnt base

### Generate fstab
1. Generate fstab
    > genfstab -U /mnt >> /mnt/etc/fstab
2. Verify fstab
    > cat /mnt/etc/fstab 
    > should return 4 partitions

### Chroot into Arch
> arch-chroot /mnt

### Adjust time
1. Link zone info
    > ln -sf /usr/share/zoneinfo/Europe/Stockholm /etc/localtime
2. Update hardware clock
    > hwclock --systohc

### Set locale
1. Generate locale
    > * nano /etc/locale.gen  
    > * uncomment en_US.UTF-8 UTF-8  
    > * locale-gen  
    > * echo "LANG=en_US.UTF-8" >> /etc/locale.conf
    > * export LANG=en_US.utf8
2. Set vconsole layout
    > echo "KEYMAP=sv-latin1" >> /etc/vconsole.conf

### Set computer name
> echo "computername" >> /etc/hostname

### Set hosts
> nano /etc/hosts
```
127.0.0.1       localhost
::1             localhost
127.0.0.1       computername.domain.local   computername
```

### Install core packages
> pacman -S linux-headers grub efibootmgr networkmanager ufw base-devel vim git xorg-xinit nvidia(if using nvidia gaming nvidia card) xf86-video-fbdev (if using hyper-v)

### Enable network manager
> systemctl enable NetworkManager

### Update mkinitcpio
> nano /etc/mkinitcpio.conf
> append to HOOKS, after fsck "shutdown", and if using fake raid append before filesystems "mdadm_udev"  
> example: HOOKS=(base udev autodetect modconf block mdadm_udev filesystems keyboard fsck shutdown)  
> mkinitcpio -p linux

### Configure Grub2 boot loader
> grub-install --target=x86_64-efi --efi-directory=/boot  
> grub-mkconfig -o /boot/grub/grub.cfg

### Configure users
1. Allow wheel group to execute sudo command
    > visudo  
    > uncomment %wheel ALL=(ALL) AL
2. Create new user
    > useradd -m -G wheel -s /bin/bash cortana  
    > passwd cortana
2. Set root password
    > passwd
### Finish the installation
> exit  
> umount -a  
> reboot

## Post reboot
### Enable multilib
> sudo vi /etc/pacman.conf  
> uncomment [multilib] and Include  
> sudo pacman -Sy
### Configure firewall
> sudo ufw default deny  
> sudo ufw allow from 192.168.1.0/24(your ip-net)  
> sudo ufw enable  
> sudo ufw reload
### Intall virtualbox
> sudo pacman -S virtualbox  
> sudo modprobe vboxdrv  
> sudo /sbin/rcvboxdrv setup

# Install Plasma
### Install packages
1. Install packages
    > sudo pacman -S code pulseaudio tilix dolphin plasma discover packagekit-qt5 sddm noto-fonts ttf-dejavu ttf-liberation firefox chromium neofetch
2. Enable sddm
    > sudo systemctl enable sddm  
    > sudo reboot
3.  Create home folders
    > mkdir ~/Downloads Pictures Coding Documents
### Install my apps
1. p7zip
    > sudo pacman -S p7zip wxgtk2 nasm python yasm kservice  
    > cd ~/.Downloads  
    > git clone https://aur.archlinux.org/p7zip-gui.git  
    > cd p7zip-gui  
    > makepkg -Acs  
    > sudo pacman -U p7zip*.tar.xz
2. PowerShell
    > sudo pacman -S icu openssl-1.0 dotnet-sdk  
    > git clone https://aur.archlinux.org/powershell.git  
    > cd powershell  
    > makepkg -Acs  
    > sudo pacman -U powershell*.tar.xz  
3. Install image applications
    > sudo pacman -S nomacs krita spectacle

### Configure Plasma
> see my_plasma.md
