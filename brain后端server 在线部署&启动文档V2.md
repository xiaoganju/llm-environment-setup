# 启智大脑后端server 在线部署&启动文档V2

# 一、文档说明

本文将介绍，在一台只有显卡的服务器上，如何从0到1在线部署启智大脑工程。

本文将逐一详细阐述每个安装步骤，并在每个步骤随后提供验证方式，目的是让阅读者能够仅凭文档指导就能成功完成安装部署。

下面以一台Ubuntu系统服务器为例，开始介绍本文内容。

# 二、部署要求

在开始着手部署以前，先列举一下启智大脑有关部署方面的要求。

|   |  **部署要求**  |  **备注**  |
| --- | --- | --- |
|  系统  |  ubuntu20.04  |  本文基于ubuntu20.04整理，其他系统版本可能一些差异，但可实现  |
|  显存  |  GPU显存40G以上  |  根据启动模型size而定，6b模型需要20G以上  |
|  基础编译环境  |  gcc g++  |   |
|  显卡驱动  |  nvidia  |   |
|  计算推理  |  cuda、cudnn、nccl  |   |
|  依赖组件  |  mysql、redis、docker、nvidia-docker2、ElasticSearch  |   |
|  依赖模型  |  大模型文件、embedding 模型文件  |  大模型文件：比如qwen embedding 模型文件：m3e-large  |
|  依赖数据  |  图形数据库、语义索引库、分词包  |   |
|  代码运行环境  |  python3.9、python-requirements  |   |

# 三、部署流程

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/a/R1LXqJEYYhBgGErr/5738d3564de949f196f8cc9c1927441c1215.png)

如上图，启智大脑环境搭建支持两种方式，源码方式和docker方式。

5.1章节源码方式搭建，可同步了解启智大脑部分工作原理。

5.2章节docker部署，适合快速搭建。

读者根据需要自行选择。

# 四、搭建大模型运行环境

启智大脑的基座是大模型，在部署启智大脑之前，需要确保服务器可以运行大模型。

1.  服务器要求
    

|   |  要求  |  备注  |
| --- | --- | --- |
|  显存  |  GPU显存40G以上  |  根据启动模型大小而定  |
|  基础编译环境  |  gcc g++  |   |
|  显卡驱动  |  nvidia  |   |
|  计算推理、通信  |  cuda、cudnn、nccl  |  需要和系统、显卡版本对应  |

2.  环境搭建文档
    

[《大模型服务器基础环境安装》](https://alidocs.dingtalk.com/i/nodes/20eMKjyp81d4XKzzSjOvBrnLJxAZB1Gv?corpId=dingc72a04295b93ad7dbc961a6cb783455b&doc_type=wiki_doc&iframeQuery=utm_source%3Dportal%26utm_medium%3Dportal_new_tab_open&rnd=0.383927174320126&sideCollapsed=true&utm_scene=person_space)详细介绍了如何在一台新的，只有显卡的服务器上，如何从0到1 搭建大模型部署环境。

# 五、搭建启智大脑

## 5.1 启智大脑环境搭建-源码方式

### 1、搭建启智大脑依赖组件和服务

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/a/R1LXqJEYYhBgGErr/f833c2fa384448548c6a06c43ac51ffc1215.png)

启智大脑的聊天应用和并行访问，需要搭建存储组件 **mysql和redis**支持。

启智大脑包含知识图谱功能，需要访问**图谱服务**。

启智大脑包含 ElasticSearch和rag 联合知识检索功能，需要搭建检索组件**ElasticSearch**。

|  依赖服务  |  服务名称  |
| --- | --- |
|  存储服务  |  **mysql和redis**  |
|  知识图谱服务  |  **graph**  |
|  检索服务  |  **ElasticSearch**  |

1.  部署依赖服务
    

1）安装mysql：

```shell
sudo apt update
sudo apt install mysql-server
sudo systemctl start mysql
sudo systemctl enable mysql

mysql -u root -p

CREATE USER 'qiyuan'@'localhost' IDENTIFIED BY 'Qizhi1016-DB';

GRANT ALL PRIVILEGES ON *.* TO 'qiyuan'@'localhost';

FLUSH PRIVILEGES;
```

2）安装redis：

```shell
sudo apt install redis-server
sudo systemctl start redis-server
sudo systemctl enable redis-server
```

