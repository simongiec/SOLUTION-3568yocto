# SOLUTION-3568yocto
RK3568 yocto系统介绍

一、板卡端口

本机型端口见如下截图：

![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/67870304-222e-4e1a-9db1-401d49c7fbc4)


二、SDK结构介绍

1、SDK具体代码目录结构概览如下图

![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/99ece9ee-bb82-4a29-beb6-fcf92a4e7ae7)


SDK⽬录包含有 buildroot、debian、app、kernel、u-boot、device、docs、external 等⽬录。每个⽬录或其⼦⽬录会对应⼀个 git ⼯程，提交需要在各⾃的⽬录下进⾏。 

app：存放上层应⽤ APP，主要是⼀些应⽤Demo。 

buildroot：基于 Buildroot（2021）开发的根⽂件系统。 

debian：基于 Debian bullseye(11) 开发的根⽂件系统。 

device/rockchip：存放芯⽚板级配置以及⼀些编译和打包固件的脚本和⽂件。 

docs：存放开发指导⽂件、平台⽀持列表、⼯具使⽤⽂档、Linux 开发指南等。 

external：存放第三⽅相关仓库，包括显⽰、⾳视频、摄像头、⽹络、安全等。 

kernel：存放 Kernel 开发的代码。 

output：存放每次⽣成的固件信息、编译信息、XML、主机环境等。 

prebuilts：存放交叉编译⼯具链。 

rkbin：存放 Rockchip 相关⼆进制和⼯具。 

rockdev：存放编译输出固件,实际软链接到 output/firmware 。 

tools：存放 Linux 和 Window 操作系统下常⽤⼯具。

u-boot：存放基于 v2017.09 版本进⾏开发的 U-Boot 代码。 

yocto：存放基于 Yocto 4.0开发的根⽂件系统。 

2、yocto目录介绍

RK3568的yocto系统配置都在linux-debian-rk356x/yocto目录，详细代码截图如下：
![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/e19beaab-71fc-4025-9f24-3b6954b14010)


二、sdk环境配置和编译介绍

环境配置和编译具体可参考官方文档Rockchip_Developer_Guide_Linux_Software_EN.pdf章节6.3和章节8，文档链接如下：https://drive.google.com/file/d/1u5arX0oeAv20RYMpMTIa7Pu2xslg_PoY/view?usp=drive_link


1、开发环境配置

我们推荐使⽤ Ubuntu 22.04 或更⾼版本的系统进⾏编译。其他的 Linux 版本可能需要对软件包做相应调整。除了系统要求外，还有其他软硬件⽅⾯的要求。 

硬件要求：64 位系统，硬盘空间⼤于 40G。如果您进⾏多个构建，将需要更⼤的硬盘空间。 

软件要求：Ubuntu 22.04 或更⾼版本系统。

安装库和⼯具集，使⽤命令⾏进⾏设备开发时，可以通过以下步骤安装编译SDK需要的库和⼯具。使⽤如下apt-get命令安装后续操作所需的库和⼯具：

sudo apt-get update && sudo apt-get install git ssh make gcc libssl-dev \ 

liblz4-tool expect expect-dev g++ patchelf chrpath gawk texinfo chrpath \ 

diffstat binfmt-support qemu-user-static live-build bison flex fakeroot \ 

cmake gcc-multilib g++-multilib unzip device-tree-compiler ncurses-dev \

libgucharmap-2-90-dev bzip2 expat gpgv2 cpp-aarch64-linux-gnu libgmp-dev \ 

libmpc-dev bc python-is-python3 python2
 
2、编译命令

SDK可通过./build.sh 加⽬标参数进⾏相关功能的配置和编译，具体可通过 ./build.sh help 查看具体编译命令。具体编译过程首先需要运行./build.sh lunch，选择工程5，rockchip_rk3568_evb1_ddr4_v10_defconfig，然后输入export RK_ROOTFS_SYSTEM=yocto，之后可以运行./build.sh all全编译uboot、kernel、rootfs、recovery等全部模块并打包成update.img，或者用./build.sh yocto编译生成rootfs.img，这就是yocto的根文件系统。



四、RK3568 yocto创建过程介绍

