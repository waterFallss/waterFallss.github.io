# ARCH-LINUX
![ARCH](ARCH_PIC.jpg)

#### Table of Contents 📑
1. [Arch Linux Downloads and Pre-Installation Steps](#preinstall)
2. [Installation Steps on VMWare](#vmware)
3. [Check Connection with the Internet](#connection)
4. [Update System Clock](#clock)
5. [Partitioning](#partition)
6. [Formatting the Partitions](#format)
7. [Mirror Selection](#mirror)
8. [Mount File Systems](#mount)
9. [Package Installation Including Another Shell](#package)
10. [System Configuration](#configure)
11. [Network Configuration](#network)
12. [Username and Password](#userPass)
13. [Setting Up the Desktop Environment](#desktop)
14. [Shut Down](#off)
15. [Installing SSH](#ssh)
16. [Terminal Color-Coding](#terminalColor)
17. [Adding Aliases](#alias)
18. [Error: Quick Fixes](#error)
19. [References](#reference)

#### Arch Linux Downloads and Pre-Installation Steps 🅰️ <a name="preinstall"></a>
***
1. [Arch Linux Installation Guide](https://wiki.archlinux.org/title/Installation_guide)
2. Download ISO from your country: [ISO Download](https://archlinux.org/download/)
3. For this Walkthrough: [MIT ISO](http://mirrors.mit.edu/archlinux/iso/2023.10.14/)
4. Use ISO Image Signature for verification of file (Example: archlinux-2023.10.14-x86_64.iso.sig)
5. If gpg is not already installed: [GPG INSTALL](https://gpg4win.org/get-gpg4win.html)
```GPG Verification
 gpg --keyserver-options auto-key-retrieve --verify archlinux-version-x86_64.iso.sig   
```
#### Installation Steps on VMWare ✏️ <a name="vmware"></a>
***
1. Click on File
2. Click on "New Virtual Machine"
3. Click on "Typical Configuration"
4. After selecting "Installer Disc Image File (iso)", find your iso file.
5. Guest Operating System: Linux
6. Version: Other Linux 5.x kernel 64-bit
7. Maximum DIsk Size: 20 GB (or Greater)
8. Store Virtual Disk as a Single File
9. Go to the setting on the Arch VM you created -> Click on "Options" -> "Advanced" -> Change Firmware type to UEFI
10. Power On
11. Click Enter on "Arch Linux Install Medium (x86_64, UEFI)"
12. Ensure that the system was booted in UEFI mode.
```
cat /sys/firmware/efi/fw_platform_size
```
#### Check Connection with the Internet 🖨️ <a name="connection"></a>
***
1. Check to see if your network interface is listed and enabled:
```
ip link
```
2. Ping archilinux.org for verification
```
ping archlinux.org
```
#### Update System Clock ⏰ <a name="clock"></a>
***
1. Check the time zone.
```
timedatectl
```
2. If incorrect timezone, list out the time zones and set the it to the correct time zone.
```
timedatectl list-timezones
```
```
timedatectl set-timezone [TIME_ZONE]
```
#### Partitioning 🗃️ <a name="partition"></a>
***
1. Create EFI System Partition
```
fdisk /dev/sda
Command (m for help): n (create new Partition)
Partition type: p (primary partition)
Partition number: 1 (results in /dv/sda1)
First Sector: 2048 (default)
Last Sector: +500M
Command (m for help): t (Change partition type)
Hex Code or Alias: EF (Changes partition type from "Linux" to "EDI (FAT-12/16/32)"
```
2. Create Root Partition:
```
Command (m for help): n
Partition Type: p
Partition Number: 2
First Sector: [TYPE_DEFAULT_VALUE]
Last Sector: [VALUE_OF_CHOICE] (For this, +18G was added)
Command (m for help): w (Write table to disk and exit.)
```
#### Formatting the Partitions ℹ️ <a name="format"></a>
***
1. For the Root Partition
```
mkfs.ext4 /dev/[ROOT_PARTITION]
Example: mkfs.ext4 /dev/sda2
```
2. For the EFI System Partition
```
mkfs.fat -F 32 /dev/[EFI_SYSTEM_PAARTITION]
Example: mkfs -F 32 /dev/sda1
```
#### Mirror Selection 🪞 <a name="mirror"></a>
***
1. Enable pacman to be able to download and install software
```
pacman -Syy
```
2. Install Refelector
```
pacman -S reflector
```
3. Create a Backup for the Mirror List
```
cp /etc/pacman.d/mirrorlist /etc/pacman.d/mirrorlist.bak
```
4. Obtain a mirror list from the reflector and save it.
```
reflector -c "US" -f 12 -l 10 -n 12 --save /etc/pacman.d/mirrorlist
```
#### Mount File Systems 📂 <a name="mount"></a>
***
1. Mount Root Partition
```
mount /dev/[ROOT_PARTITION] /mnt
Example: mount /dev/sda2 /mnt
```
2. Mount UEFI System
```
mount --mkdir /dev/[UEFI_SYSTEM_PARTITION] /mnt/boot
Example: mount --mkdir /dev/sda1 /mnt/boot
```
3. Check Partition Layout
```
sudo fdisk -l
OR
df -h
```
#### Basic Installation 🖥️ <a name="package"></a>
***
1. Install Basic Packages
```
pacstrap -K /mnt base linux linux-firmware sudo vim zsh man base-devel grub efibootmgr mkinitcpio nano
```
#### System Configuration ☑️ <a name="configure"></a>
***
1. Generate fstab file
```
genfstab -U /mnt >> /mnt/etc/fstab
```
2. Check to see if the fstab file has been generated correctly.
```
cat /mnt/etc/fstab
```
3. Change root into the new system (Once doing so, stay as root for the rest of the instruction until told to exit)
```
arch-chroot /mnt
```
4. Time Zone:
```
ln -sf /usr/share/zoneinfo/[REGION]/[CITY] /etc/localtime
Example: ln -sf /usr/share/zoneinfo/America/Chicago /etc/localtime
```
5. Localization
```
vim /etc/locale.gen
Remove # in front of the desired localization (e.g. en_US.UTF-8 UTF-8)
Check if /etc/local.conf file can be created correctly
echo 'LANG=en_US.UTF-8' >> /etc/local.conf
cat /etc/locale.conf
```
#### Network Configuration ☎️ <a name="network"></a>
***
1. Create a Hostname
```
echo [HOSTNAME] > /etc/hostname
Example: echo archUser > /etc/hostname
```
2. Configure the /etc/hosts
```
127.0.0.1  localhost
:1         localhost
127.0.1.1  localhost
```
#### GRUB Installation 💻 <a name="grub"></a>
***
1. Type down these commands (Make sure you are still under arch-chroot.)
```
pacman -S grub efibootmgr
mkdir /boot/efi
mount /dev/sda1 /boot/efi
grub-intall --target=x86_64-efi --bootloader-id=GRUB --efi-directory=/boot/efi
grub-mkconfig -o /boot/grub/grub.cfg
```
#### Username and Password 📔 <a name="userPass"></a>
***
1. Create User
```
useradd -m [username]
```
2. Create Password for the User
```
passwd [username]
```
3. Permissions for the User
```
EDITOR=nano visudo
[username] ALL=(ALL:ALL) ALL (This provides sudo privilege to the user.)
```
#### Setting Up the Desktop Environment 🖥️ <a name="desktop"></a>
***
1. Install xorg
```
sudo pacman -S --needed xorg
```
2. Install lxqt
```
sudo pacman -S --needed lxqt xdgutils ttf-freefont sddm
```
3. Install Extra Components for the Desktop
```
sudo pacman -S --needed libpulse libstatgrab libsysstat lm_sensors network-manager-applet oxygen-icons pavucontrol-qt
```
4. Install a browser and other applications
```
sudo pacman -S --needed firefox vlc filezilla leafpad xscreensaver archlinux-wallpaper
```
5. Enable sddm
```
systemctl enable sddm
```
6. Enable Network Manager
```
systemctl enable NetworkManager
```
7. Enable WPA Supplicant Service
```
systemctl enable wpa_supplicant.service
```
#### Shut Down 🚪 <a name="off"></a>
***
```
exit
umount -l /mnt
shutdown now
```
1. After Shutting down, do not forget to remove the live USB before powering on again. (Look at [Quick Fixes](#error) for more information)

#### Installing SSH 🖱️ <a name="ssh"></a>
***
1. Install ssh package
```
pacman -S openssh
```
2. To connect to a gateway
```
ssh -p [PORT_NUMBER] [USER_NAME]@[SERVER_IP_ADDRESS]
Example: ssh -p 22 sysadmin@10.1.1.325
```
#### Terminal Color-Coding 🖍️ <a name="terminalColor"></a>
***
1. First Create a Backup of the Settings File
```
(Not System Wide)
cp .bashrc .bashrc.backup
(For System-Wide bashrc)
sudo cp /etc/bash.bashrc /etc/bash.bashrc.backup
```
2. Configure the files given in [COLOR_CODE](https://wiki.archlinux.org/title/Bash/Prompt_customization)
3. Sample Configuration Files are given in: [SAMPLE_COLOR](https://averagelinuxuser.com/linux-terminal-color/)
   - Things that should be downloaded before downloading the configured files.
```
sudo pacman -S zip
sudo pacman -S unzip
sudo pacman -Sy wget
```
4. Move the Configured files from the given sample to the specific /etc directories.
```
sudo mv bash.bashrc /etc/bash.bashrc
sudo mv DIR_COLORS /etc/
mv .bashrc ~/.bashrc
```
#### Adding Aliases 📓 <a name="alias"></a>
***
Reference: [MORE_ON_ALIAS](https://www.cyberciti.biz/faq/create-permanent-bash-alias-linux-unix/)
1. Open the ~/.bashrc file
```
nano ~.bashrc
```
2. Add an alias command
```
Example: alias update='sudo pacman -Syu'
```
3. Activate the alias
```
source ~/.bashrc
```
4. To list all the aliases
```
alias
```
5. Unactivate an alias
```
unalias [ALIAS_NAME]
```

#### Error: Quick Fixes ❓ <a name="error"></a>
***
1. __Icon Not Showing on LXQT Desktop Environment__
```
   - Click on the logo at the left corner
   - Click on "Preferences" -> "LXQT Settings" -> "Appearance" -> "Icons Theme" -> "Oxygen (Oxygen Team)"
```
2. __"Not Enough Free Disk Space"__
```
  - Not enough room was reserved for partition number 2 (i.e. the Root Partition)
  - umount -l /dev/sda1
  - Check to see if sda1 has actually been unmounted: df -h
  - If it has not been been unmounted: pkill target_process -> umount /dev/sda1
  - fdisk /dev/sda
  - Command(m for help): n -> Partition type: p -> Partition number: 2 -> first sector: [DEFAULT_VALUE] -> last sectore: [CHOOSE_LARGER VALUE] (e.g. +60G)
  - Command (m for help): w
```
3. __After Setting up the Arch Linux and Rebooting it, it completely starts over the Arch Settings.__
```
  - Do not forget to remove the live USB before powering on the Arch again.
  - Go to "Settings"
  - Click on "CD/DVD (SATA)"
  - Uncheckmark the "Connect at power on"
  - Click "OK"
```
4. __"Cannot find the EFI Directory" although the firmware is UEFI__
```
  - This is caused by the fact that the /dev/sda1 was not mounted into /boot/efi
```
### REFERENCES <a name="reference"></a>
1. https://wiki.archlinux.org/title/Installation_guide
2. https://wiki.archlinux.org/title/Bash/Prompt_customization
3. https://itsfoss.com/install-arch-linux/#:~:text=How%20to%20install%20Arch%20Linux%201%20Step%201%3A,USB%20...%204%20Step%204%3A%20Partition%20the%20disks
4. https://www.debugpoint.com/lxqt-arch-linux-install/
5. https://averagelinuxuser.com/linux-terminal-color/
6. https://www.cyberciti.biz/faq/create-permanent-bash-alias-linux-unix/

