title: beautifulSoup爬取同步加载页面数据
author: Elijah Zheng
tags:
  - beautifulSoup
  - python
categories:
  - 爬虫
date: 2019-11-15 11:00:00
---
beautifulSoup 可以很方便的获取 html 页面的标签节点，而且也很容易获取到标签的属性和文本内容。我们就运用 beautifulSoup 来爬取同步加载页面数据，然后我们将获取到的数据存入到 excel 中。
<!--more-->

## 爬取同步加载页面数据

我们这里爬取豆瓣电影Top 250 的数据做描述，``request``库请求页面，``beautifulSoup``用来获取页面节点，所使用到的库自行安装。

### 获取页面源码
```
from urllib import request
from chardet import detect
from bs4 import BeautifulSoup


def get_soup(page_url):
    """获取源码"""
    with request.urlopen(page_url) as fp:
        byt = fp.read()
        det = detect(byt)
        return BeautifulSoup(byt.decode(det['encoding']), 'lxml')
```
``page_url``为页面地址，``lxml``为``python``库自行安装。

### 获取页面数据
我们运用``beautifulSoup``来获取页面的数据，数据一般存放在页面的节点内容中（如：标题、价格、数量）和节点的属性中（图片链接），所以我们需要找到这些存放数据的相应节点。

![douban电影top250数据节点](https://cdn.zhengxiangling.com/superbed/2019/11/15/5dce0edf8e0e2e3ee92f82d9.jpg)

我们需要从中找到一些规律，每部电影的数据都存放在一个``li``中，所有的``li``都存放在一个``ol``中，图片的链接在``li``的``img``标签的``src``属性中，其他的数据都放在带有``class``属性的``span``标签中。
```
# 正则式
import re

def get_data(page):
    """
    获取数据
    selelct 可以用css语法获取标签，find可以获取标签里的 attrs 属性值
    """
    data = []
    ol = page.find('ol', attrs={'class': 'grid_view'})
    for li in ol.select('li'):
        # 单元
        tmp = []
        # 多个 title
        titles = []
        img_url = li.find('img').attrs['src'].strip()
        tmp.append(img_url)
        for span in li.findAll('span', attrs={"class": re.compile('')}):
            if span.attrs['class'][0] == 'title':
                titles.append(span.string.strip())
            # 评价
            if span.attrs['class'][0] == 'rating_num':
                tmp.append(span.string.strip())
            # 简评
            if span.attrs['class'][0] == 'inq':
                tmp.append(span.string.strip())
        tmp.insert(0, titles)
        data.append(tmp)
    return data
```
### 获取下一页的数据
我们先需要在页面中找到下一页数据请求的参数
![下一页按钮](https://cdn.zhengxiangling.com/superbed/2019/11/15/5dce1b588e0e2e3ee931cd7c.jpg)
我们可以看到下一页的参数为``?start=25&filter=``，所以获取到这个参数然后加入到之前的网址当中，获取到的数据就是下一页的数据了。
```
def next_url(page):
    """获取下一页链接后缀"""
    a = page.find('a', text=re.compile("^后页"))
    if a:
        return a.attrs['href']
    else:
        return None
```

### 保存数据到 excel 中
```
import xlwt

def xls_save(workbook, data, count):
    """保存数据到excel"""
    for d in data:
        for i in range(len(d)):
            # print(d[i])
            workbook.write(count, i, d[i])
        count = count + 1
    return workbook, count
```
``xlwt``为``python``库自行安装，``workbook``为要保存的``excel``对象，``data``为``beautifulSoup``对象，``count``为要写入数据的行数。

### 完整代码
```
# 爬取豆瓣电影 top250

from urllib import request
from chardet import detect
from bs4 import BeautifulSoup
import re
import xlwt
import os


def get_soup(page_url):
    """获取源码"""
    with request.urlopen(page_url) as fp:
        byt = fp.read()
        det = detect(byt)
        return BeautifulSoup(byt.decode(det['encoding']), 'lxml')


def get_data(page):
    """
    获取数据
    selelct 可以用css语法获取标签，find可以获取标签里的 attrs 属性值
    """
    data = []
    ol = page.find('ol', attrs={'class': 'grid_view'})
    for li in ol.select('li'):
        # 单元
        tmp = []
        # 多个 title
        titles = []
        img_url = li.find('img').attrs['src'].strip()
        tmp.append(img_url)
        for span in li.findAll('span', attrs={"class": re.compile('')}):
            if span.attrs['class'][0] == 'title':
                titles.append(span.string.strip())
            if span.attrs['class'][0] == 'rating_num':
                tmp.append(span.string.strip())
            if span.attrs['class'][0] == 'inq':
                tmp.append(span.string.strip())
        tmp.insert(0, titles)
        data.append(tmp)
    return data


def next_url(page):
    """获取下一页链接后缀"""
    a = page.find('a', text=re.compile("^后页"))
    if a:
        return a.attrs['href']
    else:
        return None


def xls_save(workbook, data, count):
    """保存数据到excel"""
    for d in data:
        for i in range(len(d)):
            # print(d[i])
            workbook.write(count, i, d[i])
        count = count + 1
    return workbook, count


if __name__ == '__main__':
    url = 'https://movie.douban.com/top250'
    soup = get_soup(url)
    # print(get_data(soup))
    path = os.path.join(os.getcwd() + "\\" + "douban-top250.xls")
    xls_file = os.path.exists(path)
    if xls_file:
        os.remove(path)
    wb = xlwt.Workbook(encoding='utf-8')
    xls = wb.add_sheet('top250')
    head = ['标题', '图片地址', '评分', '简评']  # 表头
    row = 1
    for h in range(len(head)):
        xls.write(0, h, head[h])
    xls, row = xls_save(xls, get_data(soup), 1)
    nxt = next_url(soup)
    while nxt:
        soup = get_soup(url + nxt)
        xls, row = xls_save(xls, get_data(soup), row)
        nxt = next_url(soup)
    wb.save('douban-top250.xls')
```