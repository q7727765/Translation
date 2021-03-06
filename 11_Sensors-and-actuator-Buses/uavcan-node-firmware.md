# UAVCAN 固件升级

官网英文原文地址：http://dev.px4.io/uavcan-node-firmware.html

## 电子调速器(ESC)矢量控制代码库 (Pixhawk ESC 1.6 and S2740VC)

下载ESC代码:

<div class="host-code"></div>

```sh
git clone https://github.com/thiemar/vectorcontrol
cd vectorcontrol
```

### 刷新UAVCAN启动引导程序

PIxhawk ESC 1.6在通过UAVCAN设备更新固件之前, 首先要求刷新UAVCAN的启动引导程序。为了生成启动引导程序，运行：

<div class="host-code"></div>

```sh
make clean && BOARD=px4esc_1_6 make -j8
```

启动引导程序生成之后，其image文件存放路径为 `firmware/px4esc_1_6-bootloader.bin`, OpenOCD的配置文档为 `openocd_px4esc_1_6.cfg`。可以通过 [如下教程](../11_Sensors-and-actuator-Buses/uavcan-node-enumeration.md) 初始化ESC的启动程序。

### 编译主要的二进制（.bin）文件Compiling the Main Binary

<div class="host-code"></div>

```sh
BOARD=s2740vc_1_0 make && BOARD=px4esc_1_6 make
```

这将会生成两个UAVCAN的节点固件，它们都支持ESCs。它们固件image文件存放路径为`com.thiemar.s2740vc-v1-1.0-1.0.<git hash>.bin` 和`org.pixhawk.px4esc-v1-1.6-1.0.<git hash>.binn`。

## Sapog 代码库 (Pixhawk ESC 1.4和 Zubax Orel 20)

下载Sapog代码库:

<div class="host-code"></div>

```sh
git clone https://github.com/PX4/sapog
cd sapog
git submodule update --init --recursive
```

### 烧写UAVCAN启动引导程序

在通过UAVCAN更新固件之前，ESC需要烧写UAVCAN引导加载程序。引导程序可以使用如下指令构建：

<div class="host-code"></div>

```sh
cd bootloader
make clean && make -j8
cd ..
```

启动引导程序的image文件存放路径为 `bootloader/firmware/bootloader.bin`, OpenOCD的配置文档为`openocd.cfg`。可以通过 [此处教程](../11_Sensors-and-actuator-Buses/uavcan-bootloader-installation.md) 初始化ESC的起始引导程序。

### 编译主要的二进制（.bin）文件

<div class="host-code"></div>

```sh
cd firmware
make RELEASE=1 # RELEASE is optional; omit to build the debug version
```

Beware, some newer version of GCC lead to segfaults during linking. Version 4.9 did work at the time of writing. The firmware image will be located at `firmware/build/io.px4.sapog-1.1-1.7.<xxxxxxxx>.application.bin`, where `<xxxxxxxx>` is an arbitrary sequence of numbers and letters. There are two hardware version of the Zubax Orel 20 (1.0 and 1.1). Make sure you copy the binary to the correct folder in the subsequent description. The ESC firmware will check the hardware version and works on both products.1


## Zubax GNSS

请参考 [项目网页](https://github.com/Zubax/zubax_gnss) 去学习如何生成和刷新固件。
Zubax GNSS 出厂时就带有支持UAVCAN的启动引导程序，因此其固件可以通过UAVCAN使用统一方式进行更新，具体更新方式如下所述。

## Autopilot的固件安装

UAVCAN节点的文档命名遵循约定的命名方式，这种命名方式允许Pixhawk更新网络内所有的UAVCAN设备，无需考虑是哪个制造商生产的。上述步骤产生的固件文件必须要复制到SD卡或PX4 ROMFS的正确的位置，以确保设备能够很好的更新。

固件image名称通常是:

  ```
  <uavcan name>-<hw version major>.<hw version minor>-<sw version major>.<sw version minor>.<version hash>.bin
  ```

  e.g. 
  ```
  com.thiemar.s2740vc-v1-1.0-1.0.68e34de6.bin
  ```

然而，由于空间和性能的限制（命名不能够超过28个字符），UAVCAN固件升级需要将这些文件名分割存储在下面的目录结构里： 
```
/fs/microsd/fw/<node name>/<hw version major>.<hw version minor>/<hw name>-<sw version major>.<sw version minor>.<git hash>.bin

```

 e.g. 
 ```
  s2740vc-v1-1.0.68e34de6.bin 
 /fs/microsd/fw/io.px4.sapog/1.1/sapog-1.7.87c7bc0.bin
 ```

基于ROMFS的更新遵循以下的模型，但是文件名中包含```_```前缀，因此我们添加的固件在:

  ```
  /etc/uavcan/fw/<device name>/<hw version major>.<hw version minor>/_<hw name>-<sw version major>.<sw version minor>.<git hash>.bin
  ```

## 将二进制文件放入PX4 ROMFS

最终生成的文件的位置为:

- S2740VC ESC: `ROMFS/px4fmu_common/uavcan/fw/com.thiemar.s2740vc-v1/1.0/_s2740vc-v1-1.0.<git hash>.bin`
- Pixhawk ESC 1.6: `ROMFS/px4fmu_common/uavcan/fw/org.pixhawk.px4esc-v1/1.6/_px4esc-v1-1.6.<git hash>.bin`
  - Pixhawk ESC 1.4: `ROMFS/px4fmu_common/uavcan/fw/org.pixhawk.sapog-v1/1.4/_sapog-v1-1.4.<git hash>.bin``
  - Zubax GNSS v1: `ROMFS/px4fmu_common/uavcan/fw/com.zubax.gnss/1.0/gnss-1.0.<git has>.bin`
  - Zubax GNSS v2: `ROMFS/px4fmu_common/uavcan/fw/com.zubax.gnss/2.0/gnss-2.0.<git has>.bin`

注意ROMFS/px4fmu_common目录将会挂载在Pixhawk的/etc目录下。

### 开始固件升级过程

<aside class="note">
当使用的是 [PX4飞行控制栈](../2_Concepts/flight_stack.md)时, 在‘电源配置’部分中启用UAVCAN，并在尝试升级UAVCAN固件之前要重启系统。
</aside>

或者可以通过以下方式在NSH上手动启动UAVCAN固件升级进程：

```sh
uavcan start
uavcan start fw
```
