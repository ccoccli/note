# 1.工具链

## 1.1编译服务器

```shell
192.168.5.31
```

## 1.2路径

```shell
/usr/local/linaro/arm-gnu-toolchain-12.3.rel1-x86_64-aarch64-none-linux-gnu
```

# 2.编译

## 2.1代码路径

```shell
git clone git@192.168.5.92:sym6_ddk
```

## 2.2编译配置

![1](..\res\1.png)

## 2.3编译命令

```shell

source envsetup symphony6.cfg
make
```

# 3.运行

```shell
export LD_LIBRARY_PATH=/mnt/nfs/git/sym6_ddk/linux/pub/shared_lib_striped:$LD_LIBRARY_PATH

mkdir /mnt/nfs && udhcpc && mount -t nfs -o intr,nolock,rsize=1024,wsize=1024 192.168.5.31:/home/gengnx /mnt/nfs && cd /mnt/nfs/git/sym6_ddk/sdkproduct/mico/test && ./symphony.bin
```
