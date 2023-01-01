# Flatpak
Flatpak is a utility for software deployment and package management which is advertised as offering a sandbox environment in which users can run application software in isolation from the rest of the system.

> If you use Gnome Software, I totally recommend to install the plugin to show apps from FlatHub.

## Installation

```sh
# Install flatpak and portal dependencies.
apt install flatpak 
apt install flatpak-xdg-utils
apt install xdg-dbus-proxy 
apt install libportal-gtk4-1
apt install gir1.2-flatpak-1.0

# Gnome Softwar Plugin (Optional)
apt install gnome-software-plugin-flatpak
```

### Stable Repositories
```sh
# Stable
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
flatpak update --appstream
```

### Beta Repositories
> This only install apps per user.
``` sh
# Beta
flatpak remote-add --user flathub-beta https://flathub.org/beta-repo/flathub-beta.flatpakrepo
flatpak update --appstream
```

## Permissions

**Flatseal** is a graphical utility to review and modify permissions your Flatpak applications have got.

```sh
# I really recommend this utility for all flatpak installation
flatpak install flathub com.github.tchx84.Flatseal
```

## Themes

### Adwaita Dark Theme
Gnome & Plasma have a dark mode which change apps appearance based on dark schemes, you must install this package in order to add dark mode compability.
```sh
flatpak install org.gtk.Gtk3theme.Adwaita-dark
```

### Override System themes in flatpak

You must have custom themes in your system, the next command show you how to enable them in some Flatpak's Apps.

> Optional
```sh
# Also you could use "--filesystem=/usr/share/themes" if your theme is that path
sudo flatpak override --filesystem=~/.themes
```

## Runtimes

Runtime describes software/instructions that are executed while your program is running, especially those instructions that you did not write explicitly, but are necessary for the proper execution of your code.

### Freedesktop
```
flatpak install org.freedesktop.Platform
```

### Gnome Runtimes
```
flatpak install org.gnome.Platform
```
### KDE Runtimes

```
flatpak install org.kde.Platform
```

***
<p style="text-align: center;">You finished it. Enjoy tour system ;)</p>