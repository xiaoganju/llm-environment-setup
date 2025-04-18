# 启智大脑在线部署&启动文档

# 一、文档说明

本文将介绍，在一台只有显卡的服务器上，如何在线部署启智大脑工程。

本文将逐一详细阐述每个安装步骤，并在每个步骤随后提供验证方式，目的是让阅读者能够仅凭文档指导就能成功完成安装部署。

下面以一台Ubuntu系统服务器为例，开始介绍本文内容。

# 二、部署要求

在开始着手部署以前，先列举一下启智大脑有关部署方面的要求。

|   |  部署要求  |  备注  |
| --- | --- | --- |
|  系统  |  ubuntu20.04  |  本文基于ubuntu20.04整理，其他系统版本可能一些差异，但可实现  |
|  显存  |  GPU显存16G以上  |  chatglm-6B :16G 以上 qianwen：147G以上  |
|  基础编译环境  |  gcc g++  |   |
|  显卡驱动  |  nvidia  |   |
|  计算推理  |  cuda&cudnn  |   |
|  依赖组件  |  mysql、redis、docker、nvidia-docker2  |   |
|  依赖数据  |  图形数据库、词向量数据  |   |
|  推理大模型  |  chatglm、llama2、qianwen  |   |
|  语言环境  |  python3.9、python依赖  |   |

# 三、基础编译环境安装

后续的显卡驱动和cuda安装需要基础的编译环境，下面是编译库的安装方式：

1、gcc安装

sudo apt-get install build-essential

2、验证gcc是否安装成功

        gcc --version

3、g++安装

sudo apt-get install g++

4、安装编译

sudo apt-get install make

成功后将显示如下信息：

gcc (Ubuntu 9.4.0-1ubuntu1~20.04.1) 9.4.0

        g++ is already the newest version (4:9.3.0-1ubuntu2)

        make is already the newest version (4.2.1-1.2)

# 四、显卡驱动安装

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

# 五、cuda安装

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

# 六、cudnn安装

1、下载cuda版本对应的deb cudnn包   [下载地址](https://developer.nvidia.com/rdp/cudnn-download)

 2、添加密钥

```shell
sudo dpkg -i cudnn-local-repo-ubuntu2004-8.9.7.29_1.0-1_amd64.deb
sudo cp /var/cudnn-local-repo-*/cudnn-local-*-keyring.gpg /usr/share/keyrings/

```

**注意：替换为您实际下载的包**

3、安装

 1）先找到cuda和cudnn版本

```shell
nvcc  -V  
```
```shell
进入/var/cudnn-local-repo* ，ls一下，找到 $name = xxx
```

2）编辑安装命令

```shell
sudo apt-get install libcudnn8=8.x.x.x-1+cudaX.Y
sudo apt-get install libcudnn8-dev=8.x.x.x-1+cudaX.Y
sudo apt-get install libcudnn8-samples=8.x.x.x-1+cudaX.Y
```

**注：将命令****X.Y** **and** **8.x.x.x** **替换为 CUDA and cuDNN 版本**。比如：

```shell
sudo apt-get update
sudo apt-get install libcudnn8=8.9.7.29-1+cuda12.2
sudo apt-get install libcudnn8-dev=8.9.7.29-1+cuda12.2
sudo apt-get install libcudnn8-samples=8.9.7.29-1+cuda12.2

```

3）验证是否安装成功

cudnn带有测试用例，跑一下测试用例看是否成功。

```shell
cp -r /usr/src/cudnn_samples_v8/ $HOME
cd  $HOME/cudnn_samples_v8/mnistCUDNN
make clean && make
./mnistCUDNN
```

**注：如果在运行make指令的时候，出现了不能编译，提示缺少FreeLmage.h与资源库的问题，则运行下面指令安装FreeLmage相关文件：**

```shell
sudo apt-get install libfreeimage3 libfreeimage-dev
```

# 七、nvidia-docker2安装

1、安装docker

```shell
sudo apt update
sudo apt install -y docker.io

```

2、安装NVIDIA Container Toolkit

安装NVIDIA Container Toolkit以支持在Docker容器中运行NVIDIA GPU工作负载。

```shell
sudo distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
&& curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
&& echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/$distribution $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null \
&& sudo apt-get update \
&& sudo apt-get install -y nvidia-docker2

```

3、启动docker

```shell
sudo systemctl start docker
```

4、验证安装

```shell
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

**注：替换成实际的镜像标签**

# 八、工程部署

经过上述步骤，启智大脑依赖组件的安装部署已经基本完成，接下来开始工程本身部署。

1、工程介绍

此处不深入探讨工程的具体实现细节，而是专注于介绍与部署相关的信息。目的是使得部署人员对整个工程有一定了解，从而更轻松地进行部署工作。

|  语言栈  |  python  |   |
| --- | --- | --- |
|  框架  |  Flask  |   |
|  本地存储  |  mysql、redis  |   |
|  本地模型  |  glm、m3elarge  |   |
|  本地数据  |  语义索引库 图数据库  |   |
|  外部服务  |  图谱服务  |   |

2、部署方式

启智大脑工程部署支持两种方式：

1）docker部署

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/2d02d11f-8b9e-4fae-be4b-944514f70ef3.png)

如上图所示：

所有服务均打包成docker，docker通过compose方式管理

```shell
sh deploy.sh
```

2）conda环境移植部署

```shell
1、
 bash Miniconda3-latest-Linux-x86_64.sh
2、
source .bashrc

3、create conda env
which conda 找到conda env 目录
如：/home/qiyuan/miniconda3/bin/conda
那么：/home/qiyuan/miniconda3/envs


将env.tar.gz 放在这里 解压
tar -xzf environment.tar.gz -C unpacked_environment
conda activate <environment_name>

4、工程启动
1）创建目录，放置代码
2）创建pretrain_models目录，放置3个模型文件
3）修改图谱服务地址
4）CUDA_VISIBLE_DEVICES=2 python main.py
```