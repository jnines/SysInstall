#### Format

```
lsblk -o name,fssize,fstype,mountpoint,uuid,model
cfdisk
```

- ==EFI partition 500MB - 2GB==
- ==/ partition remaining==

```
mkfs.vfat -n BOOT /dev/***?1
mkfs.ext4 -L ROOT /dev/***?2

mount /dev/***?2 /mnt
mkdir /mnt/boot
mount /dev/***?1 /boot
```

#### Pacstrap

```
pacstrap /mnt archlinux-keyring base base-devel neovim rsync openssh reflector git
&&
genfstab -U /mnt >> /mnt/etc/fstab
&&
cat /mnt/etc/fstab
&&
mkdir -p /mnt/remove
&&
git clone https://github.com/jnines/installs.git /mnt/remove
&&
arch-chroot /mnt
```

```
ln -sf /usr/share/zoneinfo/America/Chigaco /etc/localtime
&&
hwclock --systohc
&&
reflector --country US --latest 5 --sort rate --save /etc/pacman.d/mirrorlist
&&
echo 'en_US.UTF-8 UTF-8' >> /etc/locale.gen
&&
locale-gen
&&
echo 'LANG=en_US.UTF-8' > /etc/locale.conf
&&
echo 'archbox' > /etc/hostname
&&
echo "127.0.0.1   localhost\n::1         localhost\n127.0.1.1   archbox.lan archbox" >> /etc/hosts
&&
passwd
&&
nvim /etc/pacman.conf
```

- ==Change==
  > NoExtract=usr/lib/security/pam_systemd_home.so
- ==Uncomment==
  > Color
  > CheckSpace
  > VerbosePkgLists
  > ParallelDownloads
  > ILoveCandy
  > [multilib]
  > Include = /etc/pacman.d/mirrorlist

```
pacman -Syy
&&
cd /remove/arch || exit
```

`pacman -S --needed - < ` #PackageLists from /remove/arch/

### Refind

```
refind-install
blkid ROOT
nvim /boot/EFI/refind/refind.conf
```

> timeout 3
> resolution 2560 1440
> use_graphics_for linux
> scanfor manual
> dont_scan_volumes "Recovery HD",Data,Win,Home,Root,1TB,"Microsoft reserved partition"
> fold_linux_kernels false
> default_selection "TKG"

```
menuentry "Arch" {
    volume   "Arch Linux"
    icon     /EFI/refind/icons/os_arch.png
    loader   /vmlinuz-linux
    initrd   /initramfs-linux.img
    options  "root=PARTUUID=***PARTUUID_OF_ROOT*** rw nouveau.modeset=0 processor.max_cstate=5 initrd=amd-ucode.img"
    submenuentry "Boot to terminal" {
        add_options "systemd.unit=multi-user.target"
    }
}
```

```
useradd -mg users -G wheel -s /bin/zsh jason
&&
passwd jason
&&
export EDITOR=nvim
&&
visudo #(uncomment wheel)
&&
systemctl enable NetworkManager bluetooth sddm rngd fstrim.timer updatedb.timer cups cronie avahi-daemon.service
&&
exit
&&
umount -a
&&
reboot
```

#### Post install

```
mkdir $HOME/.local/bin/git
&&
git clone https://aur.archlinux.org/yay-bin.git $HOME/.local/bin/git/
&&
(cd $HOME/.local/bin/git/yay-bin && makepkg -si)
&&
cd /remove/aur
```

`yay -S - < ` #PackageLists from /remove/aur/

```
rm -r /remove
&&
exit
```

```
mkdir -p $HOME/.local/bin/git/tkg
&&
git clone --separate-git-dir="$HOME"/.local/bin/git/dotfiles https://github.com/jnines/dotfiles.git "$HOME"/.local/bin/git/dotf
&&
git clone https://github.com/Frogging-Family/linux-tkg.git $HOME/.local/bin/git/tkg/
&&
git clone https://github.com/Frogging-Family/nvidia-all.git $HOME/.local/bin/git/tkg/
```

https://github.com/Frogging-Family/wine-tkg-git/actions/workflows/wine-arch.yml
https://github.com/sahibjotsaggu/San-Francisco-Pro-Fonts
https://github.com/ryanoasis/nerd-fonts/blob/master/patched-fonts/SourceCodePro/Regular/complete/Sauce%20Code%20Pro%20Nerd%20Font%20Complete%20Mono%20Windows%20Compatible.ttf
https://www.lunarvim.org/docs/installation
