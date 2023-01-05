# Gnome
In this guide you will learn to manually install [Gnome Desktop Enviroment](https://www.gnome.org/) to take control in package's number in order to install what you need.

> This was thought to be used in **Debian Unstable**

## Configure APT

Enable Sid's repositories in `/etc/apt/sources.list`
> nano /etc/apt/sources.list

``` md
# You may have something like this
deb http://deb.debian.org/debian/ unstable main contrib non-free
#deb-src http://deb.debian.org/debian/ unstable main contrib non-free
```

Create a file in `etc/apt/apt.conf.d/` and name `99local`, then add the lines bellow:

> nano /etc/apt/apt.conf.d/99local

``` md
# You could use any other text editor.
APT::Install-Suggests "0";
APT::Install-Recommends "0";
```

## Network Manager
``` md
# Install Network Manager to  start using wifi
apt install network-manager network-manager-gnome

# Restart system service
systemctl restart NetworkManager.service
```

### Enable Interface Management

Modify `/etc/NetworkManager/NetworkManager.conf`

``` md
# Set true in:
managed=false

# Then restart the system service
systemctl NetworkManager restart
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
# Filemanager
apt install nautilus

# Showcase thumbnails
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