针对客户提出的step1~8，回复如下：

Step 1: Set Up a New Yocto Repository for RK3568

linux-debian-rk356x/yocto目录就是创建后的yocto-rk3568工程，官方yocto代码路径见linux-debian-rk356x/yocto/poky目录，kirkston 4.0.13版本，编译环境配置按照上文开发环境配置要求即可。

Step 2: Create and Add Layers

linux-debian-rk356x/yocto目录下meta-browser、meta-clang、meta-poky、meta-rockchip、meta-openembedded、meta-qt5就是新建的layer，且在bblayers.conf中有添加配置，详细路径linux-debian-rk356x/yocto/build/conf/bblayers.conf，配置如下：

POKY_BBLAYERS_CONF_VERSION = "2"

BBPATH = "${TOPDIR}"
BBFILES ?= ""

BBLAYERS ?= " \
  ${TOPDIR}/../meta-openembedded/meta-oe \
  ${TOPDIR}/../meta-openembedded/meta-python \
  ${TOPDIR}/../meta-openembedded/meta-networking \
  ${TOPDIR}/../meta-openembedded/meta-multimedia \
  ${TOPDIR}/../meta-qt5 \
  ${TOPDIR}/../meta-clang \
  ${TOPDIR}/../meta-rockchip \
  ${TOPDIR}/../poky/meta \
  ${TOPDIR}/../poky/meta-poky \
  ${TOPDIR}/../poky/meta-yocto-bsp \
"

Step 3: Configure Machine Settings

Machine定义在device/rockchip/common/configs/Config.in.yocto中，如下：

![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/4ab0ba5f-099d-4f3e-b962-0352e76f8599)

   
   
RK3568的machine默认rockchip-rk3568-evb，其他系统信息如cpu架构等可查看rockchip-rk3568-evb.conf文件，路径linux-debian-rk356x\yocto\meta-rockchip\conf\machin。

Step 4: Create Recipes for Kernel and Bootloader

Kernel Recipe在linux-debian-rk356x\yocto\meta-rockchip\recipes-kernel目录下，包括WiFi/BT的firmware等。

U-Boot Recipe在linux-debian-rk356x\yocto\meta-rockchip\recipes-bsp目录下，包括rkbin、npu等。

Step 5: Add Security Features to U-Boot and Kernel

默认Secureboot安全启动没有开启，需要开启RK_SECURITY=y，可以在如下文件中添加：

kernel/arch/arm64/configs/rockchip_linux_defconfig。

Kernel部分还要加如下配置：

CONFIG_BLK_DEV_DM=y 

CONFIG_DM_CRYPT=y 

CONFIG_BLK_DEV_CRYPTOLOOP=y 

CONFIG_DM_VERITY=y 

CONFIG_TEE=y 

CONFIG_OPTEE=y 

u-boot添加相关配置，位置u-boot/configs/rk3568_deconfig:

CONFIG_FIT_SIGNATURE=y 

CONFIG_SPL_FIT_SIGNATURE=y

buildroot添加相关配置，位置buioldroot/configs/rockchip_rk3568_defconfig

BR2_ROOTFS_OVERLAY="board/rockchip/common/security-system-overlay"

Step 6: Kernel Configuration and Device Drivers

Device Tree Files: rockchip-rk3568-evb.conf中有KERNEL_DEVICETREE，指定了device tree 

KERNEL_DEVICETREE = "rockchip/rk3568-evb1-ddr4-v10-linux.dtb"

Kernel Configuration:在yocto/meta-rockchip/conf/machine/include/rockchip-common.inc中有配置，具体：KBUILD_DEFCONFIG ?= "rockchip_linux_defconfig"，如需修改，只需要在kernel/arch/arm64/configs/rockchip_linux_defconfig中修改即可。

Network and Storage Drivers:  Ethernet, Wi-Fi, Bluetooth, USB, SATA等所有驱动都是在meta-rockchip\recipes-kernel\linux-libc-headers下定义的，通过github上下载，参考linux-libc-headers_4.19-custom.bb，

SRC_URI = " \
	git://github.com/JeffyCN/mirrors.git;protocol=https;nobranch=1;branch=kernel-4.19-2022_01_10; \
 
