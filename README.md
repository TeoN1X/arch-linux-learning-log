[*] = command runs silently but takes effect  
[#] = command produces harmless output  
:warning: = important caution or risk  

== env-setup ==  
// Disable terminal bell/beep and shell history beep  
setterm -blength 0  # [*]  
set bell-style none  # [*]  

== mnt-backup ==  
// If you're reinstalling Arch, back up important files before wiping the disk  
mount /dev/your_usb /mnt  # Mount backup USB device [#]  
lsblk  # List block devices to identify your USB [#]  
mkdir /mnt/your_usb  # Create mount point for USB [*]  
ls /mnt/your_usb  # List files inside mounted USB [#]  
df -h /mnt/your_usb  # Show USB disk usage in human-readable format [#]  
cp -r ~/your_folder /mnt/your_usb  # Copy folder recursively to USB  
cp your_file.txt /mnt/your_usb  # Copy single file to USB  
cp -a your_folder /mnt/your_usb  # Copy with attributes (permissions, timestamps)

== boot ==  
// BIOS (UEFI) setup: move "EFI USB Device" to top in boot order  
// Use F2/F12/DEL to enter BIOS depending on your device

== wifi-setup ==  
rfkill unblock wifi  # Enable Wi-Fi device [*]  
iwctl  # Enter interactive Wi-Fi shell

// Inside iwctl:
device list  # Show Wi-Fi devices; check "Powered: on" [#]  
station your_device_name scan  # Scan for networks (no output = silent) [#]  
station your_device_name get-networks  # Show available SSIDs [#]  
station your_device_name connect "your_SSID"  # :warning: SSID must be quoted if it has spaces or symbols

== debug-tools ==  
dmesg | grep firmware  # Check for missing firmware errors [#]  
ip link  # Show all network interfaces [#]  
iw dev  # Show Wi-Fi device state [#]  

== timezone ==  
ln -sf /usr/share/zoneinfo/Your_Region/Your_City /etc/localtime  # Set system timezone [*]  
hwclock --systohc  # Generate /etc/adjtime by syncing hardware clock [*]  
timedatectl set-ntp true  # Enable NTP auto time sync [*]  
timedatectl status  # Check system time, NTP, timezone status [#]  

== disk-prep ==  
lsblk  # List current block devices and partition layout [#]  
cfdisk /dev/your_disk  # Open partition editor

// Inside cfdisk:
 - Delete existing partitions if any  
 - Create partitions as follows:
     - EFI Partition: 512M, Type "EFI System"  
     - Swap Partition: 4G or same as your RAM, Type "Linux swap"  
     - Root Partition: remaining space, Type "Linux filesystem"  
 - Choose [Write], type 'yes', then [Quit]

== mount ==  
// Mounting tells the system where each storage volume is located  
mkswap /dev/your_swap_partition  # Format swap  
mkfs.ext4 /dev/your_root_partition  # Format root with ext4  
swapon /dev/your_swap_partition  # Enable swap partition [*]  
mount /dev/your_root_partition /mnt  # Mount root to /mnt [#] (/mnt is temp mount point)  
mkdir -p /mnt/boot/efi  # Create EFI mount point [*]  
mount /dev/your_efi_partition /mnt/boot/efi  # Mount EFI partition to correct location [#]  
// Use `lsblk` to verify partition names before mounting

== mirror-setup ==  
// :warning: If you're in China, default mirrors may fail. Replace mirrorlist manually:
cat > /etc/pacman.d/mirrorlist <<EOF  
Server = https://mirrors.hit.edu.cn/archlinux/$repo/os/$arch  
Server = https://mirrors.163.com/archlinux/$repo/os/$arch  
Server = https://mirror.sjtu.edu.cn/archlinux/$repo/os/$arch  
EOF

curl -I https://your.mirror.url/archlinux/core/os/x86_64/  # Test mirror: look for HTTP/2 200 [#]  
reflector --country China --age 12 --sort rate --save /etc/pacman.d/mirrorlist  # Update mirrorlist [#]  
// You can use ↑ and ↓ to reuse previously typed commands

== base-install ==  
pacstrap -K /mnt base linux linux-firmware neovim kitty zsh starship  # Install base system and terminal tools [#]
genfstab -U /mnt >> /mnt/etc/fstab  # Generate system mount config file [*] (/etc/fstab defines what to mount on boot)

== zsh-starship-setup ==  
chsh -s /bin/zsh your_username  # Set Zsh as default shell for your user [*]  
echo 'eval "$(starship init zsh)"' >> /home/your_username/.zshrc  # Load Starship prompt in Zsh [*]  

mkdir -p /home/your_username/.config  # Create config directory if not exist [*]  
nano /home/your_username/.config/starship.toml  # Manually edit Starship config

// In the editor, type or paste the following content:
add_newline = false

[character]
success_symbol = "->"
error_symbol = "!!"

[directory]
truncation_length = 3
style = "blue"

[username]
show_always = true
style_user = "yellow"

[hostname]
ssh_only = false
style = "green"

== chroot ==  
arch-chroot /mnt  # Enter the newly installed system environment [#]

== locale ==  
ln -sf /usr/share/zoneinfo/Your_Region/Your_City /etc/localtime  # Set timezone [*]  
hwclock --systohc  # Sync hardware clock to system time [*]

echo "your_locale UTF-8" > /etc/locale.gen  # Enable desired UTF-8 locale (e.g. zh_CN.UTF-8) [*]  
locale-gen  # Generate compiled locale info [#]  
echo "LANG=your_locale" > /etc/locale.conf  # Set global language environment [*]

== hostname ==  
echo "your_hostname" > /etc/hostname  # Set your system's hostname [*]  

cat > /etc/hosts <<EOF  
127.0.0.1   localhost  
::1         localhost  
127.0.1.1   your_hostname.localdomain your_hostname  
EOF

== root-password ==  
passwd  # Set root password [#]

== paru-install ==  
git clone https://aur.archlinux.org/paru-bin.git  # Clone paru AUR helper [#]  
cd paru-bin  # Enter paru-bin directory [#]  
cd ~  # Return to home to ensure re-entry path is correct [*]  
cd paru-bin  # Re-enter paru-bin under home path  
makepkg -si  # Build and install paru from PKGBUILD [#]  
// If `cd ~` is skipped, makepkg may fail due to incorrect working directory

== user-setup ==  
useradd -m your_username  # Create new user and home directory [*]  
passwd your_username  # Set user password [#]  
usermod -aG wheel your_username  # Add user to sudo group [*]  

EDITOR=your_editor_name visudo  # Safely edit sudoers file [#]  
// Uncomment this line to allow sudo for wheel group:  
// %wheel ALL=(ALL:ALL) ALL  
// Exit visudo properly based on your editor (e.g. :wq in nvim)

== user-setup ==  
useradd -m your_username  # Create new user and home directory [*]  
passwd your_username  # Set user password [#]  
usermod -aG wheel your_username  # Add user to sudo group [*]  

EDITOR=your_editor_name visudo  # Safely edit sudoers file [#]  
// Uncomment this line to allow sudo for wheel group:  
// %wheel ALL=(ALL:ALL) ALL  
// Exit visudo properly based on your editor (e.g. :wq in nvim)

== nvidia-driver ==  
sudo pacman -S nvidia nvidia-utils nvidia-settings  # Install proprietary NVIDIA driver and tools [#]  
sudo nvidia-xconfig  # Generate /etc/X11/xorg.conf (required by some tools even if Wayland is used) [#]  
// You can later remove /etc/X11/xorg.conf if not using X11

== hyprland-setup ==  
sudo pacman -S hyprland xdg-desktop-portal-hyprland xdg-desktop-portal  # Install Hyprland and portal interface [#]  

mkdir -p /home/your_username/.config/hypr  # Create Hyprland config directory [*]  

pacman -Ql hyprland | grep hyprland.conf  # Locate default config file path [#]  
cp /usr/share/hypr/hyprland.conf /home/your_username/.config/hypr/  # Copy config to user folder [*]  
// If unsure of path, run `pacman -Ql hyprland | grep hyprland.conf`

echo '[ "$(tty)" = "/dev/tty1" ] && exec Hyprland' >> /home/your_username/.zprofile  # Auto-start Hyprland on TTY1 login [*]  
// This ensures Hyprland starts only when logging in from tty1 using zsh

== system-network ==  
systemctl enable systemd-networkd  # Enable built-in systemd network manager [#]  
systemctl enable systemd-resolved  # Enable DNS resolver service [#]  
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf  # Link DNS config path [*]  

== waybar-setup ==  
sudo pacman -S waybar  # Install Waybar [#]  
mkdir -p /home/your_username/.config/waybar  # Create config directory [*]  
pacman -Ql waybar | grep config  # Find config paths [#]  
cp /usr/share/waybar/config.jsonc /home/your_username/.config/waybar/  # Copy config file [*]  
cp /usr/share/waybar/style.css /home/your_username/.config/waybar/  # Copy style file [*]

== exit-chroot ==  
exit  # Exit from chroot environment  
umount -R /mnt  # Recursively unmount all mounted filesystems [*]  
reboot  # Reboot into your new Arch system  















