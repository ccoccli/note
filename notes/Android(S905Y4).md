> 代码地址：git clone git@192.168.5.92:s905_y4

# 1.编译环境配置

## 0.编译服务器配置说明

> 1.服务器系统：Ubuntu22.04.4LTS
>
> 2.gcc : 9(不能大于9， 否则编译报错)
>
> 3.软件包安装列表 : `openjdk-8-jdk automake make git gperf zip dos2unix bison perl gcc(9) g++ tig pkg-config cpp-aarch64-linux-gnu gcc-4.8-aarch64-linux-gnu unzip lib32z1 libx11-dev lib32z-dev ccache gitk xmllint libxml2-utils libssl-dev`
>
> 4.编译前需要运行`B:\scripts\s905_y4.sh`配置工具链，否则编译报错

## 1.Bootloader

> Q : 在哪里修改工具链？(riscv)
>
> A : 在`A:\S905Y4\bootloader\uboot-repo\bl30\src_ao\demos\amlogic\makedefs`

```makefile
ifeq (${COMPILER}, gcc)

#
# The command for calling the compiler.
#
#ifeq ($(ARCH_CPU), RISC_V_N205)
CC=/home/zhidong/toolchains/riscv-none-gcc/8.2.0-2.2-20190521-0004/bin/riscv-none-embed-gcc
CPP=/home/zhidong/toolchains/riscv-none-gcc/8.2.0-2.2-20190521-0004/bin/riscv-none-embed-g++

AFLAGS+=-march=rv32imc -mabi=ilp32 -mcmodel=medany \
	--specs=nano.specs --specs=nosys.specs -ffunction-sections \
	-fdata-sections -fno-common -fgnu89-inline

WARNINGS=-Werror -Wall -Wextra -Wshadow -Wpointer-arith -Wbad-function-cast -Wsign-compare \
	-Waggregate-return -Wstrict-prototypes -Wmissing-prototypes -Wmissing-declarations -Wunused
CFLAGS+= -g -march=rv32imc -mabi=ilp32 -mcmodel=medany -O2 \
	--specs=nano.specs --specs=nosys.specs -ffunction-sections \
	-fdata-sections -fno-common -fgnu89-inline

CFLAGS+=$(WARNINGS)

AR=/home/zhidong/toolchains/riscv-none-gcc/8.2.0-2.2-20190521-0004/bin/riscv-none-embed-ar
LD=/home/zhidong/toolchains/riscv-none-gcc/8.2.0-2.2-20190521-0004/bin/riscv-none-embed-ld
OBJCOPY=/home/zhidong/toolchains/riscv-none-gcc/8.2.0-2.2-20190521-0004/bin/riscv-none-embed-objcopy
NM=/home/zhidong/toolchains/riscv-none-gcc/8.2.0-2.2-20190521-0004/bin/riscv-none-embed-nm
OBJDUMP=/home/zhidong/toolchains/riscv-none-gcc/8.2.0-2.2-20190521-0004/bin/riscv-none-embed-objdump
#endif
endif
```

# 2.编译

> 编译完成后不要轻易执行clean,否则会编译很长时间！！！！！！！！！！！

## 0.bootloader部分

> Q:多重定义yylloc'错误的解决方案
>
> A:
>
> ​	1.卸载原有的gcc(`sudo apt-get purge gcc`), 
>
> ​	2.安装gcc9(`sudo apt-get install gcc-9`)
>
> ​	3.配置符号链接(`sudo update-alternatives --install /usr/bin/gcc gcc /usr/bin/gcc-9 100`),
>
> ​	4.`sudo ln -s /usr/bin/gcc /usr/bin/cc`
>
> 详细解决方案见：`https://github.com/BPI-SINOVOIP/BPI-M4-bsp/issues/4`

## 1.Android部分

> Q:出现`playready/bgroupcert.dat', missing and no known rule to make it`如何解决？
>
> A:补丁:`http://192.168.5.67/rd1/rd1/Amlogic/S905Y4/patch/0001-fix-drm-image-build-failed-issue-1-1.patch,`根据补丁需将
>
> `A:\S905Y4\device\amlogic\common\media.mk`中的如下部分屏蔽(180-191行)

