---

---
This is a simple guide to install Arch Linux on most computers and how to setup  the `kde plasma` desktop environment and it's useful `pcksgs`, including essentials like the `ssdm` display manager, GPU drivers, `firefox` browser and a GUI file manager `thunar`.

>*Note:* this guide isn't perfect and you're always welcome to suggest changes

>**Progress:**
>	*Installation:* All steps have been implemented.
>	*Desktop Environment:* Work has just begun xd
---
## set font, preferred keymap and fix time

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
>*note:* if you're using ethernet it should set itself up automatically, if it doesn't, it's likely caused by MAC address filtering

*active wlan check*
```
ip link
```

*wlan connection*
```
iwctl
device list
station wlan0 scan
station wlan0 get-networks
station wlan0 connect "your_wifi_name"
exit
```

*internet check:*
```
ping archlinux.org
```
> *note:* press `Ctrl`+`C` to exit
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

*set city and region*
```
ln -sf /usr/share/zoneinfo/Region/City /etc/localtime
```
>*note:* to see available Cities for selected Region: 
>enter `zoneinfo` using `cd /usr/share/zoneinfo`
>select prefeed Region by `cd /Region`, example: `cd /Europe`
>list available Cities for Region by `ls` (usually nation capitals)

*sync hw-clock*
```
hwclock --systohc
```
---
## setup locale

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
>use `vconsole.conf` to configure default `tty` keymap

*generate locales*
```
locale-gen
```

*set sys-lang*
```
echo "LANG=en_US.UTF-8" > /etc/locale.conf
```
---
## 11. hostname and host

*set the sys-hostname*
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

*install necessary pckgs and `networkmanager`*
```
pacman -S networkmanager vim grub efibootmgr base-devel
```

*enable `networkmanager`*
```
systemctl enable NetworkManager
```
---
## install and configure bootloader

*install GRUB*
```
grub-install --target=x86_64-efi --efi-directory=/boot --bootloader-id=GRUB
```

*generate GRUB config*
```
grub-mkconfig -o /boot/grub/grub.cfg
```
---
## create user-acc

*create new-user and add to `wheel` (admin group)*
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

*change default `tty` keymap*
```
sudo nano /etc/vconsole.conf
```
>*note:* use the same keymap name as you did with `loadkeys` during installation, then `reboot`

*check network status*
```
systemctl status NetworkManager
```

