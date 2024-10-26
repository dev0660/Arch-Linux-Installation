# Arch Linux Installation Documentation

## Step 1: Download ISO and Verify Signature
Download the Arch Linux ISO from the [official Arch Linux download page](https://archlinux.org/download/) and verify the checksum.

## Step 2: Load ISO into VM Software to Create Virtual Machine
Using VirtualBox, create a new VM and allocate at least 20GB of disk space and 2GB of RAM. Set the type to **Linux** and version to **Arch Linux (64-bit)**.

**##Step 3: Set up Internet Connection**
1. Use `ip link` to see if there are any available network interfaces. I had enp0s3.
2. Use `ip link set enp0s3 up` to bring the interface up
3. 3. Use `ping archlinux.org` to verify connection
**Issue:** When trying to download firefox for some reason my VM wasnt connected to the internet anymore so I had to redo these steps. 

## Step 4: Partition the Disk
First, I listed the disks using `fdisk -l` to identify the primary disk. I then used `fdisk /dev/sda` to create at least two partitions: an EFI partition (for UEFI) and a root partition (/).  

**##Step 5: Format Partitions**
After partitioning, I formatted the EFI partition using `mkfs.fat -F 32 /dev/sda1` for UEFI, the root partition with `mkfs.ext4 /dev/sda2`. 

**Step 6: Mount Partitions**
I mounted the root partition using `mount /dev/sda2 /mnt`, then created a directory for the EFI partition with `mkdir /mnt/boot` and mounted it using `mount /dev/sda1 /mnt/boot`. 

**Step 7: Install the Base System**
To optimize download speed, I selected the fastest mirrors using `reflector --verbose --latest 5 --sort rate --save /etc/pacman.d/mirrorlist`. 
After that, I installed the base system with `pacstrap /mnt base linux linux-firmware`, which installs the necessary components of Arch Linux. 

**Step 8: Generate fstab**
I generated the fstab file by running `genfstab -U /mnt >> /mnt/etc/fstab` to ensure the partitions would be mounted automatically during future boots. 

**Step 9: Chroot into the New System** 
I entered the new system using `arch-chroot /mnt` to begin configuring it. 

**Step 10: Configure the System**
I set the time zone using `ln -sf /usr/share/zoneinfo/Region/City /etc/` and synchronized the hardware clock with `hwclock --systohc`. 
For localization, I edited /etc/locale.gen using `nano /etc/locale.gen` and uncommented en_US.UTF-8 UTF-8, then generated the locale using `locale-gen`. 
I set the system's language with `echo "LANG=en_US.UTF-8" > /etc/locale.conf`, configured the hostname with `echo "yourhostname" > /etc/hostname`, and set the root password using passwd.
**Issue:** I encountered the issue where the nano command was not found during the localization step. This was resolved by installing nano using `pacman -S nano`.

**Step 11: Install Bootloader**
I installed the bootloader. For UEFI, I installed GRUB and efibootmgr with `pacman -S grub efibootmgr`, followed by `grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB`. 
For BIOS systems, I used `pacman -S grub` and `grub-install /dev/sda`. Afterward, I generated the GRUB configuration with `grub-mkconfig -o /boot/grub/grub.cfg`. 
**Issue:** When trying to run grub-install, I initially encountered the error "grub command not found." This was resolved by ensuring GRUB was installed using `pacman -S grub`. 

**Step 12: Create Users and Configure Sudo**
I created two user accounts using `useradd -m -G wheel justin` and `useradd -m -G wheel codi`, then set their passwords using passwd justin and passwd codi. 
To grant these users sudo privileges, I installed sudo with `pacman -S sudo`, then edited the sudoers file using `EDITOR=nano visudo`, where I uncommented the line "%wheel ALL=(ALL) ALL". 
**Issue:** I encountered a "Permission denied" error when trying to run visudo as a non-root user. This was resolved by switching to the root user using `su -` 
and then editing the sudoers file. 

**Step 13:** Install a Desktop Environment (DE) 
I installed the LXQt desktop environment by running `pacman -S lxqt sddm`, then enabled the display manager with `systemctl enable sddm`. 

**Step 14: Install a Different Shell**
I installed Zsh by running `pacman -S zsh` and changed the default shell for both justin and codi with `chsh -s /bin/zsh justin` and `chsh -s /bin/zsh codi`. 
**Issue:** When trying to enable terminal colors using `autoload -U colors && colors`, I encountered an "autoload command not found" error. This was resolved by ensuring I was in the Zsh shell, 
as the autoload command only works in Zsh, not Bash.

**Step 15:** Install SSH 
I installed and enabled SSH by running `pacman -S openssh` and `systemctl enable sshd`. 

**Step 16:** Add Terminal Colors and Aliases 
To add color support for Zsh, I edited the .zshrc file (nano ~/.zshrc) and added the line "autoload -U colors && colors". 
I also added useful aliases like "alias ll='ls -lah'" and "alias update='sudo pacman -Syu'" to the .zshrc file. 

**Step 17: Reboot**
After finishing all configurations, I exited the chroot environment with `exit`, unmounted the partitions using `umount -R /mnt`, and rebooted the system with `reboot`. 
**Issue:** I encountered the error "running in chroot, ignoring request" when trying to reboot from within the chroot environment. This was resolved by exiting chroot,
unmounting the partitions manually, and rebooting from the live environment. 