```makefile
ifeq ($(BUILD_WITH_PLAYREADY_DRM),true)
PRODUCT_PACKAGES += \
  libplayreadymediadrmplugin \
  libplayready \
  9a04f079-9840-4286-ab92-e65be0885f95

PRODUCT_COPY_FILES += \
    $(BOARD_AML_VENDOR_PATH)/prebuilt/libmediadrm/playready/keycert/zgpriv.dat:$(TARGET_COPY_OUT_VENDOR)/etc/drm/playready/zgpriv.dat \
    $(BOARD_AML_VENDOR_PATH)/prebuilt/libmediadrm/playready/keycert/bgroupcert.dat:$(TARGET_COPY_OUT_VENDOR)/etc/drm/playready/bgroupcert.dat \
    $(BOARD_AML_VENDOR_PATH)/prebuilt/libmediadrm/playready/keycert/zgpriv_protected.dat:$(TARGET_COPY_OUT_VENDOR)/etc/drm/playready/zgpriv_protected.dat

endif
```

> Q:出现`/bin/bash: vendor/amlogic/common/tdk_v3/ta_export/scripts/sign_ta_auto.py: No such file or directory`该如何解决？
>
> A:首先检查TDK仓库的代码是否已经下载完整，如果下载完整后仍然存在此问题，需要去`vendor/amlogic/common/tdk_v3/ta_export/`目录下执行`git checkout -f`,然后回到项目目录重新执行编译命令

## 2.编译命令

### 1.bootloader部分

```shell
source ~/.bashrc
cd S905Y4/bootloader/uboot-repo/
./mk s4_ap222 --vab --avb2

cp build/u-boot.bin.signed ../../device/amlogic/oppen/bootloader.img
cp build/u-boot.bin.sd.bin.signed ../../device/amlogic/oppen/upgrade/
cp build/u-boot.bin.usb.signed ../../device/amlogic/oppen/upgrade/
```

### 2.kernel部分

```shell
cd S905Y4
./mk oppen -v 5.4 -t user(userdebug)
```

> userdebug可进入debug模式

### 3.Android部分

```shell
cd S905Y4
source build/envsetup.sh
export BOARD_COMPILE_ATV=false
export BOARD_COMPILE_CTS=true
lunch oppen-user(oppen-userdebug)
make otapackage -j8
```

> userdebug可进入debug模式

# 3.开发说明

## 0.打印kernel日志

```shell
su
```
```shell
dmesg
```

## 1.IR

### 1.物理键值到Linux键值的映射

```shell
B:\AOSP\common\arch\arm64\boot\dts\amlogic\meson-s4.dtsi(1769~1778行)
```

```dtsi
	ir: ir@8000 {
		compatible = "amlogic, meson-ir";
		reg = <0x0 0xfe084040 0x0 0xA4>,
			<0x0 0xfe084000 0x0 0x20>;
		status = "okay";
		protocol = <REMOTE_TYPE_NEC>;
		interrupts = <GIC_SPI 22 IRQ_TYPE_EDGE_RISING>;
		map = <&custom_maps>;
		max_frame_time = <200>;
	};
```

> 1.status：okay/disable 是否使能遥控器
>
> 2.protocol: 遥控器协议类型，定义：common/include/dt-bindings/input/meson_rc.h 
>
> #define REMOTE_TYPE_UNKNOWN 0x00 
>
> #define REMOTE_TYPE_NEC 0x01 
>
> #define REMOTE_TYPE_DUOKAN 0x02 
>
> #define REMOTE_TYPE_XMP_1 0x03 
>
> #define REMOTE_TYPE_RC5 0x04 
>
> #define REMOTE_TYPE_RC6 0x05 
>
> #define REMOTE_TYPE_TOSHIBA 0x06 
>
> #define REMOTE_TYPE_RCA 0x08
>
>  #define REMOTE_TYPE_RCMM 0x0a

```shell 
B:\AOSP\common\arch\arm64\boot\dts\amlogic\meson-ir-map.dtsi(203~233行)
```

