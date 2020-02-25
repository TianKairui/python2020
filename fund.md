## 使用python爬虫爬取天天基金网的单位净值数据
TianKairui   
2020/02/20


### 正文
#### 引入python库
首先需要引入BeautifulSoup,requests,time等python库。
为了读写改Excel文件，还需引入openpyxl库。

#### 爬取数据
F12键查看所需网页的审查元素。
天天基金的所有基金数据储存在一个div里，使用BeautifulSoup的find_all方法，
通过class获取div。
`exts = bf.find_all('div', class_='dataOfFund')`
再使用同样的方法，获取包含最终所需数据的两个span。
`span = dd[0].find_all('span')`
将两个span中的数据拼接成想要的格式。
`fundData = span[0].text + '(' + span[1].text + ')'`

#### 获取日期
获取系统日期。
`todayTime = time.strftime("%Y-%m-%d", time.localtime())`
这行代码还可以获取当前时间，但是这次我并不需要。
Excel把文件中的第一列写入了每天的日期，读取这一信息
`str(ws.cell(row, 1).value)[0:10]`
使用str()将数据转换为str格式，后边的切片是为了去掉日期后自动添加的时间。
将系统日期与Excel的日期注意比较，好确定将数据填入哪一行。
```
for row in range(2, 100):
	if str(ws.cell(row, 1).value)[0:10] == todayTime
```
注意，Excel的row属性及column属性从1开始。
因为我从第2行开始写入日期，因此此处的row值从2开始。

#### 写入Excel
定位到当前日期后，将数据写入Excel文件。
python的xlrd库仅支持读取Excel，而xlwt库仅支持写入Excel。
因为我们每天的数据都存储在同一个Excel中，不能每次运行程序都新建一个Excel文件。
因此需要使用openpyxl库来实现读写改。
```
wb = openpyxl.load_workbook('fund.xlsx')  # 载入excel文件
ws = wb[wb.sheetnames[0]]  # 读取第一个sheet存入ws
ws.cell(row, i + 2).value = fundData #写入数据
wb.save('fund.xlsx') #保存
```

### 代码
```
# -*- coding:UTF-8 -*-
from bs4 import BeautifulSoup
import requests
import sys
import openpyxl
import time


todayTime = time.strftime("%Y-%m-%d", time.localtime())

wb = openpyxl.load_workbook('fund.xlsx')  # 载入excel文件
ws = wb[wb.sheetnames[0]]  # 读取第一个sheet存入ws

for row in range(2, 100):
    if str(ws.cell(row, 1).value)[0:10] == todayTime:
        row = row - 1
        server = "http://fund.eastmoney.com/"  # 服务器地址
        target = ["http://fund.eastmoney.com/161631.html",
                  "http://fund.eastmoney.com/161723.html",
                  "http://fund.eastmoney.com/000294.html",
                  "http://fund.eastmoney.com/000294.html",
                  "http://fund.eastmoney.com/000961.html",
                  "http://fund.eastmoney.com/519674.html",
                  "http://fund.eastmoney.com/001071.html"]  # 目标网页地址
        for i in range(len(target)):
            req = requests.get(url=target[i])  # 获取目标网页
            html = req.text  # 将目标网页存到html中
            bf = BeautifulSoup(html)  # 将html转化为BeautifulSoup格式
            # 找到html中所有指定div，存到texts中。返回值为列表。
            texts = bf.find_all('div', class_='dataOfFund')
            dl = texts[0].find_all(
                'dl', class_='dataItem02')  # 找到texts中的指定dl
            dd = dl[0].find_all('dd', class_='dataNums')  # 找到dl中的指定dd
            span = dd[0].find_all('span')  # dd中只有两个span，分别存储昨日单位净值及昨日变化
            fundData = span[0].text + '(' + span[1].text + ')'
            ws.cell(row, i + 2).value = fundData

        break

wb.save('fund.xlsx')
```
