---
title: SCRAPY 简单入门
date: 2016-12-19 18:37:56
tags: [Hello world,Python]
toc: true
description: SCRAPY是由Python开发的，为爬取网站数据，提取结构性数据而编写的应用框架。可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。
---
>SCRAPY是由Python开发的，为爬取网站数据，提取结构性数据而编写的应用框架。可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。

下面通过一个小例子，爬取RS05网站的电影链接，熟悉该框架的基本功能。
闲话少叙，下面开始:)

### 安装SCRAPY
相信有一半的同学，学习SCRAPY从安装到放弃。
我也是折腾了好久，从baidu到google，再到stackoverflow，才完完整整的安装到了电脑上。高兴的我，又在树莓派Raspbian系统上安装了一遍。为了节省大家的时间，这里把Linux和Windows的安装方法都列举出来，方便以后查询。
Python版本为2.7

#### Windows平台：
首先到[这里](https://www.microsoft.com/en-us/download/details.aspx?id=44266)下载VC For Python27
然后按照下列顺序安装前置包
* zope.interface
* pyopenssl
* twisted
* lxml
* scrapy

不能使用pip下载的，直接google安装包好了。

#### Linux平台：
准确的说应该是Raspbian平台，因为我用的树莓派是XD。相比win平台就简单多了，
```bash
apt-get install libxml2-dev libxslt1-dev python-dev
apt-get install python-lxml
if error:
pip install pyasn1 --upgrade
```
好的，大功告成！enjoy！

### 选择网站
我真是太懒了，懒到不想点开浏览器去找电影下载。为了节省十几秒找电影的时间，花去了几个小时研究爬虫框架自动爬取链接。想想还是挺值的！
这里我选择RS05为目标网站，提取每个电影页面的迅雷url并写入文件，然后直接复制到迅雷里就可以下载了，是不是很简单很开心？

### 新建项目
好的，终于要开始了。
CD到项目文件夹，执行命令：
```bash
scrapy startproject Rs05
```
随后项目文件夹内变生成了一系列文件，其结构如下：

>Rs05/
　　scrapy.cfg
　　Rs05/
　　　　`__init__.py`
　　　　items.py
　　　　piplines.py
　　　　settings.py
　　　　spiders/
　　　　　　`__init__.py`
　　　　　　...

简单介绍一下各个文件：
`scrapy.cfg` 是项目的配置文件；
`__init__.py` 这个文件不用管；
`items.py` 文件里编写爬取内容的Field；
`piplines.py` 文件里编写爬取后过滤内容的规则或者其他操作，如写入文件；
`settings.py` 是项目的设置文件；
`spiders` 内放置爬虫文件

### 设置规则
首先，需要定义items，可以理解为放置爬取内容的地方，与字典类似。
编写方法非常简单，
```python
import scrapy
class Rs05Item(scrapy.Item):
　　#name = scrapy.Field()
　　title = scrapy.Field()
　　url = scrapy.Field()
```
name根据你的需要随便起名字，数量由爬取种类的多少决定。
然后CD到spider文件，新建爬虫文件rs05_spider.py
打开文件编写
```python
#coding:utf-8
import re
import scrapy
from scrapy.contrib.spiders import CrawlSpider, Rule
from scrapy.contrib.linkextractors import LinkExtractor
from rs05.items import Rs05Item
class Rs05Spider(CrawlSpider):
　　name = 'rs05'
　　start_urls = ['http://www.rs05.com/']
　　rules = (
　　　　Rule(LinkExtractor(allow=('[a-z0-9]+\.html')),callback='parse_item'),
　　)
　　def parse_item(self,response):			
　　　　items = []
　　　　item = Rs05Item()
　　　　for sel in response.xpath('/html/body/div[4]/div[1]/div[1]'):
　　　　　　title = sel.xpath('h1/text()').extract()
　　　　　　item['title'] = title #t.encode('utf-8') for t in title
　　　　　　pattern = re.compile('href="(.*?)"',re.S)			
　　　　　　url = re.findall(pattern, sel.extract())			
　　　　　　item['url'] = url
　　　　　　items.append(item)
　　　　return items
```
根据代码可以看出，爬虫使用了scrapy内置的CrawlSpider，用于跟进链接继续爬取。而linkextractors用于设置跟进链接的规则，使用正则匹配。`name`为爬虫的名称，该名称唯一。`start_urls`为开始爬取的链接。需要注意的是，当要设置`allowed_domains`时，`start_urls`与`allowed_domains`不能相同，否则会出现错误。`parse_item`函数用于解析response的内容，并将结果保存到items中。上述代码使用xpath与re提取内容，当然也可以使用其他方法，比如beautifulsoup。无论用什么方法，最终将需要提取的内容放到items中即可。

### 写入文件
爬取内容后，如何输出呢？
此时打开pipelines.py，输入以下代码：
```python
import json
import codecs
from collections import OrderedDict
class Rs05Pipeline(object):
　　def __init__(self):
　　　　self.file = codecs.open('rs05.json','wb',encoding='utf-8')
　　def process_item(self, item, spider):
　　　　line = json.dumps(OrderedDict(item)) + '\n'
　　　　self.file.write(line.decode("unicode_escape"))
　　　　return item
　　def close_spider(self, spider):
　　　　self.file.close()
```
其中,`__init__()`和`close_spider()`相当于构造、析构函数，写不写都可以。但是process_item是一定要写的。这里用于处理爬取内容，丢弃或者保存。从上面的代码可以看出，爬取内容保存到了rs05.json文件中。
还没有完事，打开settings.py找到
```python
ITEM_PIPELINES = {
    'rs06.pipelines.Rs06Pipeline': 300,
}
```
取消注释，并将名字改为爬虫的名字，后面的数字为优先级，如果你有多个pipelines，可以为它们设定执行顺序。

### 开始爬取
好了，一切都设置好之后，CD到项目根目录可以开始爬取了。
执行下列命令：
```bash
scrapy crawl Rs05
```
一大串字母和数字闪过之后，在根目录生成了rs05.json, 打开文件看看是否提取到了想要的内容。我相信多数情况下，都没有一次成功的时候吧……
下面介绍两种调试的方法：
* 1.检查spider文件的过滤规则，是否提取到内容。如果使用xpath，可以通过xpath插件，或者chrome shift + i 查看xpath路径；
* 2.输入命令 `scrapy shell "http://xxx"` 然后输入 `response.selector.xpath('your rules')`查看内容，验证xpath路径是否正确。


### 小结
SCRAPY框架适用于中型大型爬虫项目，使我们专注于内容的提取，而不是爬虫的构建。小型项目可以使用urllib,repuests库等等。
简单来讲，网络爬虫就是获取网页内容后提取有价值信息的一个技术。我们设置好过滤内容规则后，让电脑自动提取，大大加强了我们提取信息的效率。当然，这一过程中，也会遇到一些困难，如网页改版，封禁IP等等。此时就需要将爬虫升级为机器学习或者分布式爬虫，这里需要更多更深的知识。
