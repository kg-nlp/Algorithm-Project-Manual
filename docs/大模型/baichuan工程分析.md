---
sort: 8
---

# baichuan工程分析

## 简介

baichuan-7B 是由百川智能开发的一个开源可商用的大规模预训练语言模型。基于 Transformer 结构，在大约1.2万亿 tokens 上训练的70亿参数模型，支持中英双语，上下文窗口长度为4096。在标准的中文和英文权威 benchmark（C-EVAL/MMLU）上均取得同尺寸最好的效果。



## 部署

* [Baichuan-13B](https://github.com/baichuan-inc/Baichuan-13B/tree/main)

* 下载模型

```
git lfs install  (用不了)
git clone https://huggingface.co/baichuan-inc/Baichuan-13B-Chat

```

* 启动

```bash
/opt/conda/bin/streamlit run web_demo.py --server.port 8010
或者
ln -s /opt/conda/bin/streamlit /usr/local/bin/streamlit
streamlit run web_demo.py --server.port 7000

访问 http://10.0.79.103:7000
```

```
curl -X POST "http://10.0.79.103:7010/baichuan" \
     -H 'Content-Type: application/json' \
     -d '{"prompt": "你好", "history": []}'
```


* API请求
  * url: http://10.0.79.103:7010/baichuan
  * 请求格式: 
  ```
    {
        "prompt":""
    }
  ```
  * 返回格式:
  ```
     {
    "response": "",
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
