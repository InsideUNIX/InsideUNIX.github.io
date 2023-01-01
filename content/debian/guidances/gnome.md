# Gnome
In this guide you will learn to manually install [Gnome Desktop Enviroment](https://www.gnome.org/) to take control in package's number in order to install what you need.

> This was thought to be used in **Debian Unstable**

## Configure APT

Enable Sid's repositories in `/etc/apt/sources.list`
> nano /etc/apt/sources.list

``` md
# You may have something like this
deb http://deb.debian.org/debian/ unstable main contrib non-free
deb-src http://deb.debian.org/debian/ unstable main contrib non-free
```

Create a file in `etc/apt/apt.conf.d/` and name `99local`, then add the lines bellow:

> nano /etc/apt/apt.conf.d/99local

``` md
# You could use any other text editor.
APT::Install-Suggests "0";
APT::Install-Recommends "0";
```

## Dependencies
``` md
# Install xserver. Only execute one of this commands depending if you use amd or intel.
apt install xserver-xorg-video-amdgpu
apt install xserver-xorg-video-intel

# Install the next dependencies in order to use the keyboard
# and adding home's folders per user.
apt install xserver-xorg-input-libinput
apt install xdg-user-dirs
apt insatll xdg-utils
```

### Desktop Portal
> This enable screenmdaring/screenrecording in Wayland Session (also configure the the app that needs it).
```md
# Install next dependencies to screenmdaring and enable utilities
apt install xdg-desktop-portal
apt install xdg-desktop-portal-gnome
apt install xdg-desktop-portal-gtk
```
## Desktop
``` md
# gnome-session may install the the session with the mdell
# the most minimal way possible.
apt install gnome-session
apt install fonts-cantarell

# Gnome Control center & adwaita dark theme
apt install gnome-control-center 
apt install gnome-themes-extra
```

## Gnome Keyring
```md
# GNOME Keyring is "a collection of components in GNOME
# that store secrets, passwords, keys, certificates.
apt install gnome-keyring 
apt install libsecret-1-0 
apt install libpam-gnome-keyring
apt install policykit-1-gnome

# Optional 
apt install gnome-keyring-pkcs11
```

## Network Manager
``` md
# Install Network Manager to  start using wifi
apt install network-manager network-manager-gnome

# Restart system service
systemctl restart NetworkManager.service
```

### Wifi Driver
> Make sure you have enable nonfree.

#### iwlwifi
> Make sure you installed the correct drive, in case you didn't read [documentation](https://wiki.debian.org/iwlwifi) or search your driver in GitHub/GitLab and compile it.
```
sudo apt install firmware-iwlwifi
sudo modprobe -r iwlwifi ; sudo modprobe iwlwifi
sudo systemctl restart NetworkManager.service
```

#### Broadcom wl
``` md
# Install dkms driver
apt install linux-image-$(uname -r|sed 's,[^-]*-[^-]*-,,') linux-headers-$(uname -r|sed 's,[^-]*-[^-]*-,,') broadcom-sta-dkms
apt install -f
dpkg-reconfigure broadcom-sta-dkms

# Unload conflict  modules
sudo modprobe -r b44 b43 b43legacy ssb brcmsmac bcma

# Load the module
modprobed wl
```

#### Realtek
> There are many Realtek driver, if your firmware isn't in that package, search it on the 
web and comile it

**Examples:**
- [RTL8822ce-dkms](https://github.com/juanro49/rtl88x2ce-dkms)
- [RTL88](https://github.com/lwfinger/rtw88)
- [rtkbt (bluetooth driver)](https://github.com/radxa/rtkbt)
- [rtl8723au bt (bluetooth drivers)](https://github.com/lwfinger/rtl8723au_bt)

``` md
# Install realtek's firmware
apt install firmware-realtek
```

### Enable Interface Management

Modify `/etc/NetworkManager/NetworkManager.conf`
> nano /etc/NetworkManager/NetworkManager.conf
``` md
# Set true in:
managed=false

# Then restart the system service
systemctl NetworkManager restart
```
## Bluetooth
```md
#  Dependencies
apt install bluetooth
apt install gnome-bluetooth
apt install blueman
apt install bluez

 # Pipewire's library - bluetooth plugins
apt install libspa-0.2-bluetooth
apt install pipewire-audio-client-libraries

# Cups' dependency
apt install bluez-cups
```

> Enable & restart bluetooth service
```md
# Use sudo/doas
systemctl enable bluetooth
systemctl start bluetooth
```

## CUPS

```
sudo apt install cups system-config-printer printer-driver-cups-pdf
sudo systemctl enable cups.service --now
```

> Por this is necessary to investigate about your printe model, in this case I use an Epson L575 so for this kind of printers you have to:

### All Printer Driver
```
sudo apt install printer-driver-all
```

### Epson
```
sudo apt install printer-driver-escpr
```

## Programs
The next packages is a "basic kit" for start using the Desktop Enviroment

### Browser
``` md
# 'apt install firefox-esr' if you use Debian Bullseye
apt install firefox
```
### Terminal
``` md
apt install gnome-terminal
```
### Filemanager
``` md
# This install nautilus and images thumbnail.
apt install nautilus 
apt gnome-sumdi 
apt libgdk-pixbuf2.0-bin 
apt libpixman-1-0
```

## Display Manager
``` md
# For a complete experience install GDM3
# in other cases, if you don't want to; install whatever display manager you like.
apt install --no-install-recommends gdm3

# Start the service
sudo systemctl enable gdm3
```

****

<p style="text-align: center;">You finimded it. Enjoy tour system ;)</p>