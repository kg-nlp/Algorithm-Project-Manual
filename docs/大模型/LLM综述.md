---
sort: 5
---

# LLM综述

[最新算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/LLM综述.html)
[有道云脑图](https://note.youdao.com/s/A5txdQdE)

> 关于综述论文,看了多遍,摘抄了大部分内容放到这里;每次实践其中一个小点后再回过头来看,才更加理解了作者的观点。


![脑图](https://note.youdao.com/yws/api/personal/file/WEB3af5a35b6c428fec507aa7451df36edb?method=download&shareKey=0b33dec4c5a1643a63e8f519baf729eb)

![数据预处理流程](https://note.youdao.com/yws/api/personal/file/WEB7f33bf4659b4edb0c738dc7a94e16007?method=download&shareKey=125e8321029f77c7645cffdacf29f45e)

![现有大预言模型的详细优化设置](https://note.youdao.com/yws/api/personal/file/WEB36f94cb314023ea64d5d72d893c0f82e?method=download&shareKey=fd3569d1ae77d3705d8a90edf68a6991)



## 引言

* 语言建模旨在对词序列的生成概率进行建模,以预测未来(或缺失)单词的概率
* 统计语言模型SLM
    * 基于马尔可夫假设建立词预测模型
* 神经语言模型NLM
    * 引入词的分布式表示,聚合上下文特征构建词预测函数,代表word2vec
* 预训练语言模型PLM
    * 在大规模无标签语料库进行预训练
* 大预言模型LLM
    * PLM基础上扩展模型,增加数据

## 概述

* 目前采用比较宽松的定义
    * 模型大小大于10B的语言模型是LLM
* 扩展定律
    * 任务性能大致随着模型大小的增加而显著提高
* 大模型涌现能力
    * 上下文学习,指令遵循,逐步推理
* LLM的关键技术
    * 模型大小,数据大小,总计算量扩展;分布式训练;能力引导;对齐微调;工具辅助LLM

## 大语言模型资源

* Flan-T5（11B 版本）
    * 可以作为研究指令微调的首选模型，因为它从三个方面探索指微调：增加任务数量、扩大模型规模和使用思 维链提示数据进行微调 
* CodeGen（11B）
    * 是一个为生成代码设计的自回归语言模型，可用作探索代码生成能力的候选模型
* mT0（13B）
    * 对于多语言任务，是一个比较好 的候选模型，因为它在多语言任务中使用多语言提示进行微 调。
* PanGu-α 
    * 对于中文的下游任务，具有较好的表现，特别是在零样本或小样本的设置下，该模型基于深度学 习框架MindSpore开发，拥有多个参数版本（最大版本200B参数），而最大的公开版本只有13B 参数。
* LLaMA（65B） 
    * 在与指令遵循相关的 任务中展现了卓越的性能
* OPT（175B）
    * 专注于复现和开源，旨在使研究人员能够进行大规模可重复研究
* BLOOM（176B）和 BLOOMZ（176B）
    * 对于跨语言泛化研究，可以用作基础模型，因为其在多语言语言建模任务中具有较好的能力
* OPT-IML 
    * 进行了指令微调，是研究指令微调效果的较好选择

## 常用算法库

* Transformers
* DeepSpeed 
    * 是由Microsoft开发的深度学习优化库（与PyTorch兼容），已用于训练多个大语言模型，例如MT-NLG 和 BLOOM。它提供了各种分布式训练优化技术的支持，例如内存优化（ZeRO技术、梯度检查点）和 管道并行
* Megatron-LM 
    * 是由 NVIDIA 开发的深度学习库，用于训练大语言模型。它提供了丰富的分布式训练优化技术，包括模型和数据并行、混合精度训练和FlashAttention。这些优化技术可以大大提高训练效率和速度，并实现 GPU 间 的高效分布式训练。
* JAX 
    * 是由 Google 开发的用于高性能机器学习算 法的 Python库，允许用户在带有硬件加速（例如GPU 或 TPU）的情况下进行数组的高效运算。它可以在各种设备上进行高效计算，还支持自动微分和即时编译等特色功能。
* Colossal-AI 
    * 是由 HPC-AI Tech 开发的用于训练大规模人工智能模型的深度学习库。它基于PyTorch实现，并支持丰富的并行训练策略。此外，它还可以使用PatrickStar提出的方法优化异构内存管理。最近，使用Colossal-AI 基于LLaMA开发的类ChatGPT模型ColossalChat（7B和13B版本）已经公开发布
* BMTrain 
    * 是由 OpenBMB 开发的用于以分布式方式训练大规模参数模型的高效库，强调代码简洁、低资源占 用和高可用性。BMTrain 已经将一些常见的大语言模型（如Flan-T5和GLM）迁移到到其ModelCenter 中，用户可以直接使用这些模型
* FastMoE 
    * 是一种专门用于MoE（即混合专家）模型的训练库。它基于PyTorch开发，注重效率和用户友好 性。FastMoE简化了将Transformer模型转换为MoE模型的过程，并支持数据并行和模型并行训练

## 数据收集

* 网页
    * 获得多样化的语言 知识并增强大语言模型的泛化能力
* 对话文本
    * 对话数据可以增强大语言模型的对话能力，可能也提高了大语言模型在问答任务上的表现。然而，过度引入 对话数据来训练大语言模型可能会导致一个潜在的风险：声明性指令和直白的疑问会被错误地认为是对话的开始，从 而导致指令的有效性下降
* 书籍
    * 提供了更正式的长文本，这对于大语言模型学习语言知识、建模长期依赖关系和生成连贯的文本可能带来了好处
* 多语言文本
    * 除了在单目标语言上进行训练外，整合多语言语料库可以增强多语言的理解和生成能力
* 科学文本
    * 在大量 科学文本上进行预训练，大语言模型可以在科学和推理任务中取得出色的性能。由于科学领域数据的复杂性，例如数学符号和蛋白质序列，通常需要特定的标记化和预处理技术将这些不同格式的数据转换为可以被语言模型处理的统一形式。
* 代码
    * 与自然语言文本相比，代码以编程语言的格式呈现，对应着长距离依赖和准确的执行逻辑，训练代 码可能是复杂推理能力（如思维链能力）的来源

## 数据处理

* 质量过滤
    * 基于分类器过滤掉低质量数据，关键在于自己怎么选择负样本。基于启发式方法设计一组规则消除低质量文本，• 基于语言的过滤：如果大语言模型主要用于某项语言的任务中，那么其他语言的文本可以被过滤掉。 •基于度量的过滤：可以利用生成文本的评估度量，例如困惑度（perplexity），来检测和删除不自然的句子。•基于统计的过滤：可以利用语料库的统计特征，例如标点符号分布、符号与单词比率和句子长度，来衡量文本质量并过滤低质量数据。•基于关键词的过滤：基于特定的关键词集合，可以识别和删除文本中的噪声或无用元素，例如HTML标签、超链接、模板和攻击性词语；以上几种流程规则很好实现。 
* 去重
    * 现有的研究发现，语料库中的重复数据会降低语言模型的多样性，可能导致训练过程不稳定，从而影响模型性能。在句子级别上，应删除包含重复单词和短语的低质量句子，因为它们可能会在语言建模中引入重复模式。在文 档级别上，现有研究主要依靠文档之间的表层特征（例如单 词和 n-gram 的重叠）重叠比率来检测和删除包含相似内容的重复文档。训练集中删除测试集可能出现的重复文本
* 隐私去除
    * 大多数预训练文本数据来自网络来源，包括涉及敏感或个人信息的用户生成内容，这可能增加隐私泄露的风 险。因此，需要从预训练语料库中删除可识别个人信息（PII）。一种直接有效的方法是采用基于规则的方法 
* 分词
    * 虽然直接利 用已有的分词器是方便的，但是使用专门为预训练语料库设计的分词器可能会更加有效，特别是对于由多种领域、语言和格式组成的语料库。最近的几个大语言模型使用SentencePiece 为预训练语料库训练定制化的分词器。同时利用字节级BytePairEncoding(BPE)算法确保分词后信息不丢失。需要注意的是，BPE 中的规范 化技术，例如 NFKC ，可能会降低分词性能 

## 数据影响

* 混合来源和预训练数据的数量，质量对实验影响没啥参考价值，说了跟没说一样
* 重复的数据会降低大语言模型从上下文中复制的能力，这可能进一步影响大语言模型在上下文学习中的泛化能力

## 架构

### 主流架构

* 编码器-解码器
    * 编码器采用堆叠的多头自注意层对输入序列进行编码以生成其潜在表示，而解码器对这些表示进行交叉 注意并自回归地生成目标序列。T5  BART Flan-T5 
* 因果解码器
    * 因果解码器架构采用单向注意力掩码，以确 保每个输入标记只能关注过去的标记和它本身 GPT OPT BLOOM Gopher 
* 前缀解码器
    * 前缀解码器架构（也称非因果解码器架构）修正了因果解码器的掩码机制，以使其能够对前缀标记执行双向注意力，并仅对生成的标记执行单向注意力。这样，与编码器-解码器架构类似，前缀解码器可以双向编码前缀序 列并自回归地逐个预测输出标记，其中在编码和解码过程中 共享相同的参数。PaLM GLM-130B 

### 详细配置

* 标准化 
    * 训练不稳定是预训练大语言模型的一个难题。为了 缓解这个问题，层标准化 (Layer Norm, LN) 被广泛应用 于 Transformer 架构中。LN 的位置对大语言模型的性能至关 重要。大多数 大语言模型采用前置 LN 以实现更稳定的训练。由于 RMS Norm 在训练速度和性能方面 的优越性，其在 Gopher 和 Chinchilla 中被采 用。与 LN 相比，DeepNorm 已经表现出更好的训练稳 定性，和后标准化一起被 GLM-130B 采用。 
* 激活函数
    * 为了获得良好的性能，在前馈网络中也需要设置合适的激活函数。在现有的大语言模型中，广泛使用 GeLU 激活函数。此外，在最新的大语言模型 (e.g., PaLM 和 LaMDA) 中，也使用了 GLU 激活函数的变体，特 别是 SwiGLU 和 GeGLU 变体，在实践中通常可以获得更好 的性能 。然而，与 GeLU 相比，它们在前馈网络中需要 额外的参数（约 50%） 
* 位置编码
    * 相对位置编码根据键和查询之间的偏移量生成嵌入，因此它可以在训练中看到的长度范围之外的更长序列上表现良好，即外推。 
* 注意力机制和偏差
    * GPT-3 采用了更低计算复杂度的稀疏注意力机制，即分解注意力。为了有效且高效地建模更长的 序列，研究者们探索了引入特殊的注意力模式或考虑GPU内存访问（即FlashAttention）。此外，与原始 Transformer一样，大多数大语言模型在每个线性层和层标准化中保留了偏置。然而，在PaLM和Galactica中，偏置被移除。研究表明，对于大语言模型来说，去除偏置可以 增强训练的稳定性。

### 预训练任务

* 语言模型GPT3
* 去噪自编码T5 和 GLM- 130B

## 模型训练

### 并行训练

 * 3D 并行
    * 数据并行 
        * 它将模型参数和优化器状态复制到多个 GPU 上，然后 将整个训练语料库分配到这些 GPU 上。这样，每个 GPU 只需要处理分配给它的数据，并执行前向和反向传播以获取梯度。在不同 GPU 上计算的梯度将进一步聚合以获得整个批次 的梯度，以更新所有 GPU 上的模型 
    * 流水线并行
        * 流水线并行旨在将大语言模型的不同层分布到多个GPU上。特别是在Transformer模型的情况下， 流水线并行将连续的层加载到同一GPU上，以减少在GPU之间传输计算隐藏状态或梯度的成本。然而，流水线并行的一种朴素实现可能导致 GPU利用率降低，因为每个GPU必须等待前一个 GPU 完成计算，从而导致不必要的气泡开 销 。为了减少流水线并行中的这些气泡，GPipe 和 PipeDream 提出了填充多个数据批次和异步梯度更新技 术，以提高流水线效率。 
    * 张量并行
        * 张量并行也是一种常用的技术，旨在将大语言模型分解为多GPU加载。与流水线并行不同，张量并行专注于分解大语言模型的张量（参数矩阵）。 
 * ZeRO
    * ZeRO（Zero Redundancy Optimizer）技术,专注于解决数据并行中的内存冗余问题。如前所述，数据并行需要每个 GPU 存储大语言模型的相同副本，包括模型参数、模型梯度和优化器参数。然而，并非所有上述数据都需要在每个 GPU 上保留，这将导致内存冗余 问题。为了解决这个问题，ZeRO 技术旨在仅在每个 GPU 上 保留部分数据，而当需要时，其余数据可以从其他GPU中检索。具体而言，ZeRO提供了三种解决方案，具体取决于三个数据部分的存储方式，即优化器状态分区、梯度分区和参数分区。实证结果表明，前两种解决方案不会增加通信开销，第三种解决方案会增加约50%的通信开销，但可节省与 GPU 数 量成比例的内存。PyTorch 实现了与 ZeRO 类似的技术，称 为 FSDP 
 * 混合精度训练
    * 对于预训练，BF16 通常比 FP16 在表示准确性方面表现更好 


### 优化设置

* 批量训练
    * 对于语言模型的预训练，现有的研究通常将批量 大小设置为较大的数字（例如 8,196 个样例或 1.6M 个标记），以提高训练的稳定性和吞吐量。对于像GPT-3和PaLM这样的大语言模型，它们引入了一种新的策略，在训练过程中动态增加批量大小，最终达到百万级别。具体而言，GPT-3的批量大小从32K逐渐增加到3.2M个标记。实证结果表明，批量大小的动态调整策略可以有效地稳定大语言模型的训练过程 
* 学习率
    * 现有的大语言模型通常在预训练过程中采用类似的 学习率调整策略，包括 warm-up 和 decay。 
* 优化器
    * Adam优化器和AdamW优化器被广泛应用于训练大语言模型,（例如GPT-3），这些优化器基于第一阶梯度的自适应估计的低阶矩。Adafactor 优化器也被用于训练大语言模型（例如PaLM和T5），它是 一种 Adam 优化器的变体，专门设计用于在训练过程中保存 GPU 内存 
* 稳定训练
    * 在大语言模型的预训练过程中，常常会遇到训练不稳定的问题，这可能会导致模型崩溃。为了解决这个问题， 通常会广泛使用权重衰减和梯度裁剪，其中现有的研究通常将梯度裁剪的阈值设置为 1.0，将权重衰 减率设置为0.1。然而，随着大语言模型的扩展，训练损失的峰值也更容易发生，导致训练不稳定。为了缓解这个问题，PaLM和OPT使用了一种简单的策略，即从峰值之前的一个检查点重新开始训练过程，并跳过可能导致问题的数据。此外，GLM发现嵌入层的异常梯度通常会导致峰值，因此提出缩小嵌入层梯度以缓解这个问题。 


## 参考 
* [1] [A Survey of Large Language Models](https://arxiv.org/abs/2303.18223)  
* [2] [A Survey of Large Language Models github](https://github.com/RUCAIBox/LLMSurvey/tree/main)  
* [3] [人大发表迄今为止最大最全的大模型综述](https://mp.weixin.qq.com/s?__biz=MzI3ODgwODA2MA==&mid=2247521215&idx=1&sn=bd4ac9718edd2e67e27a124441c94cc8&chksm=eb53872cdc240e3a6946073ccfdcbf2d4563d39d0fd42884e33b387c67ee6c2070fea2494965&mpshare=1&scene=2&srcid=0707gJFH8AWsFrWym2TJwqto&sharer_sharetime=1688710023091&sharer_shareid=225eb5292c43eed8f6e53f73d8e5205d#rd)
* [4] [大语言模型综述中文版-202308-LLM_Survey_Chinese](https://kdocs.cn/l/ce64NiRAZDeO)

## 资源汇总


![大模型面试基础｜秋招
](https://pic1.zhimg.com/v2-f8edb6f67f0aade01a70c198999c92c2_1440w.jpg?source=172ae18b)
* [大模型面试基础｜秋招](https://zhuanlan.zhihu.com/p/649241113)



### 模型
* [Lawyer LLaMA](https://github.com/AndrewZhe/lawyer-llama)
    * [基于chinese-llama-plus北大团队推出法律大模型，数据与模型全部开源，模型合并使用全流程](https://mp.weixin.qq.com/s/WtBwSPZ7jCzJNk3aQVllUQ)
* [中文 LLaMA & OpenLLaMA & Falcon 大模型](https://github.com/CVI-SZU/Linly)
    * [中文Falcon基础模型：代码实现与增量训练](https://zhuanlan.zhihu.com/p/636994073)
* [baichuan-7B](https://github.com/baichuan-inc/baichuan-7B)
* [ChatGLM-6B](https://github.com/THUDM/ChatGLM-6B)
* [Firefly](https://github.com/yangjianxin1/Firefly)
    * [Firefly QLoRA+百万数据，多卡高效微调bloom-7b1模型](https://mp.weixin.qq.com/s/lA4YUJ9XGpKlUUUjz0Le-g)  
* [ChatGLM2-6B 发布：性能大幅提升，8-32k上下文，推理提速42%](https://mp.weixin.qq.com/s/zDf9YbOEc681Otcjh0FJxw)
* [GLM-130B：开放的中英双语预训练模型](https://github.com/THUDM/GLM-130B/blob/main/README_zh.md)
* [Baichuan-13B](https://github.com/baichuan-inc/Baichuan-13B)
* [OpenBuddy](https://github.com/OpenBuddy/OpenBuddy/blob/main/README.zh.md)
* [MOSS](https://github.com/OpenLMLab/MOSS)
* [书生·浦语](https://github.com/InternLM/InternLM/blob/main/README-zh-Hans.md)
* [通义千文](https://github.com/QwenLM/Qwen-7B/blob/main/README_CN.md)
* [XVERSE-13B](https://github.com/xverse-ai/XVERSE-13B)



### 微调

*  [llm-action-github](https://github.com/liguodongiot/llm-action)
    * 🔥 LLM训练
        * 🐫 LLM训练实战
        * 🐼 LLM参数高效微调技术原理综述
        * 🐰 LLM参数高效微调技术实战
        * 🐘 LLM分布式训练并行技术
        * 🌋 分布式AI框架
        * 📡 分布式训练网络通信
        * 大模型结构与算法
    * 🐎 LLM推理
        * 🚀 模型推理加速
        * ✈  型推理服务化
        * 📐 LLM量化
    * 🧩 LLM应用开发
    * 🀄 LLM国产化适配
    * 🍄 LLM生态相关技术
*  [LLaMA, ChatGLM, BLOOM的参数高效微调实践](https://zhuanlan.zhihu.com/p/635710004)
* [650亿参数，8块GPU就能全参数微调：邱锡鹏团队把大模型门槛打下来了](https://mp.weixin.qq.com/s/339iXf2bimusfq6zQmFpWw)
* [微调百川Baichuan-13B保姆式教程，手把手教你训练百亿大模型](https://mp.weixin.qq.com/s/ZBY6kbogHjbCQvZBzNEqag)
* [Firefly增强Baichuan-13B的多轮对话能力](https://mp.weixin.qq.com/s/djO8Tg3emmy6wzw_rTUlcw)
* [大模型LLM微调经验总结&项目更新](https://mp.weixin.qq.com/s/wBWpjoMSLgUXammnv0kuAw)
### 其他

* [Awesome Pretrained Chinese NLP Models](https://github.com/lonePatient/awesome-pretrained-chinese-nlp-models/tree/main)
* [大模型微调究竟需要多少数据？](https://mp.weixin.qq.com/s/DVH-vlOpGik8iwW4KnPlkw)
* [ACL2023大模型如何快速构建指令遵循数据集？Self-Instruct：只需175条种子数据追上InstructGPT](https://mp.weixin.qq.com/s/ehEM04xmeJyqB4z7rmLKBQ)
* [从 FlashAttention 到 PagedAttention, 如何进一步优化 Attention 性能](https://zhuanlan.zhihu.com/p/638468472)
* [LLM从零开始训练大模型](https://zhuanlan.zhihu.com/p/636270877)
* [大模型LLM知识整理](https://zhuanlan.zhihu.com/p/641109766)
* [预训练模型微调的原理](https://zhuanlan.zhihu.com/p/35890660)
* [Don't Stop Pretraining](https://zhuanlan.zhihu.com/p/375203594)
* [浅谈一下「继续预训练」](https://zhuanlan.zhihu.com/p/545092184)
* [大模型时代-行业落地的再思考](https://mp.weixin.qq.com/s/ewI4FKX_R30E5fF6p2GYUQ)
* [灾难性遗忘](https://www.zhihu.com/question/265056068)
* [腾讯预训练模型框架](https://github.com/Tencent/TencentPretrain/blob/main/README_ZH.md)
    * TencentPretrain 是 UER-py 预训练框架的多模态版本，支持 BERT、GPT、T5、ViT、Dall-E、Speech2Text 等模型，支持文本、图像和语音模态预训练及下游任务。TencentPretrain 基于模块化设计，用户可以通过模块组合的方式构成各种模型，也可以通过复用已有的模块进行少量修改来实现新的模型。 
* [训练中文LLaMA大规模语言模型](https://zhuanlan.zhihu.com/p/612752963)
    * 基于 TencentPretrain 预训练框架训练 LLaMA 模型。 
* [LLaMA 2技术细节详细介绍！](https://mp.weixin.qq.com/s/zGQpxp865xuOIKD6e6dBVQ)
    *  Llama 2 模型接受了 2 万亿个标记的训练，上下文长度是 Llama 1 的两倍。Llama-2-chat 模型还接受了超过 100 万个新的人类注释的训练。
    *  Llama2训练语料相比LLaMA多出40%，上下文长度是由之前的2048升级到4096，可以理解和生成更长的文本。
* [GPT类模型的几个常用参数](https://juejin.cn/post/7236558485290631205)
    * temperature:用于控制模型输出的结果的随机性，这个值越大随机性越大。一般我们多次输入相同的prompt之后，模型的每次输出都不一样。一般来说，prompt 越长，描述得越清楚，模型生成的输出质量就越好，置信度越高，这时可以适当调高 temperature 的值；反过来，如果 prompt 很短，很含糊，这时再设置一个比较高的 temperature 值，模型的输出就很不稳定了。
    *    greedy decoding: 总是选择最高分的 token，有用但是有些弊端，详见下文
    *    top-k: 从 tokens 里选择 k 个作为候选，然后根据它们的 likelihood scores 来采样
    *    top-p: 候选词列表是动态的，从 tokens 里按百分比选择候选词
* [0718-LLaMA2讨论-Memo](https://d7mv45xi4m.feishu.cn/docx/OOhedFKGao2jlmxgsKGcCTnEnUc)
* [0723-LLaMA2第二次讨论-Memo](https://d7mv45xi4m.feishu.cn/docx/DOHIdmpbCoXhRwx62cCc3RcEnCh)
* [大模型人类对齐方法综述](https://mp.weixin.qq.com/s/Df_GbvhLRndeold4lkHRaA)
* [Transformer for Machine Learning: A Deep Dive](https://github.com/CRCTransformers/deepdive-book)
* [Transformer深入浅出](https://mp.weixin.qq.com/s/iBVNkvfz8usZTpPLRcghRA)
* [综述LLM的当前挑战和应用](https://mp.weixin.qq.com/s/wih8sNHCQKEfazpYwGhXaA)
* [关于检索增强下大模型知识边界的探索](https://mp.weixin.qq.com/s/G60VI3sJiMXmZxGoIBUO6Q)
* 



## 数据处理+模型训练


### 数据处理

参考 [LLM数据处理](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/LLM数据处理.html)

* 将所有数据转为excel或txt格式
* 总结数据问题,提炼规则
* 开始提取数据(PT),常规预训练模型需要的数据,编写相应规则
    * 去重算法准备
    * 分类算法准备(需要准备数据集)保证精确率
    * 流畅度模型准备
* 开始提取提示数据(SFT),需要格式清洗的文档构建,减少规则编写



## 应用想法

* 解决方案编写上下文理解问题