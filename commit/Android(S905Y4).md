# 1.交叉编译工具链

## 1.Bootloader

> Q : 在哪里修改工具链？(linaro)
>
> A : 在`A:\S905Y4\bootloader\uboot-repo\bl33\v2019\Makefile`

```Makefile
# set default to nothing for native builds
ifeq ($(HOSTARCH),$(ARCH))
CROSS_COMPILE ?=
endif

CROSS_COMPILE ?=/opt/gcc-linaro-7.3.1-2018.05-i686_aarch64-elf/bin/aarch64-elf-
export CROSS_COMPILE
```

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

## 1.Android部分

> Q:出现`playready/bgroupcert.dat', missing and no known rule to make it`如何解决？
>
> A:补丁:http://192.168.5.67/rd1/rd1/Amlogic/S905Y4/patch/0001-fix-drm-image-build-failed-issue-1-1.patch,根据补丁需将
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

cp build/u-boot.bin.signed /home/zhidong/S905Y4/device/amlogic/oppen/bootloader.img
cp build/u-boot.bin.sd.bin.signed /home/zhidong/S905Y4/device/amlogic/oppen/upgrade/
cp build/u-boot.bin.usb.signed /home/zhidong/S905Y4/device/amlogic/oppen/upgrade/
```

### 2.kernel部分

```shell
cd S905Y4
./mk oppen -v 5.4 -t user
```

### 3.安卓部分

```shell
cd S905Y4
source build/envsetup.sh
export BOARD_COMPILE_ATV=false
export BOARD_COMPILE_CTS=true
lunch oppen-user
make otapackage -j8
```

