# 1.编译方法

`A:\NationalChip\GX6613H1\gx_api_dev`目录下的`mico_customer_evn.sh`中的`export BUILD_NAYTAL_DVBC=yes`其余为no

# 2.USB升级文件制作方法

 1.将编译产生的`ecos.img`复制到`A:\NationalChip\GX6613H1\suma_mergebin_nayatel\bin_ecos\`,然后执行`mksign.sh`

2.将生成的`ecos_signed.img`复制到USB升级包中，执行`upgrade_file_merge.exe`

3.注意修改`FileDescriptor.xml`中的内容

```xml
	<!--FileDescriptor>
		<FileNmae>flash.bin</FileNmae>
		<FlashType>EM_FLASH_TYPE_SPI</FlashType>
		<FlashOffset>0x0</FlashOffset>
	</FileDescriptor-->

 	<FileDescriptor>
		<FileNmae>ecos_signed.img</FileNmae>
		<FlashType>EM_FLASH_TYPE_SPI</FlashType>
		<FlashOffset>0x50000</FlashOffset>
	</FileDescriptor>

 	<FileDescriptor>
		<FileNmae>root_cramfs.img</FileNmae>
		<FlashType>EM_FLASH_TYPE_SPI</FlashType>
		<FlashOffset>0x650000</FlashOffset>
	</FileDescriptor>
```

如果升级了固件，需要将`root_cramfs.img`从编译目录拷贝至制作USB升级文件的目录中

# 2.国芯系列项目增加新工程方法

1.`A:\NationalChip\GX6613H1\gx_api_dev`目录下的`mico_customer_evn.sh`增加如下内容，以`naytal`为例:

```sh
export BUILD_NAYTAL_DVBC=yes
```

```sh
if [ $BUILD_NAYTAL_DVBC == yes ];
then
	export CONFIG_MEDIA=yes
	export SUMACA_SUPPORT=yes
	export DVB_HDCP_SUPPORT=no
	export SUMACA_DMCA_BENGAL=no
	export CABLE=yes
	export BOARD_MC6827_GX6617_VER2=no
fi
```

```sh
echo BUILD_NAYTAL_DVBC=$BUILD_NAYTAL_DVBC
```

2.`A:\NationalChip\GX6613H1\gx_api_dev\tools`目录下的`mkimg`文件增加如下内容：

```sh
if [ $BUILD_VMX_SOURCES_DVBC == yes ]||[ $BUILD_SUMA_DVBC == yes ]||[ $BUILD_NAYTAL_DVBC == yes ]||[ $BUILD_SUMA_DVBC_DMCA == yes ]||[ $BUILD_CDCA_DVBC == yes ]; 
then
	cp $GXSRC_PATH/src/driver/libgxlowpower/$GX_CHIP_SERIES_TYPE/mc6827_gxlowpower.fw rootfs_ecos/lib/firmware/gxlowpower.fw	
fi
```

3.`A:\NationalChip\GX6613H1\gx_api_dev`目录下的`unsetenv.sh`文件增加如下内容：

```sh
unset BUILD_NAYTAL_DVBC
```

4.`A:\NationalChip\GX6613H1\gx_api_dev\src`目录下的Makefile文件增加如下内容：

```make
ifeq ($(BUILD_NAYTAL_DVBC), yes)
MICO_SRC_PATH=api_suma/naytal_dvbc
MICO_INCLUDE_PATH=api_suma/naytal_dvbc/include
endif
```

```make
ifeq ($(BUILD_NAYTAL_DVBC), yes)
CFLAGS += -I./$(MICO_SRC_PATH)/APP/Gui/pvr_7581
else
```

# 3.如何更改Tuner型号

在`A:\NationalChip\GX6613H1\gx_api_dev\env\gx3113u1\demo\ecos`的`setenv.sh`文件中修改如下内容：

```sh
#export DVB_TUNER_TYPE=TUNER_R836_C
#export DVB_TUNER_ATTACH=R836_C_ATTACH
export DVB_TUNER_TYPE=TUNER_MXL608_C
export DVB_TUNER_ATTACH=MXL608_C_ATTACH
```

```sh
export TUNER_I2C_CHIPADDRESS=0xc0
#export TUNER_I2C_CHIPADDRESS=0xF8
```

# 4.如何更新国芯提供的固件

1.将国芯提供的固件文件放到`A:\NationalChip\GX6613H1\gx_api_dev\client\ecos\gx3113u1\suma_naytal_dvbc\file_tree\rootfs_ecos\lib\firmware`

2.将`main.c`中的如下位置的`.dst_addr`按照国芯提供的petch进行修改

```c
GxSecureLoader firm = 
{
	.loader   = 0,
	.size	  = 0,
	.dst_addr = 0x40000,
};
```

