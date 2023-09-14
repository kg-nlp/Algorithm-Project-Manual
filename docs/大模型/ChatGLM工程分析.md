---
sort: 7
---

# ChatGLM工程分析


* [算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/ChatGLM工程分析.html)

* [个人知乎](https://www.zhihu.com/people/zhangyj-n)

[TOC]

## 资料

[GLM-130B](https://github.com/THUDM/GLM-130B/blob/main/README_zh.md)

GLM-130B 是一个开源开放的双语（中文和英文）双向稠密模型，拥有1300亿个参数，模型架构采用通用语言模型（GLM）。

[ChatGLM2-6B](https://github.com/THUDM/ChatGLM2-6B)

ChatGLM2-6B 是开源中英双语对话模型 ChatGLM-6B 的第二代版本，在保留了初代模型对话流畅、部署门槛较低等众多优秀特性的基础之上，ChatGLM2-6B 引入了如下新特性：

* **更强大的性能**：基于 ChatGLM 初代模型的开发经验，我们全面升级了 ChatGLM2-6B 的基座模型。ChatGLM2-6B 使用了 GLM 的混合目标函数，经过了 1.4T 中英标识符的预训练与人类偏好对齐训练，评测结果显示，相比于初代模型，ChatGLM2-6B 在 MMLU（+23%）、CEval（+33%）、GSM8K（+571%） 、BBH（+60%）等数据集上的性能取得了大幅度的提升，在同尺寸开源模型中具有较强的竞争力。
* **更长的上下文**：基于 FlashAttention 技术，我们将基座模型的上下文长度（Context Length）由 ChatGLM-6B 的 2K 扩展到了 32K，并在对话阶段使用 8K 的上下文长度训练，允许更多轮次的对话。但当前版本的 ChatGLM2-6B 对单轮超长文档的理解能力有限，我们会在后续迭代升级中着重进行优化。
* **更高效的推理**：基于 Multi-Query Attention 技术，ChatGLM2-6B 有更高效的推理速度和更低的显存占用：在官方的模型实现下，推理速度相比初代提升了 42%，INT4 量化下，6G 显存支持的对话长度由 1K 提升到了 8K。
* **更开放的协议**：ChatGLM2-6B 权重对学术研究完全开放，在获得官方的书面许可后，亦允许商业使用。如果您发现我们的开源模型对您的业务有用，我们欢迎您对下一代模型 ChatGLM3 研发的捐赠。



[2023/07/04] 发布 P-Tuning v2 与 全参数微调脚本，参见 [P-Tuning](https://github.com/THUDM/ChatGLM2-6B/tree/main/ptuning)。

[ChatGLM 的 Prompt 工程实践教程](https://lslfd0slxc.feishu.cn/docx/Nqm9dX81hotVYUxFQuxcVR82n2g)


## 部署
* ### 网页端启动

```bash

ln -s /opt/conda/bin/streamlit /usr/local/bin/streamlit
streamlit run web_demo2.py --server.port 8003
网页访问:
访问 http://10.0.79.103:7020
```

```
post请求
curl -X POST "http://10.0.79.103:7030/chatglm" \
     -H 'Content-Type: application/json' \
     -d '{"prompt": "你好", "history": []}'
```


* ### API请求
  
  * url: http://10.0.79.103:8001/chatglm
  * 请求格式: 
  
  ```
    {
        "prompt":"",
        "history": [],
        "max_length": "",
        "top_p":"",
        "temperature":""
    }
  ```
  * 返回格式:
  ```
     {
    "response": "。",
    "history": [
        [
            "",
            ""
        ]
    ],
    "status": 200,
    "time": "2023-07-25 09:47:00"
    }
  ```
  
  * 请求示例

```python
	# -*- coding:utf-8 -*-
'''
# @FileName    :request.py
---------------------------------
# @Time        :2023/7/25 
# @Author      :
# @Email       :
---------------------------------
# 目标任务 :   chatglm-2-6b 和 baichuan13b 模型服务
# 二期环境: 10.0.79.103
# chatglm-2-6b 网页端  http://10.0.79.103:7020
# baichuan13b 网页端  http://10.0.79.103:7000
---------------------------------
'''

import json
import requests
headers = {
  "Content-Type": "application/json"
}
def get_chatglm_info(data:dict)->dict:
    raw_data = json.dumps(data)
    res = requests.post("http://10.0.79.103:7030/chatglm",headers=headers, data=raw_data)  # ChatGLM访问

    result = json.loads(res.text)
    print(json.dumps(result,indent=False,ensure_ascii=False))
    return result

def get_baichuan_info(data:dict)->dict:
    raw_data = json.dumps(data)
    res = requests.post("http://10.0.79.103:7010/baichuan",headers=headers, data=raw_data)  # baichuan访问

    result = json.loads(res.text)
    print(json.dumps(result,indent=False,ensure_ascii=False))
    return result



if __name__ == "__main__":

    # chatglm请求示例
    chatglm_data = {
        "prompt":"",
        "history": [],
        "max_length": "",
        "top_p":"",
        "temperature":""
    }

    # baichuan请求示例
    baichuan_data = {
        "prompt":""
    }

    get_chatglm_info(chatglm_data)
    get_baichuan_info(baichuan_data)
```



## prompt设计-ChatGLM2-6B



* Prompt万能公式： 任务+生成主体+细节（可选）+形式（可选）
* 原则一:编写清晰,具体的指令
  - 可以使用更长的指令,分行编写
  - 使用明显的分隔符**待解析的文本**和**prompt**区分开,分隔符可以使用`''',<>,<tag>,<\tag>`等
      - 比如:三个反引号括起来的文本
  - 结构化输出
      - 比如json: 以JSON格式提供,其中包含以下键:xxx,xxx,xxx
  - 要求模型检查是否满足条件,给出其他解,防止模型自由发挥
      - 比如: 如果文本无法接下,则直接输出xxx
  - 提供少量示例
      - 模仿输出风格,few-shot任务
- 原则二:给模型时间思考
    - 将一个任务拆开几步去解决,理解过程,输出结果
    - 涉及计算方案比较,先让模型自己解决,再去做对比(给出模型怎么比较的流程prompt)
    - 模型预测有一定的局限性,会生成一些虚假知识,最好提示模型找到文本中的一些引用,根据引用进行回答，或者进行模型检查其他另外说明。

