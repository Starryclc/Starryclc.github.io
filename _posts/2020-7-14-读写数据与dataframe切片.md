---
layout: post
title: 读写数据与dataframe
categories: [数据处理]
---

---

常常使用pandas处理数据转化成DataFrame类型，但时间一久总是忘记某些操作QAQ，今天整理一下。基本上根据 *Wes McKinney*的《利用python进行数据分析》。

---

## 1 pandas读取数据

### 常用函数如下

| Function       | Description                                                  |
| -------------- | ------------------------------------------------------------ |
| read_csv       | 从文件,URL,文件型对象中加载带分隔符的数据.默认分隔符为','    |
| read_table     | 从文件,URL,文件型对象中加载带分隔符的数据.默认分隔符为'/t'   |
| read_fwf       | 读取定宽列格式数据(即没有分隔符)                             |
| read_clipboard | 读取剪贴板中的数据.可以看作是read_table的剪贴板版,在将网页转换为表格时很有用 |

### read_csv()

> 究极常用,这里重点记录下它的用法,其他函数的参数用法也大致相同.该函数参数太多，记录几个常用的，其他的参见官网指南( https://pandas.pydata.org/pandas-docs/stable/reference/api/pandas.read_csv.html )

`header和names:` 用来设置列名。header的作用是说明原文档有没有列明，如果没有则设置为None，如果不设置则默认首行为列名。

```python
# 不设置header变量，自动把第一行作为列属性
pd.read_csv('./example.csv')
# 将header设置为None，含义为原文档没有列名，这时设置默认列名为0，1，2....
pd.read_csv('./example.csv',header=None) 
# 自定义
pd.read_csv('./example.csv',header=None,names=['x1','x2','x3','y'])
or
data = pd.read_csv('./example.csv',header=None)
data.columns = ['x1','x2','x3','y']
#（注意这个地方的header需要根据实际情况是否有列名来设置，设置错误会导致第一条数据被吞）
```

`index_col： `  设置索引列

```python
#设置y列为索引
pd.read_csv('./example.csv',header=None,names=['x1','x2','x3','y']，index_col='y')
#将多个列设置成一个层次化索引
pd.read_csv('./example.csv',header=None,names=['x1','x2','x3','y']，index_col= ['x1','x2'])
#（这时会首先按照x1索引，然后再按照x2索引）
```

`skiprows:` 跳过读取某些行

```python
#跳过读取1，3，5行
data = pd.read_csv('./example.csv',skiprows=[0,2,4])
```

`na_values:` 缺失值处理

> pands会用一组经常出现的值识别缺失值,如NA,-1,NULL等.但我们可以用na_values接受一组用于表示缺失值的字符串,来帮助判断文档中的缺失值.

```python 
#全局设置
data = pd.read_csv('./example.csv',na_values=['NA','foo'])
#用字典为各列制定不同的标记值
data = pd.read_csv('./example.csv',na_values={'y':['NA','foo'],'x1':['null']})
```

`nrows:` 读取文件前几行

`chunksize:` 控制逐块读取文件

## 2 导出数据

* DataFrame的to_csv方法,将数据写入到逗号分隔的文件中

```python
data = pd.read_csv('./example.csv')
data.to_csv('./out.csv')
```

`seq:` 设置分隔符

`na_rep:` 设置缺失值标记

`header和index:` 设置行列标签,False表示禁用

`cols:` 设置列,可仅设置一部分

## 3 获取DataFrame元素

```python
data = pd.DataFrame=([[1,2,3,1],[2,1,4,1],[1,1,1,0]],columns = ['x1','x2','x3','y'])
>>
   x1  x2 x3 y
0  1  2  3  1
1  2  1  4  1
2  1  1  1  0
```

### loc() && iloc()

`df.loc:` 通过索引标签名获取数据

`df.iloc:` 通过位置获取数据

```python
# 取一行 (返回Series)
data.loc[0]
# 取多行 (返回DataFrame)
data.loc[[0,1]]
# 连续取多行 (返回DataFrame)
data.loc[0:2]

# 取单行单列 (返回Str)
data.loc[0,'x1']
# 取单行多列  (返回Series)
data.loc[0,['x1','y']]
...类推
```

### at() && iat()

* at: 根据行索引和列名获取元素值

```python
#  (返回Str)
data.at[0,'x1']
or
data.iloc[0].at['x1']
```

* iat : 根据行索引和列索引获取元素值

```python
#  (返回Str)
data.iat[0,0]
or
data.iloc[0].iat[0]
```

## 4 缺失值处理

`DataFrame.dropna(axis=0, how='any', thresh=None, subset=None, inplace=False)` 

| 参数    | 含义                                                       |
| ------- | ---------------------------------------------------------- |
| axis    | axis = 0 删除行 ；axis = 1 删除列                          |
| how     | how = 'all' 全部为NAN时才删除 ； how = ‘any' 含有NAN即删除 |
| thresh  | axis中至少有thresh个非缺失值，否则删除                     |
| subset  | 在哪些列中查看是否有缺失值                                 |
| inplace | 是否在原数据上操作.True则返回None，否则返回新的copy        |

> 关于`Inplace`的提示：
>
> 有一次直接使用df.dropna()后再查看数据集，发现NAN行还存在，数据维度没有发生任何改变，排查后发现原来是忽略了inplace。
>
> **inplace默认为None，此时dropna()默认返回没有NA的数据集，需要分配一个新的变量去接收它。**
>
> **如果希望dropna()修改原数据，需要明确指定inplace=True.**