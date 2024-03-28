---
title: SCRAPY爬取智联招聘信息
date: 2017-02-07 13:41:00
tags: Python
toc: true
description: 新的一年准备换一个新工作，如何快速获取大量的职位信息呢？干脆编写一个爬虫好了，顺带巩固一下知识。
---
网上找工作这件事，首先要登录主站搜索职位名称，然后在一大串列表里逐个查看职位需求，查看完毕关闭页面，再打开新页面，想一想都觉得是费心费力的一件事。俗话说：“磨刀不误砍柴工”，再加上自己懒癌发作，心想着一定要一劳永逸的解决这件事。正好SCRAPY框架能派上用场，大致整理了一下思路开始撸代码。

### 思路
根据文章开头部分的话，可以整理出`网上找工作`这一系列动作的流程

1. 打开主站搜索`职位`，并选择`地点`
2. 网站返回一系列`<li><a>职位名称</a></li>`
3. 点击`<a>`进入详情页，里面包含着职位和公司的详细信息
4. 查看完毕后关闭，换下一个`<a>`

好了，流程整理好了，怎么实现呢？
显然第一步打开网站获取网页信息可以使用`urllib`或者`requests`模块，第二步提取`URL`信息交给`re`和`BeautifulSoup`处理，第三步使用`SCRAPY`处理第二步获取的`URL`到逐个页面提取职位详细信息，使用`SCRAPY`内置的`xpath`即可。
思路整理完毕，let's coding!

### 获取URL
秉着模块化的精神，我把提取URL这一部分单独写一个文件，然后在`SCRAPY`调用。这样不仅方便调试，还能减少`SCRAPY`内爬虫文件代码的冗杂程度。
首先，引入必要的模块
```python
# -*-coding:utf-8-*-
import urllib2
import re
from bs4 import BeautifulSoup

```
为了在其他模组内调用，创建一个`Geturls`类
```python
class Geturls():
```
其他方法都在这里完成。
定义打开网页的方法
```python
def open(self, url):
	try:
		request = urllib2.Request(url)
		response = urllib2.urlopen(request)
		content = response.read().decode('utf-8')
		return content
	except urllib2.URLError, e:
		if hasattr(e,"code"):
			print e.code
		if hasattr(e,"reason"):
			print e.reason
```
打开网页成功后将返回网页内所有的内容，然后提取关键信息
```python
def distill(self, content):
		soup = BeautifulSoup(content, 'html.parser').find_all(href=re.compile("http://jobs.zhaopin.com/"))
		urls_temp = soup[:60]
		urls = []

		for url in urls_temp:
			pattern = re.compile('href="(.*?)"', re.S)
			url_temp = re.findall(pattern, str(url))
			urls.append(url_temp)
		return urls
```
原本想把方法名设置为`extract`，但是`SCRAPY`内有相同的方法名，只好换成`distill`(蒸馏)，我觉得反而更加形象XD.
智联网页上的东西很多，而我只需要职位的链接，所以使用正则把链接提出来即可。这些链接的规律就是开头皆为`http://jobs.zhaopin.com/`,后面附带一系列数字。查看网页源代码后，发现还是蛮有规律的，在第一链接之前没有其他的广告链接，而且每页的职位信息顺序排列只有60个，在这之后又会有11个广告链接，如下图

