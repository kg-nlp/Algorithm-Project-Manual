---
sort: 4
---
# ColossalAI源码分析

## 环境配置

* ![镜像下载](https://note.youdao.com/yws/api/personal/file/WEB386ef86df22f59c31dd7847ea71e8c79?method=download&shareKey=0a1ccc7142b789db2d98ca350aa84568)
* [docker-hub地址](https://hub.docker.com/r/hpcaitech/colossalai)
    
### 容器环境配置
* [有道云链接](https://note.youdao.com/s/SyuVJga)

```sh
# 拉取最新的镜像
docker pull hpcaitech/colossalai:0.3.0  
# 启动镜像
docker run -it -d --name colossal_env -p 8095:22 -p 7000:7000 -p 7010:7010 -p 7020:7020 -v /data/user/zhangyj/LLM:/workspace/LLM --gpus all --ipc=host hpcaitech/colossalai:0.3.0 bash

# 容器工作目录: /workspace
# 在当前容器中查找python 和 pip 的位置 /opt/conda/bin/python

# 原容器ssh无法启动,copy pytorch_env下的配置文件
docker cp pytorch_env:/etc/ssh/sshd_config /home/zhangyj/
docker cp /home/zhangyj/sshd_config colossal_env:/etc/ssh/sshd_config

# sshd_config 内容
ChallengeResponseAuthentication no
UsePAM yes
X11Forwarding yes
PrintMotd no
AcceptEnv LANG LC_*
Subsystem	sftp	/usr/lib/openssh/sftp-server
PubkeyAuthentication yes
RSAAuthentication yes
PermitRootLogin yes

# 安装ssh-server
apt-get install -y openssh-server
service ssh restart

# 创建软连接
ln -s /opt/conda/bin/python /usr/local/bin/python
ln -s /opt/conda/bin/pip /usr/local/bin/pip

# 永久更新清华源
pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple

# commit新镜像
docker commit colossal_env registry.cn-beijing.aliyuncs.com/sg-gie/colossalai:0.3.0

# 每次启动镜像时,进入容器内部执行 service ssh restart  可以在容器外通过pycharm 远程访问,访问端口8095
```

## Chat示例
### 官方chat训练流程
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
### Chat环境搭建
* 安装chat环境
```sh
# 进入容器
docker exec -it colossal_env bash
cd /workspace
git clone https://github.com/hpcaitech/ColossalAI.git
cd ColossalAI/applications/Chat
pip install -r requirements.txt 
```
* 安装Transformers环境
```sh
cd /workspace
git clone https://github.com/hpcaitech/transformers
cd transformers
pip install .
```
### 数据集准备
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

### 模型准备
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
### 训练脚本

* 监督指令微调: examples/train_sft.sh
* 训练奖励模型: examples/train_rm.sh
* 基于人类反馈的强化学习训练模型: examples/train_prompts.sh



## 源码解析
* [Chat源码地址](https://github.com/hpcaitech/ColossalAI/tree/main/applications/Chat) 截止实际时间20230612


### 监督质量微调
> 使用前面提到的数据集执行有监督指令微调，以微调模型
  
  
```sh
cd /workspace/ColossalAI/applications/Chat/examples/
pip install -r requirements.txt -i https://pypi.tuna.tsinghua.edu.cn/simple
```

* 策略选择     
```python
from coati.trainer.strategies import ColossalAIStrategy, DDPStrategy, NaiveStrategy
```
```python
class ColossalAIStrategy(DDPStrategy):  
 
    Args:  # 参数说明
        stage(int): The stage to use in ZeRO. Choose in (1, 2, 3)
        precision(str): The precision to use. Choose in ('fp32', 'fp16'). Stage 3 only supports fp16.
        seed(int): The seed for the random number generator.
        shard_init(bool): Whether to shard the model parameters during initialization. Only for ZeRO-3.  初始化时是否对模型参数进行分片
            This is not compativle with `from_pretrained()`. We temporarily disable this and will support it in the future.
        placement_policy(str): The placement policy for gemini. Choose in ('cpu', 'cuda')
                          If it is “cpu”, parameters, gradients and optimizer states will be offloaded to CPU,
                          If it is “cuda”, they will not be offloaded, which means max CUDA memory will be used. It is the fastest.
        pin_memory(bool): Whether to pin the memory for the data loader. Only for ZeRO-3.  # 是否在数据加载时固定内存
        force_outputs_fp32(bool): Whether to force the outputs to be fp32. Only for ZeRO-3.
        search_range_mb(int): The search range in MB for the chunk size. Only for ZeRO-3.
        hidden_dim(optional, int): The hidden dimension for the gemini. Only for ZeRO-3.
        min_chunk_size_mb(float): The minimum chunk size in MB. Only for ZeRO-3.  Chunk即一段连续的内存空间
        gpu_margin_mem_ratio(float): The margin memory ratio for the GPU. Only for ZeRO-3.  GPU的剩余内存比例
        reduce_bugket_size(int): The reduce bucket size in bytes. Only for ZeRO-1 and ZeRO-2.  # 减少桶大小(以字节为单位)
        overlap_communication(bool): Whether to overlap communication and computation. Only for ZeRO-1 and ZeRO-2.  通信和计算是否重叠
        initial_scale(float): The initial scale for the optimizer. gradient scaler 的初始值 初始缩放因子
        growth_factor(float): The growth factor for the optimizer.  loss scale 的增长率,如果在growth_interval连续迭代过程中没有出现 inf/NaN 梯度，则在update中乘以比例系数
        backoff_factor(float): The backoff factor for the optimizer.  loss scale 的下降率 如果在迭代中出现 inf/NaN 梯度，则在update中乘以比例系数
        growth_interval(int): The growth interval for the optimizer. 在指定次数的连续迭代中，若没有出现 inf/NaN 梯度，则乘以growth_factor
        hysteresis(int): The hysteresis for the optimizer.  动态 loss scaling 的延迟偏移
        min_scale(float): The minimum scale for the optimizer.  loss scale 的最小允许值
        max_scale(float): The maximum scale for the optimizer.  loss scale 的最大允许值
        max_norm(float): The maximum norm for the optimizer.  梯度正则策略
        norm_type(float): The norm type for the optimizer.  正则类型
```
```python
args.strategy == 'colossalai_gemini'
# 策略说明 异构内存空间管理  https://colossalai.org/zh-Hans/docs/advanced_tutorials/meet_gemini/
```

* 转为lora模型   
```python
from coati.models import convert_to_lora_module
```

```python
# 大模型的参数被固定，只有低秩矩阵参数被调整
def convert_to_lora_module(module: nn.Module, lora_rank: int, lora_train_bias: str = 'none') -> nn.Module:
    """Convert a torch.nn.Module to a LoRA module.

    Args:
        module (nn.Module): The module to convert.
        lora_rank (int): LoRA rank.

    Returns:
        nn.Module: The converted module.
    """
    if lora_rank <= 0:
        return module
    convert_to_lora_recursively(module, lora_rank)
    lora.mark_only_lora_as_trainable(module, lora_train_bias)
    return module
```

* 启动脚本

```sh
#!/usr/bin/env bash
torchrun --standalone --nproc_per_node=4 train_sft.py \
    --pretrain "/workspace/ColossalAI/LLM/llama-7b-hf" \
    --model 'llama' \
    --strategy colossalai_zero2 \
    --log_interval 10 \
    --save_path  /workspace/ColossalAI/Saved/Coati-7B-sft \
    --dataset /workspace/ColossalAI/data/InstructionWild/instinwild_ch.json \
    --batch_size 2 \
    --accumulation_steps 8 \
    --lr 2e-5 \
    --max_datasets_size 500 \
    --max_epochs 1 \
```
### 训练奖励模型
  
> 训练奖励模型，通过手动对同一提示的不同输出进行排序来分配相应的分数，然后有监督奖励模型的训练。

* RM加载数据集     

```python

# 原始数据集anthropic-hh-rlhf和rm-static的格式都为parquet
# 更改train_reward_model.py 中数据加载的代码
# 原代码:
if args.subset is not None:
    data = load_dataset(args.dataset, data_dir=args.subset)
else:
    data = load_dataset(args.dataset)
# 更改后:
if 'rm-static' in args.dataset or 'hh-rlhf' in args.dataset:
    data = load_dataset("parquet", data_files={'train': os.path.join(args.dataset,'train.parquet'), 'test': os.path.join(args.dataset,'test.parquet')})
else:
    raise ValueError('数据集有问题')

# 命令行参数类型进行修改
parser.add_argument('--dataset',
                type=str,
                choices=['Anthropic/hh-rlhf', 'Dahoas/rm-static'],
                default='Dahoas/rm-static')
# 原始代码限定了choices,为了加载自定义数据集,将choices注释掉
```
 
  
```python
# 为了测试ColossalAI训练流程,本次不加载全量数据
# 设置args.test == True
# 更改测试数据范围
if args.test:
    train_data = data['train'].select(range(10000))
    eval_data = data['test'].select(range(1000))
else:
    train_data = data['train']
    eval_data = data['test']
```  

* 启动脚本    
 
```sh 
set_n_least_used_CUDA_VISIBLE_DEVICES() {
    local n=${1:-"9999"}
    echo "GPU Memory Usage:"
    local FIRST_N_GPU_IDS=$(nvidia-smi --query-gpu=memory.used --format=csv \
        | tail -n +2 \
        | nl -v 0 \
        | tee /dev/tty \
        | sort -g -k 2 \
        | awk '{print $1}' \
        | head -n $n)
    export CUDA_VISIBLE_DEVICES=$(echo $FIRST_N_GPU_IDS | sed 's/ /,/g')
    echo "Now CUDA_VISIBLE_DEVICES is set to:"
    echo "CUDA_VISIBLE_DEVICES=$CUDA_VISIBLE_DEVICES"
}

set_n_least_used_CUDA_VISIBLE_DEVICES 2

torchrun --standalone --nproc_per_node=2 train_reward_model.py \
   --pretrain /workspace/ColossalAI/Saved/Coati-7B-sft \
   --model 'llama' \
   --strategy colossalai_zero2 \
   --loss_fn 'log_exp'\
   --save_path /workspace/ColossalAI/Saved/Coati-7B-rw.pt \
   --dataset '/workspace/ColossalAI/data/anthropic-hh-rlhf' \
   --test True
```  
  
### 基于人类反馈的强化学习训练模型  


> 在第一阶段的监督微调模型和第二阶段的奖励模型的基础上，使用强化学习算法进一步训练大型语言模型。该阶段是RLHF训练的核心部分，在强化学习中使用近端策略优化（PPO）算法来引入奖励信号，并生成更符合人类偏好的内容。

![训练流程](https://raw.githubusercontent.com/hpcaitech/public_assets/main/applications/chat/stage-3.jpeg)


> 在PPO部分，ColossalChat遵循两个阶段的过程：首先，构造实验阶段，它使用SFT（有监督微调）、Actor、RM（奖励模型）和Critic模型来计算生成的实验并将其存储在经验回放缓冲区中。然后是参数更新阶段，使用经验计算策略损失和价值损失。

> 在PTX部分，ColossalChat计算Actor的输出响应和输入语料库的响应部分之间的交叉熵损失。这种损失用于将预训练梯度添加到PPO梯度中，以保持语言模型的原始性能并防止遗忘。最后，总结了策略损失、价值损失和PTX损失，用于反向传播和参数更新。


   
* 数据集  

```  
Pretrain dataset 使用第一阶段数据
Prompt dataset 可以使用第一阶段数据,也可以使用examples下的脚本generate_prompt_dataset.py生成部分prompt
```

* 启动脚本 

```sh
set_n_least_used_CUDA_VISIBLE_DEVICES() {
    local n=${1:-"9999"}
    echo "GPU Memory Usage:"
    local FIRST_N_GPU_IDS=$(nvidia-smi --query-gpu=memory.used --format=csv \
        | tail -n +2 \
        | nl -v 0 \
        | tee /dev/tty \
        | sort -g -k 2 \
        | awk '{print $1}' \
        | head -n $n)
    export CUDA_VISIBLE_DEVICES=$(echo $FIRST_N_GPU_IDS | sed 's/ /,/g')
    echo "Now CUDA_VISIBLE_DEVICES is set to:"
    echo "CUDA_VISIBLE_DEVICES=$CUDA_VISIBLE_DEVICES"
}

set_n_least_used_CUDA_VISIBLE_DEVICES 2

# torchrun --standalone --nproc_per_node=2 train_prompts.py prompts.csv --strategy colossalai_zero2

torchrun --standalone --nproc_per_node=4 train_prompts.py \
    --prompt_dataset /workspace/ColossalAI/data/InstructionWild/instinwild_ch.json \
    --pretrain_dataset /workspace/ColossalAI/data/InstructionWild/instinwild_ch.json \
    --strategy colossalai_zero2 \
    --pretrain /workspace/ColossalAI/Saved/Coati-7B-sft \
    --model 'llama' \
    --rm_pretrain /workspace/ColossalAI/Saved/Coati-7B-sft \
    --rm_path /workspace/ColossalAI/Saved/Coati-7B-rw.pt
```  


### 资料

[中文教程](https://colossalai.org/zh-Hans/docs/get_started/installation/)

#### PPO详解

* [trl](https://github.com/lvwerra/trl)& [huggingface文档](https://huggingface.co/docs/trl/index)的ppo算法,已集成在了transformers中,大致过程如下:
    * rollout: 语言模型生成回复或是生成query之后的token序列
    * evaluation: 对query和响应通过函数,模型,人工反馈等操作进行评估,每一对都要有一个结果
    * optimization: 在优化步骤中，计算query和response序列对数概率。通过训练的模型和参考模型完成，参考模型通常是在微调之前预训练的模型。两个输出之间的kl散度被用作额外的奖励信号，以确保生成的响应不会偏离参考语言模型太远。然后使用PPO对active语言模型进行训练
    ![流程图](https://camo.githubusercontent.com/85d00cf9bca67e33c2d1270b51ff1ac01853b26a8d6bb226b711f859d065b4a6/68747470733a2f2f68756767696e67666163652e636f2f64617461736574732f74726c2d696e7465726e616c2d74657374696e672f6578616d706c652d696d616765732f7265736f6c76652f6d61696e2f696d616765732f74726c5f6f766572766965772e706e67)
    ![详细流程图](https://note.youdao.com/yws/api/personal/file/WEB9db34c99ec6c971798a72b744d3ded86?method=download&shareKey=7028569247d53a04a3dd8d397e1f7c45)
    
#### ColossalAI优势

* 系统性能优化与开发加速

```
ColossalChat 能够快速跟进 ChatGPT 完整 RLHF 流程复现，离不开 AI 大模型基础设施 Colossal-AI 及相关优化技术的底座支持，相同条件下训练速度相比 Alpaca 采用的 FSDP(Fully Sharded Data Parallel) 可提升两倍以上。
```

* 减少内存冗余的 ZeRO + Gemini

```
Colossal-AI 支持使用无冗余优化器 (ZeRO) 提高内存使用效率，低成本容纳更大模型，同时不影响计算粒度和通信效率。自动 Chunk 机制可以进一步提升 ZeRO 的性能，提高内存使用效率，减少通信次数并避免内存碎片。异构内存空间管理器 Gemini 支持将优化器状态从 GPU 显存卸载到 CPU 内存或硬盘空间，以突破 GPU 显存容量限制，扩展可训练模型的规模，降低 AI 大模型应用成本。
```
* 使用 LoRA 低成本微调

```
Colossal-AI 支持使用低秩矩阵微调（LoRA）方法，对 AI 大模型进行低成本微调。LoRA 方法认为大语言模型是过参数化的，而在微调时，参数改变量是一个低秩矩阵。因此，可以将这个矩阵分解为两个更小的矩阵的乘积。在微调过程中，大模型的参数被固定，只有低秩矩阵参数被调整，从而显著减小了训练所需的参数量，并降低成本。 低成本量化推理
```
* 量化推理

```   
为降低推理部署成本，Colossal-AI 使用 GPTQ 4bit 量化推理。在 GPT/OPT/BLOOM类模型上，它比传统的RTN(rount-to-nearest) 量化技术能够获得更好的 Perplexity效果。相比常见的 FP16推理，它可将显存消耗降低75%，只损失极少量的吞吐速度与Perplexity 性能。 以 ColossalChat-7B 为例，在使用 4bit 量化推理时，70亿参数模型仅需大约 4GB 显存即可完成短序列（生成长度为 128）推理，在普通消费级显卡上即可完成
```
## 其他

[参考MedicalGPT](https://github.com/shibing624/MedicalGPT)
![GPT-4训练流程](https://github.com/shibing624/MedicalGPT/raw/main/docs/GPT_Training.jpg)
* 第一阶段：PT(Continue PreTraining)增量预训练，在海量领域文档数据上二次预训练GPT模型，以注入领域知识
* 第二阶段：SFT(Supervised Fine-tuning)有监督微调，构造指令微调数据集，在预训练模型基础上做指令精调，以对齐指令意图
* 第三阶段：RM(Reward Model)奖励模型建模，构造人类偏好排序数据集，训练奖励模型，用来对齐人类偏好，主要是"HHH"原则，具体是"helpful, honest, harmless"
* 第四阶段：RL(Reinforcement Learning)基于人类反馈的强化学习(RLHF)，用奖励模型来训练SFT模型，生成模型使用奖励或惩罚来更新其策略，以便生成更高质量、更符合人类偏好的文本
