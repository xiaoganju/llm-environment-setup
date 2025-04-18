# 显卡驱动&cuda&pytorch

# 一、显卡驱动安装

1、查看显卡 型号

1）查询命令

 lspci | grep -i vga

命令结果如下：

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/e1d0e2fb-bff9-4c9e-8bbc-833bca71976f.png)

2）根据型号device number查询型号

有的显卡会直接返回显卡所属系列以及型号，有的返回结果中不会显式给出，只提供了一个device number。这里给出根据device number查询型号方式。

如图中所示，可以看到十六进制的数字代码2504，这个就是device number，

1.  进入这个网址 [通过device number查询型号](https://admin.pci-ids.ucw.cz/mods/PC/10de?action=help?help=pci)
    
2.  输入2504
    

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/c3597d4d-a30b-4826-b364-b1c721771b57.png)

3.  查询结果说明
    

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/b5fae8a3-c014-4246-abdc-14b076e2802e.png)

2、安装驱动

1）根据显卡型号，下载对应的显卡驱动

1.  上面查到的显卡是 GA106 \[GeForce RTX 3060 Lite Hash Rate\]
    
2.  进入NVIDIA官网下载驱动地址 [NVIDIA Driver Downloads](https://www.nvidia.com/Download/index.aspx?lang=en-us)，点击search下载
    

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/4ee1d006-8784-47cd-b0fb-ac47506c8c7d.png)

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/e77c8683-225d-48f6-a69e-2701a8176b77.png)

2）安装驱动

1.  禁用nouveau
    

sudo vim /etc/modprobe.d/blacklist.conf

在文件末尾添加：

blacklist nouveau

blacklist lbm‐nouveau

options nouveau modeset=0

alias nouveau off

alias lbm‐nouveau off

对刚才修改的文件进行更新：sudo update-initramfs -u

重启计算机：sudo reboot

查看nouveau是否禁用成功：lsmod | grep nouveau，执行完这句，如没有任何输出，表示禁用成功。

2.  进入驱动所在目录，给驱动文件付权限，然后安装
    

sudo chmod +x NVIDIA-Linux-x86\_64-510.68.02.run

sudo sh NVIDIA-Linux-x86\_64-510.68.02.run  -no-opengl-files -no-x-check -no-nouveau-check

**注：将安装文件.run替换为您实际下载的版本**

3.  挂载 Nvidia 驱动
    

sudo modprobe nvidia

4.  查看驱动是否安装成功
    

nvidia-smi

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/c5a7a0ba-5511-434b-8002-3c69614ee15f.png)

# 二、cuda安装

1、下载cuda

1）找到正确的cuda版本

要注意，cuda版本需要与nvidia版本对应。二者版本对应关系在 [这里](https://docs.nvidia.com/cuda/cuda-toolkit-release-notes/) 查找。

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/5867c38f-af79-494b-8b08-e97a4143a127.png)

2）下载cuda，在[这里](https://developer.nvidia.com/cuda-downloads?target_os=Linux&target_arch=x86_64&Distribution=Ubuntu&target_version=20.04&target_type=runfile_local)下载

如下图，选择好后进行下载。

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/80ce9ffe-3bfa-442b-90fb-a3df65208d3b.png)

文件较大，建议通过浏览器下载，如果通过wget下载，可能会遇到“wget下载cuda 段错误核心已转储  ”错误

解决方式：

```shell
ulimit -a
ulimit -s unlimited
wget -c
https://developer.download.nvidia.com/compute/cuda/12.3.2/local_installers/cuda_12.3.2_545.23.08_linux.run

```

1.  安装cuda
    

```shell
sudo sh cuda_11.6.2_510.47.03_linux.run
```

4）为cuda添加环境变量

```shell
export LD_LIBRARY_PATH=$LD_LIBRARY_PATH:/usr/local/cuda-11.8/lib64
export PATH=/usr/local/cuda-11.8/bin:$PATH
source /etc/profile
```

5）验证是否安装成功

```shell
nvcc  -V
```

# 三、Pytorch

根据cuda版本和python版本自己所需要的torch和torchvision

下载地址：[https://download.pytorch.org/whl/torch\_stable.html](https://download.pytorch.org/whl/torch_stable.html)

下载后，上传到服务器，安装即可