![](http://okzvvfkr0.bkt.clouddn.com/zhilian_urls.jpg)

所以只需要提取出前60个即可。

### 获取信息
#### 准备工作
接下来的工作是，在上一步获得了所有的url之后，传到`SCRAPY`逐个打开页面获取职位信息。在目标目录内打开命令行执行
```bash
scrapy startproject recruit
```
好吧，这个项目名字不是很高大上，起码能够望文生义吧……
命令执行后，会自动生成下列文件夹
```
|--recruit/
|	|--scrapy.cfg
|	|--recruit/
|		|--`__init__.py`
|		|--items.py
|		|--piplines.py
|		|--settings.py
|		|--spiders/
|		|--`__init__.py`
```
将上一步完成的代码保存为`geturls.py`放到`recruit`(与items.py等文件同级)目录下。

#### 定义items
先打开网页的职位信息页面看看需要提取哪些信息吧

![](http://okzvvfkr0.bkt.clouddn.com/description.jpg)

除了这些信息外，下面还有职位描述和地址信息等，把这些信息定义为items，即为放置这些信息的容器。
打开`items.py`敲入以下代码
```python
# -*- coding: utf-8 -*-
import scrapy

class RecruitItem(scrapy.Item):
    position = scrapy.Field()
    company = scrapy.Field()
    salary = scrapy.Field()
    location = scrapy.Field()
    date = scrapy.Field()
    nature = scrapy.Field()
    experience = scrapy.Field()
    education = scrapy.Field()
    number = scrapy.Field()
    category = scrapy.Field()
    description = scrapy.Field()
    address = scrapy.Field()
```
保存后退出。

#### 定义spider
在`spiders`文件夹内新建文件`recruit_spider.py`,这是爬虫的核心文件。
首先引入需要的模块
```python
#coding:utf-8
import scrapy
from recruit.items import RecruitItem
from recruit.geturls import Geturls
from scrapy.http import Request
```
这里要引入前面编写的`Geturls类`和定义的`items`.
新建类，继承框架的爬虫基类，并注明唯一的爬虫名称
```python
class RecruitSpider(scrapy.Spider):
	name = 'recruit'
```
其余的方法在这个类中编写。第一个方法是
```python
def start_requests(self):
	base_url = "http://sou.zhaopin.com/jobs/searchresult.ashx?jl=%E5%A4%A7%E8%BF%9E&kw=php&sm=0&p=1"
	geturls = Geturls()
	content = geturls.open(base_url)
	suburls = geturls.distill(content)
	
	for url in suburls:
		url_str = str(url)
		url_temp = url_str.strip("'[]'")
		yield Request(url_temp, self.parse)
```
这是`scrapy`内置的方法名称，依据名称可知是开启`requests`执行的函数，这里用到了第一步编写的提取url类，直接使用。需要注意的是传过来的url可能包含多余字符，使用`strip()`去除。`base_url`是在智联主页搜索后跳转的链接，可以发现这个链接也是有规律可循的，规律是这样的
```python
jl=地点，kw=职位名称，sm=页面状态码(默认=0，可以无视)，p=页码;
```
可以根据这个规律，自行创建。地点职位页码，根据自身情况修改即可，或者将页码代入循环自动生成一系列地址逐个爬取，这里就不再拓展了……
然后将`url`传给scrapy内置方法`Request()`，并且使用`parse`方法作为回调函数进行筛选。

#### 爬取信息
爬取信息更加简单了，根据`xpath`获取信息位置取得文本，再剔除多余的空格和字符再保存到`items`内即可。
```python
def parse(self, response):
		items = RecruitItem()

		items['position'] = "".join(response.xpath('/html/body/div[5]/div[1]/div[1]/h1/text()').extract()).strip()
		items['company'] = "".join(response.xpath('/html/body/div[5]/div[1]/div[1]/h2/a/text()').extract()).strip()
		items['salary'] = "".join(response.xpath('/html/body/div[6]/div[1]/ul/li[1]/strong/text()').extract()).strip()
		items['location'] = "".join(response.xpath('/html/body/div[6]/div[1]/ul/li[2]/strong/a/text()').extract()).strip()
		items['date'] = "".join(response.xpath('//*[@id="span4freshdate"]/text()').extract()).strip()
		items['nature'] = "".join(response.xpath('/html/body/div[6]/div[1]/ul/li[4]/strong/text()').extract()).strip()
		items['experience'] = "".join(response.xpath('/html/body/div[6]/div[1]/ul/li[5]/strong/text()').extract()).strip()
		items['education'] = "".join(response.xpath('/html/body/div[6]/div[1]/ul/li[6]/strong/text()').extract()).strip()
		items['number'] = "".join(response.xpath('/html/body/div[6]/div[1]/ul/li[7]/strong/text()').extract()).strip()
		items['category'] = "".join(response.xpath('/html/body/div[6]/div[1]/ul/li[8]/strong/a/text()').extract()).strip()
		items['description'] = "".join(response.xpath('/html/body/div[6]/div[1]/div[1]/div/div[1]/p[1]/text()').extract()).strip()
		items['address'] = "".join(response.xpath('/html/body/div[6]/div[1]/div[1]/div/div[1]/h2/text()').extract()).strip()

		return items

```

#### 定义pipelines
`pipelines`模块是将`items`内的信息做过滤，持久化或其他工作的。
这里的代码跟之前`SCRAPY简单入门`中几乎一样，就不再赘述了。
```python
# -*- coding: utf-8 -*-
import json  
import codecs
from collections import OrderedDict

class RecruitPipeline(object):
	def __init__(self):
		self.file = codecs.open('recruit.json','wb',encoding='utf-8')

	def process_item(self, item, spider):
		line = json.dumps(OrderedDict(item))
		self.file.write(line.decode("unicode_escape"))
		self.file.write("\n")
		return item

	def close_spider(self, spider):
		self.file.close()
```
将`items`内的信息写入json文件
不要忘记在`setting.py`文件中，取消以下几行的注释,并修改成自己项目的名字。
```python
ITEM_PIPELINES = {
   'recruit.pipelines.RecruitPipeline': 300,
}
```
OK，所有准备工作完成！打开命令行键入
```bash
scrapy crawl recruit
```
见证奇迹的发生吧～

### 总结
这个小项目整体来说并不难，应该说麻烦多一些。其实数据导出之后，才做好了一个开头工作，本打算将这些数据做成一个可视化的图表，这样才能更直观的感受到信息的价值。数据处理稍微查了一下也是个大坑，更何况这种中文信息处理，想想就有点头疼。话不多说，继续学习，待将数据处理完成后写一篇新的文章。