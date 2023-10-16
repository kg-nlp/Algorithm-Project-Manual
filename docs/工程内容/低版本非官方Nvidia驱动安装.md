---
sort: 4
---

# 低版本非官方Nvidia驱动安装

[🔨算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/工程内容/低版本非官方Nvidia驱动安装.html)
[🔨个人知乎](https://www.zhihu.com/people/zhangyj-n)

## **nvidia官方安装**

https://docs.nvidia.com/datacenter/cloud-native/container-toolkit/install-guide.html#docker

有效卸载https://blog.csdn.net/qq_36287702/article/details/122549400 必须重启



修改docker默认存储位置

https://blog.csdn.net/qq_39698985/article/details/126598627



查看驱动位置 locate 卸载    NVIDIA-Linux  

https://blog.csdn.net/Aaron_qinfeng/article/details/106939938

sh NVIDIA-Linux-x86_64-418.126.02.run --uninstall  



Linux下nvcc -V 与 nvidia -smi 已安装但command not found问题

https://blog.csdn.net/ifenghua135792468/article/details/101436856

```
sudo echo 'export PATH=/usr/local/cuda-11.2/bin/:$PATH'>>~/.bashrc
sudo echo 'export LD_LIBRARY_PATH=/usr/local/cuda-11.2/lib64:$LD_LIBRARY_PATH'>>~/.bashrc
sudo source ~/.bashrc

vim ~/.bashrc
cat ~/.bashrc
```



## **内核禁止更新(旧版本)**

```
法一 安装Centos 6.5后，系统yum自动更新状态默认为开启，若禁止系统自动更新需要手动关闭。
1.进入yum目录
[root@localhost ~]$ cd /etc/yum
2.编辑yum-cron.conf文件
[root@localhost yum]$ vi yum-cron.conf
3.把download_updates = yes的值改为no，保存更改后即可
```



## **centos环境配置(新环境安装)**

```
sudo yum install -y yum-utils \
  device-mapper-persistent-data \
  lvm2
sudo yum-config-manager \
    --add-repo \
    http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
sudo yum install docker-ce docker-ce-cli containerd.io 
sudo systemctl start docker   
```



## **将docker权限添加给普通用户**

```
sudo gpasswd -a $USER docker     #将登陆用户加入到docker用户组中
newgrp docker     #更新用户组
```



## **安装nvidia-docker-gpu驱动**

```
sudo rpm -ivh libnvidia-container1-1.1.1-1.x86_64.rpm libnvidia-container-tools-1.1.1-1.x86_64.rpm
sudo rpm -ivh nvidia-container-runtime-3.2.0-1.x86_64.rpm nvidia-container-toolkit-1.1.2-2.x86_64.rpm nvidia-docker2-2.3.0-1.noarch.rpm 

sudo systemctl restart docker  重启
--gpus all  在起服务时，加入这句话

docker run -it --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```



**驱动链接**

https://pan.baidu.com/s/1ksbir29R4eMmSP5fq-W5Ng

提取码：6kdi 	



## **如何干净卸载docker**

docker卸载提示Device or resource busy

https://blog.csdn.net/weixin_50762970/article/details/126298881

## **ubuntu环境配置**

**安装nvidia-docker-gpu**

```
apt-get install nvidia-container-runtime  先安装这个
sudo systemctl restart docker  重启
--gpus all  在起服务时，加入这句话

docker run -it --rm --gpus all nvidia/cuda:11.0-base nvidia-smi
```



**其他参考**

**1** [**Linux查看系统基本信息，版本信息（最全版）**](https://blog.csdn.net/qq_31278903/article/details/83146031)

**2** [Linux下如何查看NVIDIA显卡信息](https://zhidao.baidu.com/question/746165463732375292.html)

3 [linux下使用conda创建python虚拟环境](https://www.cnblogs.com/zuoxiaodragon/p/12730240.html)

4 [Ubuntu用户之间相互切换方法(推荐)](https://www.jb51.net/article/109471.htm)

5 https://zhuanlan.zhihu.com/p/106708516 ubuntu18.0安装cuda

https://developer.nvidia.com/cuda-10.2-download-archive?target_os=Linux&target_arch=x86_64&target_distro=Ubuntu&target_version=1804&target_type=deblocal

**windows10 下显卡配置**

https://developer.nvidia.com/rdp/cudnn-download
