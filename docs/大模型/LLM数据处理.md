---
sort: 11
---


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

数据来源
数据组成(哪些字段,哪种类型)
数据解析(提取txt)
数据清洗(清除杂质,文本拼接)
数据过滤(去重,无效数据)
数据质量(完整语句,清晰语义)


* 第一阶段提取数据

| 措施  | 方法 | 完成情况 | 备注 | 
|--- | --- | --- |---|
| 迁移目录下所有文件| 自定义脚本 | √ | |
| 文本格式转换| 自定义脚本 | √| |
| pdf批量转docx| 使用WPS    | √ | |
| 转换前后文件对比查漏  | 自定义脚本| √| |
| 过滤：文件名存在副本,文件大小一致判断 | 自定义脚本| √| |
| 压缩包解压 | 手动解压,自定义脚本 | √ | |
| 文档只保留正文,过滤导航栏文本,标题,脚注,页眉页脚| 自定义脚本| √x | |
| 去除 html 标签、去除异常字符、去除冗余字符、去除 URL、去除 E-mail、去除电话号码，将全角字母数字空格替换为半角 | pip工具+自定义脚本 | √ | |
| 过滤非中文占比高的句子 |自定义脚本|√| |
| 整理无效禁用词，标点符号过多的行  | 自定义脚本 |√x| |
| 过滤：去重| 自定义脚本，分阶段过滤：完全匹配，近似匹配，hash去重 | √(需要增加规则) | |
| 标点符号切割句子,保证句子完整性|自定义脚本|√| |
| 繁体转简体 |Pip工具+自定义脚本|√| |
| 过滤构建重组句子后使用文本流畅度检验|自定义脚本|√x| |
| 去除章节号 |自定义脚本|√| |
| 多层过滤:1 只保留有章节序列号,去除表格和图片 |自定义脚本|√| |
| 全角转半角 |自定义脚本|√| |
|人名过滤|自定义脚本|√|
|判断句尾符号拼接文本|自定义脚本|√|
|文本起始位置格式化|自定义脚本|√|
|去除连续空格占比高的文本|自定义脚本|√|
|只提取中文去重|自定义脚本|√|针对业务数据集中含有规格型号|
|近似匹配循环去重|自定义脚本|√|相似数据保留最长|
|公共子串过滤|自定义脚本|√||
|所有数据长度小于10的过滤掉|自定义脚本|√|



* 第二阶段 提取指令集
       
    * 根据标题序号制作指令集

## 数据源文件整理

> 只处理领域数据

数据目录: D:\项目-预训练模型\数据\业务数据

| 目录   | 格式           | 目录大小  |
| ---- | ------------ | ----- |
| 工艺手册 | 多为pdf和doc    | 225M  |
| 施工方案 | doc,txt          | 472M  |
| 规范搜索 | txt,doc,pdf,docx  | 9.41G  |
| 论文数据 | docx         | 1.4G  |
| 合同数据 | pdf,doc,xls等 | 9.44G |
| 业务数据 | jsonl等|53.5M|

## 数据处理结果

> 结果文件用于LLM/PTM, 用于业务标注数据(弥补原标注数据不足的问题)

### 格式

*   每篇token数/每篇样本数

```
{   
    doc_id: id,
    doc_name: 文档名,
    doc_hash": 文档hash值
    doc_tokens: 总token数,
    doc_sentences: 句子数量,
    text: 整篇文档内容"
}

```

## 任务阶段

### 20230714 第一轮数据处理结果

* 数据源 
    *  工艺手册35本
* 数据预处理

> 脚本位置:  /home/Algorithm_Frame/LLM/process/core/CDE工艺数据处理.py

| 数据 | 句子数量 | tokens数 | 备注 |
| --- | --- | --- | --- |
| 原数据 | 206396 | 8201759 | 转成excel格式 docx2excel |
| filter | 135139 | 4768389 | 应用过滤规则 get_filter | 
| mid    | 120919 | 4286773 | 合并去重分组操作 get_mid|
| final  | 57708| 4282668 | 文本拼接 get_final |


