<!-- markdownlint-disable MD013 -->

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
lsblk -o name,fssize,fstype,mountpoint,uuid,partuuid,model
cfdisk
```

- EFI partition 500MB - 2GB
- / partition remaining

```zsh
lsblk -o name,fssize,fstype,mountpoint,uuid,partuuid,model
boot_part=**
root_part=**
mkfs.vfat -n BOOT /dev/${boot_part}
mkfs.ext4 -L ROOT /dev/${root_part}

mount /dev/${root_part} /mnt
mkdir /mnt/boot
mount /dev/${boot_part} /mnt/boot
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
sed -i \
-e '/\[Manager\]/aDefaultLimitNOFILE=524288\nDefaultTimeoutStopSec=20s' \
/etc/systemd/system.conf &&
sed -i \
-e '/\[Manager\]/aDefaultLimitNOFILE=524288' \
/etc/systemd/user.conf &&
passwd &&
pacman -Syy &&
cd /remove/arch || exit
```

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
    options  "root=PARTUUID=***PARTUUID_OF_ROOT*** rw rootflags=subvol=@ initrd=amd-ucode.img nowatchdog amd_pstate=guided split_lock_detect=off fbcon=font:VGA8x16"
    submenuentry "Boot to terminal" {
        add_options "systemd.unit=multi-user.target"
    }
}
```

`nvim /etc/mkinitcpio.conf`

```zsh
useradd -mg users -G wheel,users,rtkit,realtime,libvirt,gamemode,docker,input -s /bin/zsh jason &&
passwd jason &&
export EDITOR=nvim &&
visudo &&
systemctl enable NetworkManager bluetooth sddm rngd fstrim.timer plocate-updatedb.timer cups cronie avahi-daemon.service logrotate.timer paccache.timer &&
exit

umount -a &&
reboot
```

### Post install

```zsh
sudo dd if=/dev/zero of=/swapfile bs=1M count=8k status=progress &&
sudo chmod 600 /swapfile &&
sudo mkswap -U clear /swapfile &&
sudo swapon /swapfile &&
echo "/swapfile    none    swap    defaults,pri=2    0  0" | sudo tee -a /etc/fstab &&

cat <<EOF > /etc/systemd/zram-generator.conf
[zram0]
zram-size = min(ram / 4, 6144)
compression-algorithim = zstd
EOF

cat <<EOF > /etc/sysctl.d/99-sysctl.conf
kernel.sysrq = 1
fs.inotify.max_user_watches=524288
vm.swappiness=180
vm.watermark_boost_factor = 0
vm.watermark_scale_factor = 125
vm.page-cluster = 0
EOF

mkdir -p $HOME/.local/bin/git &&
git clone https://aur.archlinux.org/packages/paru-bin.git $HOME/.local/bin/git/paru-bin &&
(cd $HOME/.local/bin/git/paru-bin && makepkg -si) &&
cd /remove/aur
```

`paru -S - <` #PackageLists from /remove/aur/

```zsh
sudo rm -r /remove &&
```

```zsh
curl -sL https://raw.githubusercontent.com/jnines/dotfiles/refs/heads/main/install.sh | sh
```

### KDE

[Transparency Panel](https://github.com/TheEssem/paneltransparencybutton)
[BreezeX Cursors](https://github.com/ful1e5/BreezeX_Cursor)
[Rmuxinator](https://github.com/ethagnawl/rmuxinator)

### Extra fonts

[SF Pro font](https://github.com/sahibjotsaggu/San-Francisco-Pro-Fonts)
