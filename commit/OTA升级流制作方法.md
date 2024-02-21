![image-20230704103932077](..\res\image-20230704103932077.png)

# 1.手动升级

## 1.选择不带7890的文件夹，把编译出来的symphony_lzma.img拷贝进去，然后注意xml文件的修改，具体如下：

![image-20230704104046686](..\res\image-20230704104046686.png)

1.<PID>7890</PID>自动触发时用到，手动无需关心

2.<tableid>0xaa</tableid>自动触发时用到，手动无需关心

3.<MID>0x84666663</MID>在代码的`stbinfo.c`的宏定义中，需要保持一致

4.<software_version>02</software_version>升级的软件版本

5.<software_version_needed>02</software_version_needed>需要与`software_version`保持一致

6.<download_pid>7895</download_pid>与manual OTA界面的download PID保持一致

7.<Freq>474</Freq> OTA流的频点

8.<Modulation>3</Modulation> 仅为C专用，其中3为64QAM, 4为128QAM， 5为256QAM

9.<Symbolrate>6875</Symbolrate> 符号率

## 2.修改好信息后，先用下图工具生成`UPG.pnd`,具体配置如下图：

![1688438888029](C:\Users\ccocc\Documents\WeChat Files\wxid_pfr3410alsi422\FileStorage\Temp\1688438888029.png)

## 3.生成TS流

![1688439001703](C:\Users\ccocc\Documents\WeChat Files\wxid_pfr3410alsi422\FileStorage\Temp\1688439001703.png)

# 2.自动触发

## 1.选择带7890的文件夹，把编译出来的symphony_lzma.img拷贝进去，然后注意xml文件的修改，和手动修改方法一样

## 2.修改好信息后，先用下图工具生成`UPG.pnd`,具体配置如下图：
![1688438888029](C:\Users\ccocc\Documents\WeChat Files\wxid_pfr3410alsi422\FileStorage\Temp\1688438888029.png)
## 3.生成TS流

![1688439267927](C:\Users\ccocc\Documents\WeChat Files\wxid_pfr3410alsi422\FileStorage\Temp\1688439267927.png)