```dtsi
custom_maps: custom_maps {
	mapnum = <5>;
	map0 = <&map_0>;
	map1 = <&map_1>;
	map2 = <&map_2>;
	map3 = <&map_3>;
	map4 = <&map_4>;

	map_0: map_0{
	};
	map_1: map_1{
	};
	map_2: map_2{
	};
	map_3: map_3{
	};

	map_4: map_4{
		mapname = "mico-3766";
		customcode = <0x00df>;
		release_delay = <80>;
		size  = <25>;
		keymap = <REMOTE_KEY(0xDC, KEY_POWER)
			REMOTE_KEY(0x8B, KEY_MUTE)
			REMOTE_KEY(0xC7, KEY_RED)
			REMOTE_KEY(0xC6, KEY_GREEN)
			REMOTE_KEY(0xC5, KEY_YELLOW)
			REMOTE_KEY(0xC4, KEY_BLUE)
			REMOTE_KEY(0xDE, KEY_UP)
			REMOTE_KEY(0xD6, KEY_DOWN)
			REMOTE_KEY(0xDB, KEY_LEFT)
			REMOTE_KEY(0xD8, KEY_RIGHT)
			REMOTE_KEY(0xDA, KEY_ENTER)
			REMOTE_KEY(0xD4, KEY_MENU)
			REMOTE_KEY(0xD7, KEY_TAB)
			REMOTE_KEY(0xD0, KEY_VOLUMEUP)
			REMOTE_KEY(0xCC, KEY_VOLUMEDOWN)
			REMOTE_KEY(0x9A, KEY_0)
			REMOTE_KEY(0x8E, KEY_1)
			REMOTE_KEY(0x86, KEY_2)
			REMOTE_KEY(0x8F, KEY_3)
			REMOTE_KEY(0x92, KEY_4)
			REMOTE_KEY(0x87, KEY_5)
			REMOTE_KEY(0x93, KEY_6)
			REMOTE_KEY(0x96, KEY_7)
			REMOTE_KEY(0x82, KEY_8)
			REMOTE_KEY(0x97, KEY_9)>;
	};
};
```

> 1.mapnum = <5>;支持多少种遥控器
>
> 2.mapname = "mico-3766";遥控器名字
>
> 3.customcode = <0x00df>;遥控器客户码
>
> 4.size  = <25>;遥控器有多少个按键
>
> 5.REMOTE_KEY(0x82, KEY_8)0x82为遥控器的物理键值， KEY_8为映射到Linux的键值

### 2.linux映射到Android的键值

```shell
B:\AOSP\device\amlogic\common\products\mbox\Vendor_0001_Product_0001.kl
```

```kl
key 116 KEYCODE_POWER
key 113 KEYCODE_MUTE
key 0x18e KEYCODE_PROG_RED
key 0x18f KEYCODE_PROG_GREEN
key 0x190 KEYCODE_PROG_YELLOW
key 0x191 KEYCODE_PROG_BLUE
key 103 KEYCODE_DPAD_UP
key 108 KEYCODE_DPAD_DOWN
key 105 KEYCODE_DPAD_LEFT
key 106 KEYCODE_DPAD_RIGHT
key 96 KEYCODE_ENTER
key 139 KEYCODE_MENU
key 15 BACK
key 158 KEYCODE_BACK
key 115 KEYCODE_VOLUME_UP
key 114 KEYCODE_VOLUME_DOWN
key 11 KEYCODE_0
key 2 KEYCODE_1
key 3 KEYCODE_2
key 4 KEYCODE_3
key 5 KEYCODE_4
key 6 KEYCODE_5
key 7 KEYCODE_6
key 8 KEYCODE_7
key 9 KEYCODE_8
key 10 KEYCODE_9
```

> 1.key为固定格式
>
> 2.第二列的数字为Linux的键值，定义在`B:\AOSP\common\include\uapi\linux\input-event-codes.h`
>
> 3.第三列为Android中的键值，定义在`B:\AOSP\frameworks\base\core\java\android\view\KeyEvent.java`

编辑完后，需要重新编译kernel,并且安卓部分直接执行make即可。

### 3.如何通过debug模式获取到遥控器的键值？

> 1.首先重新编译kernel --->      `./mk oppen -v 5.4 -t userdebug`
>
> 2.重新lunch android  --->        `lunch oppen-userdebug`
>
> 3.重新编译Android --->           `make`
>
> 4.烧录进去后开始执行以下命令
>
> ​	1.`su`
>
> ​	2.`echo 1 > sys/class/remote/amremote/debug_enable`
>
> ​	3.`echo 8 > proc/sys/kernel/printk`
>
> 5.开始按遥控器，串口则会打印出相关的键值

### 4.查看支持的IR协议

```shell
cat sys/class/remote/amremote/protocol
```

### 5.查看支持的遥控器用户码

```shell
cat sys/class/remote/amremote/map_tables
```

### 6.查看支持的遥控器

```shell
echo 0x00df > sys/class/remote/amremote/keymap
```

```shell
cat sys/class/remote/amremote/keymap
```

### 7.待机唤醒按键设置

```shell
B:\AOSP\bootloader\uboot-repo\bl30\src_ao\demos\amlogic\n200\s4\s4_ap222\power.c
```

```c
static IRPowerKey_t prvPowerKeyList[] = {
	{ 0xef10fe01, IR_NORMAL}, /* ref tv pwr */
	{ 0xba45bd02, IR_NORMAL}, /* small ir pwr */
	{ 0xef10fb04, IR_NORMAL}, /* old ref tv pwr */
	{ 0xf20dfe01, IR_NORMAL},
	{ 0xe51afb04, IR_NORMAL},
	{ 0xff00fe06, IR_NORMAL},
	{ 0x3ac5bd02, IR_CUSTOM},
	
	//mico 3766 power
	{ 0x23dc00df, IR_CUSTOM},
};
```

