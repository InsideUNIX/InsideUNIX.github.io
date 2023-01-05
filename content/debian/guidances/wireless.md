# How to install my driver?
Next steps were taken from [debian's documentation](https://wiki.debian.org/Wireless) in order to enable the network or bluetooth cards, this is problem has been too common due to Debian hasn't non-free repositories and firmware enabled by default. Verify wireless cards and follow next steps in order to activate it.

1. Use wire connection from usb (celphone) or ethernet.
2. Make sure you know your wifi/bluetooth chipset. If don't, [see this explanation](https://www.cyberciti.biz/faq/linux-list-network-cards-command/)

## iwlwifi
```
apt install firmware-iwlwifi
modprobe -r iwlwifi ; sudo modprobe iwlwifi
```

## Broadcom
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

## Realtek
``` md
# Install realtek's firmware
apt install firmware-realtek
```

## Troubleshooting
Your network card isn't in those package due to different versions were removed or it wasn't activate. Try restarting your system or search driver's name in GitHub or Gitlab repositories, in this cases the same repo could explain how to install and configure them. 

- [RTL8822ce-dkms](https://github.com/juanro49/rtl88x2ce-dkms)
- [RTL88](https://github.com/lwfinger/rtw88)
- [rtkbt (bluetooth driver)](https://github.com/radxa/rtkbt)
- [rtl8723au bt (bluetooth drivers)](https://github.com/lwfinger/rtl8723au_bt)