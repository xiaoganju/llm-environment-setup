# 安装pytorch

1、查看cuda版本

nvcc  -V

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4j6OJQQwBBmKq3p8/img/5fd1290f-640f-4e82-8a38-6861f17580a6.png)

2、查看python版本

python -V

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/4j6OJQQwBBmKq3p8/img/567234e7-9a3d-477e-b549-215770602155.png)

3、下载pytorch

下载地址：[https://download.pytorch.org/whl/torch\_stable.html](https://download.pytorch.org/whl/torch_stable.html)

如上cuda和python版本应下载：

torchaudio-2.0.0+cu118-cp39-cp39-linux\_x86\_64.whl  

torchvision-0.15.1+cu118-cp39-cp39-linux\_x86\_64.whl

torch-2.0.0+cu118-cp39-cp39-linux\_x86\_64.whl

4、安装pytorch

pip install torch-2.0.0+cu118-cp39-cp39-linux\_x86\_64.whl

pip install torchvision-0.15.1+cu118-cp39-cp39-linux\_x86\_64.whl