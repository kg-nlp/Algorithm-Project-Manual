---
sort: 3
---

# conda环境配置

[🔨算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/工程内容/conda环境配置.html)
[🔨个人知乎](https://www.zhihu.com/people/zhangyj-n)

[TOC]

## **Linux下安装miniconda**

https://blog.csdn.net/m0_46336568/article/details/127836072

```
#第一步下载miniconda安装包 
#安装包连接https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/，查找相应安装包
wget -c https://mirrors.tuna.tsinghua.edu.cn/anaconda/miniconda/Miniconda3-py39_4.9.2-Linux-x86_64.sh
#第二步安装
bash Miniconda3-py39_4.9.2-Linux-x86_64.sh
#第三部配置conda镜像
source ~/.bashrc
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/pkgs/free
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/conda-forge
conda config --add channels https://mirrors.tuna.tsinghua.edu.cn/anaconda/cloud/bioconda
conda config --set show_channel_urls yes
#第四步创建一个名称为python38的环境，python版本等于3.8
conda create -n tf2.3torch1.7 python=3.8
conda create -n paddle_env python=3.8
#第五步 激活环境
source activate python38
```



## **创建用户和密码**

```
useradd zhangyj
passwd zhangyj
zhangyj-n  zhangyj-n
接下来安装pip包
```



## **配置源**

```
# 设置conda命令启动
vi ~/.bashrc
内容如下
export PATH=/home/zhangyj/miniconda3/bin/:$PATH


#设置pip清华源
#用户的文件夹下新建目录.pip，并在目录新建配置文件pip.conf
#命令
mkdir ~/.pip
cd ~/.pip
vi pip.conf
内容如下
[global]
index-url=https://pypi.tuna.tsinghua.edu.cn/simple
[install]
trusted-host=pypi.tuna.tsinghua.edu.cn
disable-pip-version-check = true
timeout = 6000
```



## **导出虚拟环境pip包**

```
conda install -n env_name python=3.7

pip批量导出包含环境中所有组件的requirements.txt文件
pip freeze > requirements.txt

pip批量安装requirements.txt文件中包含的组件依赖
pip install -r requirements.txt

conda批量导出包含环境中所有组件的requirements.txt文件
conda list -e > requirements.txt

conda批量安装requirements.txt文件中包含的组件依赖
conda install --yes --file requirements.txt
```



更多内容查看

[环境框架安装](https://kg-nlp.github.io/Algorithm-Project-Manual/算法框架/框架环境.html)