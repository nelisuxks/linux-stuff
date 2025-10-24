---

---
This is a simple guide to install Arch Linux on most computers and how to setup  the `kde-plasma` desktop environment and it's useful `pcksgs`, including essentials like the `ssdm` display manager, GPU drivers, `firefox` browser and a GUI file manager `thunar`.

>*Note:* this guide isn't perfect and you're always welcome to suggest changes

>**Progress:**
>	*Installation:* All steps have been implemented. Future changes are possible.
>	*Desktop Environment:* initial setup, dual-boot setup and `tty` QoL fixes implemented
---

## user interface and time

*set readable console font:*
```
setfont ter-132b
```
>*note:* you can proceed with installation using the default font, however (and especially when using VM's) it's better to use HiDPI fonts for better readability

*set preferred keymap*
```
loadkeys cz-qwertz
```

*network time-sync:*
```
timedatectl set-ntp true
```

*time-sync check:*
```
timedatectl status
```
>*note:* never install arch without time-sync because package manager operations and secure mirror connections rely on it
---
## network setup
>*note:* if you're using ethernet it should set itself up automatically - if it doesn't, it's likely caused by MAC address filtering (router feature)

*active `wlan` devices check*
```
ip link
```
>*note:*
>`wlan0` is typically the tag for your primary Wi-Fi device 

*connect to Wi-Fi and check connection*
```
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "your_wifi_name"
exit
---
ping archlinux.org
```
> *note:* press `Ctrl`+`C` to exit the `ping` command output
---
## drive partitioning

*list all*
```
lsblk
```

*open `cfdisk` cli for selected drive*
```
cfdisk /dev/sdx
```
>*note:* 
>SATA drives will appear as `sdx` 
>	`x`=physical SATA connector the drive is connected to on your motherboard
>NVME drives will appear as `nvmexny` 
>	`x`=physical M.2 position on your motherboard
>	`y` stands for drive namespace (usually `1`, consumer NVME's never use multiple namespaces, check NVME's manual if unsure)

*recommended partition layout*:

| type | size     | cap formula                  |
| ---- | -------- | ---------------------------- |
| efi  | <512 MiB | none                         |
| root | <16 GiB  | `disk` - `swap`              |
| swap | <1,5 GiB | `ram` + 1,5 GiB + swap space |
>*note:*
>5 GiB of swap space is generally recommended if you wish to not use hibernation modes
>entire `ram` capacity and ± 500 MiB are required for hibernation modes to work
---
## formatting partitions

*format efi as fat32*
```
mkfs.fat -F32 /dev/sdx1
```

*format root as ext4*
```
mkfs.ext4 /dev/sdx2
```
>**note:** you can use `btrfs`, but it requires additional setup

*format swap as swap-space*
```
mkswap /dev/sdx3
```
>*note:* activate swap with `swapon /dev/sdx3`
---
## mounting partitions

*mount root*
```
mount /dev/sdx2 /mnt
```

*creati efi-mount and mount it*
```
mkdir /mnt/boot
mount /dev/sdx1 /mnt/boot
```
---
## install sys-base

*install base-pckgs*
```
pacstrap /mnt base linux linux-firmware
```
---
## generate fstab

*generate file-sys table*
```
genfstab -U /mnt >> /mnt/etc/fstab
```

*check fstab*
```
cat /mnt/etc/fstab
```
---
## chroot into the sys

*change root to sys*
```
arch-chroot /mnt
```
---
## set time-zone

*set `Area` and `Location`*
```
timedatectl set-timezone Area/Location
```
>*note:* to see available timezones use `timedatectl list-timezones`

*sync `hw-clock`*
```
hwclock --systohc
```
---
## setup locale and keymap

*open locale config in nano*
```
nano /etc/locale.gen
```
>**note:** nano might not be installed, install it via `pacman -S nano`

*uncomment preffered locale(s)*
```
en_US.UTF-8 UTF-8
```
>*note:* when locale is uncommented=is enabled for the system
>locales for DE/WM are configured in respective config files

*generate locales*
```
locale-gen
```

*set sys-lang*
```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```

*create the default `tty` keymap config file*
```
nano /etc/vconsole.conf
```

*enter and save*
```
KEYMAP=ln-layout
```
>*note:* 
>`ln`=language layout (eg. `us` for american)
>`layout`=specific key layout (eg. `cz-qwertz` for czech QWERTZ layout)
---
## 11. hostname and host

*set the system name
```
echo "sys-hostname" > /etc/hostname
```

*edit host files*
```
nano /etc/hosts
```

*add*
```
127.0.0.1   localhost
::1         localhost
127.0.1.1   hostname.localdomain hostname
```
---
## set root password

*root-passwd*
```
passwd
```
---
## install necessary utils and enable networking

*install necessary `pckgs`*
```
pacman -S iwd dhcpcd vim grub efibootmgr base-devel
```

*enable networking*
```
systemctl enable iwd dhcpcd
```
---
## install and configure bootloader

*install GRUB*
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=btid
```
>*note:* `btid` can be replaced by any desired name, chosen name will show up in BIOS when selecting boot options

*generate GRUB config*
```
grub-mkconfig -o /boot/grub/grub.cfg
```
---
## create user-acc

*create new-user and set user-passwd*
```
useradd -m -G wheel -s /bin/bash new-user
passwd new-user
```
>*note:* `new user` is to be replaced by desired username

*enable sudo for new-user*
```
EDITOR=nano visudo
```

*uncomment*
```
%wheel ALL=(ALL:ALL) ALL
```
---
## finalization

*exit chroot*
```
exit
```

*unmount all mounted partitions*
```
umount -R /mnt
```

*reboot into new system*
```
reboot
```
---
Congratulations, You have now successfully installed Arch! Login via user credentials and continue on your own, or follow the guide to install the `kde plasma` desktop environment, and various necessities.
## initial steps

*check network status*
```
sudo systemctl status iwd dhcpd
```
>*note:* 
>sometimes networking services deactivate after finishing installation
>re-activate them using `systemctl enable --now iwd dhcpcd`

*connect to Wi-Fi and verify connection*
```
sudo iwctl
station wlan0 connect "your_wifi_name"
exit
---
ping archlinux.org
```
>*note:* 
>setup is similar to the one you've done during installation
>press `Ctrl`+`C` to exit the `ping` command output

## **OPTIONAL:** setting up dual boot

*download `os prober`*
```
sudo pacman -S os-prober
```

*enable OS probing in GRUB config*
```
nano /etc/default/grub
---
GRUB_DISABLE_OS_PROBER=true
```

*rebuild GRUB config*
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
```
>*note:* 
>reboot to enter GRUB menu
>	detected OS entries will be automatically added to boot selection in GRUB menu
>macOS/ZFS, encrypted (BitLocker, LUKS, FileVault etc.) and LVM partitions will not be detected 
>	optionally you can check the output of  `os prober` to verify that your OS was detected
## fixing `tty` font and resolution

*download and test out high-res console fonts*
```
sudo pacman -S terminus-fonts
---
setfont ter-132b
```
>*note:* 
>to see available fonts use `ls /usr/share/kbd/consolefonts/` 
>font `ter-132b` is generally recommended for high-res screens

*set desired font permanently*
```
sudo nano /etc/vconsole.conf
---
FONT=ter-132b
```
>*note:* applying requires reboot

*change framebuffer resolution GRUB config*
```
sudo nano /etc/default/grub
---
GRUB_GFXMODE=res_widthxres_height
GRUB_GFXPAYLOAD_LINUX=keep 
```
>*note:* 
>check display resolution parameters before entering
>also confirm that your resolution is supported by GRUB
>	hit 'c' at the GRUB menu during boot
>	type 'vbeinfo' or 'videoinfo' to list all valid resolutions
>**failing to confirm your resolution and it's availability may result in an unreadable UI**

*rebuild GRUB config and reboot*
```
sudo grub-mkconfig -o /boot/grub/grub.cfg
---
sudo reboot
```