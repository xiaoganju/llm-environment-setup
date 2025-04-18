# nvidia问题排查

1、查看nvidia驱动版本日志

cat /var/log/dpkg.log | grep nvidia

2、nvidia-smi 命令报错，无法查看版本，用下面命令替换

cat /proc/driver/nvidia/version

3、遇到nvidia驱动和显卡版本不匹配问题

报错信息：Failed to initialize NVML: Driver/library version mismatche

 解决方式：

1）卸载赶紧旧版本驱动 

    sudo apt-get --purge remove nvidia\*

    sudo apt-get purge nvidia\*

    sudo apt-get purge libnvidia\*

    （直到命令不输出任何内容：sudo dpkg --list | grep nvidia-\*）

  2）执行后，仍然出现：

    ii  linux-objects-nvidia-525-5.15.0-67-generic 5.15.0-67.74~20.04.1                amd64        Linux kernel nvidia modules for version 5.15.0-67 (objects)

    ii  linux-objects-nvidia-525-5.15.0-91-generic 5.15.0-91.101~20.04.1+1             amd64        Linux kernel nvidia modules for version 5.15.0-91 (objects)

    ii  linux-signatures-nvidia-5.15.0-67-generic  5.15.0-67.74~20.04.1                amd64        Linux kernel signatures for nvidia modules for version 5.15.0-67-generic

    ii  linux-signatures-nvidia-5.15.0-91-generic  5.15.0-91.101~20.04.1+1             amd64        Linux kernel signatures for nvidia modules for version 5.15.0-91-generic

    对这些进行卸载

    sudo apt purge linux-objects-nvidia-525-\* linux-signatures-nvidia-5.15.0-\*

**注：将命令中的驱动版本，替换为您实际下载的版本**