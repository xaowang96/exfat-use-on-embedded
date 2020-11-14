

## 嵌入式Linux兼容exFAT格式

SD卡多数使用的是FAT32格式的文件系统，也有exFAT格式的，而部分Linux内核是没有支持exFAT格式的，需要使用第三方工具支持.本文简单介绍下在嵌入式平台下编译使用exfat-nofuse的过程,仅供参考.

### 1. SD卡文件系统

在Windows系统下，格式化SD卡会有三种文件系统可选择，分别是NTFS、FAT32和exFAT。

+ 其中NTFS格式是微软为硬盘或固态硬盘（SSD）创建的默认新型文件系统，在Windows系统下使用起来没有任何问题，而在Mac OSX和LInux是不能直接使用的，需要使用第三方软件。
+ FAT32 文件格式是一种通用格式，任何USB存储设备都可以使用此格式，可以在Windows、Linux和Mac OSX系统上使用。但是其最大支持的单个文件为4GB。
+ exFAT是微软创建的用于替代FAT32文件格式的新型文件格式。其最大支持1EB大小的文件（虽然我们好像用不到\~\_\~)，可以在Windows和Mac OSX上使用，但是没有文件日志功能。

### 2. PC端

在PC端加载第三方工具，兼容exFAT格式是比较容易的，只需要执行以下命令即可：
```C
Ububtu:
sudo apt install exfat-fuse exfat-utils
```
### 3. 嵌入式平台

在嵌入式端支持exFAT就没那么容易了(并不能直接安装23333)，我们可以编译exfat-nofuse的源码,生成内核驱动，在使用的时候加载驱动即可,以下以某mips架构平台为例（~_~）

#### 3.1 下载源码

在GitHub上可以下载源码，[exfat-nofuse](https://github.com/dorimanx/exfat-nofuse)

```C
git clone https://github.com/dorimanx/exfat-nofuse.git
```

#### 3.2 编译

+ 进入源码目录，`cd exfat-nofuse`

+ 修改Makefile 
  
  ```C
  #指定kernel编译结果路径和exfat-nofuse驱动install路径
  #KDIR	?= /lib/modules/$(shell uname -r)/build
  #MDIR	?= /lib/modules/$(shell uname -r)
  KDIR	?= PATH_FOR_LINUX_KERNEL        #linux kernel编译结果路径
  MDIR    ?= $(shell pwd)/__install 
  ```
  
+ 编译源码
  
  ```C
  #需要指定编译链，PATH_FOR_TOOLCHAIN表示使用的编译链路径 
  make CROSS_COMPILE=PATH_FOR_TOOLCHAIN/bin/mips-linux-
  
  #或者将编译链路径加入环境变量PATH中
  export PATH=$PATH:PATH_FOR_TOOLCHAIN/bin/
make    

  ```
  如果kernel没有编译过或者指定的路径不是kernel编译路径,则会编译出错,错误信息如下,此时只需要编译kernel或者指定正确的kernel编译路径即可.
  
  ```C
  #PATH_FOR_KERNEL_SOURCE 表示Linux kernel源码路径，而非编译结果路径
  make -C /PATH_FOR_KERNEL_SOURCE/platform/source/kernel/linux-4.9 M=/home/wxd/Host/Workspace/git/exfat-nofuse modules
  make[1]: Entering directory '/PATH_FOR_KERNEL_SOURCE/platform/source/kernel/linux-4.9'
  
    ERROR: Kernel configuration is invalid.
           include/generated/autoconf.h or include/config/auto.conf are missing.
           Run 'make oldconfig && make prepare' on kernel src to fix it.
  
  
    WARNING: Symbol version dump ./Module.symvers
             is missing; modules will have no dependencies and modversions.
  
    Building modules, stage 2.
  scripts/Makefile.modpost:42: include/config/auto.conf: No such file or directory
  make[2]: *** No rule to make target 'include/config/auto.conf'.  Stop.
  Makefile:1530: recipe for target 'modules' failed
  make[1]: *** [modules] Error 2
  make[1]: Leaving directory '/PATH_FOR_KERNEL_SOURCE/platform/source/kernel/linux-4.9'
  Makefile:37: recipe for target 'all' failed
  make: *** [all] Error 2
  
  ```

#### 3.3 使用

+ 加载驱动
  ```C
  # insmod /ipc/modules/exfat.ko 
[   31.728000] exFAT: Version 1.2.9
  ```
  
+ 挂载/卸载SD卡

  ```C
  # mount /dev/mmcblk0p1 /mnt/disc1/
  [  106.936000] [EXFAT] trying to mount...
  [  106.992000] [EXFAT] mounted successfully
      
  # umount /mnt/disc1/
  [  136.080000] [EXFAT] trying to unmount...
  [  136.084000] [EXFAT] unmounted successfully   
  
  ```

+ 卸载驱动

  ```C
  # lsmod 
  Module                  Size  Used by    Tainted: G  
  exfat                  95200  0 
  #     
  # rmmod exfat    
  #     
  # lsmod     
  Module                  Size  Used by    Tainted: G  
      
  ```

  

