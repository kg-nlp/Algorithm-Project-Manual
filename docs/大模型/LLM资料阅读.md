---
sort: 15
---



# LLM资料阅读

*   [算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/LLM资料阅读.html)

*   [个人知乎](https://www.zhihu.com/people/zhangyj-n)



> 参考综述 [LLM综述](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/LLM综述.html)



[TOC]



## 综合资料

### 大模型研发核心：数据工程、自动化评估及与知识图谱的结合

* [原文地址-老刘说NLP](https://mp.weixin.qq.com/s/l9k49nf93nj1o3oEx3ya6w)

* [Data-centric Artificial Intelligence: A Survey](https://arxiv.org/abs/2303.10158)

**Q: 数据工程包括哪几部分**

A : 现在大家去做GPT模型或者BERT等模型，都会有两个方向。第一个是以模型为中心，不怎么关注数据，不断地优化模型的结构；第二个是以数据为中心（Data-Driven），也是目前做算法的一个共识，算法本质上是在做数据，核心是说模型不变，通过改进数据质量来提升模型效果，不断提升训练数据的质量。

以数据为中心的 AI 核心在于**训练数据开发**，**推理数据开发**以及**数据维护**。  

**训练数据开发**:怎么收集数据,怎么确实数据源,怎么标注数据,怎么预处理数据

**推理数据开发**:怎么评估训练样本,怎么评估测试集数据

**数据维护**:闭环要求,训练过程发现问题怎么定位,优化



**Q: 非中文的大模型数据构成**

A : 维基百科,书籍,期刊,社交新闻论坛网站Reddit,网站爬虫数据集,专利,arxiv论文,github,



Q: 中文的模型数据构成

A : 开源数据百度QA、DuReader、CAIL2018法律文本（几百万的法律文书数据）、搜狗 CA（搜狗的一个文本分类的数据集）,百科数据包括百度百科，搜狗百科等，以及之前大家卷知识图谱的时候开放的百科的三元组以及内部信息。电子书也有应用，但是国外有zlibary这样比较大型的书籍集合。Common Crawl,论坛,新闻,学术著作,社区QA,百科全书



Q: 我们需要怎样的预训练数据

A :

- 相关性，回答是否和问题相关，不要答非所问，体现了对问题的理解能力；
- 准确性，事实性要求回答要求完全一致，不要产生错的答案，开放性回答要求语义相近
- 完备性，是否涵盖了所有要点
- 连贯性，语言上是否表达流畅
- 安全性，是否符合地方法规以及人的价值观
- 专业性，不口水话，不啰嗦，坦白说ChatGPT比较啰嗦。
- 敏感性，是否涉及到政治理念、黄反、敏感事件等负面信息

拿到质量要求后，可以得出大模型需要高质量、大规模、多样性的数据。

**（1）高质量**

- 高质量数据集能够提高模型精度与可解释性，并且减少收敛到最优解的时间，减少训练时长；
- 高质量数据的选择依据是信源权威可靠、内容价值观对齐、专业领域知识，不会选择不入流的站点数据或者大家随便写的文章；
- 高质量的数据具有规范性、完整性、准确性、一致性、时效性，比如说GPT的时效只到2021年，那2022年、2023 年的数据也要去收集，实现时效性上的高质量。

**（2）大规模**

预训练的数据量越多，大模型的拟合能力就越强，效果就会越来越好。如果数据规模太小的话，模型学的东西不会多，记得也不够深。

**（3）多样性**

数据丰富性能够提高大模型的泛化能力，模型预训练数据足够多，其生产内容也能更多样。在准备预训练数据的时候尽可能准备更多的数据，数据多了，模型的泛化能力就会更强；而且数据足够丰富，在训练时就不会偏向某一类，导致过拟合问题的出现。所以需要对预训练数据做严格的去重，有各种花式的玩法。



Q: 如何构造领域预训练模型

A :以浙江大学CaMA模型为例,首先继续预训练，然后在预训练之后进行微调,从语种上有中英文；从类型上有代码、文本；在领域上有百科、维基等



Q: 做预训练模型要做哪些数据工作,详细说一下

A : ![](https://mmbiz.qpic.cn/mmbiz_png/zHbzQPKIBPhlaUHIqJNZPg7JDND9IHUiazqyJ3e0YYCscaOBUB0GJEYBIYUmrgz5qqTnUXC3Ps10VcEfckiaT8dw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

**为了实现数据的大规模、多样性和高质量，大致的流程包括如下几步**

一、站点过滤，站点过滤的方法也有很多，就包括基于图的过滤方法、基于单点的过滤方法、基于规则的过滤方法等；

二、敏感与隐私过滤，语言或者噪声过滤等；

三、文章去重，做不同粒度的去重；

四、网页主题建模，要提升多样性，主题就一定要好，所以会做大量的主题挖掘的工作，这里搜索有天然的优势；

五、数据质量评分，包括数据质量版本控制等。

**如何衡量标注数据集的质量**:

图像标注质量评估的MV 算法、文本质量评估的BLEU算法等

**站点过滤和噪声信息清洗有很多方法**

质量分档模型，使用fasttext分类器分为四档（0,1,2,3），2、3 为优质数据，训练时，正样本是人工标注的一些比较好的样本，负样本采用比较垃圾的文本，特征使用包含title以及CEloss。

边缘文本剔除模型，需要将广告位文本、杂七杂八的推广文本识别出来。

垂直网页处理，包括用大量的Pattern做高优语料提取以及定制化的边缘文本剔除。

基于规则的噪音清洗，包括空格、特殊符号的处理、语种检测，敏感信息检测、隐私数据识别与处理等等。

基于模型的噪声清洗，包括使用PPL判定模型，剔除不连贯的文本等

**更多内容参考**解决方案:以数据为中心的大模型预训练数据工程

Q: 怎么进行模型性能的自动化评分

A : 模型性能的自动化评估基本上有三种方式。

- 第一、基于人工业务评估，人工根据特定的业务场景找到需要评估的能力点（如摘要能力，生成能力等）通过列举相关测试样本，建立评估维度，完成多维度打分；
- 第二、基于下游任务评测，利用下游评测榜单，任务数据集，进行性能评估。客观题比较适合用下游任务去评测，但主观题的话不是特别适合，比如评估生成的好不好等。
- 第三、基于ChatGPT打分，现在有一个风向，大家用 ChatGPT 打分，利用ChatGPT 的专业能力，充当裁判，完成打分评估。



Q: 知识图谱和大模型的相同点

A :



Q: 知识图谱和大模型的不同点

A :



Q: 大模型出现幻觉怎么解决

A :





## 问答相关的内容



### [LangChain+LLM大模型问答能力搭建与思考](https://zhuanlan.zhihu.com/p/644740531)

1. 数据处理和清洗

   * **关于目录：**目录部分应提前识别并删除
     这部分关键词巨多（误命中）、各种标点符号巨多（误分段）、大多不是自然语言（用来做目录索引的，没实际的文本含义），引入目录，会在文档召回阶段通过向量或关键词产生大量误召回，建议直接咔嚓掉

   * **文档分段长度：**模型允许的情况下，能长则长；可根据数据酌情向后扩展
     * **在大模型可接受文本总字数有限的情况下，增加召回数量和增加单文档长度的权衡**
     * 规范的内容质量较高,但有些章节重复程度很高,需要处理
     * 分段采用普通符号,或者langchain的chunk方法,如果是截成短文本,对于有序号的这种,可以把上下文给输出(我们暂且任务大模型对长文本的总结推理能力很强)
   * 文件格式多种多样,是否要内置很多模型进行处理
   * 有研究表明[Lost in the Middle: How Language Models Use Long Contexts](https://arxiv.org/abs//2307.03172)长文本作为输入LLM输入时，LLM倾向于更关注长文本的开头、结尾处，然而中间部分的语义可能会被忽略。

2. 模型

   1. 温度系数设置，如果是chatgpt,temperature<=0.1较好,但chatglm设置成0.01都不怎么样
   2. LangChain+LLM,是针对**多文档下的知识点问答**的，目标是在海量语料中精准找到几小段参考文档然后总结出答案；单文档总结,类似ChatPDF,范围小
   3. 文本还是用检索+问答的方式解决,表格转换成NL2SQL解决。prompt尽量增加一步步思考引导
   4. 对于表格的数据,最好引导做推理,可以先输出代码,再输出计算结果

```
res = chatglm.chat(tokenizer, promptValue, history=[],do_sample=False)
do_sample=False，可以使结果稳定，且是好的结果
```



### [LLM+Embedding构建问答系统的局限性及优化方案](https://zhuanlan.zhihu.com/p/641132245)



**领域知识入库**

1. 数据源可能来自于网络（游戏已经对外的攻略）、本地文本文件（技术文档、设计稿）或者数据库（业务自己维护的 UGC，比如用户帖子、评论等）。采用合适的方式收集这些数据并整理为纯文本的格式。这里提供一个python 库[**textract**](https://link.zhihu.com/?target=https%3A//textract.readthedocs.io/en/stable/)，支持从多种类型文件中提取文字信息，普通文本文件自不必说，其它各种常用格式文件也都支持，比如：Microsoft 全家桶 docx, xlsx；图像gif, jpg等；音频文件mp3, ogg等。

2. 生成分词器 tokenizer，将文本分成一个个词元，保证各个词元拥有相对完整和独立的语义，以供后续任务比如 Embedding 使用。[tiktoken](https://link.zhihu.com/?target=https%3A//github.com/openai/tiktoken)是一种 [Byte Pair Encoding(BPE)](https://link.zhihu.com/?target=https%3A//en.wikipedia.org/wiki/Byte_pair_encoding) 分词器，有多种编码方法可选，如：r50k_base, p50k_base, cl100k_base等。面向 OpenAI 的gpt-4, gpt-3.5-turbo和text-embedding-ada-002模型通常使用cl100k_base编码方法。

3. 分片。将原始知识库拆分为若干个独立、较短的知识点。每个知识点会作为问答的最小记录，与问题进行匹配。在实际使用过程中有以下几点建议：

4. 1. 原始内容在编写、组织时最好原子化、正交化。对于树状结构的知识点，可以按层级关系表示，最好不要混为一谈。比如倚天剑可能基础属性，也有适合的打法，偏向的英雄天赋，那么三者应该独立描述，而不要混杂在一起。
   2. 可以在原始语料中设计明确的分片标记，简化处理过程。对于 html、markdown 等类型的文档而言，天然结构化处理会简单一些。
   3. 基本的分片方式。粒度从细到粗可以使用，标点符号、段落、章节等进行区分。分片粒度过细，知识点会比较零碎影响了相互间的关系；分片粒度过粗，在匹配时可能会携带冗余信息，另外对 Embedding、处理、索引的效率也有影响。
   4. 分片要使用 tokenizer，原始文本经过分词然后再进行 embedding，分片大小需要考虑分词之后生成的 token 数量。基本目标是：分片不能破坏知识点的完整性，生成的分片对应的 token 数量应该在预设范围内，不要过小或过大。

5. 词嵌入(Embedding)。使用 OpenAI API 对每个分片后的每个知识点进行处理，获得向量化的结果。这里需要调用 `openai.Embedding.create` 接口。

6. 存储。将 Embeddings 生成的向量连同原始分片（知识点），以 kv 形式存储，便于后续快速匹配索引。专业的解决方案是 [vector database](https://link.zhihu.com/?target=https%3A//learn.microsoft.com/en-us/semantic-kernel/concepts-ai/vectordb)，但实际上很多传统的数据库或存储中间件也已经提供了支持，比如：

- **RediSearch 提供的 [Vector Similarity](https://link.zhihu.com/?target=https%3A//redis.io/docs/stack/search/reference/vectors/)** ，支持使用向量字段和向量相似性查询。它可以加载、索引和查询存储在 Redis 哈希或 JSON 文档（通过与 RedisJSON 模块集成）中的向量。Vector Similarity 提供了实时向量索引、实时向量更新/删除、K-最近邻（KNN）搜索和范围过滤等功能。
- [pgvector](https://link.zhihu.com/?target=https%3A//github.com/pgvector/pgvector)基于 PostgresQL，提供了类似的向量索引支持。和 Redis 的基本功能差不多，在向量距离计算方面，也提供了：L2、点积和 COSINE 这三种方法。

使用 Redis 比较简单高效，接口和文档非常丰富，如果没有特别要求可以直接使用。



重点还是数据处理,数据存储,数据索引,还是用faiss或hnswlib吧



**Q:[LangChain](https://link.zhihu.com/?target=https%3A//python.langchain.com/) + LLM 局限性**

A:LLM意图识别准确性较低，交互链路长导致时间开销大；Embedding 不适合多词条聚合匹配等



**Q:该项目embedding-search召回精度低的问题**

A:问题询问知识点多,某个概念下,很多范围数据,进行查询,对比,统计(皮蓬、英格利什和布兰德谁的第一位置是 PF？,皮蓬、英格利什和布兰德谁的金徽章数最多？)

问题复杂,我觉的可以理解为核心词和附属属性多,查询范围大



**Q:Embedding 缺陷**

A:本地知识建立索引时，通常对单个知识点进行 Embedding,而不会对知识点进行排列组合存储;

为了增加召回数量,降低相似度阈值,召回的结果无效信息,噪声太多,还有知识点容易被截断;召回结果和LLM交互时间长,总结结果也不太好



**Q:针对上面Embedding召回的缺陷和LLM对冗余数据的总结推理出错的问题有什么可以参考的解决方案**

A:

一.问题理解——准确识别用户意图(对于多轮交互的方法,在问答比赛中不合适)

[Precise Zero-Shot Dense Retrieval without Relevance Labels](https://link.zhihu.com/?target=https%3A//arxiv.org/abs/2212.10496)

1. 使用基础模型在训练过程中已经掌握的相关语料，面向用户问题，生成虚构的文档
2. 将虚构的文档向量化,和本地知识库进行相似性检索

二.关键词提取

关键词召回,增加同义词,去除停用词(可以大量召回数据,但原文本长度是个问题)

想法:百度解语 应用场景A：[文本挖掘/解析模板生成与匹配](https://github.com/PaddlePaddle/PaddleNLP/tree/develop/examples/text_to_knowledge)

谓语。体现名词短语之间的事件。比如：评价。在更多场景中是：比较、查询、过滤、统计等，存在动态计算、处理的需求。因此即使忽略谓语，仅通过名词短语的组合也可以获得完备的知识。谓语所代表的事件，可以交给 LLM 处理

* 关键词只考虑名词短语，将其作为知识点的索引用于召回。

- 事件体现的是不同知识点的关系。重点在于事件关联者，获取事件关联者对应的语料后，通过 LLM 理解事件进行处理：

- - 知识点通常不是按照事件组织的，而是按实体。因此索引建立和关键词抽取，应该以名词短语为主要目标。
  - 主语、宾语之间存在并列和限定关系。将限定关系按级联方式处理，同一名词短语内如果存在并列关系则与其他部分叉乘。

微调指令,多些这种复杂问题的数据,把相关的知识要放到LLM中,从提取的数据中生成问答对(能不能行,是个问题)



**Q: 基于LLM提取的一些问题**

A:可以支持部分程度的 NER；对非专业名词、或业务自定义术语识别不好，需要Fine-Tuning。

- 可以提取名词短语，但无法做精确的语义分析，大量测试案例下总是存在不少偏差。
- 即使明确了输出格式，但还是频繁出错。
- 耗时较长。提取 6 条测试案例耗时 12s。



**Q: ==方法总结==**

A: 多知识点聚合处理问题的方案：

* 基于传统 NLP 的成分句法分析，提取名词短语；再通过短语间的依存关系，生成关键词列表
* 从完整语句的 Embedding，切换为关键词 Embedding：
  * 知识库构建时。基于单知识点入库，入库时提取关键词列表进行 Embedding，用于检索。
  * 查询时。对用户的问题提取关键词列表进行 Embedding 后，从本地知识库命中多条记录。
* 将单问句中的多知识点拆解后检索，将召回的多条记录交付给 LLM 整合

优势:

* 相比传统 Embedding，大幅提升召回精准度。

* 支持单次交互，对多知识点进行聚合处理。而不必让用户，手动分别查询单个知识点，然后让 LLM 对会话历史中的单个知识点进行汇总。
* 使用传统 NLP 在专项问题处理上，相比 LLM 提供更好的精度和性能。
* 减少了对 LLM 的交互频次；提升了交付给 LLM 的有效信息密度；大大提升问答系统的交互速度。



### [稠密检索模型的zero-shot能力究竟如何？](https://zhuanlan.zhihu.com/p/506405755)



**Q: 背景问题**

A:





### 如何避免大语言模型绕过知识库乱答的情况？LlamaIndex 原理与应用简介

[链接](https://mp.weixin.qq.com/s/D6_pUv7hHZHRrKSXqo0u2w)

[链接](https://zhuanlan.zhihu.com/p/638827267)





### ChatPDF | LLM文档对话 | pdf解析关键问题

[链接](https://zhuanlan.zhihu.com/p/652975673)





### 大模型外挂知识库优化技巧-如何更有效的利用召回的文档

[链接](https://mp.weixin.qq.com/s/C5abuYtUYI1vzh7C3EGJUQ)















