5-1

Scrapy-Wiki

----------


## 学习Scrapy框架爬虫

*什么是Scrapy？*

> Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 其可以应用在数据挖掘，信息处理或存储历史数据等一系列的程序中。其最初是为了页面抓取 (更确切来说, 网络抓取 )所设计的， 也可以应用在获取API所返回的数据(例如 Amazon Associates Web Services ) 或者通用的网络爬虫。Scrapy用途广泛，可以用于数据挖掘、监测和自动化测试。

Scrapy 使用了 Twisted异步网络库来处理网络通讯。整体架构大致如下

![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190501093101.png)



### 0x01安装Scrapy

```
pip install -i https://pypi.douban.com/simple/ scrapy
```
他能将所用的大多数依赖包都安装
![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190501091844.png)


pip安装过程+截图

### 0x02创建项目

使用pacharm是不能创建scrapy项目的


命令行创建项目过程

```
scrapy startproject ArticleSpider
```
![](https://raw.githubusercontent.com/Hatcat123/GraphicBed/master/Img/20190501092116.png)

创建的过程使用默认的模板创建。后期熟悉后可使用自己定制的模板创建

开始使用项目

使用pycharm打开项目使用



### 0x03 项目结构讲解

初始化每个文件是什么样子，都有哪些作用

```
文件夹 PATH 列表
卷序列号为 7C2B-B6D8
D:.
│  scrapy.cfg
│          
└─ArticleSpider
    │  items.py
    │  middlewares.py
    │  pipelines.py
    │  settings.py
    │  __init__.py
    │  
    ├─spiders
          __init__.py

```
### 文件的作用：

**scrapy.cfg**

项目的配置信息，主要为Scrapy命令行工具提供一个基础的配置信息。（真正爬虫相关的配置信息在settings.py文件中）

**ArticleSpider**

项目子目录

**items.py**

设置数据存储模板，用于结构化数据，如：Django的Model

**middlewares.py**

**pipelines.py**

数据处理行为，如：一般结构化的数据持久化

**settings.py**

配置文件，如：递归的层数、并发数，延迟下载等

**spiders**

爬虫目录，如：创建文件，编写爬虫规则
   


### 测试项目

进入到项目爬取目录

```
cd AriticSpider
```
生成spider爬虫文件
```
scrapy genspider jobbole blog.jobbole.com
```
在spider目录下,依靠基础模板自动生成

```
# -*- coding: utf-8 -*-
import scrapy


class JobboleSpider(scrapy.Spider):
    name = 'jobbole'
    allowed_domains = ['blog.jobbole.com']
    start_urls = ['http://blog.jobbole.com/']

    def parse(self, response):
        pass
```
查看源码，发现有个`start_requests`函数,他将for循环start_urls，自动爬取，将结果yeid到parse

```
    def start_requests(self):
			······
            for url in self.start_urls:
                yield self.make_requests_from_url(url)
        else:
            for url in self.start_urls:
                yield Request(url, dont_filter=True)
```

设置不遵循robot协议
```
# Obey robots.txt rules
ROBOTSTXT_OBEY = True
```

使用命令行运行

```
输入scrapy查看命令

 ArticleSpider

Usage:
  scrapy <command> [options] [args]

Available commands:
  bench         Run quick benchmark test
  check         Check spider contracts
  crawl         Run a spider
  edit          Edit spider
  fetch         Fetch a URL using the Scrapy downloader
  genspider     Generate new spider using pre-defined templates
  list          List available spiders
  parse         Parse URL (using its spider) and print the results
  runspider     Run a self-contained spider (without creating a project)
  settings      Get settings values
  shell         Interactive scraping console
  startproject  Create new project
  version       Print Scrapy version
  view          Open URL in browser, as seen by Scrapy

Use "scrapy <command> -h" to see more info about a command

```
运行命令
```
scrapy crawl jobbole

```
如果你是用的新环境，第一次运行可能会出现`ImportError: No module named 'win32api'`没有导入win32api的包的问题。

pip 安装 pypiwin32

```
pip install pypi32
```
再次运行
```
scrapy crawl jobbole

```
运行结果

```
 'start_time': datetime.datetime(2019, 5, 1, 2, 4, 29, 29653)}
2019-05-01 10:04:29 [scrapy.core.engine] INFO: Spider closed (finished)

```
**使用main.py程序运行**

一般情况下，我们为了是方便开发的时候调试程序，不使用命令号直接运行项目，而是使用`scrapy.cmdline`提供的包。

在项目下创建main.py文件，使用execute创建命令号的命令。

```
from scrapy.cmdline import  execute
import sys
import os

sys.path.append(os.path.dirname(os.path.abspath(__file__)))
execute(['scrapy','crawl','jobbole'])
```
这时候运行main文件一样能达到运行的效果，然后还可以使用debug模式下断点调试。


### 0x04 爬虫分析

介绍爬取目标网站的网站结构，分析网站是否存在验证、限制、等等反爬机制，是否能适用于scrapy。

分析爬取的目标的url、想要获取的数据，如何解析数据，数据存储到哪里

对于目标页面`http://blog.jobbole.com/110287`提取标题、时间、点赞数、收藏数、评论数、正文

### xpath提取数


使用浏览器开发者工具copy xpath路径

```spider/jobbole.py

    def parse(self, response):
        title = response.xpath('//*[@id="post-110287"]/div[1]/h1/text()')


```
获取到title的对象，使用`extract`获取匹配到的列表数据。

在开发过程中频繁启动main函数调试是一件效率很低的事情，每次都要访问一次网站，对于网站结果解析的分析过程，我们可以使用命令行中的`scrapy shell`，他能对网站进行一次请求，可以根据需求对请求的结果分析，减少了对网站的请求次数。

```
scrapy shell  http://blog.jobbole.com/110287
```
请求结果
```
2019-05-01 10:40:39 [scrapy.core.engine] DEBUG: Crawled (200) <GET http://blog.jobbole.com/110287/> (referer: None)
[s] Available Scrapy objects:
[s]   scrapy     scrapy module (contains scrapy.Request, scrapy.Selector, etc)
[s]   crawler    <scrapy.crawler.Crawler object at 0x0000000004BBFAC8>
[s]   item       {}
[s]   request    <GET http://blog.jobbole.com/110287>
[s]   response   <200 http://blog.jobbole.com/110287/>
[s]   settings   <scrapy.settings.Settings object at 0x0000000004BBF5F8>
[s]   spider     <JobboleSpider 'jobbole' at 0x5d97e48>
[s] Useful shortcuts:
[s]   fetch(url[, redirect=True]) Fetch URL and update local objects (by default, redirects are followed)
[s]   fetch(req)                  Fetch a scrapy.Request and update local objects
[s]   shelp()           Shell help (print this help)
[s]   view(response)    View response in a browser

```

命令行中获取标题的过程方法

```
>>> title = response.xpath('//*[@id="post-110287"]/div[1]/h1/text()')
>>> title
[<Selector xpath='//*[@id="post-110287"]/div[1]/h1/text()' data='2016 腾讯软件开发面试题（部分）'>]
>>> title.extract()
['2016 腾讯软件开发面试题（部分）']
>>> title.extract()[0]
'2016 腾讯软件开发面试题（部分）'
>>>


```



### 0x05 详细编程过程


介绍编程的每一步都代表什么含义、尽量详细详细


### 0x06 测试代码&寻找bug

展示编程过程中陷入哪些坑，哪些地方需要注意



## 总结

这将写一串总结，介绍爬虫的心得，最好对比分析出于其他方式爬虫的区别。

