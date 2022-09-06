# Notes for installing Ubuntu on Asus N61Jv laptop without build-in screen

I have an old Asus N61Jv laptop from which the build-in LCD screen is broken. I want to use this 
computer as a _VT100_ like terminal or HP/UX running CDE like computer, and connect it to an external screen as the primary screen. However
that is not as simple as it seems.  The computer has an NVidia GeForce GT 325M graphics chip that has Nvidia driver 340 as 
last supported version, with kernel 5.4. Do not use a higher kernel. It won't compile.
The most reliable solution is to disable Nvidia GPU and use the Intel i915 GPU only.

## Info

```
ubuntu@asus:~/Development/git/asus$ sudo lshw -C display
  *-display                 
       description: VGA compatible controller
       product: GT216M [GeForce GT 325M]
       vendor: NVIDIA Corporation
       physical id: 0
       bus info: pci@0000:01:00.0
       version: a2
       width: 64 bits
       clock: 33MHz
       capabilities: pm msi pciexpress vga_controller bus_master cap_list rom
       configuration: driver=nvidia latency=0
       resources: irq:16 memory:e2000000-e2ffffff memory:d0000000-dfffffff memory:e0000000-e1ffffff ioport:d000(size=128) memory:e3000000-e307ffff
  *-display
       description: VGA compatible controller
       product: Core Processor Integrated Graphics Controller
       vendor: Intel Corporation
       physical id: 2
       bus info: pci@0000:00:02.0
       version: 18
       width: 64 bits
       clock: 33MHz
       capabilities: msi pm vga_controller bus_master cap_list rom
       configuration: driver=i915 latency=0
       resources: irq:38 memory:e3400000-e37fffff memory:c0000000-cfffffff ioport:e080(size=8) memory:c0000-dffff
```
## GRUB

To disable the laptop panel at boot time, add the following option to grub:
```
GRUB_CMDLINE_LINUX_DEFAULT="video=LVDS-1-1:d"
```
Then run:
```sh
sudo update-grub
```

## Kernel drivers

You can choose between the dedicated Nvidia driver or the Intel i915 video driver. Note that the latest 
supported Nvidia driver for the 325M is nvidia-driver-340. This driver supports up to kernel version 5.4.0. 
For newer kernel versions, only the Intel video driver will work. You have to blacklist the nvidia driver.

To select nvidia only as the dedicated driver, you can use the program `nvidia-settings` or the command
line program `/usr/bin/prime-select nvidia|intel|on-demand|query`.

### Blacklist nouveau and/or nvidia

In /etc/modprobe.d/blacklist-nvidia.conf:

```
blacklist nvidia
blacklist nvidia_uvm
blacklist nouveau
options nouveau modeset=0

```
Then run:

```sh
sudo update-initramfs -u
```

## X11

### CDE

Add the following lines at the top of `/usr/dt/bin/Xsession`, just under `set +a`:
```sh
xrandr --output LVDS-1 --off
xrandr --output HDMI-1 --primary
xrandr --output HDMI-1 --preferred
```


### GDM3

Uncomment in `/etc/gdm3/custom.conf`:

```
WaylandEnable=false
```


### Monitor settings in Gnome desktop

Config is stored in `~/.config/monitors.xml`.

### xrandr

- Disable the laptop panel.
- Set the HDMI as primary output.
- Set the output resolution to 'preferred'.

```sh
xrandr -q
xrandr --listactivemonitors 
xrandr --output LVDS-1 --off
xrandr --output HDMI-1 --primary
xrandr --output HDMI-1 --preferred
```

## References

https://unix.stackexchange.com/questions/13619/how-do-i-prevent-xorg-using-my-linux-laptops-display-panel
https://stackoverflow.com/questions/20114918/xlib-extension-glx-missing-on-display-0
