# Installation (UEFI)
1. Select keyboard layout 
   > loadkeys sv-latin1
2. Update system clock
    > timedatectl set-ntp true
3. Partition the disks
    1. List avaliable disks
       > fdisk -l
    2. Note the disk you want to partition (eg. /dev/sda)
    3. Partition the disk
       > - cfdisk /dev/sda
       > - choose gpt
       > - New -> 512MB -> Type -> Efi System
       > - New -> RAM*2 -> Type -> Linux Swap
       > - New -> 80GB  -> Type -> Linux filesystem
       > - New -> Rest  -> Type -> Linux filesystem
       > - Write -> yes -> quit
    4. Format the disk
        > - mkfs.fat -F32 /dev/sda1
        > - mkswap /dev/sda2
        > - swapon /dev/sda2
        > - mkfs.ext4 /dev/sda3
        > - mkfs.ext4 /dev/sda4
4. Modify mirror list
    > - pacman -Sy
    > - pacman -S git
    > - git clone https://github.com/MichalKir/myarch.git
    > - mv /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.old
    > - mv myarch/mirrorlist /etc/pacman.d/mirrorlist
    > - pacman -Sy
5. Mount the system
    > - mount /dev/sda3 /mnt
    > - mkdir /mnt/boot
    > - mkdir /mnt/home
    > - mount /dev/sda1 /mnt/boot
    > - mount /dev/sda4 /mnt/home
6. Install the system
    > - pacstrap /mnt base
7. Generate Fstab
    > - genfstab -U /mnt >> /mnt/etc/fstab
    > - cat /mnt/etc/fstab (4 partitions should show)
8. Chroot("logon") into the Arch
    > - arch-chroot /mnt
9. Adjust time
    > - ln -sf /usr/share/zoneinfo/Europe/Stockholm /etc/localtime
    > - hwclock --systohc
10. Set localization
    > - nano /etc/locale.gen -> uncomment LANG=en_US.UTF-8 UTF-8
    > - locale-gen
    > - nano /etc/locale.conf -> LANG=en_US.UTF-8
    > - nano /etc/vconsole.conf -> KEYMAP=sv-latin1
11. Configure network
    > - nano /etc/hostname -> computername
    > - nano /etc/hosts <br>
    > 127.0.0.1   localhost <br>
    > ::1 localhost <br>
    > 127.0.0.1 computername.domainlocal computername
    > - pacman -S networkmanager
    > - systemctl enable NetworkManager
    > - pacman -S ufw
    > - ufw default deny
    > - ufw allow from 192.168.1.0/24
    > - ufw enable
12. Install additional packages and configurations
    > - groupadd approvedsudoers    
    > - pacman -S base-devel (aur)
    > - pacman -S vim
    > - pacman -S tilix
    > - pacman -S git
    > - pacman -S dolphin
    > - pacman -S fish
    > - pacman -S pulseaudio
    > - visudo -> below %wheel ALL=(ALL) ALL add: <br>
    > %approvedsudoers ALL=(ALL) ALL
    > - useradd -m -G approvedsudoers -s /usr/bin/fish cortana
    > - passwd cortana
13. Configure mkinitcpio
    > - vi /etc/mkinitcpio.conf ->
    > append into HOOKS (after fsck) shutdown + if fake raid add (before filesystems) mdadm_udev
    > - mkinitcpio -p linux
14. Install GRUB
    > - pacman -S grub efibootmgr
    > - grub-install --target=x86_64-efi --efi-directory=/boot
    > - grub-mkconfig -o /boot/grub/grub.cfg
    > - (if VirtualBox)
    > - mkdir /boot/EFI/boot
    > - cp /boot/EFI/arch/grubx64.efi /boot/EFI/boot/bootx64.efi
15. Set root password
    > - passwd
# System (if any widget is missing, install)
1. Install plasma and sddm
    > - sudo pacman -S plasma discover packagekit-qt5
    > - sudo pacman -S sddm
    > - sudo systemctl enable sddm
    > - sudo reboot
2. Language settings (System Settings)
    > #### Regional Settings
    > - In language add American English
    > - Formats change to Sverige
    > #### Input Devices
    > - Layouts -> check configure layouts -> add Swedish -> Layout = Swedish -> Ok -> remove English
3. UI settings (System Settings)
    > #### Startup and Shutdown
    > - Login Screen -> Install Chili login theme
    > #### Workspace Theme
    > - User Breeze Dark on Look And Feel, Desktop Theme, and Cursor Theme, install Arch Space for Splash screen
    > #### Colors
    > - Breeze High Contrast
    > #### Fonts
    > - pacman -S noto-fonts ttf-dejavu ttf-liberation
    > - General: Noto Sans 10, Fixed: Noto Sans 9, Small: Noto Sans 8, Rest: Noto Sans 10
    > #### Icons
    > - git clone https://aur.archlinux.org/packages/papirus-icon-theme-git/
    > - cd papirus-icon-theme
    > - makepkg -Acs
    > - sudo pacman -U papirus*.tar.xz
    > - reboot
4. Install core applications
    > - sudo pacman -S firefox
    > - Start Firefox -> type about:config -> change gfx.font_rendering.fontconfig.max_generic_substitutions to 127
    > - sudo pacman -S p7zip wxgtk2 nasm python yasm kservice
    > - git clone https://aur.archlinux.org/p7zip-gui.git
    > - cd p7zip-gui
    > - makepkg -Acs
    > - sudo pacman -U p7zip*.tar.xz
    > - sudo pacman -S icu openssl-1.0 dotnet-sdk
    > - git clone https://aur.archlinux.org/powershell.git
    > - cd powershell
    > - makepkg -Acs
    > - sudo pacman -U powershell*.tar.xz
    > - sudo pacman -S electron libsecret libxkbfile gulp nodejs-lts-carbon npm python python2 yarn
    > - git clone https://aur.archlinux.org/code-git.git
    > - cd code-git
    > - makepkg -Acs
    > - sudo pacman -U code*.tar.xz
    > - sudo pacman -S linux-headers
    > - sudo modprobe vboxdrv
    > - sudo /sbin/rcvboxdrv setup
    > - sudo pacman -S virtualbox
    > - sudo pacman -S nomacs
    > - sudo pacman -S krita
5. Start menu
    > - Replace current task manager with icons only task manager
    > - sudo pacman -S make cmake extra-cmake-modules
    > - Download separator from kde and cd to the folder
    > - sudo sh ./build.sh
    > - Add seperator between start icon and task manager
    > - Right click clock -> Configure Digital Clock -> Check Show Date
    > - Right click menu icon -> Application Launcher Settings -> Select Arch Icon
    > - Add Minimize Windows next to System Tray
    > - Add new vertical panel and move it to secondary screen alligin to the right
    > - Add icons only task manager and add following widgets: thermal monitor, turn off, clock
6. Install other applications
    > - sudo pacman -S spectacle
    > - sudo pacman -S kdeconnect
7. Install my root certificates
    > - sudo trust anchor --store ca.cer
    > - sudo trust anchor --store intermediate.cer
