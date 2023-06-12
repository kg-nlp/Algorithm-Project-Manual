---
sort: 4
---
# 环境配置

* ![镜像下载](https://note.youdao.com/yws/api/personal/file/WEB386ef86df22f59c31dd7847ea71e8c79?method=download&shareKey=0a1ccc7142b789db2d98ca350aa84568)
    * [docker-hub地址](https://hub.docker.com/r/hpcaitech/colossalai)
    
## 容器环境配置
* [有道云链接](https://note.youdao.com/s/SyuVJga)

```
#拉取最新的镜像
docker pull hpcaitech/colossalai:0.3.0  
#启动镜像
docker run -it -d --name colossal_env -p 8095:22 -p 7000:7000 -p 7010:7010 -p 7020:7020 -v /data/user/zhangyj/LLM:/workspace/LLM --gpus all --ipc=host hpcaitech/colossalai:0.3.0 bash

#容器工作目录: /workspace
#在当前容器中查找python 和 pip 的位置 /opt/conda/bin/python

#原容器ssh无法启动,copy pytorch_env下的配置文件
docker cp pytorch_env:/etc/ssh/sshd_config /home/zhangyj/
docker cp /home/zhangyj/sshd_config colossal_env:/etc/ssh/sshd_config

#sshd_config 内容
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem	sftp	/usr/lib/openssh/sftp-server
PubkeyAuthentication yes
RSAAuthentication yes
PermitRootLogin yes

#安装ssh-server
apt-get install -y openssh-server
service ssh restart

#创建软连接
ln -s /opt/conda/bin/python /usr/local/bin/python
ln -s /opt/conda/bin/pip /usr/local/bin/pip

#永久更新清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

#commit新镜像
docker commit colossal_env registry.cn-beijing.aliyuncs.com/sg-gie/colossalai:0.3.0

# 每次启动镜像时,进入容器内部执行 service ssh restart  可以在容器外通过pycharm 远程访问,访问端口8095
```

# Chat示例
## 官方chat训练流程
[知乎参考资料](https://www.zhihu.com/tardis/zm/art/618048558?source_id=1005)
![流程图](https://raw.githubusercontent.com/hpcaitech/public_assets/main/applications/chatgpt/chatgpt.png) 
*  支持全面的大型模型训练加速能力的ColossalAI，不需要复杂的分布式训练算法的知识
*  监督数据集收集
*  监督指令微调
*  训练奖励模型
*  基于人类反馈的强化学习
*  量化推理
*  快速部署
*  与HuggingFace生成集成,模型自由定制
## Chat环境搭建
* 安装chat环境
```
#进入容器
docker exec -it colossal_env bash
cd /workspace
git clone https://github.com/hpcaitech/ColossalAI.git
cd ColossalAI/applications/Chat
pip install -r requirements.txt 
```
* 安装Transformers环境
```
cd /workspace
git clone https://github.com/hpcaitech/transformers
cd transformers
pip install .
```
## 数据集准备
* SFT指令微调数据集
    * 本地路径:D:\项目-预训练模型\ColossalAI-main\data\InstructionWild 
    * [InstructionWild数据集](https://github.com/XueFuzhao/InstructionWild/tree/main)
    * 52K instructions for Chinese.
    * 数据集特点
        * 我们的新数据集提高了模型在生成、开放QA和头脑风暴指令方面的能力。这对应于我们的数据收集过程。我们的数据是从Twitter上收集的，在那里用户倾向于分享他们的有趣提示，主要是生成，开放QA和头脑风暴类型。
    * llama微调模型的局限性
        * Alpaca和ColossalChat都是基于LLaMA。在预训练阶段，知识的缺失是很难弥补的。缺乏计数能力:无法数出列表中项目的数量。
        * 缺乏逻辑(推理和计算)。
        * 倾向于重复最后一个句子(无法产生结束标记)。
        * 多语言结果差:LLaMA主要在英语数据集上进行训练(Generation表现优于QA)。
    * 数据集的局限性
        * 缺乏总结能力:在微调数据集中没有这样的指令。
        * 缺少多回合聊天和角色扮演:在微调数据集中没有这样的指示
        * 缺乏自我识别:在微调数据集中没有这样的指令
        * 缺乏安全性:当输入包含虚假事实时，模型会编造虚假事实和解释。
        * 不能遵守OpenAI的策略:当OpenAI API生成提示时，它总是遵守OpenAI的策略。所以数据集中没有违规情况。
* 奖励模型排序数据集
     * prompt -> response -> chosen -> rejected  数据集
     * [数据集链接](https://huggingface.co/datasets/Dahoas/rm-static)
     * 本地路径: D:\项目-预训练模型\ColossalAI-main\data\rm-static
     * [数据集链接](https://huggingface.co/datasets/Anthropic/hh-rlhf/tree/refs%2Fconvert%2Fparquet)
     * 本地路径: D:\项目-预训练模型\ColossalAI-main\data\anthropic-hh-rlhf

* 人类反馈强化学习数据集
    * InstructionWild 里的中文数据集
    * 本地路径: D:\项目-预训练模型\ColossalAI-main\data\alpaca_data
    * 本地路径:D:\项目-预训练模型\ColossalAI-main\data\InstructionWild 

## 模型准备
* LLAMA模型 
    *  [facebook(meta)开源产品](https://ipfs.io/ipfs/Qmb9y5GCkTG7ZzbBWMu2BXwMkzyCKcUjtEKPpgdZ7GEFKm/)
    *  [内网下载](https://www.123pan.com/s/Su8ZVv-g97q3.html)
    *  [huggingface下载](https://huggingface.co/decapoda-research)
        ```
        git lfs install
        git clone https://huggingface.co/decapoda-research/llama-7b-hf
        git clone https://huggingface.co/decapoda-research/llama-13b-hf
        ```  
* Bloomz模型
    * BLOOM是一个在46种自然语言和13种编程语言上训练的1760亿个参数的语言模型
    * [huggingface下载](https://huggingface.co/bigscience/bloomz-7b1-mt)
## 训练脚本
![训练流程](https://raw.githubusercontent.com/hpcaitech/public_assets/main/applications/chat/stage-3.jpeg)
* 监督指令微调: examples/train_sft.sh
* 训练奖励模型: examples/train_rm.sh
* 基于人类反馈的强化学习训练模型: examples/train_prompts.sh



# 源码解析
* [Chat源码地址](https://github.com/hpcaitech/ColossalAI/tree/main/applications/Chat) 截止实际时间20230612

```
cd /workspace/ColossalAI/applications/Chat/examples/
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```
* 训练策略
```
from coati.trainer.strategies import ColossalAIStrategy, DDPStrategy, NaiveStrategy


```
* 转为lora模型
```
from coati.models import convert_to_lora_module

```
* RM加载数据集
```
#需要提前处理一下,更改下train_reward_model.py


```

## 资料
[中文教程](https://colossalai.org/zh-Hans/docs/get_started/installation/)



# 其他

![GPT-4训练流程](https://github.com/shibing624/MedicalGPT/raw/main/docs/GPT_Training.jpg)
* 第一阶段：PT(Continue PreTraining)增量预训练，在海量领域文档数据上二次预训练GPT模型，以注入领域知识
* 第二阶段：SFT(Supervised Fine-tuning)有监督微调，构造指令微调数据集，在预训练模型基础上做指令精调，以对齐指令意图
* 第三阶段：RM(Reward Model)奖励模型建模，构造人类偏好排序数据集，训练奖励模型，用来对齐人类偏好，主要是"HHH"原则，具体是"helpful, honest, harmless"
* 第四阶段：RL(Reinforcement Learning)基于人类反馈的强化学习(RLHF)，用奖励模型来训练SFT模型，生成模型使用奖励或惩罚来更新其策略，以便生成更高质量、更符合人类偏好的文本
