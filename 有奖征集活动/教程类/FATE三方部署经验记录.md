﻿@[TOC](目录)
 
# 1、准备

本篇文章主要记录并分享FATE框架的多方部署经验并尽可能详细，这里的多方以三方为例。部署FATE联邦学习的方法众多，其中最推荐的方法之一是利用Docker Compose进行部署，本文记录利用Docker Compose进行部署的方案。

**物理资源（推荐）：**
3台虚拟机服务器，每台500G硬盘，16G内存，8VCPU核，centOS7.2，10M带宽。专家提示：内存太小会有不可预见的问题!!！

首先从微众银行下载fate镜像，这里采用1.4.2版本，大小超过11G，下载方式如下
https://webank-ai-1251170195.cos.ap-guangzhou.myqcloud.com/fate_1.4.2-images.tar.gz
下载方式不是重点，反正上面这个链接可以下载到，重点是确保每台机器上都有镜像的压缩文件。

然后每台机器都载入这个镜像包，执行docker命令如下

```
docker load -i fate_1.4.2-images.tar.gz
```

由于镜像比较大，这个过程有可能会报docker镜像目录剩余空间不足的问题，这时候就需要对docker的镜像存放目录进行修改和迁移，总结后的修改与迁移脚本如下，逐行运行或保存成shell脚本直接运行。迁移完成后重新载入镜像即可。

```
# 备份fstab文件
sudo cp /etc/fstab /etc/fstab.$(date +%Y-%m-%d)
# 停止docker
sudo service docker stop
# /data/docker/为目标路径，根据机器情况设定。
export DOCKER_PATH=/data/docker/
# 用rsync同步/var/lib/docker到新位置
rsync -avPHSX /var/lib/docker/.  $DOCKER_PATH
echo $DOCKER_PATH /var/lib/docker  none bind 0 0 >> /etc/fstab
mount –a
sudo service docker start
```
然后可以检查所有镜像是否载入成功。
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200922160546338.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

# 2、docker-compose配置
***接下来是重点！！***
找到对应版本的kubefate-docker-compose.tar.gz文件，***一定要确保是对应版本的！！***
如1.4.2版本的kubefate-docker-compose.tar.gz文件在下图所示位置。
链接：https://github.com/FederatedAI/KubeFATE/releases/tag/v1.4.2
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092216075365.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
这个文件下载下来只需要放在一台部署机上就行，部署机就是运行部署脚本的机器，部署脚本会自动给设定的每一台机器进行部署，其他被部署的机器称为运行机。
然后对kubefate-docker-compose.tar.gz进行解压。

```
tar -zxvf kubefate-docker-compose.tar.gz
```
解压完之后进入docker-deploy目录配置parties.conf文件。

```
vim parties.con
```
只需要配置以下内容
![在这里插入图片描述](https://img-blog.csdnimg.cn/20200922162029754.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)

# 3、部署
然后运行生成配置文件的脚本和部署脚本如下

```
bash generate_config.sh
bash docker_deploy.sh all
```
部署期间脚本会登入到每台机器上进行自动部署，会多次询问登入密码，如果觉得麻烦可以提前设置好免密登入。
如果没有报错就可能是部署成功了，可以简单验证如下
![在这里插入图片描述](https://img-blog.csdnimg.cn/2020092216215817.png?x-oss-process=image/watermark,type_ZmFuZ3poZW5naGVpdGk,shadow_10,text_aHR0cHM6Ly9ibG9nLmNzZG4ubmV0L29xcW1vb24xMjM=,size_16,color_FFFFFF,t_70#pic_center)
最后算出一个约为2000的浮点数就是成功了。