final 是根据(末尾特定符号) 判断是否添加换行符,是否合并
* 模型数据格式处理
    * all_doc_ids [ 1390 41644 40183 ...    42  4315 12043] (4197617,)
    * lens [47 73 84 ... 47 56 39] (82609,)
    * docs [    0    42    79 ... 82517 82550 82609] (2218,)
    * Total sentences num: 82609   # 按照换行,句号,问号,叹号
    * Total documents num: 2217
    * Total tokens num: 4197617
    * Average tokens per sentence: 50.81
    * Average tokens per document: 1893.38

* 结果
[垂直领域小模型快速训练（一）](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/垂直领域小模型快速训练（一）.html)

 <br>
 
### 20230727 第二轮数据处理结果

* 本次增加施工方案数据,成本数据
    * 增加规则处理方法
    * 增加分类模型,通顺度检测模型(未增加)
* 使用预训练框架: paddle和tencent
* 场景任务验证
    * 分类任务:CDE(已验证)
    * 信息抽取任务:合同(未验证)

 
> 脚本位置:  /home/Algorithm_Frame/LLM/process/core/CDE业务数据处理.py

* 成本数据(数字,字母等符号很多)

| 数据 | 句子数量 | tokens数 | 备注 |
| --- | --- | --- | --- |
|原数据| 189248|11580038| 拉取数据|
|filter| 20072|1518167|完全匹配近似匹配去重 get_filter |
|final | 17428|1453167|公共子串过滤 get_final |
|jsonl | 17428|1453167|格式转换 get_jsonl |
 

* 成本+工艺手册
    * all_doc_ids [ 2659 40894  1320 ...    30  2817  9517] (5549871,)
    * lens [ 35  66  73 ... 122  25  32] (101057,)
    * docs [     0      1      2 ... 101055 101056 101057] (19646,)
    * Total sentences num: 101057
    * Total documents num: 19645
    * Total tokens num: 5549871
    * Average tokens per sentence: 54.92
    * Average tokens per document: 282.51 

* 结果
[垂直领域小模型快速训练（二）](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/垂直领域小模型快速训练（二）.html)

```bash
python -u -m paddle.distributed.launch \
    --gpus "0,1,2,3" \
    --log_dir "output/${MODEL_NAME}/logs" \
    run_pretrain.py \
    --model_name_or_path ${MODEL_NAME} \
    --tokenizer_name_or_path ${MODEL_NAME} \
    --input_dir  /home/Algorithm_Frame/LLM/ernie-1.0/data_file/output_data \
    --output_dir output/${MODEL_NAME} \
    --split 198,1,1 \
    --binary_head False \
    --max_seq_len 256 \
    --micro_batch_size 128 \
    --max_steps 2000 \
    --checkpoint_steps 400 \
    --save_steps 200 \
    --logging_freq 200 \
    --eval_freq 200
```

* 施工方案

> 脚本位置:  /home/Algorithm_Frame/LLM/process/core/CDE施工数据处理.py


| 数据 | 句子数量 | tokens数 | 备注 |
| --- | --- | --- | --- |
|原数据| 原文件4636|去重后文件3670|文件去重 compare_file|
|原数据| 5231733|141801639|转成excel格式 get_excel|
|filter| 2686421|85665489|应用过滤规则 get_filter |
|mid | 1547546|58761510|合并去重分组操作 get_mid |
|final | 783714|58041789|文本拼接 get_final |


* 施工方案+成本+工艺手册
    * Total sentences num: 1197619
    * Total documents num: 51171
    * Total tokens num: 62166983
    * Average tokens per sentence: 51.91
    * Average tokens per document: 1214.89

* 数据大小

|类型|大小|句子数量(累计)|tokens(累计)|
|---|---|---|---|
|工艺手册|11.7M|82609|4197617|
|成本清单|5.06M|101057|5549871|
|施工数据|159M|1197619|62166983|
    
