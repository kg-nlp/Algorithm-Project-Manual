***

## sort: 11

# LLM数据处理

*   [算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/LLM数据处理.html)

*   [个人知乎](https://www.zhihu.com/people/zhangyj-n)

## 数据处理原则

*   文件格式
    *   txt
    *   pdf
    *   doc
    *   docx
    *   parquet
    *   zip
    *   tar
    *   rar
    *   xls
    *   xlsx

*   数据解析
    *   PDF转docx,docx转txt
    *   doc转docx,docx转txt
    *   xls转xlsx,xlsx转txt
    *   其他压缩包解压至上述格式
    *   排版格式错乱数据,规则提取

*   数据过滤
    *   对解析数据执行规则过滤,处理方法见 #数据处理方法

## 数据处理方法

*   数据过滤论文 [The RefinedWeb Dataset for Falcon LLM](https://zhuanlan.zhihu.com/p/641013454)

*   方法1 [文档规则解析](https://kg-nlp.github.io/Algorithm-Project-Manual/文档解析/文档规则解析.html)

*   工具参考
    *   [JioNLP](https://github.com/dongrixinyu/JioNLP)
        *   正则抽取与解析 jionlp.clean\_text(text)
    *   [pandas](https://github.com/pandas-dev/pandas)
        *   没啥可说的
    *   [Scrubadub](https://github.com/LeapBeyond/scrubadub/tree/d95cb8d4d0f6d27691c0c2de1ee7ab54e535a970)
        *   清除电子邮件地址,网址,姓名,Skype 用户名,电话号码,密码/用户名组合,社会安全号码
    *   [Modin](https://github.com/modin-project/modin)
        *   加速pandas
    *   [Ftfy](https://github.com/rspeer/python-ftfy)
        *   乱码转换（中文不友好）
    *   [Beautifier](https://github.com/sachin-philip/beautifier)
        *   清洗 URL 和 Email 地址并让美化它们

### 方法总结

| 措施                                                                       | 方法                           | 完成情况      |
| ------------------------------------------------------------------------ | ---------------------------- | --------- |
| 迁移目录下所有文件                                                                | 自定义脚本                        | √         |
| 文本格式转换                                                                   | 自定义脚本                        | √         |
| pdf批量转docx                                                               | 使用WPS                        | √         |
| 转换前后文件对比查漏                                                               | 自定义脚本                        | x         |
| 过滤：文件名存在副本,文件大小一致判断                                                      | 自定义脚本                        | x         |
| 压缩包解压                                                                    | 手动解压,自定义脚本                   | √         |
| 文档只保留正文,过滤导航栏文本,标题,脚注,页眉页脚                                               | 自定义脚本                        | x         |
| 去除 html 标签、去除异常字符、去除冗余字符、去除括号补充内容、去除 URL、去除 E-mail、去除电话号码，将全角字母数字空格替换为半角 | pip工具+自定义脚本                  | √         |
| 整理无效禁用词，标点符号过多的行去掉                                                       | 自定义脚本                        | x         |
| 过滤：去重                                                                    | 自定义脚本，分阶段过滤：完全匹配，近似匹配，hash去重 | √(需要增加规则) |

## 数据源文件整理

> 只处理领域数据

数据目录: D:\项目-预训练模型\数据\业务数据

| 目录   | 格式           | 目录大小  |
| ---- | ------------ | ----- |
| 工艺手册 | 多为pdf和doc    | 225M  |
| 施工方案 | txt          | 472M  |
| 规范搜索 | txt,doc,pdf  | 7.6M  |
| 论文数据 | pdf          | 351M  |
| 合同数据 | pdf,doc,xls等 | 9.44G |

## 数据处理结果

> 结果文件用于LLM/PTM, 用于业务标注数据(弥补原标注数据不足的问题)

### 格式

*   每篇token数/每篇样本数

```
{   
    "doc_id":"id",
    "doc_name":"文档名",
    "doc_hash":"文档hash值"
    "doc_tokens":"总token数",
    "doc_sentences":"句子数量",
    "text":"整篇文档内容"
}
```

