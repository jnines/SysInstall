# EXT4

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
- / partition remaining

```zsh
mkfs.vfat -n BOOT /dev/***?1
mkfs.ext4 -L ROOT /dev/***?2

mount /dev/***?2 /mnt
mkdir /mnt/boot
mount /dev/***?1 /boot
```

### Pacstrap

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
echo 'archt' > /etc/hostname &&
echo "127.0.0.1   localhost
::1         localhost
127.0.1.1   archbox.lan archbox" >> /etc/hosts &&
passwd &&
nvim /etc/pacman.conf &&
pacman -Syy &&
cd /remove/arch || exit
```

- Change in pacman.conf
  > NoExtract=usr/lib/security/pam_systemd_home.so
- Uncomment
  > Color
  > CheckSpace
  > VerbosePkgLists
  > ParallelDownloads
  > ILoveCandy
  > [multilib]
  > Include = /etc/pacman.d/mirrorlist

`pacman -S --needed - <`
PackageLists from /remove/arch/

### Refind

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
    options  "root=PARTUUID=***PARTUUID_OF_ROOT*** rw initrd=amd-ucode.img rcu_nocbs=0-15 acpi_enforce_resources=lax nowatchdog nvidia-drm.modeset=1 amd_pstate=guided"
    submenuentry "Boot to terminal" {
        add_options "systemd.unit=multi-user.target"
    }
}
```

```zsh
useradd -mg users -G wheel -s /bin/zsh jason &&
passwd jason &&
export EDITOR=nvim &&
visudo &&
systemctl enable NetworkManager bluetooth sddm rngd fstrim.timer updatedb.timer cups cronie avahi-daemon.service logrotate.timer paccache.timer &&
exit

umount -a &&
reboot
```

### Post install

```zsh
mkdir -p $HOME/.local/bin/git &&
git clone https://aur.archlinux.org/yay-bin.git $HOME/.local/bin/git/yay-bin &&
(cd $HOME/.local/bin/git/yay-bin && makepkg -si) &&
cd /remove/aur
```

`yay -S - <` #PackageLists from /remove/aur/

```zsh
sudo rm -r /remove &&
```

```zsh
mkdir -p $HOME/.local/bin/git/tkg &&
git clone --separate-git-dir="$HOME"/.local/bin/git/dotfiles https://github.com/jnines/dotfiles.git "$HOME"/.local/bin/git/dotf &&
git clone https://github.com/jnines/nvim $HOME/.config/nvim &&
git clone https://github.com/Frogging-Family/linux-tkg.git $HOME/.local/bin/git/tkg/linux-tkg &&
git clone https://github.com/Frogging-Family/nvidia-all.git $HOME/.local/bin/git/tkg/nvidia-all
```

[Wine](https://github.com/Frogging-Family/wine-tkg-git/actions/workflows/wine-arch.yml)  
[SF Pro font](https://github.com/sahibjotsaggu/San-Francisco-Pro-Fonts)  
[SauceCode Pro font](https://github.com/ryanoasis/nerd-fonts/blob/master/patched-fonts/SourceCodePro/Regular/complete/Sauce%20Code%20Pro%20Nerd%20Font%20Complete%20Mono%20Windows%20Compatible.ttf)
