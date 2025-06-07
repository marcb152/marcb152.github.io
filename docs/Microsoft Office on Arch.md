---

title: Installing Office on Arch
description: Step by step guide on installing Office 2013/2016 on Arch-based distros
tags: [Linux]
keyword:
    - linux
    - office
    - word
    - excel
    - powerpoint
    - arch
    - wayland
slug: office-install-arch-linux

---

# Installing Office on Arch Linux

This short and concise guide lists all the required commands to install all the Office suite (version 2013 and 2016 supported) on Arch Linux (or any other Arch-based distro).
Works under Wayland and using Window Managers (tested using Hyprland on Arch, Artix and CachyOS).

> [!IMPORTANT]
> [Read the docs (RTFM)](https://wiki.archlinux.org/title/Wine)

## I - Install required graphic drivers

### On NVIDIA

Check the correct and working latest driver version or install the `nouveau` drivers instead.

`sudo pacman -S nvidia-utils lib32-nvidia-utils` 

### NVIDIA & all others

`sudo pacman -S mesa mesa-utils lib32-mesa lib32-mesa-utils`

## II - Install wine

`sudo pacman -S wine wine-gecko wine-mono`

`yay -S winetricks`

## III - Install dependencies

> [!NOTE]
> Samba contains the required `winbind` package.

`sudo pacman -S zenity cabextract samba gnutls lib32-gnutls lib32-libxinerama lib32-libxcomposite lib32-v4l-utils lib32-ncurses lib32-lcms`

## IV - Install fonts

`yay -S ttf-ms-fonts` OR `yay -S ttf-ms-win11-auto`

## V - Setup the wineprefix

`WINEPREFIX=~/.wine/office2013 WINEARCH=win32 winecfg`

Select Windows 7 and quit.

Install packages (not all of them are needed, at least `msxml6 riched20` should be installed, then `riched30 richtx32 vb6run gdiplus` if problems arise).

If you are lazy like me, *install them all*™:

`WINEPREFIX=~/.wine/office2013 WINEARCH=win32 winetricks msxml3 msxml6 riched20 riched30 richtx32 corefonts tahoma msftedit vb6run gdiplus`

## VI - Run the installer

`WINEPREFIX=~/.wine/office2013 WINEARCH=win32 wine /path/to/msoffice/folder/setup.exe`

## VII - Regedit

Run regedit (using winecfg, bash or winetricks) in the wine prefix.

Create a new DWORD `HKCU/Software/Wine/Direct3D/MaxVersionGL=0x30002`

Create a new DWORD `HKCU/Software/Wine/Direct2D/max_version_factory=0`

## VIII - Office 2016 & 2019 specifics

Depending on your installer, two scenarii may happen:
- Your installer enters in a infinite loop at around 75% completion (see the console logs) "finalizing stuff..." --> **kill** it (**do NOT cancel** it)
- Your installer works:
	After your installation, copy the **AppvIsvSubsystems32.dll** and the **C2R32.dll**, from `/Program Files/Common Files/Microsoft Shared/ClickToRun/` to `/Program Files/Microsoft Office/root/Office16/`

## References

- [Old arch wiki entry for Office 2007](https://wiki.archlinux.org/index.php?title=Microsoft_Office_2007&oldid=486936)
- [Latest entry for Office 2016 on WineHQ DB](https://appdb.winehq.org/objectManager.php?sClass=version&iId=34941)
- [Latest entry for Word 2016 on WineHQ DB](https://appdb.winehq.org/objectManager.php?sClass=version&iId=34964)
- [Arch entry for Office 2019 on WineHQ DB](https://appdb.winehq.org/objectManager.php?sClass=version&iId=37735&iTestingId=108623)