```bash
python -u -m paddle.distributed.launch \
    --gpus "0,1,2,3" \
    --log_dir "output/${MODEL_NAME}/logs" \
    run_pretrain.py \
    --model_name_or_path ${MODEL_NAME} \
    --tokenizer_name_or_path ${MODEL_NAME} \
    --input_dir  /home/Algorithm_Frame/LLM/ernie-1.0/data_file/output_data \
    --output_dir output/${MODEL_NAME} \
    --split 198,1,1 \
    --binary_head False \
    --max_seq_len 256 \
    --micro_batch_size 256 \
    --max_steps 11700 \
    --checkpoint_steps 2000 \
    --save_steps 1200 \
    --logging_freq 1200 \
    --eval_freq 1200
```

### 20230806 第三轮数据处理结果

* 本次增加论文数据,规范数据
    * 增加规则处理方法
    * 增加分类模型,通顺度检测模型(有时间扩充数据)
* 使用预训练框架: paddle
* 场景任务验证
    * 分类任务:CDE

 
> 脚本位置:  /home/Algorithm_Frame/LLM/process/core/CDE论文数据处理.py  
> 脚本位置:  /home/Algorithm_Frame/LLM/process/core/CDE规范数据处理.py
  
  
* 论文数据

| 数据 | 句子数量 | tokens数 | 备注 |
| --- | --- | --- | --- |
|原数据| 418555|28387238|转成excel格式 get_excel|
|filter| 221548|9794638|应用过滤规则 get_filter |
|mid | 193271|9477326|合并去重分组操作 get_mid |
|final | 57110|9400626|文本拼接 get_final |
 
* 规范数据

| 数据 | 句子数量 | tokens数 | 备注 |
| --- | --- | --- | --- |
|原数据| 882242|34736539|转成excel格式 get_excel|
|filter| 550466|19291905|应用过滤规则 get_filter |
|mid | 426012|15920029|合并去重分组操作 get_mid |
|final | 206571|15864233|文本拼接 get_final |
 
 

* 成本+工艺手册+施工方案+论文数据+规范数据
    * all_doc_ids [12052   397 12053 ...   488 40413 12043] (85769452,)
    * lens [ 31  35  35 ... 337  88  63] (85769452,)
    * docs [      0      43      73 ... 1618600 1618603 1618639] (66138,)
    * Total sentences num: 1608787
    * Total documents num: 1608787
    * Total tokens num: 85769452
    * Average tokens per sentence: 53.31
    * Average tokens per document: 53.31


* 数据大小

|类型|jsonl文件大小|句子数量(累计)|tokens(累计)|
|---|---|---|---|
|工艺手册|11.7M|82609|4197617|
|成本清单|5.06M|101057|5549871|
|施工数据|159M|1197619|62166983|
|论文+规范|26.4+43.5=69.9M|1618639|86766539|

```bash
python -u -m paddle.distributed.launch \
    --gpus "0,1,2,3" \
    --log_dir "output/20230806第三轮测试/${MODEL_NAME}/logs" \
    run_pretrain.py \
    --model_name_or_path ${MODEL_NAME} \
    --tokenizer_name_or_path ${MODEL_NAME} \
    --input_dir  /home/Algorithm_Frame/LLM/ernie-1.0/data_file/output_data \
    --output_dir output/20230806第三轮测试/${MODEL_NAME} \
    --split 198,1,1 \
    --binary_head False \
    --max_seq_len 256 \
    --micro_batch_size 256 \
    --max_steps 8000 \
    --checkpoint_steps 800 \
    --save_steps 800 \
    --logging_freq 800 \
    --eval_freq 800
```

 <br>

**以下循环操作**
* 增加数据
*  数据处理
*  基础模型选择
*  超参选择
*  模型对比验证



## 参考

* [Falcon Paper 我们是靠洗数据洗败 LLaMA 的！](https://zhuanlan.zhihu.com/p/637996787)

