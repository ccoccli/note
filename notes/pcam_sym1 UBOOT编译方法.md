# **1.UBOO编译方法**

- `/home/zhidong/MT2230_4M64_FTA_A3/UBOOT`目录下执行`./release_symphony mini_ota`生成`u-boot.bin`
- 将u-boot.bin拷贝至`/home/zhidong/pcam_sym1/sign_tools/Sym_security_tools_143/symtool_format`然后执行`./pack_uboot.sh`，将会在`file_out`文件夹中生成`u-boot_last.img`
- 用winHex将`sbootloader_final.img.bin`和`u-boot_last.img`合并为一个`boot_signed.bin`
- 将编译好的`symphony_lzma.img`和上面合并好的`boot_signed.bin`复制到`/home/zhidong/pcam_sym1/sign_tools/flash_merge`然后执行`./merge_flash.sh`生成`flash.bin`

注意事项：

- 执行`./release_symphony mini_ota`之前需要配置工具链，执行`export PATH=/home/zhidong/toolchains/mips-4.3/bin:$PATH`
- 要是找不到`sbootloader_final.img.bin`，它在此处`/home/zhidong/pcam_sym1/sign_tools/flash_merge`