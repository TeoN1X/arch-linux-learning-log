#arch linux installation without 'archinstall' from scratch

[*] = command runs silently but takes effect  
[#] = command produces harmless output  
:warning: = important caution or risk  

#env-setup
setterm -blength 0  # Disable terminal beep/bell sound [*]  
set bell-style none  # Disable shell bell on empty history access [*]  

#mnt-backup
# If you're reinstalling Arch, back up important files before wiping the disk.
# The steps below may repeat later mount instructions, but that's fine.
mount /dev/your_usb /mnt  # Attach (mount) partition your_usb to /mnt directory  
lsblk  # Show block devices and partitions [#]  
mkdir /mnt/your_usb  # Create directory for USB mount point [*]  
ls /mnt/your_usb  # List files and folders inside the mounted USB drive [#]  
df -h /mnt/your_usb  # Show disk usage of the USB in human-readable format [#]  
cp -r ~/folder /mnt/your_usb  # Copy folder and all contents to USB (recursive)  
cp file.txt /mnt/your_usb  # Copy single file to USB  
cp -a folder /mnt/your_usb  # Copy folder with attributes (permissions, timestamps, etc.)

#boot  
# BIOS boot order (UEFI)  
# Move "EFI USB Device (your USB name)" to top to boot into Arch installer  

#wifi-setup
rfkill unblock wifi  # Enable (unblock) wireless device [*]  
iwctl  # Launch iwd interactive shell  
# Inside iwctl:
device list  # List available Wi-Fi devices; look for "Powered: on" [#]  
station device_name scan  # Scan for Wi-Fi networks; no output = no signal or failed [#]  
station device_name get-networks  # Show available networks; empty = failure [#]  
station device_name connect "your_SSID"  # :warning: Important: wrap SSID in double quotes if it contains spaces or special characters  

#debug-tools (outside iwctl)  
dmesg | grep firmware  # Check for firmware errors; look for "missing" or "failed to load" [#]  
ip link  # Show network interfaces; look for DOWN state [#]  
iw dev  # Show wireless info; confirm device exists and is UP [#]  

#timezone  
ln -sf /usr/share/zoneinfo/Your_Region/Your_City /etc/localtime  # Set system timezone [*]  
hwclock --systohc  # Sync hardware clock to system time [*]  
timedatectl set-ntp true  # Enable automatic time sync (NTP) [*]  
timedatectl status  # Check current time and sync state [#]  

#disk-prep  
cfdisk /dev/your_disk  # Launch partition editor  
# Inside cfdisk:
# 1. Delete existing partitions if needed  
# 2. Create:
#    - EFI partition: 512M, Type: EFI System  
#    - Swap partition: 4G (or match RAM), Type: Linux swap  
#    - Root partition: remaining space, Type: Linux filesystem  
# 3. Write changes, confirm with 'yes', then Quit  

#mount  
# Mounting is like telling your system the route to each storage area.  
mkswap /dev/your_swap_partition  # Format swap partition  
mkfs.ext4 /dev/your_root_partition  # Format root partition  
swapon /dev/your_swap_partition  # Enable swap [*]  
mount /dev/your_root_partition /mnt  # Mount root to /mnt [#] (/mnt is a temp mount point used during installation)  
mkdir -p /mnt/boot/efi  # Create EFI mount point [*] (/boot/efi is where EFI bootloaders go)  
mount /dev/your_efi_partition /mnt/boot/efi  # Mount EFI partition [#]  
# Tip: Use `lsblk` to identify your_efi_partition etc.  

#mirror-setup  
# :warning: If you're in China, default mirrors may fail. Use working mirrors manually:
cat > /etc/pacman.d/mirrorlist <<EOF  
Server = https://mirrors.hit.edu.cn/archlinux/$repo/os/$arch  
Server = https://mirrors.163.com/archlinux/$repo/os/$arch  
Server = https://mirror.sjtu.edu.cn/archlinux/$repo/os/$arch  
EOF

# Test a mirror:
curl -I https://your.mirror.url/archlinux/core/os/x86_64/  # Look for HTTP/2 200 [#]

# Update mirrors:
reflector --country China --age 12 --sort rate --save /etc/pacman.d/mirrorlist

# Tip: Use up/down arrow keys to edit and reuse previous commands!

#base-install  
pacstrap -K /mnt base linux linux-firmware neovim  # Install base system, kernel, firmware, and editor [#]  
genfstab -U /mnt >> /mnt/etc/fstab  # Generate system mount config file [*] (/etc/fstab defines what to mount on boot)  
arch-chroot /mnt  # Enter installed system as root shell [#]  
ln -sf /usr/share/zoneinfo/Your_Region/Your_City /etc/localtime  # Set timezone [*]  
hwclock --systohc  # Sync hardware clock to system time [*]  
echo "your_locale UTF-8" > /etc/locale.gen  # Enable desired UTF-8 locale (e.g. zh_CN.UTF-8) [*]  
locale-gen  # Generate locale definitions [#]  
echo "LANG=your_locale" > /etc/locale.conf  # Set system language environment [*]  
echo "your_hostname" > /etc/hostname  # Set machine hostname [*]  
cat > /etc/hosts <<EOF  
127.0.0.1   localhost  
::1         localhost  
127.0.1.1   your_hostname.localdomain your_hostname  
EOF  
passwd  # Set root password [#]  

#paru-install

git clone https://aur.archlinux.org/paru-bin.git  # Clone paru binary build script [#]  
cd paru-bin  # Enter paru directory to verify it exists [#]  
cd ~  # Return to home directory [*]  
cd paru-bin  # Re-enter the cloned directory under home  
makepkg -si  # Build and install paru [#]

# Tip: If 'paru-bin' was not cloned under home (~), step 4 may fail.

#user-setup  
useradd -m your_username  # Create new user with home directory [*]  
passwd your_username  # Set password for your user [#]  
usermod -aG wheel your_username  # Add user to 'wheel' group for sudo access [*]  
EDITOR=your_editor_name visudo  # Safe editing of sudoers file (e.g. nvim, nano, micro) [#]  
# Uncomment: %wheel ALL=(ALL:ALL) ALL  

#hyprland-setup  
sudo pacman -S hyprland xdg-desktop-portal-hyprland xdg-desktop-portal  # Install Hyprland and Wayland portal [#]  
mkdir -p /home/your_username/.config/hypr  # Create config directory for Hyprland [*]  
pacman -Ql hyprland | grep hyprland.conf  # Locate Hyprland config file [#]  
cp /usr/share/hypr/hyprland.conf /home/your_username/.config/hypr/  # Copy default config [*]  
echo '[ "$(tty)" = "/dev/tty1" ] && exec Hyprland' >> /home/your_username/.zprofile  # Autostart Hyprland on login [*]  

#system-network  
systemctl enable systemd-networkd  # Enable basic systemd networking [#]  
systemctl enable systemd-resolved  # Enable system DNS resolver [#]  
ln -sf /run/systemd/resolve/stub-resolv.conf /etc/resolv.conf  # Symlink DNS config [*]  

#exit-chroot  
exit  # Exit chroot  
umount -R /mnt  # Unmount all target system partitions [*]  
reboot  # Reboot into your new Arch system  

#nvidia-driver-install  
sudo pacman -S nvidia nvidia-utils nvidia-settings  # Install proprietary NVIDIA driver and tools [#]  
sudo nvidia-xconfig  # Generate /etc/X11/xorg.conf for NVIDIA [#]  

#fonts-fallback  
sudo pacman -S noto-fonts ttf-dejavu ttf-liberation  # Basic font families [#]  
# Tip: Install fallback fonts if you see square or garbled characters in GUI  

#clash-setup  
export GOPROXY=https://goproxy.cn,direct  # Use reliable Go module proxy [*]  
export GO111MODULE=on  # Enable Go modules [*]  
paru -S clash-verge-rev-bin  # Install Clash Verge Rev binary from AUR [#]  
which clash-verge-rev  # Confirm installation path [#]  

#proxy-config  
export all_proxy="socks5://127.0.0.1:your_port"  # Set proxy if needed [*]  
curl https://www.google.com  # Test proxy connection [#]  



















