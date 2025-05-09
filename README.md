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

# Inside iwctl
device list  # List available Wi-Fi devices; look for "Powered: on" [#]  
station device_name scan  # Scan for Wi-Fi networks; no output = no signal or failed [#]  
station device_name get-networks  # Show available networks; empty = failure [#]  
station device_name connect SSID  # :warning: **Important:** wrap SSID in double quotes if it contains spaces or special characters  

#debug-tools (outside iwctl)

dmesg | grep firmware  # Check for firmware errors; look for "missing" or "failed to load" [#]  
ip link  # Show network interfaces; look for DOWN state [#]  
iw dev  # Show wireless info; confirm device exists and is UP [#]  

#timezone

ln -sf /usr/share/zoneinfo/Region/City /etc/localtime  # Set system timezone [*]  
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

# Reminder for partition sizes:
# - EFI: 512M  
# - Swap: Match RAM (e.g. 4G)  
# - Root: All remaining space

mkswap /dev/your_swap_partition  # Format swap partition  
mkfs.ext4 /dev/your_root_partition  # Format root partition  
swapon /dev/your_swap_partition  # Enable swap [*]  
mount /dev/your_root_partition /mnt  # Mount root to /mnt [*]  (/mnt is a temp mount point used during installation)  
mkdir -p /mnt/boot/efi  # Create EFI mount point [*] (/boot/efi is where EFI bootloaders go)  
mount /dev/your_efi_partition /mnt/boot/efi  # Mount EFI partition [*]  

#mirror-setup

# :warning: If you're in China, default mirrors may fail. Use working mirrors manually:
cat > /etc/pacman.d/mirrorlist <<EOF  
Server = https://mirrors.hit.edu.cn/archlinux/$repo/os/$arch  
Server = https://mirrors.163.com/archlinux/$repo/os/$arch  
Server = https://mirror.sjtu.edu.cn/archlinux/$repo/os/$arch  
EOF  

# To test a mirror:
curl -I https://your.mirror.url/archlinux/core/os/x86_64/  # Look for HTTP/2 200 or server info [#]

# To update mirrors with reflector:
reflector --country China --age 12 --sort rate --save /etc/pacman.d/mirrorlist  

# Tip: use up/down arrows to edit and reuse previous commands!

#base-install

pacstrap -K /mnt base linux linux-firmware neovim  # Install base system, kernel, firmware, and editor  








