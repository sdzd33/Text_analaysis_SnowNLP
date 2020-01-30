# Text analysis SnowNLP
利用SnowSNLP进行文本分析实验
## 1.将txt文件转换成csv格式
为了便于Python数据录入，我们需要将txt文件转换成csv格式。
录入数据的包为pandas，使用方式这里不详述。

具体代码为：

```
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
```
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
## 3. 


