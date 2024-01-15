# 公司名称匹配
【上市公司名称】与【本公司自有数据库内的公司名称】匹配
# 背景
公司自建了一个客户数据库，我们需要给其中A股上市的公司打上tag。但是数据库建的时候，客户名称与A股list上的标准名称会有些许差异（比如：数据库名称为xx有限公司，标准名称为：xx股份有限公司），需要手工搜索关键字匹配，比较繁琐，所以利用python的fuzz和jieba两个库进行匹配。
# 步骤
- 通过jieba分词找到自有数据库的词频，比如“公司”，“有限”，“股份”，“集团”等，这些不属于关键字，去掉冗余单字才能indentify公司名称，也提高了匹配成功率。
- 通过fuzz的算法得出两份数据的文字匹配度
- 把匹配度从高到低排列，超过50%匹配度的都可以认为是匹配成功，最后需要二次审核避免算法失误。
```
import jieba
import pandas as pd
tokenizer = jieba.dt
data=pd.read_excel(r"C:Desktop\单词相似度\company name.xlsx",sheet_name='Sheet7')
m=[]
for x in set(data['股票简称']):
    for y in tokenizer.cut(x):
        m.append(y)
 
print(pd.Series(m).value_counts())
```
得出结果：
股份    315  

科技    260  

药业     51  

集团     47  

医疗     46
```
import pandas as pd
from fuzzywuzzy import fuzz
 
data1=pd.read_excel(r"C:\Desktop\单词相似度\company name.xlsx",sheet_name='Sheet7')
data2=pd.read_excel(r"C:\单词相似度\sfdc acct.xlsx",sheet_name='Sheet1')
def sinple(x):
    temp=x
    for stopwords in ['股份','科技','药业','集团','医疗']: # 可以多加几个stopwords
        temp=temp.replace(stopwords,'')
    
    return temp
# print(sinple('广州越秀融资租赁有限公司'))
 
#给定一个待处理公司名，在标准公司名列表中找最相近的字符串
def findWord(word,list):
    maxRatio=0
    target=''
    for x in list:
        #先计算简化版的相似度，再加上未简化的相似度作为小数位
        ratio=fuzz.ratio(sinple(word),sinple(x))+fuzz.ratio(word,x)/100
        if ratio>maxRatio:
            maxRatio=ratio
            target=x
    return word,target,maxRatio


def findlist(targetList,fromList):
    data=pd.DataFrame(columns=('target', 'result', 'ratio'))
    index=0
    for x in set(targetList):
        row=findWord(x,fromList)
        index+=1
        data.loc[index]={'target':row[0],'result':row[1],'ratio':row[2]}
    return data
 
data=findlist(data1['股票简称'],data2['SFDC'])
data.to_csv(r'C:\Desktop\单词相似度\单词相似度.csv')#保存一个结果文件
```
