---

title: Manual Arch install guide
description: Step by step guide on installing Artix with dinit, an arch-based, systemd-free distro, full manual install
tags: [Linux, Arch]
keyword:
    - linux
    - arch
    - artix
    - dinit
    - systemd
    - install
    - wayland
    - guide
slug: manual-arch-install-artix

---


## Part I - Partitions

### A. Booting

1. Choose an initsystem :
	1. OpenRC --> old + lots of documentation
	2. Runit --> newer with good docs (Théo uses it)
	3. Dinit --> newer than runit, still not documented enough (I use it)
	4. S6 --> idk
2. Download the base ISO
3. If you use a VM, enable EFI
4. **If you have trouble booting the ISO, install Ventoy and boot in grub2 mode**
5. Start the computer
6. Configure Artix with the correct locale/language/keyboard layout
7. Enter the edit console
	1. Type `video=1980x1080` to setup an appropriate resolution in the VM
8. Boot into artix
	1. user `artix`
	2. passwd `artix`
	3. `sudo su`

### B. Creating partitions

1. If keyboard isn't set correctly `loadkeys fr`
2. Download & set the font
	1. `pacman -Sy terminus-font`
	2. Set **big** font `setfont ter-128n`
	3. (set less bigger font `setfont ter-d24b`)
3. Setup partitions using `cfdisk` (`gpt` file system)
	1. First must be EFI of size between 100M and 512M
	2. Then root partition
	3. Then home partition (optional)
	4. Then swap partition (optional if no hibernation)
4. See created partitions using `lsblk`
5. Format the partitions (-L to set a label)
	1. EFI `mkfs.fat -F32 /dev/sda1` and `fatlabel /dev/sda1 ESP`
	2. Root `mkfs.ext4 -L ROOT /dev/sda2`
	2. Home `mkfs.ext4  -L HOME /dev/sda3`
	3. Swap `mkswap -L SWAP /dev/sda4`
6. Mount the partitions (swap first, then root, finally boot and home)
	1. `swapon /dev/disk/by-label/SWAP` OR `swapon /dev/sda4`
	2. `mount /dev/sda2 /mnt`
	3. `mkdir -p mnt/boot/efi` (if the fs is encrypted, do not create `/efi`)
	4. `mount /dev/sda1 /mnt/boot/efi`
	5. `mkdir -p mnt/home`
	4. `mount /dev/sda3 /mnt/home`

> [!WARNING]
> The order matters!!

6. Check with `lsblk`
7. Sync the system clock `dinitctl start ntpd`
8. Install the system with interactive mode (-i) `basestrap -i /mnt base base-devel linux-zen linux-zen-headers linux-firmware dinit btop vim neovim man-db elogind seatd seatd-dinit intel-ucode grub os-prober efibootmgr networkmanager networkmanager-dinit bash-completion fastfetch` (approx 280 packages)
9. Setup fstab `fstabgen -U /mnt >> /mnt/etc/fstab`
10. Check generated fstab `cat /mnt/etc/fstab`

## Part II - chroot

### A. Various configuration

1. chroot inside the installed system `artix-chroot /mnt`
2. Update repos with `pacman -Sy`
3. Setup timezone `ln -sf /usr/share/zoneinfo/Region/City /etc/localtime`
	1. For instance, `ln -sf /usr/share/zoneinfo/Europe/Paris /etc/localtime`
4. Setup hwclock `hwclock --systohc`
5. Uncomment the locale you want to keep in `nvim /etc/locale.gen` (recommended US UTF-8)
6. Generate the locale `locale-gen`
7. Edit `locale.conf` with `nvim /etc/locale.conf`
	1. Enter `LANG=en_US.UTF-8`
8. Network config
	1. `nvim /etc/hostname` and enter your host name (aka computer name)
	2. `nvim /etc/hosts`
		1. `127.0.0.1    localhost`
		2. `::1          localhost`
		3. `127.0.1.1    HOSTNAME.localdomain    HOSTNAME`
