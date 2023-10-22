---
sort: 8
---

# Pandas

[🔨算法开发手册](https://kg-nlp.github.io/Algorithm-Project-Manual/工程内容/Pandas.html)    

[🔨个人知乎](https://www.zhihu.com/people/zhangyj-n)



## 简易教程

https://note.youdao.com/s/bRNHYJot

## 常用功能

- **某列的空值由另一列填充**

```python
data['分部分项校核'].fillna(data['分部分项工程'],inplace=True)
```

- **根据列值筛选某行数据**

```python
single_data = data.loc[data['分部分项校核'].isin(single_label)].copy()
```

- **对 列 中每种不同的值 进行计数** 

```python
df['label'].value_counts()
验证集标签分布 
给排水工程      779
主体工程       513
幕墙工程       304
```

- **统计有多少种不同的值** 

```python
df['label'].nunique()
训练集标签分布 
 27
```

- **统计有多少种不同的数据**

```python
df['label'].unique()
pd.Series([2, 1, 3, 3], name='A').unique()
array([2, 1, 3]
```

- **合并两个df**

```python
pd.concat([a,b],axis=0)

通过merge合并 如果两个表中公共字段名不同
pd.merge(result,df_anntotation,how='left',left_on='match_整句id',right_on='整句id')
```



- **处理某列数据生成新列**

```python
df_material['材料名称'] = df_material['信息json'].apply(lambda x:eval(x)['material_name'])
```



- **根据拼音排序**

```python
from pypinyin import lazy_pinyin
df['a'] = df['分部分项工程'].apply(lambda x: lazy_pinyin(x)[0][0])
df['b'] = df['分部分项工程'].apply(lambda x: lazy_pinyin(x)[1][0])
df = df.sort_values(by=['a', 'b'], ascending=True)
df.drop(['a','b'],axis=1, inplace=True)
```



- **合并两列**

```python
df_schedule['合并名称'] = df_schedule[['父级名称','任务名称']].apply('/'.join,axis=1) 
df_schedule['合并名称'] = df_schedule[['父级名称','任务名称']].apply(lambda x:x[0]+'/'+x[1],axis=1)
```



- **删除重复值**

```python
newdata.drop_duplicates(subset=['A','B'],keep='first')   
```



- **某列长度统计（适合数量少的文本）**

```python
test['contentLen2'] = test['content'].str.len()
df_train_14['length'] = df_train_14.apply(lambda x:len(str(x[1]))+len(str(x[2])),axis=1)

df_train_14['length'] = df_train_14.apply(lambda x:len(str(x['任务名称']))+len(str(x['前缀名称'])),axis=1)

对于Dataframe格式,指定axis,axis=1 遍历行,默认axis=0,遍历列
对于Series格式,不需要指定,可以认为就一列,遍历行
```



- **group by分析**

https://learnku.com/articles/39735

- **series 转 dict**

```python
s = pd.Series([1, 2, 3, 4])
>>> s.to_dict()
{0: 1, 1: 2, 2: 3, 3: 4}
>>> from collections import OrderedDict, defaultdict
>>> s.to_dict(OrderedDict)
OrderedDict([(0, 1), (1, 2), (2, 3), (3, 4)])
>>> dd = defaultdict(list)
>>> s.to_dict(dd)
defaultdict(<class 'list'>, {0: 1, 1: 2, 2: 3, 3: 4})
```



- **DataFrame转dict**

https://blog.csdn.net/hanyunkaka/article/details/120603027

```python
dict1 = dict(zip(data['key'],data['value']))

sales = [{"Fruits":"apple","Numbers":5},
         {"Fruits":"banana","Numbers":8},
         {"Fruits":"pear","Numbers":9}]
df = pd.DataFrame(sales)
df.to_dict(orient='records')  结果是上面的格式

sales = {"Fruits":["apple","banana","pear"],
         "Numbers":[5,8,9]}
df = pd.DataFrame.from_dict(sales)
df.to_dict(orient='list')  结果是上面的格式
```



- **数据筛选**

Dataframe:https://www.modb.pro/db/107549

Series:https://cloud.tencent.com/developer/article/1627715

- **判断一个表中 两列是否相同**

```python
df['result'] = np.where(df['col1']==df['col2'],'same','different')
single_data = df.loc[df['result']=='same'].copy()
或者 
df_all_match = df_all_match.loc[df_all_match['标签']!=df_all_match['match_标签']].copy()
```



- **修改某一列的值**

```python
df[column] = value
```



- **打乱顺序**

```python
df.sample(frac=1)
```



- **类别统计图**

```python
train_df['label'].value_counts().plot(kind='bar')
plt.title('News class count')
plt.xlabel("category")
```



- **句子长度统计图**

```python
_ = plt.hist(train_df['text_len'], bins=200)
plt.xlabel('Text char count')
plt.title("Histogram of char count")
```



- **绘制频率图**

https://cloud.tencent.com/developer/article/1587884

- **修改列名**

```python
1、修改列名a，b为A、B。
df.columns = ['A','B']
2、只修改特定列
df.rename(columns={'a':'A','b':'B'},inplace=True)
```



- **交换两列**

```python
order = ['date', 'time', 'open', 'high', 'low']  
df = df[order]
```



- **apply中使用tqdm进度条**

```python
from tqdm import tqdm
import pandas as pd
tqdm.pandas(desc='pandas bar')
df['title_content'] = df.progress_apply(lambda x: _title_content(x['title'], x['content']), axi
```



- **筛选某列是空的行**

```python
df_total_1 = df_total_import[df_total_import['前缀名称'].isnull()]
```



- **修改类型**

```python
>>>df=df.astype({'scores':'int32','rank':'float32'})
>>>df.dtypes
#利用字典使得任意列为任意类型
name       object
scores      int32
level      object
rank      float32
dtype: object
```



- **按行扩充表**

```python
df_data_0 = pd.DataFrame(np.repeat(df_data_1.values,10,axis=0))
df_data_0.columns = df_other.columns
```



- **df.to_parquet**

```python
df.to_parquet('必须是英文.parquet',index=False)
```



- **根据某字段排序**

```python
print(frame.sort_values(by='a'))
```



- **删除指定列**

```python
print(df.drop('state', axis=1))
print(df.drop(['state', 'point'], axis=1))
```



- **取对应两个字段的所有行数据**

```python
# 假设有一个名为 df 的 DataFrame，包含 'column1' 和 'column2' 两列

# 创建一个布尔条件，判断同时满足两个字段的条件
condition = (df['column1'] == value1) & (df['column2'] == value2)

# 通过布尔条件筛选出对应的行数据
result = df[condition]
```

