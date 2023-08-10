---
sort: 4
---


# SMP-2023-ChatGLM金融大模型挑战赛

* [算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/表格解析/SMP-2023-ChatGLM金融大模型挑战赛.html)

* [个人知乎](https://www.zhihu.com/people/zhangyj-n)

## 赛题类型
```
初级：数据基本查询（40分）

参赛者需要利用提供的ChatGLM2-6B开源模型和上市公司年报原始数据，并以此为基础创建信息问答系统。系统需能够解决基本查询，如：某公司2021年的研发费用是多少？等问题。

中级：数据统计分析查询（30分）

在初级阶段的基础上，参赛者需要进行金融数据的统计分析和关联指标查询。系统需基于各类指标，提供问题和答案，如：某公司2021年研发费用增长率为多少？等问题。

高级：开放性问题（30分）

如：某公司2021年主要研发项目是否涉及国家创新领域，如新能源技术、人工智能等？

```


## prompt设计

* [Prompt设计](https://kg-nlp.github.io/Algorithm-Project-Manual/大模型/Prompt设计.html)

* [基于ChatGLM2-6B 和 LangChain 的 Agents 实现：Prompt Template、Tools 和 Agents（下）](https://zhuanlan.zhihu.com/p/642703708)

* 常用问答设计
```bash
请根据上下文来回答问题，
如果上下文的信息无法回答问题，请回答"我不知道"。

上下文：
"""
{根据用户输入的问题，从向量数据库查询的结果。一行一个}
"""

问题："""{用户输入的问题}"""

回答：
```

```bash
请根据下面带```分隔符的文本来回答问题。
如果该文本中没有相关内容可以回答问题，请直接回复：“抱歉，该问题需要更多上下文信息。”
```{text}```
问题：{query}
```

* text2sql的prompt设计
```
chatbot_prompt = """
你是一个文本转SQL的生成器，你的主要目标是尽可能的协助用户将输入的文本转换为正确的SQL语句。
上下文开始
生成的表名和表字段均来自以下表：
"""python
# chatbot_prompt = """
# 你是一个文本转SQL的生成器，你的主要目标是尽可能的协助用户，将输入的文本转换为正确的SQL语句。
# 上下文开始
# 表名和表字段来自以下表：
# 表1: cargo
# 字段1:cargo_id(货物编号),字段2:cargo_name(货物名称),字段3:year(年份),字段4:net_yield(净收益率%),字段5:loss_rate(损失率%),字段6:month_on_month_growth_rate(环比增长率%),字段7:sales_volume(销售量),字段8:cargo_price(货物单价),字段9:cargo_category(货物品类),字段10:source_cargo(货物来源),字段11:storage_warehouse(货物仓库),字段12:sales_person_name(销售责任人名字),字段13:sales_person_id(销售负责人id),字段14:sales_department(销售部门),字段15:sales_person_numbers(销售负责人联系方式)
# 表2: sales
# 字段1:sales_person_id(销售负责人id),字段2:sales_person_name(人员名称),字段3:sales_person_level(人员等级),字段4:sales_person_work_date(入职时间),字段5:sales_person_leader_id(人员主管id),字段6:sales_person_leader_name(人员主管名字),字段7:sales_person_number(人员电话),字段8:sales_person_achievement_year(人员业绩年份),字段9:sales_person_achievement(人员业绩),字段10:sales_person_department(人员部门名称),字段11:sales_person_department_id(人员部门id)
# 表3: cargo_info
# 字段1:cargo_info_id(货物信息id),字段2:cargo_id(货物id),字段3:cargo_name(货物名称),字段4:origin_cargo(货物产地),字段5:cargo_purchase_price(货物购买价),字段6:cargo_type(货物大类),字段7:cargo_category(货物品类),字段8:cargo_supply_company(供货公司),字段9:cargo_num(货物数量),字段10:cargo_supply_aftermarket_person(货物售后负责人),字段11:cargo_supply_aftermarket_person_number(货物售后负责人联系电话),字段12:cargo_supply_market_person(货物公司销售负责人),字段13:cargo_supply_market_person_number(货物公司负责人联系电话)
# 表4: depart_list
# 字段1:department_id(部门id),字段2:department_name(部门名称),字段3:department_duty(部门职责),字段4:department_lead_name(部门负责人名字),字段5:department_lead_name_id(部门负责人id),字段6:department_person_nums(部门人数)
# 表5: purchase_info
# 字段1:purchase_company_id(货物购买方id),字段2:purchase_company_name(货物购买方名称),字段3:purchase_company_address(货物购买方地址),字段4:purchase_company_person_name(货物购买方负责人人名),字段5:purchase_company_person_numbers(货物购买方负责人联系方式),字段6:purchase_company_person_level(货物购买方负责人职位),字段7:purchase_cargo_name_id(购买货物id),字段8:purchase_cargo_name(购买货物名称),字段9:purchase_cargo_nums(购买货物数量),字段10:purchase_cargo_type(购买货物大类),字段11:purchase_cargo_category(购买货物品类)
# 表6: supply_company
# 字段1:supply_company_id(供货公司id),字段2:supply_company_name(供货公司名称),字段3:supply_company_address(供货公司地址),字段4:supply_company_product_id(供货公司货物id),字段5:supply_company_product_name(供货公司货物名称),字段6:supply_company_date(供货公司入名录时间)
# 上下文结束
# 问: 请帮我查询所有的货物名称
# 答: SELECT cargo_name FROM cargo
# 问: 请帮我查询在2019年的净收益率大于10并且销售量大于100并且业绩大于1000的销售负责人名字
# 答: SELECT sales.sales_person_name FROM sales INNER JOIN cargo on sales.sales_person_id = cargo.sales_person_id WHERE cargo.year = 2019 AND cargo.net_yield > 10 AND cargo.sales_volume > 100 AND sales.sales_person_achievement > 1000
# 问: 文本转SQL: <user input>
# 答: 
# """

# 一些学习的例子
In_context_prompt = """问: 请帮我查询所有的货物名称
答: SELECT cargo_name FROM cargo;
问: 请帮我查询在2019年的净收益率大于10并且销售量大于100并且业绩大于1000的销售负责人名字
答: SELECT sales.sales_person_name FROM sales INNER JOIN cargo on sales.sales_person_id = cargo.sales_person_id WHERE cargo.year = 2019 AND cargo.net_yield > 10 AND cargo.sales_volume > 100 AND sales.sales_person_achievement > 1000;
问: 请帮我查询购买"板材"货物的公司名称
答: SELECT purchase_company_name FROM purchase_info WHERE purchase_cargo_name = "板材";
"""
query_template = """问: <user_input>
答: 
"""
```

## 工程化

* 采取常规检索+LLM汇总方法
    * 常规检索问答是否加入模型意图识别,模型实体识别(现阶段不作)
* 整个流程分为以下几个模块:
    * 数据入库处理模块
        * 非结构化文本拼接,拆分
        * 关键词库
    * 关键词检索模块
        * 搭建ES库,构造非结构化文本索引和结构化表格索引
        * 替换分词库,如有同义词库则增加
    * 排序模块
        * 使用bge或m3e计算语义相似度召回
        * 上述效果不好使用比赛数据训练自监督语义模型,重新进行向量召回
        * 上述方法作为粗排召回,可以增加开源交互模型作为精排测试效果(不单独训练精排模型)
        * 向量召回使用milvus引擎
    * LLM模块
        * 使用上述关键词检索和排序召回的结果,TOPN的设置以tokens为限制,原则上ChatGLM2-6b可以最长32k,但采取召回数据越多,文本越长,经LLM推理,总结后结果越差,目前暂定使用TOP5数据,文本长度<1024
        * 提示词工程,针对ChatGLM需要不断试错
    
* 预计周日搭建完系统