3）图谱服务部署文档：[《图谱服务部署文档》](https://alidocs.dingtalk.com/i/nodes/20eMKjyp81d4XKzzSEgmLXMeJxAZB1Gv?doc_type=wiki_doc&utm_medium=dingdoc_doc_plugin_card&utm_scene=person_space&utm_source=dingdoc_doc)

2.  配置依赖服务地址
    

修改启智大脑配置文件，配置服务访问地址

```shell
git clone https://gitlab.qizhi.ai/hip/chatglm.git
vim config.py
```
```shell
#mysql config
MYSQL_HOST = os.environ.get('MYSQL_HOST', '172.16.170.155')
MYSQL_PORT = os.environ.get('MYSQL_PORT', '23306')
MYSQL_USER = os.environ.get('MYSQL_USER', 'qiyuan')
MYSQL_PASSWORD = os.environ.get('MYSQL_PASSWORD', parse.quote_plus('Qizhi@1016-DB'))
MYSQL_DB = 'chatglm'

```
```shell
GRAPH_URL = os.environ.get('GRAPH_URL', "http://127.0.0.1:8000/chat-process-txt")
```
```shell
ES_URL_ADD = '172.16.14.211:9200'
# INDEX CONFIG
QIZHIBRAIN_INDEX = 'qizhi_brain_es_retriever'
KGINSERT_INDEX = 'qizhi_brain_kg_insert'
QIZHIBRAIN_INDEX_TEST = 'qizhi_brain_es_retriever_test'
KGINSERT_INDEX_TEST = 'qizhi_brain_kg_insert_test'
# threadhold
QIZHIBRAIN_ES_SCORE = 1

```
```shell
class RedisConfig():
    """
    redis 配置
    """
    HOST = os.environ.get('REDIS_HOST', '172.16.170.151')
    PASSWORD = os.environ.get('REDIS_PASSWORD', 'Qizhi_1-Redis')
    DB = 0
    CELERY_DB = 1
    PORT = 16379


```

### 2、搭建启智大脑环境

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/a/R1LXqJEYYhBgGErr/ed52c256cc074d43b36a88f8526373931215.png)

在开始搭建启智大脑之前，先对启智大脑工程做简要介绍。

