---
sort: 14
---

# InternLM工程分析


* [算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/InternLM工程分析.html)

* [个人知乎](https://www.zhihu.com/people/zhangyj-n)


[InternLM](https://github.com/InternLM/InternLM/blob/main/README-zh-Hans.md)

InternLM ，即书生·浦语大模型，包含面向实用场景的70亿参数基础模型与对话模型 （InternLM-7B）。模型具有以下特点：

使用上万亿高质量语料，建立模型超强知识体系；
支持8k语境窗口长度，实现更长输入与更强推理体验；
通用工具调用能力，支持用户灵活自助搭建流程



## 部署
* 启动

```bash

ln -s /opt/conda/bin/streamlit /usr/local/bin/streamlit
streamlit run web_demo2.py --server.port 7000
网页访问:
访问 http://10.0.79.103:7000
```

```
post请求
curl -X POST "http://10.0.79.103:7010/internlm" \
     -H 'Content-Type: application/json' \
     -d '{"prompt": "你好", "history": []}'
```


* API请求
  * url: http://10.0.79.103:7010/internlm
  * 请求格式: 
  ```
    
  ```
  * 返回格式:
  ```
   
  ```
