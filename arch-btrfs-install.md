# BTRFS

## Basics

```zsh
ls /sys/firmware/efi/efivars
ping -c 1 google.com
passwd
pacman -Syy
pacman -S git --noconfirm
ip a
```

### SSH in if available

```zsh
reflector --country US --latest 5 --sort rate --save /etc/pacman.d/mirrorlist &&
timedatectl set-ntp true &&
sed -i "/ParallelDownloads/s/^#//" /etc/pacman.conf &&
pacman -Syy
```

## Format

```zsh
lsblk -o name,fssize,fstype,mountpoint,uuid,model
cfdisk
```

- EFI partition 500MB - 2GB
- / partition remaining -8GB
- SWAP 8GB Unless using ZRAM

```zsh
mkfs.vfat -n BOOT /dev/***?1
mkswap -L SWAP /dev/***?2
mkfs.btrfs -L ROOT /dev/***?3

mount /dev/***?3 /mnt
swapon /dev/***?2

cd /mnt &&
btrfs su cr @ &&
btrfs su cr @home &&
btrfs su cr @root &&
btrfs su cr @snapshots &&
btrfs su cr @games &&
btrfs su cr @cache &&
btrfs su cr @log &&
btrfs su cr @flatpak &&
btrfs su cr @vm &&
btrfs su cr @docker &&
cd / &&
umount /mnt

mount -o compress=zstd:1,noatime,subvol=@ /dev/***?3 /mnt

mkdir -p /mnt/{/boot,/home,/root,/.snapshots,/opt/games,/var/cache/pacman/pkg,/var/log,/var/lib/flatpak,/var/lib/libvrt/images}

mount -o compress=zstd:1,noatime,nodiscard,subvol=@home /dev/***?3 /mnt/home
mount -o compress=zstd:1,noatime,nodiscard,subvol=@root /dev/***?3 /mnt/root
mount -o compress=zstd:1,noatime,nodiscard,subvol=@snapshots /dev/***?3 /mnt/.snapshots
mount -o compress=zstd:1,noatime,nodiscard,subvol=@games /dev/***?3 /mnt/opt/games
mount -o compress=zstd:1,noatime,nodiscard,subvol=@cache /dev/***?3 /mnt/var/cache/pacman/pkg
mount -o compress=zstd:1,noatime,nodiscard,subvol=@log /dev/***?3 /mnt/var/log
mount -o compress=zstd:1,noatime,nodiscard,subvol=@flatpak /dev/***?3 /mnt/var/lib/flatpak
mount -o compress=zstd:1,noatime,nodiscard,subvol=@vm /dev/***?3 /mnt/var/lib/libvirt/images
mount -o compress=zstd:1,noatime,nodiscard,subvol=@docker /dev/***?3 /mnt/var/lib/docker

mount /dev/***?1 /mnt/boot
```

## Pacstrap

```zsh
pacstrap /mnt archlinux-keyring base base-devel neovim rsync openssh reflector git &&
genfstab -U /mnt >> /mnt/etc/fstab &&
cat /mnt/etc/fstab &&
mkdir -p /mnt/remove &&
git clone https://github.com/jnines/SysInstall.git /mnt/remove &&
arch-chroot /mnt
```

```zsh
ln -sf /usr/share/zoneinfo/America/Chigaco /etc/localtime &&
hwclock --systohc &&
reflector --country US --latest 5 --sort rate --save /etc/pacman.d/mirrorlist &&
echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen &&
locale-gen &&
echo 'LANG=en_US.UTF-8' > /etc/locale.conf &&
echo 'archbox' > /etc/hostname &&
echo "127.0.0.1   localhost
::1         localhost
127.0.1.1   archbox.lan archbox" >> /etc/hosts &&
sed -i \
-e '/Color/s/^#//' \
-e '/VerbosePkgLists/s/^#//' \
-e '/ParallelDownloads/s/^#//' \
-e '/^#\[multilib]/{N;s/\n#/\n/}' \
-e '/\[multilib]/s/^#//' \
-e 's/#NoExtract.*/NoExtract = usr\/lib\/security\/pam_systemd_home.so/' \
-e '/ParallelDownloads/a ILoveCandy' \
/etc/pacman.conf &&
sed -i \
-e 's/#MAKEFLAGS.*/'MAKEFLAGS="-j\$(( \$(nproc) - 2 ))"'/' \
-e '/#BUILDDIR.*/s/^#//' \
/etc/makepkg.conf &&
sed -i \
-e '/-auth.*/s/^/#/' \
-e '/-account.*/s/^/#/' \
-e '/-session.*/s/^/#/' \
/etc/pam.d/system-auth &&
passwd &&
pacman -Syy &&
cd /remove/arch || exit
```

