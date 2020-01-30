# Text analysis SnowNLP
利用SnowSNLP进行文本分析实验
## 1.将txt文件转换成csv格式
为了便于Python数据录入，我们需要将txt文件转换成csv格式。
录入数据的包为pandas，使用方式这里不详述。

具体代码为：

```py
coding = UTF-8
-*- coding: cp936 -*-
from snownlp import SnowNLP
import pandas as pd
import os
import re
import csv

os.chdir(os.path.join(os.getcwd(), 'acc_txt_5400_test1'))
file_path = os.listdir(os.getcwd())
file_list = []
for file in file_path:
    with open(file, 'r', encoding='utf-8') as f:
        stkcd = []
        year = []
        file_name = f.name.strip()
        content = f.read().strip()
        p = re.compile(r'^(\d{6}?)-')  
        # p = re.compile(r'[\u4e00-\u9fa5]{3}') 读取三个汉字
        stkcd = str(re.findall(p, file_name)[0])
        p = re.compile(r'-(\d{4}?)-')
        if re.search(r'-\d{4}-', file_name):
            year = str(re.findall(p, file_name)[0])
        else:
            year = str(re.findall(r'\d{4}', file_name)[0])


        file_list.append({'stkcd': stkcd, 'year': year, 'file_name': file_name, 'content': content})
        print(file_list)

headers = ['stkcd', 'year', 'file_name', 'content']

with open('acc_txt_5400_test1.csv', 'w', newline='') as f:
    f_csv = csv.DictWriter(f, headers)
    f_csv.writeheader()
    f_csv.writerows(file_list)
```

## 2. 对SnowNLP进行情感词汇训练
为了增加情感分析的准确性，我们加入自己搜集的程度、肯定、否定词汇表并进行训练

具体代码如下：
```py
# coding = UTF-8
# -*- coding: cp936 -*-
from snownlp import SnowNLP
import pandas as pd
import os
from snownlp import sentiment
import re
import jieba

os.chdir(os.path.join(os.getcwd(), 'traintext'))
file_path = os.listdir(os.getcwd())

sentiment.train('negative.txt', 'positive.txt')
sentiment.save('sentiment.marshal2')
```
## 3. 利用SnowNLP包进行文本分析

首先我们要用正则表达式对会议次数进行抓取：
```py
def conti(cont):
    a = 0
    p1 = re.compile(r'审计委员会.召开(.*)次会议')
    a1 = re.findall(p1, cont)

    if a1 != []:
        cn = a1[0]
        for i in cn:
            try:
                a = a + (getResultForDigit(i))

            except:
                print
                'URLError: <urlopen error timed out> All times is failed'

    elif a1 == []:
        p2 = re.compile(r'审计委员会(.{0,30})次会议')
        a2 = re.findall(p2, cont)
        if a2 != []:
            b = a2[len(a2) - 1][len(a2[len(a2) - 1]) - 1]
            try:
                a = a + (getResultForDigit(b))


            except:
                print
                'URLError: <urlopen error timed out> All times is failed'
    return a
```
其次，我们对抓取的会议次数进行汉语阿拉伯语转换：
```py
dictnum ={'零':0,'一':1,'二':2,'三':3,'四':4,'五':5,'六':6,'七':7,'八':8,'九':9,'十':10,'百':12,'千':13,'万':14,'亿':18,'两':2,
           '壹':1,'贰':2,'叁':3,'肆':4,'伍':5,'陆':6,'柒':7,'捌':8,'玖':9,'拾':10,'佰':12}
def getResultForDigit(a):
     count = len(a)-1
     result = 0
     tmp = 0

     while count >= 0:
         tmpChr = a[count:count+1]
         tmpNum = 0
         if tmpChr.isdigit():#防止大写数字中夹杂阿拉伯字母
             tmpNum=int(tmpChr)
         else:
             tmpNum = dictnum[tmpChr]
         if tmpNum >10:#获取0的个数
             tmp=tmpNum-10
         #如果是个位数
         else:
             if tmp == 0:
                 result+=tmpNum
             else:
                 result+=pow(10,tmp)*tmpNum
             tmp = tmp+1
         count = count - 1
     return result
```

阿拉伯语汉语的转换方式为：
```py
n = input("请输入阿拉伯数字：")
cn = "123456789"
cnum = ""
for i in n:
	cnum = cnum + cn[eval(i)]
print(cn[eval(i)])
print(cnum)
print("转换成汉字数字是：{}".format(cnum))
```

最后，我们对所有的文本进行文本分析：
```py
def senti(content):
   return SnowNLP(content).sentiments
```

调用函数的代码如下：
```py
os.chdir(os.path.join(os.getcwd(), 'acc_txt_5400_test1'))
file_path = os.listdir(os.getcwd())
data = pd.read_csv("acc_txt_5400_test1.csv", encoding='utf-8')
print(data)
data['time'] = data.content.apply(conti)
data['sentiment'] = data.content.apply(senti)
print(data)

data.to_csv('Result.csv')
```


