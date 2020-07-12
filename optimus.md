# Optimus on Archlinux

 * [Resources](optimus.md#resources)
 * [TL;DR](optimus.md#tldr)
   * [Install](optimus.md#install)
   * [Test](optimus.md#test)
   * [Power management](optimus.md#power-management)
      * [Bumblebee](optimus.md#bumblebee)
      * [BBSwitch](optimus.md#bbswitch)
 * [Archlinux + Nvidia + Steam](optimus.md#archlinux--nvidia--steam)


## Resources

  * [Wiki Archlinux : NVIDIA Optimus](https://wiki.archlinux.org/index.php/NVIDIA_Optimus)
  * [Wiki Archlinux : Bumblebee](https://wiki.archlinux.org/index.php/Bumblebee#Bumblebee:_Optimus_for_Linux)
  * [Wiki Archlinux : Steam](https://wiki.archlinux.org/index.php/Steam)
  * [BBSwitch documentation](https://github.com/Bumblebee-Project/bbswitch)

## TL;DR

> **Warning**: Avoid installing the NVIDIA driver through the package provided from the NVIDIA website. Installation through pacman allows upgrading the driver together with the rest of the system.

### Install
  * Package [Bumblebee](https://www.archlinux.org/packages/?name=bumblebee), 
  * Package [mesa](https://www.archlinux.org/packages/?name=mesa), 
  * An appropriate version of the [Nvidia Driver](https://wiki.archlinux.org/index.php/NVIDIA#Installation),
  * [xf86-video-intel](https://www.archlinux.org/packages/?name=xf86-video-intel)

Add regular *user* to the ``bumblebee`` group :
```
gpasswd -a user bumblebee
```

Enable service ``bumblebeed.service``, reboot etc.

### Test

Install [mesa-demos](https://www.archlinux.org/packages/?name=mesa-demos) and use ``glxgears`` :
```
optirun glxgears -info
```

### Power management

> How to use by default intel gpu then activate your Nvidia gpu only on demand.

#### Bumblebee

Your configuration files have to look like below :
```
$ cat /etc/bumblebee/bumblebee.conf | grep ^[^#]

[bumblebeed]
VirtualDisplay=:8
KeepUnusedXServer=false
ServerGroup=bumblebee
TurnCardOffAtExit=false
NoEcoModeOverride=false
Driver=
XorgConfDir=/etc/bumblebee/xorg.conf.d
[optirun]
Bridge=auto
VGLTransport=proxy
PrimusLibraryPath=/usr/lib/primus:/usr/lib32/primus
AllowFallbackToIGC=false
[driver-nvidia]
KernelDriver=nvidia
PMMethod=bbswitch
LibraryPath=/usr/lib/nvidia:/usr/lib32/nvidia:/usr/lib:/usr/lib32
XorgModulePath=/usr/lib/nvidia/xorg,/usr/lib/xorg/modules
XorgConfFile=/etc/bumblebee/xorg.conf.nvidia
[driver-nouveau]
KernelDriver=nouveau
PMMethod=bbswitch
XorgConfFile=/etc/bumblebee/xorg.conf.nouveau
```

```
$ cat /etc/bumblebee/xorg.conf.nvidia | grep ^[^#]

Section "ServerLayout"
    Identifier  "Layout0"
    Option      "AutoAddDevices" "false"
    Option      "AutoAddGPU" "false"
EndSection
Section "Device"
    Identifier  "DiscreteNvidia"
    Driver      "nvidia"
    VendorName  "NVIDIA Corporation"
    Option "ProbeAllGpus" "false"
    Option "NoLogo" "true"
    Option "UseEDID" "false"
    Option "UseDisplayDevice" "none"
EndSection
```

```
$ cat /etc/bumblebee/xorg.conf.nouveau | grep ^[^#]

Section "ServerLayout"
    Identifier  "Layout0"
    Option      "AutoAddDevices" "false"
    Option      "AutoAddGPU" "false"
EndSection
Section "Device"
    Identifier  "DiscreteNvidia"
    Driver      "nouveau"
EndSection

```

#### BBSwitch

```
$ cat /etc/modprobe.d/bbswitch.conf 

options bbswitch load_state=0 unload_state=0
```

To get status :
```
$ cat /proc/acpi/bbswitch  
0000:01:00.0 ON
```
*In this example, Nvidia gpu is activate.*

## Archlinux + Nvidia + Steam

If you want to use Steam with your NVidia chipset using optimus technology.

Install ``primusrun`` from package [primus](https://www.archlinux.org/packages/?name=primus).

Launch Steam (without any optirun/primusrun command, etc. Just launch it in the most natural way), then, for each game you want to use Nvidia Chipset :
> Settings -> Set Launch Options -> optirun -b primus %command%

If you want to deactivate vertical sync : ``vblank_mode=0 optirun -b primus %command%``