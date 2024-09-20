# LVGL ported to Verdin AM62 on Linux BSP

## Overview

This guide provides steps to cross-compile an LVGL project for the Verdin AM62 from Toradex. The project is based on the [lv_drivers](https://github.com/lvgl/lv_drivers/tree/release/v8.3) and [lvgl](https://github.com/lvgl/lvgl/tree/release/v8.3) repositories and a lvgl demo from [dialog lvgl](https://gitlab.eclipse.org/pcoval/dialog-lvgl.git).

This repository offers insstructions on how to cross-compile and run lvgl framwork and application on top of the [Weston](https://wiki.archlinux.org/title/Weston) compositor.  

## Buy

You can purchase the Verdin AM62 and compatible carrier boards directly from Toradex.'

## Specification

### CPU and Memory
- **Module:** Vedin AM62
- **RAM:** 1GB DDR RAM
- **Flash:** 8GB eMMC Flash
- **GPU:** PowerVR Rogue AXE-1-16M

## Getting started

### Hardware setup
- Insert the Verdin AM62 module into one of Toradex’s carrier boards, e,g. Dahlia with DSI to HDMI adapter.
- Connect a display to the carrier board. HDMI interface from the DSI to HDMI adapter on Dahlia can be enabled by the [device tree overlays](https://developer.toradex.cn/torizon/application-development/use-cases/multimedia/setting-up-displays-with-torizon/#HDMI).
```
root@verdin-am62:~# cat /boot/overlays.txt
fdt_overlays=verdin-am62_dsi-to-hdmi_overlay.dtbo verdin-am62_spidev_overlay.dtbo
```

### Software setup
- Ensure the Verdin AM62 module is running [reference multimedia image](https://developer.toradex.com/software/toradex-embedded-software/toradex-download-links-torizon-linux-bsp-wince-and-partner-demos/#TdxRef6QuarterlyMult). The default autostart demo can be stoped by `systemctl stop wayland-app-launch.service`. The image comes with a default root user with empty password.
- On the Linux host, install [Linux SDK](https://developer.toradex.com/linux-bsp/os-development/build-yocto/linux-sdks/) for Verdin AM62. For example, the Linux SDK is installed at `~/SDK`. Source the environment script in every shell session.
```
$ . environment-setup-aarch64-tdx-linux 
```

### Build the binary on the host
- Clone this repository repository:

```
git clone --recursive https://github.com/lvgl/lv_port_toradex_verdin_am62.git
```

- apply patchs
```
cd lvgl
patch -p 1 < ../lvgl.patch

cd lv_drivers
patch -p 1 < ../lv_drivers.patch

cd dialog-lvgl
patch -p 1 < ../dialog.patch
```

lv_drivers.patch contains an absolute path of Linux SDK that needs to be changed to your own.
```
include_directories(~/SDK/sysroots/aarch64-tdx-linux/usr/src/debug/weston/10.0.2-r0/build)
```

- build lvgl and lv_drivers

header files and built libraries are installed to a `lvgl_wayland` folder which will be copied to Linux SDK intallation path later for application compile.
```
cd lvgl
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=../../lvgl_wayland
make
make install


cd lv_driver
mkdir build
cd build
cmake .. -DCMAKE_INSTALL_PREFIX=../../lvgl_wayland
make
make install
```

- copy files to Linux SDK intallation path
Linux SDK was installed at `~/SDK` and it has to been changed to your own path.
```
cd ../../lvgl_wayland
cp -r ./* ~/SDK/sysroots/aarch64-tdx-linux/usr/
```

- build application

```
cd dialog-lvgl
make lvgl_driver=wayland
```
The binar `dialog-lvgl` will be generated.



### Run the binary on the module

copy `dialog-lvgl` to Verdin AM62, set library path and excute the bianry.
```
export LD_LIBRARY_PATH=/usr/lib/plugins/wayland-shell-integration
./dialog-lvgl
```


## Contribution and Support

If you find any issues with the development board feel free to open an Issue in this repository. For LVGL related issues (features, bugs, etc) please use the main [lvgl repository](https://github.com/lvgl/lvgl).

If you found a bug and found a solution too please send a Pull request. If you are new to Pull requests refer to [Our Guide](https://docs.lvgl.io/master/CONTRIBUTING.html#pull-request) to learn the basics.