|  类目  |  项  |  说明  |
| --- | --- | --- |
|  语言栈  |  python3.9+  |   |
|  框架  |  Flask  |   |
|  通信  |  Socketio  |  实时通信  |
|   |  HTTP  |  API 访问  |
|  服务发布  |  gunicorn server 请求排队 连接资源释放  |  [《大模型服务生产开发部署设计文档V2》](https://alidocs.dingtalk.com/i/nodes/Gl6Pm2Db8DKaoRBBt6z17LAjJxLq0Ee4?corpId=dingc72a04295b93ad7dbc961a6cb783455b&doc_type=wiki_doc&iframeQuery=utm_source%3Dportal%26utm_medium%3Dportal_new_tab_open&rnd=0.013196293357404798&sideCollapsed=true&utm_scene=person_space)  |
|  服务管理  |  supervisor  |  通过supervisor监控和管理服务运行状态  |
|  本地数据  |  语义索引库  |  数据名称：m3elarge-bookqas\_tiaoling-250-50 数据用途：语义索引库 数据存放路径： ip: 172.16.170.50 dir: /data1/zhaojinming/knowledge\_pool ![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mPdnp8LpWZ22nw98/img/be034438-6d88-48a5-b5bc-2f61bb753983.png)  |
|   |  分词包  |  数据名称：nltk\_data 数据用途：用于embedding分词 数据存放路径：和工程在一起，无需单独下载。  |
|  本地模型文件  |  大模型文件  |  用途：要访问的大模型文件 支持模型：qwen、llama、九格、微调后的模型等 模型存放路径： ip: 172.16.170.50 dir: /data1/zhaojinming/pretrained\_models ![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mPdnp8LpWZ22nw98/img/5bef00f6-bde9-4461-84b8-365c08328f4f.png)  |
|   |  embedding 模型  |  用途：实体映射成向量 文件存放路径： ip: 172.16.170.50 dir：/data1/zhaojinming/embedding\_models ![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mPdnp8LpWZ22nw98/img/b54d92f6-fc71-4578-8055-e76d70d5f11c.png)  |
|  运行环境  |  python requirements  |  支持conda python 环境移植（下文第三步介绍）  |

开始搭建：                                    

#### 第一步： 下载工程代码

```shell
git clone https://gitlab.qizhi.ai/hip/chatglm.git
git checkout guoda
```

#### 第二步： 配置config.py

配置项目根目录：

```shell
CURRENT_DIR = "/data1/xxx"
```

配置上述依赖数据路径、模型文件路径、启动模型名称

```shell
kg_index_dir = os.environ.get('EMBEDDING_INDEX', os.path.join(CURRENT_DIR, 'knowledge_pool', 'm3elarge-bookqas_tiaoling-250-50'))
embedding_model_dir = os.environ.get('EMBEDDING_MODEL', os.path.join(CURRENT_DIR, 'embedding_models'))
```
```shell
nltk_directory = os.path.dirname(os.path.abspath(__file__)) + '/nltk_data'
```

| ```shell model_config = dict(     qzbrain=dict(         path=os.environ.get('MODEL_PATH', os.path.join(CURRENT_DIR, 'pretrained_models', 'checkpoint-126000'))     ),     chatglm2=dict(         path=os.environ.get('MODEL_PATH', os.path.join(CURRENT_DIR, 'pretrained_models', 'chatglm2-6b'))     ),     qzbrain3=dict(         path=os.environ.get('MODEL_PATH', os.path.join(CURRENT_DIR, 'pretrained_models', 'checkpoint-50000'))     ),     cpm9g=dict(         pt=os.environ.get('MODEL_PATH', os.path.join(CURRENT_DIR, 'pretrained_models/cpm-9g', 'cpm-9g-8b/model.pt')),         config=os.environ.get('MODEL_PATH', os.path.join(CURRENT_DIR, 'pretrained_models/cpm-9g', 'cpm-9g-8b/config.json')),         vocab=os.environ.get('MODEL_PATH', os.path.join(CURRENT_DIR, 'pretrained_models/cpm-9g', 'cpm-9g-8b/vocab.txt')),         memory_limit= 50 << 30     ),     qwen=dict(         path=os.environ.get('MODEL_PATH', os.path.join(CURRENT_DIR, 'pretrained_models', 'Qwen-72B-Chat'))     ) ) ```  |
| --- |

```shell
model_alive = dict(
    qzbrain=False,
    qwen=False,
    chatglm2=False,
    qzbrain3=False,
    cpm9g=True
)

```

配置使用的显卡

```shell
CUDA_VISIBLE_DEVICES = "0,1,2,3"
```

#### 第三步： 安装python依赖

安装python依赖环境有两种方案：1）**移植已有conda环境到新服务器  2）创建新的conda环境**

1.  移植已有conda环境到新服务器
    

1）找到新服务器上 conda 位置

```shell
which conda 找到conda env 目录
如：/home/qiyuan/miniconda3/bin/conda
那么目录是：/home/qiyuan/miniconda3/envs
```

2）激活conda环境

```shell
cp environment_name.tar.gz  /home/qiyuan/miniconda3/envs 
tar -xzf <environment_name>.tar.gz -C ./
conda activate <environment_name>
```

**注**：**移植已有conda环境由于不需要安装，所以速度较快，但可能需要修改编译器执行目录，解决方法是：**[《conda安装&离线环境移植》](https://alidocs.dingtalk.com/i/nodes/Qnp9zOoBVBdQonqqSNoogMBa81DK0g6l?corpId=dingc72a04295b93ad7dbc961a6cb783455b&doc_type=wiki_doc&iframeQuery=utm_source%3Dportal%26utm_medium%3Dportal_new_tab_open&rnd=0.2469759214784819&sideCollapsed=true&utm_scene=person_space)

2.  创建新的conda环境
    

在工程根目录下requirements 文件罗列了需要的依赖

```shell
conda create --name qzbrain_server python=3.1.0 
conda activate qzbrain_server 
pip install -r requirements.txt
```

#### 第四步： 创建数据库&表

1.  连接mysql服务
    

     连接在5.1.1章节安装的mysql服务

     mysql -u qiyuan -p

2.  创建测试库、测试表
    

      CREATE DATABASE test\_database;

use test\_database;

source /工程所在目录/chatglm.sql;

3.  连接测试库
    

修改config文件，将生产库替换为测试库![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/MeYVOL0xEzLLOpz2/img/2fccff4b-18a8-442f-8d3e-481630e07526.png)

**至此，大模型部署环境、启智大脑依赖服务搭建和访问配置、依赖数据和模型文件配置、启智大脑工程环境搭建已经全部完成，下面到最后一步，启动工程。**

### 3、启动启智大脑

![image](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/a/R1LXqJEYYhBgGErr/906703a4715b485eb700c29f517719fc1215.png)

1.  进入项目目录 
    

cd /xxxx/xxxx

2.  启动服务
    

python main.py

sh run\_celery.sh

备注，在main.py中可修改端口

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mPdnp8LpWZ22nw98/img/e5500dfc-af29-45d8-bb3e-1887e0d3a274.png)

如上图所示：启智大脑服务将以端口9025提供对外访问。

## 5.2 启智大脑环境搭建-docker方式

### 第一步：nvidia-docker2安装

1.  docker 安装
    

```shell
sudo apt update
sudo apt install -y docker.io

```

2.  安装NVIDIA Container Toolkit
    

安装NVIDIA Container Toolkit以支持在Docker容器中运行NVIDIA GPU工作负载。

```shell
sudo distribution=$(. /etc/os-release;echo $ID$VERSION_ID) \
&& curl -s -L https://nvidia.github.io/nvidia-docker/gpgkey | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg \
&& echo "deb [signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/$distribution $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null \
&& sudo apt-get update \
&& sudo apt-get install -y nvidia-docker2

```

c、启动docker

```shell
sudo systemctl start docker
```

d、验证安装

```shell
docker run --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```

**注：替换成实际的镜像标签**

### 第二步：启智大脑部署和启动

如下图所示，所有服务均打包成docker，docker通过compose方式管理。

![image.png](https://alidocs.oss-cn-zhangjiakou.aliyuncs.com/res/mxPOGyZ4N675nKa9/img/2d02d11f-8b9e-4fae-be4b-944514f70ef3.png)

在经过第四章节，完成大模型基础环境搭建后，运行sh deploy.sh

```shell
sh deploy.sh
```

该脚本完成一键完成部署和启动。