9. Install terminus font (in the installed env) `pacman -S terminus-font`
10. Configure console
	1. `nvim /etc/vconsole.conf`
	2. Enter `FONT=ter-d24b`
	3. Enter `KEYMAP=fr`
11. Setup users and passwords
	1. Setup current session (root) password `passwd`
	2. Add user `useradd -mG wheel user`
	3. Setup user password `passwd user`
	4. Make user a sudoer `EDITOR=nvim visudo`
		1. Uncomment `%wheel ALL=(ALL:ALL) ALL`
12. Setup autologin
	1. `nvim /etc/dinit.d/tty1`
	2. Add flag `--autologin user` before `tty1` in the `command=` line

### B. Grub

1. Setup the correct resolution `nvim /etc/default/grub` and add `video="1920x1080"` at the end of line `GRUB_CMDLINE_LINUX_DEFAULT="loglevel=3 quiet"`
2. Edit `GRUB_GFXMODE=1920x1080` as well
3. Uncomment `GRUB_DISABLE_OS_PROBER=false` to detect other installed OS (for windows, see [the arch wiki: grub](https://wiki.archlinux.org/title/GRUB#GUID_Partition_Table_%28GPT%29_specific_instructions))
4. Mount the other OS in read-only (to detect other installed OS)
	1. `mkdir -p /mnt/foo`
	2. `mount -o ro /dev/sda5 /mnt/foo`
	3. If the other installed Linux enters rescue mode at launch, update its `/etc/fstab` file to remove the line of the old EFI partition UUID
5. Install for UEFI `grub-install --target=x86_64-efi --efi-directory=/boot/efi --bootloader-id=GRUB`
6. `grub-mkconfig -o /boot/grub/grub.cfg`
7. [You can add custom grub entries](https://wiki.archlinux.org/title/GRUB#Boot_menu_entry_examples) in `/boot/grub/custom.cfg` (for instance, shutdown or reboot)
8. Reboot the system
	1. Exit chroot `exit`
	2. Unmount all partitions `umount -R /mnt`
	3. `reboot`

## Part III - UI

> [!WARNING]
> Now, you may need to add sudo in front of most commands since you are no longer chroot'ed

### Last details

1. Enable network connection
	1. `dinitctl enable NetworkManager` (be cautious of the uppercase letters)
2. Enable seatd
	1. `dinitctl enable seatd`
	2. Add user to group seat to allow them to start processes `usermod -aG seat <user>`
	2. Add user to group video (and group audio) `usermod -aG video <user>`
4. Enable ssh for remote control (if needed)

### A. Additional Arch repos & yay

1. Enable Arch repos
	1. `sudo pacman -Sy artix-archlinux-support`
	2. `sudo pacman-key --populate archlinux`
	3. `sudo pacman -S archlinux-mirrorlist`
	4. `sudo nvim /etc/pacman.conf`
		 `# Arch`
		 `[extra]`
		 `Include = /etc/pacman.d/mirrorlist-arch`
		 `[multilib]`
		 `Include = /etc/pacman.d/mirrorlist-arch`

> [!NOTE]
> All *Universe* packages have been moved to *Galaxy* according to the [Artix wiki](https://wiki.artixlinux.org/Main/Repositories#Arch_repositories)

	5. sudo pacman -Sy
2. Install yay
	1. Setup dependencies `pacman -S --needed git base-devel`
	2. Clone the git `git clone https://aur.archlinux.org/yay-bin.git`
	3. `cd yay-bin`
	4. Make `makepkg -si`
	5. (Next steps are not mandatory, works without)
	6. Scan for packages installed without yay `yay -Y --gendb`
	7. Check for installed package updates `yay -Syu --devel`
	8. Permanently check for dev updates `yay -Y --devel --save`
	9. Remove the unnecessary folder `cd ..` and `rm -rf yay-bin/`

### B. Installing useful stuff

> [!TIP]
> [Awesome Hyprland](https://github.com/hyprland-community/awesome-hyprland)
> [Awesome Wayland](https://github.com/rcalixte/awesome-wayland)
> [Awesome wlroots](https://github.com/solarkraft/awesome-wlroots)
> [Hyprland Wiki - Must have](https://wiki.hyprland.org/Useful-Utilities/Must-have/)

1. Install fastfetch `pacman -S fastfetch`
	1. fastfetch
	2. hyfetch
2. Install Hyprland `yay -S hyprland`
	1. Hyprland (dynamic tiling WM) [Hyprland wiki - Master tutorial](https://wiki.hyprland.org/Getting-Started/Master-Tutorial/)
	2. Sway (manual tiling WM)
3. Install a file manager (GUI) `pacman -S thunar`
	1. Dolphin (KDE)
	2. Thunar (XFCE)
	3. Nemo (Mint)
	4. Nautilus (GNOME)
	5. PCManFM (LXDE)
	6. + plugins for all of them (if needed)
4. Install a file manager (CLI, if needed) `pacman -S nnn`
	1. nnn
	2. Ranger
	3. vifm
5. Install a launcher ([see](https://mark.stosberg.com/fuzzel-a-great-dmenu-and-rofi-altenrative-for-wayland/)) `pacman -S rofi-wayland`
	1. Rofi-wayland
	2. Wofi
	3. Fuzzel
	4. bemenu
	5. dmenu-wl
	6. yofi
6. Install a wallpaper `yay -S hyprpaper`
7. Install a terminal emulator `pacman -S kitty`
	1. Kitty
	2. Alacritty
	3. Foot
8. Install fish `pacman -S fish`
	1. Edit hyprland's config to launch kitty terminal emulator by default
	2. Edit kitty's config to launch on fish by default
9. Install a clipboard manager `pacman -S cliphist wl-clipboard wl-clip-persist`
	1. 
10. Install Flatpak for even more apps `pacman -S flatpak` (Proton-QT is packed inside)

> [!WARNING]
> Flatpak requires a working xdg-portal, install `xdg-desktop-portal` ; `xdg-desktop-portal-hyprland` needs to be installed as well.

### C. Configuring Hyprland

> [!WARNING]
> **OUTDATED SECTION**

1. Create a startup script (to set up additional env variables, can be set up in /etc/environment or in your .profile or .bash_profile)
	1. `nvim starthypr.sh`
	2. `#!/bin/sh`
	3. `export WLR_NO_HARDWARE_CURSORS=1`
	4. `export WLR_RENDERER_ALLOW_SOFTWARE=1`
	5. `export XDG_SESSION_DESKTOP=Hyprland`
	6. `export XDG_CURRENT_DESKTOP=Hyprland
	7. `export XDG_SESSION_TYPE=wayland`
	8. `export HYPRLAND_LOG_WLR=1`
	9. `export LIBGL_ALWAYS_SOFTWARE=1`
	10. `export LIBSEAT_BACKEND=seatd`
	11. `exec dbus-run-session Hyprland`
	12. `chmod +x starthypr.sh`
2. Configure it or download dotfiles/scripts/rices

## Part IV - Tips & tricks

- Edit .desktop files of installed apps that doesn't start and edit `Exec = dbus-run-session program` instead of `Exec = program` (after having copied them to `~/.local/share/applications`)
- AppImage can be detected and started in rofi if you create a .desktop entry for them in `~/.local/share/applications`

## References

- Artix full install tutorial with dinit: https://youtu.be/zfHkmhEamlc
- Artix full install tutorial with runit: https://youtu.be/mIpZA6z-Ctk
- Artix install blog: https://cheatsheets.stephane.plus/distros/arch-based/artix_installation/
- Artix install wiki: https://wiki.artixlinux.org/Main/Installation
- Arch wiki, grub config: https://wiki.archlinux.org/title/GRUB
- Hyprland full install tutorial: https://youtu.be/V7nP-To0630
- Awesome Wayland: https://github.com/rcalixte/awesome-wayland
- Awesome wlroots: https://github.com/solarkraft/awesome-wlroots
- Hyprland dots: https://github.com/prasanthrangan/hyprdots && https://github.com/end-4/dots-hyprland && https://github.com/linkfrg/dotfiles/wiki/Installation
