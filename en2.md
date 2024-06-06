# A Brief Introduction to RK3568 Yocto System Code

## 1. Board Ports

The ports for this model are shown as below picture

![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/7c32f439-205c-4491-9482-1b32a9237c67)

## 2. SDK Structure Overview
We have a git repo for all the RK3568 SDK, and the SDK directory structure is as follows:

```
linux-debian-rk3568/
├── app
├── buildroot
├── build.sh -> device/rockchip/common/scripts/build.sh
├── debian
├── device
├── docs
├── envsetup.sh -> buildroot/build/envsetup.sh
├── external
├── kernel
├── Makefile -> device/rockchip/common/Makefile
├── prebuilts
├── rkbin
├── rkflash.sh -> device/rockchip/common/scripts/rkflash.sh
├── tools
├── u-boot
└── yocto
```

The SDK directory includes buildroot, debian, app, kernel, u-boot, device, docs, external, etc.
```
app:Contains upper-level application demos.
buildroot:Root filesystem based on Buildroot (2021).
debian:Root filesystem based on Debian bullseye(11).
device/rockchip:Contains chip board configurations and scripts/files for compiling and packaging firmware.
docs:Contains development guides, platform support lists, tool usage documents, Linux development guides, etc.
external:Contains third-party related repositories, including display, audio/video, camera, network, security, etc.
kernel:Contains Kernel development code.
output:Stores firmware information, compilation information, XML, host environment, etc., generated each time.
prebuilts:Contains cross-compilation toolchains.
rkbin:Contains Rockchip-related binaries and tools.
rockdev:Stores compiled output firmware, which is actually a symlink to output/firmware.
tools:Contains commonly used tools for Linux and Windows operating systems.
u-boot:Contains U-Boot code developed based on version v2017.09.
yocto:Contains the root filesystem developed based on Yocto 4.0.
```

* Yocto Directory Overview
All configurations for the RK3568 Yocto system are in the linux-debian-rk356x/yocto directory. Detailed code is as follows

```
yocto/
├── bitbake
├── build
├── meta-browser
├── meta-clang
├── meta-openembedded
├── meta-poky
├── meta-qt5
├── meta-rockchip
├── oe-init-build-env
├── poky
└── scripts
```

