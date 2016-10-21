# FriendlyArm Tiny4412 开发板使用Uboot引导的一种可行方案

## step1

使用uboot_tiny4412_0929版本，修改include/configs/tiny4412.h

```c
#define CONFIG_BOOTCOMMAND	"movi read kernel 0 40008000;movi read rootfs 0 41000000 400000;bootm 40008000 41000000"
#define CONFIG_BOOTARGS	""
//修改为
#define CONFIG_BOOTCOMMAND	"mmc read 0 40008000 427 3000;mmc read 0 41000000 3427 800;bootm 40008000 41000000"
#define CONFIG_BOOTARGS	"console=ttySAC0,115200n8 androidboot.console=ttySAC0 ctp=2 skipcali=y vmalloc=384m ethmac=1C:6F:65:34:51:7E androidboot.selinux=permissive lcd=S702"
```

    make tiny4412_config
    make

生成u-boot.bin文件

## step2

生成FakeSuperboot.bin文件，sd_fuse目录下

    make
    ./mkbl2 ../u-boot.bin tiny4412/bl2.bin 14336

进入tiny4412目录

```sh
dd if=E4412_N.bl1.bin of=FakeSuperboot.bin
dd if=bl2.bin of=FakeSuperboot.bin seek=16
dd if=../../u-boot.bin of=FakeSuperboot.bin seek=48
dd if=E4412_tzsw.bin of=FakeSuperboot.bin seek=704
```


## step3

配置内核

    cp tiny4412_android_defconfig .config
    make menuconfig

System Type下取消Support TrustZone-enabled Trusted Execution Environment
(uboot可以引导)
和Exynos4 CPUIDLE Feature(防止崩溃)

或者使用提供的.config文件，编译内核

## step4

用Superboot.bin烧写SD卡，用FakeSuperboot.bin烧写开发板，注意修改配置文件