Rebuild the Kernel: RK3568的kernel是linux-rockchip，编译可用命令 bitbake linux-rockchip -C compile。

编译完kernel后打包rootfs.img，测试机器USB/BT/Ethernet/HDMI/WiFi等都正常，USB/HDMI测试可用对应的设备，BT/Ethernet/WiFi可用命令测试，ifconfig 和hciconfig -a。

Step 7: Build the Yocto Image

单独编译yocto的rootfs.img，可用命令./build.sh yocto，编译完成后会在yocto\build\tmp\deploy\images\rockchip-rk3568-evb下生成rootfs.img，另外rockdev下也会有。

Step 8: Flash and Test the Image

烧录rootfs.img，可用RKDevTool烧录，如下图

![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/377e61a4-80fb-4008-8601-bbaed72834f7)


其中uboot、recovery等的img可以用命令./build.sh all编译或者”./build.sh  <模块名>”生成，都在rockdev目录下。

RKDevTool和烧录说明文档链接：

https://drive.google.com/drive/folders/1qXsAu01LdWnwxjY-pDFxdnjrp3m2FHbQ?usp=drive_link


烧录固件链接：
https://drive.google.com/drive/folders/1mQA5ctSpYvhAQD4HINCqMSV6uX181qP3?usp=drive_link

五、客户代码移植

1、尝试把客户的YOCTO-META-ARED、YOCTO-META-OTAB、YOCTO-META-READONLY-ROOTFS-OVERLAY、YOCTO-META-SYSTEM-INSTALLER、YOCTO-VIRTUAL-ENVIRONMENT-BUILD编译进去，放到linux-debian-rk356x/yocto目录下，在linux-debian-rk356x/yocto/build/conf/bblayers.conf中添加如下代码

![image](https://github.com/simongiec/SOLUTION-3568yocto/assets/169290270/ade58025-6b3f-40c2-ab0c-ab84010a07f6)


然后编译 ./build.sh yocto

报错ERROR: Layer readonly-rootfs-overlay is not compatible with the core layer which only supports these series: kirkstone (layer is compatible with dunfell master hardknott gatesgarth)

github上下载3mdeb/meta-readonly-rootfs-overlay: This yocto meta layer implements a read-only root filesystem with a writable overlay. (github.com)并切换到kirkstone分支，替换YOCTO-META-READONLY-ROOTFS-OVERLAY内容再编译，报错：

ERROR: No recipes in default available for:

  /home/liuqq/yocto/linux-debian-rk356x/yocto/build/../YOCTO-META-ARED/meta-ared-bsp/recipes-kernel/linux/linux-stable_%.bbappend
  
  /home/liuqq/yocto/linux-debian-rk356x/yocto/build/../YOCTO-META-ARED/meta-ared-distro/recipes-containers/balena-engine/balena-engine_%.bbappend
  
  /home/liuqq/yocto/linux-debian-rk356x/yocto/build/../YOCTO-META-ARED/meta-ared-distro/recipes-tailscale/tailscale/tailscale_%.bbappend
  
  /home/liuqq/yocto/linux-debian-rk356x/yocto/build/../YOCTO-META-ARED/meta-ared-distro/recipes-webadmin/webmin/webmin_1.850.bbappend
  

没有linux-stable、balena-engine、tailscale开头的bb文件，只能删掉对应路径的bbappend文件，再编译，继续报错：Dependency loop #1 found，log很多，去掉YOCTO-META-OTAB，并且去掉YOCTO-META-ARED中otab相关的再继续编译，
可以看到一个warning，meta-ared-distro/recipes-containers/install-containers/install-containers_0.1.bb中SRC_URI指向file://gsutil-container-${MACHINE}.tar.bz2，没有这个文件，
忽略这个warning，继续编译还有报错，meta-ared-distro/recipes-extended/procps/procps_%.bbappend 中do_install报错，没有sysctl.d目录，删掉这个sysctl.d，继续编译，最后编译成功。


2、编译烧录检查

参照step8烧录开机，没有看到ARED相关的服务，但是yocto/build/tmp/下确实能找到ARED目录的编译log。
