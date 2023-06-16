---
sort: 6
---

# MedicalGPT源码阅读  

[算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/%E5%A4%A7%E6%A8%A1%E5%9E%8B/MedicalGPT%E6%BA%90%E7%A0%81%E9%98%85%E8%AF%BB.html)
[知乎](https://zhuanlan.zhihu.com/p/637090041)


> 截止0614   


## 环境配置

[ColossalAI源码分析](https://kg-nlp.github.io/Algorithm-Project-Manual/%E5%A4%A7%E6%A8%A1%E5%9E%8B/ColossalAI%E6%BA%90%E7%A0%81%E5%88%86%E6%9E%90.html)

### 硬件信息

```bash
# 宿主机
cat /proc/cpuinfo | grep name | cut -f2 -d: | uniq -c
#     96  Intel(R) Xeon(R) Gold 5220R CPU @ 2.20GHz
# 内存:256G
# 显卡: T4  15g  4卡
# | NVIDIA-SMI 515.65.01    Driver Version: 515.65.01    CUDA Version: 11.7     |

```

```bash
# 开发环境使用colossal_env环境 
cd /workspace/MedicalGPT/
pip install -r requirements.txt
# 安装结果如下
```

```
Successfully installed 
MarkupSafe-2.1.3 
absl-py-1.4.0 
accelerate-0.20.3 
cachetools-5.3.1 
cmake-3.26.4 
google-auth-2.20.0 
google-auth-oauthlib-1.0.0 
grpcio-1.54.2 
importlib-metadata-6.6.0 
jinja2-3.1.2 
lit-16.0.5.post0 
loguru-0.7.0 
markdown-3.4.3 
mpmath-1.3.0 
networkx-3.1 
nvidia-cublas-cu11-11.10.3.66 
nvidia-cuda-cupti-cu11-11.7.101 
nvidia-cuda-nvrtc-cu11-11.7.99 
nvidia-cuda-runtime-cu11-11.7.99 
nvidia-cudnn-cu11-8.5.0.96 
nvidia-cufft-cu11-10.9.0.58 
nvidia-curand-cu11-10.2.10.91 
nvidia-cusolver-cu11-11.4.0.1 
nvidia-cusparse-cu11-11.7.4.91 nvidia-nccl-cu11-2.14.3 
nvidia-nvtx-cu11-11.7.91 
oauthlib-3.2.2 
peft-0.3.0 
pyasn1-0.5.0 pyasn1-modules-0.3.0 
requests-oauthlib-1.3.1 
rsa-4.9 
sympy-1.12 
tensorboard-2.13.0 tensorboard-data-server-0.7.1 
torch-2.0.1 
transformers-4.30.2
triton-2.0.0 
trl-0.4.4 werkzeug-2.3.6 
zipp-3.15.0
```

### 问题

* package版本

```bash
# ModuleNotFoundError: No module named 'torch._six'
# 降低torch版本 peft 0.3.0 requires torch>=1.13.0
pip uninstall torch 
pip install torch==1.13.0
pip install scikit-learn

apt-get install tree
```

* ValueError: Tokenizer class BloomTokenizer does not exist or is not currently imported.

```python
# 使用rl_training.py模型使用奖励模型的分词器为AutoTokenizer,修改分词器
reward_tokenizer = tokenizer_class.from_pretrained(
        args.reward_model_name_or_path, **tokenizer_kwargs
    )
```


## 训练流程
![GPT-4训练流程](https://github.com/shibing624/MedicalGPT/raw/main/docs/GPT_Training.jpg)

*   第一阶段：PT(Continue PreTraining)增量预训练，在海量领域文档数据上二次预训练GPT模型，以注入领域知识 &#x20;

*   第二阶段：SFT(Supervised Fine-tuning)有监督微调，构造指令微调数据集，在预训练模型基础上做指令精调，以对齐指令意图 &#x20;

*   第三阶段：RM(Reward Model)奖励模型建模，构造人类偏好排序数据集，训练奖励模型，用来对齐人类偏好，主要是"HHH"原则，具体是"helpful, honest, harmless" &#x20;

*   第四阶段：RL(Reinforcement Learning)基于人类反馈的强化学习(RLHF)，用奖励模型来训练SFT模型，生成模型使用奖励或惩罚来更新其策略，以便生成更高质量、更符合人类偏好的文本 &#x20;

## [模型训练细节](https://github.com/shibing624/MedicalGPT/wiki/%E8%AE%AD%E7%BB%83%E7%BB%86%E8%8A%82%E8%AF%B4%E6%98%8E)

* 训练脚本在scripts目录下：
    * 第一阶段：PT(Continue PreTraining)增量预训练 run_pt.sh
    * 第二阶段：SFT(Supervised Fine-tuning)有监督微调 run_sft.sh
    * 第三阶段：RM(Reward Model)奖励模型建模 run_rm.sh
    * 第四阶段：RL(Reinforcement Learning)基于人类反馈的强化学习 run_rl.sh

* 增量训练

```
使用百科类文档类数据集，用来在领域数据集上增量预训练或二次预训练，期望能把领域知识注入给模型，以医疗领域为例，希望增量预训练，能让模型理解感冒的症状、病因、治疗药品、治疗方法、药品疗效等知识，便于后续的SFT监督微调能激活这些内在知识。

这里说明一点，像GPT3、LLaMA这样的大模型理论上是可以从增量预训练中获益，但增量预训练需要满足两个要求：1）高质量的预训练样本；2）较大的计算资源，显存要求高，即使是用LoRA技术，也要满足block_size=1024或2048长度的文本加载到显存中。

其次，如果你的项目用到的数据是模型预训练中已经使用了的，如维基百科、ArXiv等LLaMA模型预训练用了的，则这些数据是没有必要再喂给LLaMA增量预训练，而且预训练样本的质量如果不够高，也可能会损害原模型的生成能力。

如果你的显存不足，可以改小batch_size=1, block_size=512（影响训练的上下文最大长度）;
如果你的显存更大，可以改大block_size=2048, 此为llama原始预训练长度，不能更大啦；调大batch_size。

```

> 后续的监督,奖励,强化可以参考作者后续的代码,写的即清晰又简洁

* 训练脚本稍稍修改

```bash
#其余参数默认
#!/usr/bin/env bash
CUDA_VISIBLE_DEVICES=1,2 torchrun --nproc_per_node 2 pretraining.py \
    --model_type bloom \
    --model_name_or_path /workspace/ColossalAI/LLM/bloom-560m \
    --train_file_dir ../data/pretrain \
    --validation_file_dir ../data/pretrain \
    --per_device_train_batch_size 1 \
    --per_device_eval_batch_size 1 \
    --do_train \
    --do_eval \
    --use_peft True \
    --output_dir outputs-pt-v1 \

#!/usr/bin/env bash
CUDA_VISIBLE_DEVICES=1,2 torchrun --nproc_per_node 2 supervised_finetuning.py \
    --model_type bloom \
    --model_name_or_path outputs-pt-v1 \
    --train_file_dir ../data/finetune \
    --validation_file_dir ../data/finetune \
    --per_device_train_batch_size 4 \
    --per_device_eval_batch_size 4 \
    --output_dir outputs-sft-v1 \
    
#!/usr/bin/env bash
CUDA_VISIBLE_DEVICES=1,2 python reward_modeling.py \
    --model_type bloom \
    --model_name_or_path outputs-sft-v1 \
    --train_file_dir ../data/reward \
    --validation_file_dir ../data/reward \
    --output_dir outputs-rm-v1 \

#!/usr/bin/env bash
CUDA_VISIBLE_DEVICES=1,2 torchrun --nproc_per_node 2 rl_training.py \
    --model_type bloom \
    --model_name_or_path outputs-sft-v1 \
    --reward_model_name_or_path outputs-rm-v1 \
    --torch_dtype float16 \
    --output_dir outputs-rl-v1 \
```


* 关于LoRA Training

默认使用LoRA训练，每个stage的LoRA模型权重都需要合并到base model中，使用以下命令合并，下一个stage的model_name_or_path指定为合并后的模型文件夹(至run_sft.sh,run_rm.sh,run_rl.sh中的model_name_or_path)
合并后的权重保存在output_dir目录下，后续可通过from_pretrained直接加载
```bash
#!/usr/bin/env bash
#python merge_peft_adapter.py \
#    --model_type bloom \
#    --base_model_name_or_path /workspace/ColossalAI/LLM/bloom-560m \
#    --peft_model_path outputs-pt-v1 \
#    --output_dir outputs-pt-v1

#python merge_peft_adapter.py \
#    --model_type bloom \
#    --base_model_name_or_path outputs-pt-v1 \
#    --peft_model_path outputs-sft-v1 \
#    --output_dir outputs-sft-v1
#
#python merge_peft_adapter.py \
#    --model_type bloom \
#    --base_model_name_or_path outputs-pt-v1 \
#    --peft_model_path outputs-rm-v1 \
#    --output_dir outputs-rm-v1

#python merge_peft_adapter.py \
#    --model_type bloom \
#    --base_model_name_or_path /workspace/ColossalAI/LLM/bloom-560m \
#    --peft_model_path outputs-rm-v1 \
#    --output_dir outputs-rm-v1

python merge_peft_adapter.py \
    --model_type bloom \
    --base_model_name_or_path outputs-pt-v1 \
    --peft_model_path outputs-rl-v1 \
    --output_dir outputs-rl-v1
```

* 测试结果

第一次运行完run_pt.sh后生成的output_dir目录内容如下

```bash
├── adapter_config.json
├── adapter_model.bin
├── all_results.json
├── eval_results.json
├── runs
│   └── Jun14_03-02-43_16fa8e855ab3
│       ├── events.out.tfevents.1686711786.16fa8e855ab3.94285.0
│       └── events.out.tfevents.1686711826.16fa8e855ab3.94285.1
├── special_tokens_map.json
├── tokenizer.json
├── tokenizer_config.json
├── train_results.json
├── trainer_state.json
└── training_args.bin
```

经过合同lora权重后的目录: 增加了config.json,generation_config.json,pytorch_model.bin

```bash
├── adapter_config.json
├── adapter_model.bin
├── all_results.json
├── config.json
├── eval_results.json
├── generation_config.json
├── pytorch_model.bin
├── runs
│   └── Jun14_03-02-43_16fa8e855ab3
│       ├── events.out.tfevents.1686711786.16fa8e855ab3.94285.0
│       └── events.out.tfevents.1686711826.16fa8e855ab3.94285.1
├── special_tokens_map.json
├── tokenizer.json
├── tokenizer_config.json
├── train_results.json
├── trainer_state.json
└── training_args.bin
```

最终结果outputs-rl-v1

```
├── adapter_config.json
├── adapter_model.bin
├── config.json
├── generation_config.json
├── pytorch_model.bin
├── special_tokens_map.json
├── tokenizer.json
├── tokenizer_config.json
└── trl
    ├── 1686733231.3986437
    │   └── events.out.tfevents.1686733231.16fa8e855ab3.9130.1
    ├── 1686733231.4012823
    │   └── hparams.yml
    ├── 1686733467.3128846
    │   └── events.out.tfevents.1686733467.16fa8e855ab3.11906.1
    ├── 1686733467.3156772
    │   └── hparams.yml
    ├── events.out.tfevents.1686733231.16fa8e855ab3.9130.0
    └── events.out.tfevents.1686733467.16fa8e855ab3.11906.0
```


### 开发环境更新

```
# 将上述对开发环境做的操作进行commit生成新镜像
# 在此环境运行原ColossalAI不受影响
```



## 作者关于大模型的一些思考答疑

* 什么情况用Bert模型，什么情况用LLaMA、ChatGLM类大模型
    * 1）NLU相关的任务，用BERT模型能处理的很好，如实体识别、信息抽取、文本分类，没必要上大模型；
    * 2）NLG任务，纯中文任务，用ChatGLM-6B，需要处理中英文任务，用 chinese-llama-plus-7b 或者 chinese-alpaca-plus-7b-hf 
* ChatGLM-6B与LLaMA-7B的区别
    *  chatGLM-6B是用的GLM模型结构，prefix LM，它的attentionmask部分，prefix部分的token是互相能看到，模型设计之初考虑NLU任务和NLG任务。
    *  LLaMA-7B是GPT模型结构，causal LM，它的attention mask部分，只有后面的token能看到前面的token，单向的从左到右，decoder only。
* 微调需要多少条数据
    *  取决于预训练数据和微调任务的数据分布是否一致，分布一致，100条就够，分布差异大就需要多些数据，千条或者万条以上为佳。自己的任务复杂或者下游任务行业比较冷门，如药品名称识别任务，则需要较多监督数据。还有微调大模型时，一遍是记不住的。100条的微调数据，epochs=20才能稳定拟合任务要求。

## 参考

[MedicalGPT](https://github.com/shibing624/MedicalGPT)
[作者blog](https://blog.csdn.net/mingzai624?type=blog)
