# SDK源码编译



## SDK下载

网盘链接



## 编译环境搭建

此编译环境配置 适用 Android/Linux SDK

### 硬件要求

| 系统环境  | 要求              |
| --------- | ----------------- |
| 系统版本  | Ubuntu22.04，64位 |
| CPU核心数 | 4核以上           |
| 内存容量  | 16GB以上          |
| 硬盘容量  | 200GB以上         |



### 安装依赖软件包

Android / Linux 

```
sudo apt update
sudo apt install autoconf bc binfmt-support bison build-essential bzip2
sudo apt install chrpath cmake cpp-aarch64-linux-gnu curl device-tree-compiler diffstat
sudo apt install expat expect expect-dev fakeroot flex
sudo apt install g++ g++-multilib gawk gcc gcc-multilib git gnupg gperf gpgv2 imagemagick
sudo apt install lib32ncurses5-dev lib32readline-dev lib32z1-dev libgmp-dev 
sudo apt install libgucharmap-2-90-dev liblz4-tool libmpc-dev
sudo apt install libncurses5-dev libsdl1.2-dev libssl-dev libwxgtk3.0-gtk3-dev 
sudo apt install libxml2 libxml2-utils live-build lzop
sudo apt install make module-assistant ncurses-dev openjdk-8-jdk 
sudo apt install patchelf pngcrush python2 python-is-python3 python-pip
sudo apt install qemu-user-static rsync schedtool squashfs-tools ssh sudo texinfo u-boot-tools unzip
sudo apt install xsltproc yasm zip zlib1g-dev pip
sudo pip install pyelftools
```

> 软件包名称 会根据UBUNTU版本更新而变化
>
> 不同UBUNTU版本安装失败，可网络搜索对应的解决方法



## Android 编译

### 配置环境

单独编译或全部编译前先配置环境

```bash
$ ./build.sh lunch
will lunch sdk

You're building on Linux
Lunch menu...pick a combo:

1. rk3576
Which would you like? [0]: 1

You're building on Linux
Lunch menu...pick a combo:

1. BoardConfig.mk
2. BoardConfig-rk3576-kickpi-k7.mk
Which would you like? [0]: 2
switching to board: /home/huangcm/A/sdk/rk3576-android14.0/device/rockchip/rk3576/BoardConfig-rk3576-kickpi-k7.mk
```



### 全部编译

```
./build.sh -AUCKu
```



### 单独编译

单编Uboot

```
./build.sh -Uu
```



单编安卓

```
./build.sh -Au
```



单编kernel

```
./build.sh -CKu
```



**固件说明**

完整编译后会生成如下文件：

```
(源码)/rockdev/Image-rk3576_u/
├── boot-debug.img
├── boot.img
├── config.cfg
├── dtbo.img
├── MiniLoaderAll.bin
├── misc.img
├── parameter.txt
├── pcba_small_misc.img
├── pcba_whole_misc.img
├── recovery.img
├── resource.img
├── super.img
├── uboot.img
├── update.img
└── vbmeta.img
```

烧写的镜像为 `(源码)/rockdev/Image-rk3576_u/update.img`

烧录详见 - `10-系统镜像烧录`



**userdebug 和 user 编译**

默认为userdebug模式编译，如果需要user版本镜像则需要修改对应编译mk

```diff
vim device/rockchip/rk3576/BoardConfig-rk3576-kickpi-k7.mk
-export BUILD_VARIANT=userdebug
+export BUILD_VARIANT=user
```



**android 环境配置**

配置环境，可以使用 mm / get_build_var 等安卓环境下命令

```shell
// 配置环境
source build/envsetup.sh
lunch rk3576_u-userdebug

// 比如
$ get_build_var PRODUCT_HAVE_RKPHONE_FEATURES
true
```



dts 路径

```
kernel-6.1/arch/arm64/boot/dts/rockchip/rk3576-kickpi-k7-android.dts
```



### 常见问题

**由于代码位置移动，路径错误导致 android 编译错误**

![image-20241125155203743](C:\Users\16708\AppData\Roaming\Typora\typora-user-images\image-20241125155203743.png)

需要清除数据

```shell
source build/envsetup.sh
lunch rk3576_u-userdebug
make clean -j32
```

重新全部编译



**编译空间不足**

目前默认线程 -j32，通过降低线程数进行编译

方式一： 

修改编译方式，带 -J 线程

```shell
如： 修改线程为16，-J16
./build.sh -AUCKu -J16
```



方式二：

修改默认线程

```diff
$ vim device/rockchip/common/build/rockchip/build.sh
-BUILD_JOBS=32
+BUILD_JOBS=16
```



## Linux 编译

### 配置环境

单独编译或全部编译前先配置环境

```
$ ./build.sh lunch
Log colors: message notice warning error fatal

Log saved at /home/huangcm/A/sdk/rk3576/rk3576-linux/output/sessions/2024-12-05_16-25-44
Pick a defconfig:

1. rockchip_defconfig
2. rockchip_rk3576_kickpi_k7_buildroot_defconfig
3. rockchip_rk3576_kickpi_k7_debian_defconfig
4. rockchip_rk3576_kickpi_k7_ubuntu_defconfig
Which would you like? [1]:
```

根据需要选择对应的主板和系统



### 全部编译

```
$ ./build.sh 
```



### 单独编译

**kernel 编译**

```
./build.sh kernel
```

或

```
export CROSS_COMPILE=../prebuilts/gcc/linux-x86/aarch64/gcc-arm-10.3-2021.07-x86_64-aarch64-none-linux-gnu/bin/aarch64-none-linux-gnu-
make ARCH=arm64 rockchip_linux_defconfig rk3576.config
make ARCH=arm64 rk3576-kickpi-k7-linux.img -j24
```



## Debian 编译

用于自行下载定制化 Debian 操作系统

```
# lsb_release -a
No LSB modules are available.
Distributor ID: Debian
Description:    Debian GNU/Linux 12 (bookworm)
Release:        12
Codename:       bookworm
```



### 配置环境

```
sudo dpkg -i debian/ubuntu-build-service/packages/*
sudo apt-get install -f
```



### rootfs.img 编译

```
./build.sh debian
或
参考 debian/readme.md
```



## 文档参考

源码下有 RK 官方的指导文件

```
Android:
	（Android源码）/RKDocs/
Linux:
	（Linux源码）/docs/
```



优先推荐查看，仔细的了解 RK 相关 SDK 指导说明

```
Android:
	（Android源码）/RKDocs/android/RK3576_Developer_Guide_Android14_SDK_CN.pdf
Linux:
	（Linux源码）/docs/readme_cn.md
	（Linux源码）/docs/cn/Rockchip_Developer_Guide_Linux_Software_CN.pdf
```