`pacman -S --needed - <`
PackageLists from /remove/arch/

## Refind

```zsh
refind-install
blkid ROOT
nvim /boot/EFI/refind/refind.conf
```

```zsh
timeout 3
resolution 2560 1440 # Pick your resolution
use_graphics_for linux
scanfor manual
fold_linux_kernels false
default_selection "Arch"

menuentry "Arch" {
    volume   "Arch Linux"
    icon     /EFI/refind/icons/os_arch.png
    loader   /vmlinuz-linux
    initrd   /initramfs-linux.img
    options  "root=PARTUUID=***PARTUUID_OF_ROOT*** rw rootflags=subvol=@ initrd=amd-ucode.img rcu_nocbs=0-15 acpi_enforce_resources=lax nowatchdog nvidia-drm.modeset=1 amd_pstate=guided"
    submenuentry "Boot to terminal" {
        add_options "systemd.unit=multi-user.target"
    }
}
```

`nvim /etc/mkinitcpio.conf`

> binaries (btrfs)

```zsh
mkinitcpio -p linux &&
useradd -mg users -G wheel -s /bin/zsh jason &&
passwd jason &&
export EDITOR=nvim &&
visudo &&
systemctl enable NetworkManager bluetooth sddm rngd fstrim.timer updatedb.timer cups cronie avahi-daemon.service logrotate.timer paccache.timer &&
exit

umount -a &&
reboot
```

## Post install

```zsh
mkdir -p $HOME/.local/bin/git &&
git clone https://aur.archlinux.org/yay-bin.git $HOME/.local/bin/git/yay-bin &&
(cd $HOME/.local/bin/git/yay-bin && makepkg -si) &&
cd /remove/aur
```

`yay -S - <` #PackageLists from /remove/aur/

```zsh
sudo su

umount /.snapshots &&
rm -r /.snapshots &&
snapper -c root create-config / &&
snapper -c home create-config /home &&
btrfs su del /.snapshots &&
mkdir /.snapshots &&
mount -a &&
btrfs su list /

btrfs su set-default 256 / # Or whatever it is

nvim /etc/snapper/configs/root &&
nvim /etc/snapper/configs/home
```

> ALLOW_GROUPS="wheel"  
> NUMBER_LIMIT="10"  
> Change TIMELINE\*\*

`nvim /etc/updatedb.conf`

> PRUNENAMES = ".snapshots"

```zsh
nvim /etc/fstab &&
systemctl enable --now snapper-timeline.timer snapper-cleanup.timer &&
nvim /etc/systemd/system/timers.target.wants/snapper-cleanup.timer &&
snapper -c root create -d BASE &&
refind-btrfs &&
systemctl enable --now refind-btrfs
```

> OnUnitActiveSec=1h

```zsh
chown -R :wheel /.snapshots /home/.snapshots &&
rm -r /remove &&
exit
```

```zsh
mkdir -p $HOME/.local/bin/git/tkg &&
git clone --separate-git-dir="$HOME"/.local/bin/git/dotfiles https://github.com/jnines/dotfiles.git "$HOME"/.local/bin/git/dotf &&
git clone https://github.com/jnines/nvim $HOME/.config/nvim &&
git clone https://github.com/Frogging-Family/linux-tkg.git $HOME/.local/bin/git/tkg/linux-tkg &&
git clone https://github.com/Frogging-Family/nvidia-all.git $HOME/.local/bin/git/tkg/nvidia-all
```

### KDE

[Event Calendar](https://store.kde.org/p/998901/)  
[Fancy Tasks](https://store.kde.org/p/1928026/)
[Kargos](https://github.com/sanniou/kargos6)

### Extra fonts

[SF Pro font](https://github.com/sahibjotsaggu/San-Francisco-Pro-Fonts)  
[SauceCode Pro font](https://github.com/ryanoasis/nerd-fonts/blob/master/patched-fonts/SourceCodePro/Regular/complete/Sauce%20Code%20Pro%20Nerd%20Font%20Complete%20Mono%20Windows%20Compatible.ttf)
