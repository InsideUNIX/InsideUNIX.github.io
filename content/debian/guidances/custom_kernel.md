# Building a custom Kernel
The kernel is the core of the operating system. Containing most of the device drivers, the kernel offers interfaces for programs to access system hardware such as memory, graphic cards, and block devices.
> This guide will show you how to build a custom kernel for `Debian` and `Debian-based` distributions.
## Dependencies

```sh
$ sudo apt install bc \
             git \
            make \
            kmod \
            cpio \
            flex \
           bison \
           rsync \
           meson \
         dwarves \
        flex-doc \
      libelf-dev \
      libssl-dev \
     ncurses-doc \
 build-essential \
 libncurses5-dev \
 ca-certificates
```

### Clang & LLVM
Clang is a compiler front end for the C, C++, Objective-C, and Objective-C++ programming languages... It acts as a drop-in replacement for the GNU Compiler Collection (GCC), supporting most of its compilation flags and unofficial language extensions.
```sh
sudo apt install lld \
          llvm \
         clang \
   clang-tools
```
## Download
Before start, download the source code's kernel you want to compile. There are many linux kernel focused with different porpuse, check the one you want to install and get the code.

- [Vanilla](https://www.kernel.org/)
- [LTS](https://www.kernel.org/)
- [Zen](https://github.com/zen-kernel/zen-kernel)
- [Hardened](https://github.com/anthraxx/linux-hardened)
- [Xanmod](https://xanmod.org/)
- [Liquorix](https://aur.archlinux.org/linux-lqx.git)
- [Linux TKG](https://github.com/Frogging-Family/linux-tkg)
- [Vanilla Release Candidate](https://github.com/torvalds/linux)

## Configuring
> Before start, go to your kernel path with `cd` command. Then execute `make clean && make mrproper` in order to prepare the kernel, also use the command again if something failed. 

<!-- tabs:start -->

#### **Generic**
This will use a generic config from other kernel that you have in your system.
```md
# Check files in /boot. It will show many files, but we're going to pay attention to `config-(numbers)` file
$ cd /boot & ls

config-5.15.72-xanmod1  initrd.img-5.15.72-xanmod1  vmlinuz-5.15.72-xanmod1
config-6.0.0-rc7+       initrd.img-6.0.0-rc7+       vmlinuz-6.0.0-rc7+

# Copy-paste the file to your kernel path. Example:
/boot $ config-6.0.0-rc7+ ~/linux/.config
```

#### **Manually**
This will show a interface in your terminal wich you could choose modules you want to enable.
```sh
$ make menuconfig
```

#### **modprobed-db**
[modprobed-db](https://github.com/graysky2/modprobed-db) is a tool that save the modules in a data base that you're using in the moment, this helps to compile a kernel with minimum number of modules in order to apply to your hardware. **Make sure you already installed it and have used the tool with all modules you'll use.**
> Warning. Using this tool may disable modules that you will want to use in a future. Example: you were not using your webcam and compile your custom kernel; then you want to use your webcam and it doesn't work.
```sh
$ make LSMOD=$HOME/.config/modprobed.db localmodconfig
```

<!-- tabs:end -->

## Compiling
In this guide we're going to create a `.deb` package for an easy install and remove. This section will take some time depending with your hardware and number of modules you used.

> In -j`` could change `nproc` with core's number you want to use.

### GCC

```sh
nice make -j`nproc` bindeb-pkg
```

### Clang
```md
nice make LLVM=1 -j`nproc` bindeb-pkg
```

## Installing

<!-- tabs:start -->

## **Manually**
```md
$ sudo dpkg -i <package name>.deb
```

## **All packages**
> $ sudo apt install ~/path/to/files/*.deb
```sh
$ sudo apt install ~/*.deb
```

<!-- tabs:end -->

## DKMS

DKMS will fail after you install a kernel using clang. Add `LLVM=1`to dkms.conf to your module (/usr/src/module_name/dkms.conf)

```
$ cd /usr/src/*module_name

$ cat dkms.conf
PACKAGE_NAME="broadcom-sta"
PACKAGE_VERSION="6.30.223.271"
MAKE[0]="make KVER=$kernelver" # <- You have to add it here.
BUILT_MODULE_NAME[0]="wl"
DEST_MODULE_LOCATION[0]="/updates/dkms"
AUTOINSTALL="YES"
REMAKE_INITRD="YES"
```
## Security Boot

### Signing a custom kernel for Secure Boot

Instructions are for ubuntu, but should work similar for other distros, if they are using shim
and grub as bootloader. If your distro is not using shim (e.g. Linux Foundation Preloader), there
should be similar steps to complete the signing (e.g. HashTool instead of MokUtil for LF Preloader)
or you can install shim to use instead. The ubuntu package for shim is called `shim-signed`, but
please inform yourself on how to install it correctly, so you do not mess up your bootloader.

Since the most recent GRUB2 update (2.02+dfsg1-5ubuntu1) in Ubuntu, GRUB2 does not load unsigned
kernels anymore, as long as Secure Boot is enabled. Users of Ubuntu 18.04 will be notified during
upgrade of the grub-efi package, that this kernel is not signed and the upgrade will abort.

Thus you have three options to solve this problem:

1. You sign the kernel yourself.
2. You use a signed, generic kernel of your distro.
3. You disable Secure Boot.

Since option two and three are not really viable, these are the steps to sign the kernel yourself.

Instructions adapted from [the Ubuntu Blog](https://blog.ubuntu.com/2017/08/11/how-to-sign-things-for-secure-boot).
Before following, please backup your /boot/EFI directory, so you can restore everything. Follow
these steps on your own risk.

1. Create the config to create the signing key, save as mokconfig.cnf:

```
# This definition stops the following lines failing if HOME isn't
# defined.
HOME                    = .
RANDFILE                = $ENV::HOME/.rnd 
[ req ]
distinguished_name      = req_distinguished_name
x509_extensions         = v3
string_mask             = utf8only
prompt                  = no
[ req_distinguished_name ]
countryName             = <YOURcountrycode>
stateOrProvinceName     = <YOURstate>
localityName            = <YOURcity>
0.organizationName      = <YOURorganization>
commonName              = Secure Boot Signing Key
emailAddress            = <YOURemail>
[ v3 ]
subjectKeyIdentifier    = hash
authorityKeyIdentifier  = keyid:always,issuer
basicConstraints        = critical,CA:FALSE
extendedKeyUsage        = codeSigning,1.3.6.1.4.1.311.10.3.6
nsComment               = "OpenSSL Generated Certificate"

```
Adjust all parts with <YOUR*> to your details.

2. Create the public and private key for signing the kernel:
```
openssl req -config ./mokconfig.cnf \
        -new -x509 -newkey rsa:2048 \
        -nodes -days 36500 -outform DER \
        -keyout "MOK.priv" \
        -out "MOK.der"
```

3. Convert the key also to PEM format (mokutil needs DER, sbsign needs PEM):
```
openssl x509 -in MOK.der -inform DER -outform PEM -out MOK.pem
```

4. Enroll the key to your shim installation:
```
sudo mokutil --import MOK.der
```
You will be asked for a password, you will just use it to confirm your key selection in the
next step, so choose any.

5. Restart your system. You will encounter a blue screen of a tool called MOKManager.
Select "Enroll MOK" and then "View key". Make sure it is your key you created in step 2.
Afterwards continue the process and you must enter the password which you provided in
step 4. Continue with booting your system.

6. Verify your key is enrolled via:
```
sudo mokutil --list-enrolled
```

7. Sign your installed kernel (it should be at /boot/vmlinuz-[KERNEL-VERSION]-surface-linux-surface):
```
sudo sbsign --key MOK.priv --cert MOK.pem /boot/vmlinuz-[KERNEL-VERSION]-surface-linux-surface --output /boot/vmlinuz-[KERNEL-VERSION]-surface-linux-surface.signed
```

8. Copy the initram of the unsigned kernel, so we also have an initram for the signed one.
```
sudo cp /boot/initrd.img-[KERNEL-VERSION]-surface-linux-surface{,.signed}
```

9. Update your grub-config
```
sudo update-grub
```

10. Reboot your system and select the signed kernel. If booting works, you can remove the unsigned kernel:
```
sudo mv /boot/vmlinuz-[KERNEL-VERSION]-surface-linux-surface{.signed,}
sudo mv /boot/initrd.img-[KERNEL-VERSION]-surface-linux-surface{.signed,}
sudo update-grub
```

Now your system should run under a signed kernel and upgrading GRUB2 works again. If you want
to upgrade the custom kernel, you can sign the new version easily by following above steps
again from step seven on. Thus BACKUP the MOK-keys (MOK.der, MOK.pem, MOK.priv).

## References
- https://wiki.debian.org/BuildADebianKernelPackage
- https://wiki.archlinux.org/title/Modprobed-db
- https://www.reddit.com/r/Fedora/comments/hp9bns/zen_and_xanmod_kernels_on_fedora_32_howto/
- https://github.com/dell/dkms/issues/124
- https://apt.llvm.org/


***
<p style="text-align: center;">Enjoy tour system ;)</p>