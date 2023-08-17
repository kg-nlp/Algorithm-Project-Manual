---
sort: 4
---


# chat_table

* [算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/表格解析/chat_table.html)

* [个人知乎](https://www.zhihu.com/people/zhangyj-n)



## 赛题类型

## 初赛

> 2023金融大模型比赛
* [代码仓库](https://github.com/kg-nlp/SMP_2023_ChatGLM/tree/main)
> 目前只有小组成员可以访问

[赛事介绍](https://tianchi.aliyun.com/competition/entrance/532126/introduction)

```
初级：数据基本查询（40分）

参赛者需要利用提供的ChatGLM2-6B开源模型和上市公司年报原始数据，并以此为基础创建信息问答系统。系统需能够解决基本查询，如：某公司2021年的研发费用是多少？等问题。

中级：数据统计分析查询（30分）

在初级阶段的基础上，参赛者需要进行金融数据的统计分析和关联指标查询。系统需基于各类指标，提供问题和答案，如：某公司2021年研发费用增长率为多少？等问题。

高级：开放性问题（30分）

如：某公司2021年主要研发项目是否涉及国家创新领域，如新能源技术、人工智能等？

```

### baseline

* [SMP 2023 ChatGLM 金融大模型挑战赛 60 分 Baseline 思路](https://github.com/RonaldJEN/FinanceChatGLM/tree/main)



### 金融类大模型的应用

FinGPT
FINMA
轩辕(XuanYuan 2.0)
聚宝盆
BloombergGPT
BBT-Fin

## 复赛

### 复赛说明

```text
复赛 A 榜：
- 提交时间：8 月 18 日中午 12 点开始。
- 提交方式：需使用阿里天池的 docker 环境提交。
- 权限开放：A 榜期间，我们将开放日志权限。
- 数据量：pdf 数据量小于 2000。
- 测试问题数量：等于 2000。
- 数据范围：初赛子集。

复赛 B 榜：
- 提交时间：8 月 22 日中午 12 点开始。
- 提交方式：需使用阿里天池的 docker 环境提交。
- 权限开放：B 榜期间，不开放日志权限。
- 数据量：pdf 数据量等于 2000。
- 测试问题数量：等于 2000。
- 数据范围：与初赛有差异。

复赛期间，我们将开启周冠军模式：
- 时间：8 月 20 日、8 月 27 日，分别选出 2 个不同的周冠军。
- 要求：不需要分享代码，但需要分享经验。
- 奖励：将为分享经验的周冠军团队提供 1000 元的现金奖励。

此外，我们将在 8 月 20 日晚组织一次答疑活动，欢迎各位选手提问和分享经验。

我们期望每位选手都能充分发挥自己的实力，取得优异的成绩，同时也希望各位能够在比赛中得到成长和收获。

再次祝愿各位参赛者取得好成绩！

——————————————
补充：
从0开始，大赛docker提交：https://tianchi.aliyun.com/competition/entrance/531863/customize253
```

### 补充docker常用命令

* Dockerfile 文件(本地打包镜像的配置文件)

```示例
FROM registry.cn-beijing.aliyuncs.com/xxx/xxx:xxx  # 基础镜像
ENV MYPATH /project/  # 定义变量MYPATH
WORKDIR $MYPATH   # 指定上面路径为工作目录
COPY ./run.py /project/  # 拷贝本地文件到工作目录
COPY ./supervisord.conf /project/
COPY ./requirements.txt /project/
COPY ./db /cde_project/db  # 拷贝db文件夹到工作目录下
# COPY . /project/  # 也可以把本地当前路径下所有内容拷贝到工作目录下

#RUN pip install --upgrade pip
#RUN pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple/
ENTRYPOINT ["supervisord", "-n", "-c", "/cde_project/supervisord.conf"]  # docker启动时自动运行的脚本
```
* 制作镜像(根据上面Dockerfile)
docker build -f Dockerfile -t registry.cn-beijing.aliyuncs.com/{命名空间}/{镜像仓库}:{版本号} .  

> 注意上面最后的点. 表示当前目录下

* 启动镜像

```bash
docker run -it -d \ 
--name paddle_env \
-p 8075:22 -p 8040:8040 -p 8030:8030 -p 8020:8020 \
-v /data/zhangyj:/home/zhangyj \
--gpus all \
--shm-size 64g \
--restart=always \
-e NVIDIA_DRIVER_CAPABILITIES=compute,utility \
-e NVIDIA_VISIBLE_DEVICES=all \
registry.cn-beijing.aliyuncs.com/sg-gie/paddle:2.4.1-gpu-cuda10.2-cudnn7.6-trt7.0 /bin/bash
```

> 说明  
> --name  镜像启动后正常运行后称为容器,这个就是容器名称,以后方便找
> -p 本地(宿主机)端口和容器端口映射  
> -v 本地(宿主机)数据位置和容器内自定义的路径,这个叫数据挂接,可以实时同步容器内和本地的数据,防止容器意外挂掉,数据丢失  
> --gpus 对于有GPU配置的镜像,需要设置
> 其他的默认即可  
> 当这个镜像作为开发环境使用时,我一般找对应的算法框架提供的镜像启动,之后在里面启动ssh,将容器端口22映射出来,外部用xshell/finalshell/pycharm远程连接

* 常用的命令
```bash
docker images   查看当前服务器上的镜像
docker ps  查看正在运行的镜像
docker ps -a 查看所有启动成功和失败的镜像
docker logs 容器名或容器id  查看容器运行日志,在容器启动失败时查看原因
docker exec -it 容器名 bash 进入容器内操作
docker stop 容器名  停止容器
docker rm 容器名  删除容器
docker rmi 镜像id  删除镜像
```