## 3.SDK Environment Configuration
> For detailed instructions on environment configuration and compilation, refer to 
 [Rockchip_Developer_Guide_Linux_Software_EN.pdf](https://drive.google.com/file/d/1u5arX0oeAv20RYMpMTIa7Pu2xslg_PoY/view?usp=drive_link), chapters 6.3 and 8: Rockchip Developer Guide.

Development Environment Configuration
We recommend using Ubuntu 22.04 or higher. Other Linux versions may require package adjustments.
* Hardware Requirements: 64-bit system, more than 40GB disk space.
* Software Requirements: Ubuntu 22.04 or higher.

Install necessary libraries and tools using:
```
sudo apt-get update && sudo apt-get install git ssh make gcc libssl-dev \
liblz4-tool expect expect-dev g++ patchelf chrpath gawk texinfo chrpath \
diffstat binfmt-support qemu-user-static live-build bison flex fakeroot \
cmake gcc-multilib g++-multilib unzip device-tree-compiler ncurses-dev \
libgucharmap-2-90-dev bzip2 expat gpgv2 cpp-aarch64-linux-gnu libgmp-dev \
libmpc-dev bc python-is-python3 python2
```
## 4.Compilation Commands
Use `./build.sh` with target parameters for configuration and compilation. Specific commands can be found using `./build.sh help`.

To compile:

Run `./build.sh lunch`, select project 5:  `rockchip_rk3568_evb1_ddr4_v10_defconfig`.

Set the root filesystem: `export RK_ROOTFS_SYSTEM=yocto`.
You can compile all modules (uboot, kernel, rootfs, recovery) and package them into update.img 

To compile all modules and package them into update.img:`./build.sh all`

To compile and generate rootfs.img for Yocto root filesystem:`./build.sh yocto`

## 5.Introduction to our RK3568 Yocto Migration Process
### Step 1: Set Up a New Yocto Repository for RK3568
The linux-debian-rk356x/yocto directory is the created Yocto RK3568 project. 
Official Yocto code can be found in linux-debian-rk356x/yocto/poky, version kirkstone 4.0.13. Follow the development environment configuration mentioned above [Rockchip_Developer_Guide_Linux_Software_EN.pdf](https://drive.google.com/file/d/1u5arX0oeAv20RYMpMTIa7Pu2xslg_PoY/view?usp=drive_link).

### Step 2: Create and Add Layers
The layers meta-browser, meta-clang, meta-poky, meta-rockchip, meta-openembedded, and meta-qt5 are created in the linux-debian-rk356x/yocto directory and added to bblayers.conf:
```
POKY_BBLAYERS_CONF_VERSION = "2"
BBPATH = "${TOPDIR}" BBFILES ?= ""
BBLAYERS ?= "
${TOPDIR}/../meta-openembedded/meta-oe
${TOPDIR}/../meta-openembedded/meta-python
${TOPDIR}/../meta-openembedded/meta-networking
${TOPDIR}/../meta-openembedded/meta-multimedia
${TOPDIR}/../meta-qt5
${TOPDIR}/../meta-clang
${TOPDIR}/../meta-rockchip
${TOPDIR}/../poky/meta
${TOPDIR}/../poky/meta-poky
${TOPDIR}/../poky/meta-yocto-bsp
```
### Step 3: Configure Machine Settings
The machine is defined in device/rockchip/common/configs/Config.in.yocto:

![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/a696b0ef-7300-4045-af38-48c8ee89cdfa)


The default machine for RK3568 is rockchip-rk3568-evb. 
Additional system information, such as CPU architecture, can be found in rockchip-rk3568-evb.conf at linux-debian-rk356x/yocto/meta-rockchip/conf/machine.

### Step 4: Create Recipes for Kernel and Bootloader
Kernel recipes are in linux-debian-rk356x/yocto/meta-rockchip/recipes-kernel, including WiFi/BT firmware. 
U-Boot recipes are in linux-debian-rk356x/yocto/meta-rockchip/recipes-bsp, including rkbin, npu, etc.

### Step 5: Add Security Features to U-Boot and Kernel
Secureboot is disabled by default. Enable it by setting RK_SECURITY=y 
* in kernel/arch/arm64/configs/rockchip_linux_defconfig, add:
```
CONFIG_BLK_DEV_DM=y
CONFIG_DM_CRYPT=y
CONFIG_BLK_DEV_CRYPTOLOOP=y
CONFIG_DM_VERITY=y
CONFIG_TEE=y
CONFIG_OPTEE=y
```
* in u-boot/configs/rk3568_defconfig, add:

```
CONFIG_FIT_SIGNATURE=y
CONFIG_SPL_FIT_SIGNATURE=y
```

* in buildroot/configs/rockchip_rk3568_defconfig, add:

```
BR2_ROOTFS_OVERLAY="board/rockchip/common/security-system-overlay"
```

### Step 6: Kernel Configuration and Device Drivers
Device tree files are specified in rockchip-rk3568-evb.conf with KERNEL_DEVICETREE:

`KERNEL_DEVICETREE = "rockchip/rk3568-evb1-ddr4-v10-linux.dtb"`

Kernel configuration is in `yocto/meta-rockchip/conf/machine/include/rockchip-common.inc` with KBUILD_DEFCONFIG. 
Network and storage drivers are defined in `meta-rockchip/recipes-kernel/linux-libc-headers`.

Compile the kernel using:
`bitbake linux-rockchip -C compile`

After compiling the kernel, package rootfs.img. Test USB, BT, Ethernet, HDMI, and WiFi using the respective devices and commands such as `ifconfig` and `hciconfig -a`.

### Step 7: Build the Yocto Image
Compile the Yocto root filesystem using:
`./build.sh yocto`

The rootfs.img will be generated in `yocto/build/tmp/deploy/images/rockchip-rk3568-evb` and `rockdev`.

### Step 8: Flash and Test the Image
Flash the rootfs.img using `RKDevTool`. 
![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/d514be85-36b4-4520-ba3b-4d0e2c3e4d0f)

Additional images such as U-Boot and recovery can be compiled using `./build.sh all`

[RKDeevTool and Flash Instructions](https://drive.google.com/drive/folders/1qXsAu01LdWnwxjY-pDFxdnjrp3m2FHbQ?usp=drive_link)

[The Firmware we build](https://drive.google.com/drive/folders/1mQA5ctSpYvhAQD4HINCqMSV6uX181qP3?usp=drive_link)

## 6.Our Recent Activities
Incorporating and Compiling Yocto Layers

We are integrating the customer's Yocto meta layers: YOCTO-META-ARED, YOCTO-META-OTAB, YOCTO-META-READONLY-ROOTFS-OVERLAY, YOCTO-META-SYSTEM-INSTALLER, and YOCTO-VIRTUAL-ENVIRONMENT-BUILD. 
These layers are placed under the linux-debian-rk356x/yocto directory. We added the following code in linux-debian-rk356x/yocto/build/conf/bblayers.conf:
![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/d2da6490-c4a6-4f20-98e7-3f1839581a5d)


Then we compile using the command `./build.sh yocto`.
However, we encountered an error:
```
Layer readonly-rootfs-overlay is not compatible with the core layer which only supports these series: kirkstone (layer is compatible with dunfell master hardknott gatesgarth)
```
To resolve this error, we downloaded the 3mdeb/meta-readonly-rootfs-overlay from GitHub, switched to the kirkstone branch, and replaced the content of YOCTO-META-READONLY-ROOTFS-OVERLAY before recompiling. Despite this, we encountered another error:
```
ERROR: No recipes in default available for:
/home/liuqq/yocto/linux-debian-rk356x/yocto/build/../YOCTO-META-ARED/meta-ared-bsp/recipes-kernel/linux/linux-stable_%.bbappend
/home/liuqq/yocto/linux-debian-rk356x/yocto/build/../YOCTO-META-ARED/meta-ared-distro/recipes-containers/balena-engine/balena-engine_%.bbappend
/home/liuqq/yocto/linux-debian-rk356x/yocto/build/../YOCTO-META-ARED/meta-ared-distro/recipes-tailscale/tailscale/tailscale_%.bbappend
/home/liuqq/yocto/linux-debian-rk356x/yocto/build/../YOCTO-META-ARED/meta-ared-distro/recipes-webadmin/webmin/webmin_1.850.bbappend
```
Since there were no .bb files starting with linux-stable, balena-engine, or tailscale, we deleted the corresponding .bbappend files and recompiled. This resulted in another error:

Dependency loop #1 found
The logs were extensive. We removed YOCTO-META-OTAB and the related components from YOCTO-META-ARED and continued compiling. A warning was observed:
```
meta-ared-distro/recipes-containers/install-containers/install-containers_0.1.bb: SRC_URI points to file://gsutil-container-${MACHINE}.tar.bz2, which is missing.
```
Ignoring this warning, we continued compiling and encountered another error in meta-ared-distro/recipes-extended/procps/procps_%.bbappend where do_install failed due to the absence of the sysctl.d directory. After removing the sysctl.d directory, the compilation succeeded.

Compilation and Burning Check
Following Step 8 for flashing and power on the device, we did not see any ARED-related services. However, the yocto/build/tmp/ directory contained the compilation logs for the ARED directory.