## 2.tuner

### 0.驱动路径

```shell
AOSP/device/amlogic/common/tuner/64/cxd2856_fe_64.ko
```

### 1.锁频测试

```shell
echo debug_on > /sys/class/cxd2856/cxd2856_debug
echo init 0 > /sys/class/cxd2856/cxd2856_debug
echo tune 0 1 474000000 6900000 8000000 6 > /sys/class/cxd2856/cxd2856_debug
echo tuner_status > /sys/class/cxd2856/cxd2856_debug
cat /sys/class/cxd2856/cxd2856_debug
```

### 2.DTS文件修改

```shell
AOSP/common/arch/arm64/boot/dts/amlogic/s4_s905y4_ap222_drm.dts
```

```
dvb-extern {
		compatible = "amlogic, dvb-extern";
		dev_name = "dvb-extern";
		status = "okay";

		/* dvb hardware main power. */
		dvb_power_gpio = <&gpio GPIOZ_5 GPIO_ACTIVE_LOW>;
		dvb_power_value = <0>;
		dvb_power_dir = <0>; /* 0: out, 1: in. */

		fe_num = <1>; /* interal 2 * dvb-c, extern 1 * dvb-s */
		fe0_demod = "cxd2856";
		fe0_i2c_adap_id = <&i2c4>;
		fe0_demod_i2c_addr = <0xD8>;
		fe0_reset_value = <0>;
		fe0_reset_gpio = <&gpio GPIOZ_4 GPIO_ACTIVE_HIGH>;
		fe0_reset_dir = <0>; /* 0: out, 1: in. */
		fe0_ant_poweron_value = <0>;
		fe0_ant_power_gpio = <&gpio GPIOH_9 GPIO_ACTIVE_HIGH>;
		fe0_ts = <0>;
		fe0_tuner0 = <0>; /* use tuner3, only S. */

		tuner_num = <1>; /* tuner number, multi tuner support */
		tuner0_name = "r850_tuner";
		tuner0_i2c_adap = <&i2c4>;
		tuner0_i2c_addr = <0x1c>; /* 7bit */
	};

	dvb-demux {
		compatible = "amlogic sc2, dvb-demux";
		dev_name = "dvb-demux";
		status = "okay";

		reg = <0x0 0xfe000000 0x0 0x480000>;

		dmxdev_num = <6>;

		tsn_from = "demod";

		/*single demod setting */
		ts0_sid = <0x20>;
		ts0 = "serial-4wire"; /* tsinA: serial-4wire, serial-3wire */
		ts0_control = <0x0>;
		ts0_invert = <0>;

		ts1_sid = <0x21>;
		ts1 = "parallel";
		ts1_control = <0x0>;
		ts1_invert = <0>;

		ts2_sid = <0x22>;
		ts2 = "parallel";
		ts2_control = <0x0>;
		ts2_invert = <0>;

		pinctrl-names = "s_ts0";
		pinctrl-0 = <&dvb_s_ts0_pins>;
	};
```

## 3.如何打补丁

### 1.在源码目录下

```shell
repo forall -c "git checkout -f"
```

```shell
repo forall -c "git am --abort"
```

### 2.在补丁目录下

```shell
./apply_patches.sh -a . /home/ccoccli/sourceAOSP
```

## 4.DVB编译命令

### 0.APK说明

> LiveTV APK：   packages/apps/TV/  ----Jave UI
> Tif适配代码：  vendor/amlogic/common/apps/TvInput/DroidLogicTvInput/ ----管理多个输入接口的标准
> TVserver代码：vendor/amlogic/common/tv/------DVB 服务器
> AOSP\vendor\amlogic\common\external\dvb ----DVB中间件
> 通过linux 内核走到
> AOSP\common\drivers\amlogic\dvb dvb ----driver

### 1.编译

```shell
cd /AOSP/packages/apps/TV
```

```shell
m -j LiveTv
```

### 2.目标文件路径

```shell
AOSP/out/target/product/oppen/product/priv-app/LiveTv/LiveTv.apk
```

### 3.更新APK

```shell
adb root
```

```shell
adb remount
```

```shell
adb push LiveTv.apk  /product/priv-app/LiveTv/LiveTv.apk
```

```shell
adb reboot
```

### 4.查看APK打印

```shell
adb logcat
```

