---
layout: post
title:  "详解Python 采用 requests + Beautiful Soup 爬取房天下新楼盘推荐"
date:   2018-01-07 11:44:30
categories: Python
tags: Python requests BeautifulSoup Spider
author: Michael
mathjax: true
---

* content
{:toc}

最近一直在关注Python写爬虫相关的知识，尝试了采用requests + Beautiful Soup来爬取房天下（原搜房网）的推荐新楼盘。

不用不知道，一用发现有惊喜也有惊吓，本文就一同记录下惊喜和踩的一些乱码的坑。



首先，觉得Beautiful soup解析网页更加符合人类的常规思维，比使用正则表达式（python中的re库）更容易理解。 同时关于requests遇到了中文字符和特殊字符解码的问题。本文都将给于深入的解说。



### 软件环境

Python    : 3.6.0   

PyCharm: Community 2017.2 

库1 : requests

库2 : beautifulsoup


## Requests知识概要

Requests 是用Python语言编写，基于 urllib，采用 Apache2 Licensed 开源协议的 HTTP 库。它比 urllib 更加方便，可以节约我们大量的工作，完全满足 HTTP 测试需求。完全支持Python3哦

1. 安装requests 

   可以采用pip安装requests,具体代码是：
```
pip install requests
```
2. 简单例子

   下面例子中是几个常用的属性，response.text可以得到html的源代码，response.encoding默认为ISO-8859-1，此编码针对英文没有问题，如果我们的网页中带有中文，将会引起乱码的问题。后面有解决方案。
```python
import requests
response = requests.get("http://sh.fang.com/") #获取一个Http请求
print(response.encoding)          #当前编码，默认为 ISO-8859-1
print(response.apparent_encoding) #当前网页的内容的实际编码
print(response.content)  #content返回的是bytes型的原始数据
print(response.text)     #text返回的是处理过的Unicode型的数据
```

## Beautiful Soup简要知识

1. Beautiful Soup 是一个非常流行的 Python 模块。 该模块可以解析网页， 并提供定位内容的便捷接口。 如果你还没有安装该模块， 可以使用下面的命令
安装其最新版本：
```
pip install beautifulsoup4
```
使用 Beautiful Soup 的第一步是将下载的 HTML 内容解析为 soup 文档 。

2. 如何引用Beautiful Soup
```python
from bs4 import BeautifulSoup
```

3. 一个简单例子
```python

from bs4 import BeautifulSoup
html = "<ul class='country'><li>中国</li><li>美国</li></ul>"
#解析html然后得到一个soup文档
soup = BeautifulSoup(html,'html.parser')
html = soup.prettify()
ul = soup.find('ul',attrs={'class':'country'})

#下面代码只返回一个匹配的元素
print(ul.find('li'))     # returns the first match element

#下面代码返回又有匹配的元素
for countryname in ul.find_all('li'):
    print(countryname)

```

## 实战爬取房天下推荐新楼盘


1. 在Chrome中打开sh.fang.com地址，按F12观察新盘推荐tab的网页源代码结构

主要结构是: 顶层为一个名为ti011的div，下面有四个class为tenrtd的div用于四个楼盘的现实。每个楼盘下面有一个class = "text1"的div存储了楼盘名称，另一个class = "text2"的存储了楼盘的价格。

<img width="100%" src="https://yuanzhitang.github.io/images/fang-code-layout.png"/>

2. 第一版代码完成如下，但是发现有一个中文乱码的问题
```python
from bs4 import BeautifulSoup
import requests
rep = requests.get("http://sh.fang.com/")
html = rep.text
soup = BeautifulSoup(html, 'html.parser')
#获取顶层 新盘推荐 的整个div
div = soup.find('div',attrs={'id':'ti011'})
#获取四个楼盘的div，更具他们的class = "tenrtd"
for house in div.find_all('div', attrs={'class': 'tenrtd'}):
    
    # 根据class="text1"获取存储楼盘标题的div
    titleDiv = house.find('div', attrs={'class': 'text1'})
    title = titleDiv.find('a').text
    # 根据class="text2"获取存储楼盘价格的div
    priceDiv = house.find('div', attrs={'class': 'text2'})
    price = priceDiv.find('b').text
    print(title, " ", price)
 ```

输出的结果尽然是这样， Why？

 ÖÐ½ðº£ÌÄÍå     48000Ôª/©O 
 »ªÒêÒÝÆ·À½Íå     43057Ôª/©O 
 ÖÐ¹ú¹é¹È´´¿ÍSOHO     ¼Û¸ñ´ý¶¨ 
 ÐÂÎ÷ÌÁ¿×È¸³Ç     14000Ôª/©O


3. 研究中文乱码问题

   中文乱码也算是requests常见的一个问题，为什么会这样的呢，看bs自己的文档描述

    Encodings
    When you receive a response, Requests makes a guess at the encoding to use for decoding the response 
    when you access the Response.text attribute. Requests will first check for an encoding in the HTTP 
    header, and if none is present, will use chardet to attempt to guess the encoding.

    The only time Requests will not do this is if no explicit charset is present in the HTTP headersand 
    the Content-Type header contains text. In this situation, RFC 2616 specifies that the default charset 
    must be ISO-8859-1. Requests follows the specification in this case. If you require a different encoding, 
    you can manually set the Response.encoding property, or use the rawResponse.content.

其实重要的也就是如果没有在requests的header部分里面指定encoding的话，那么默认采用ISO-8859-1这个编码去解码response.content里面的字节数据。ISO-8859-1针对英文是没有问题的，但是针对中文就不行了。


解决思路当然就是换encoding，但是我们应该用什么编码呢？下面这个代码可以看到网页的实际encoding，下面代码将输出：GB2312 ，所以我们采用GB2312来解码我们的数据。
```python
print(rep.apparent_encoding)
```

4. 换编码解决中文字符

    经过上面的研究，我们修订代码对response设定encoding = "GB2312"

rep.encoding = "GB2312"
   运行后结果如下：

  中金海棠湾     48000元/�O 
 华谊逸品澜湾     43057元/�O 
 中国归谷创客SOHO     价格待定 
 新西塘孔雀城     14000元/�O
   发现代码显示的结果中㎡又乱码了，不应该啊，这又是为什么呢？ 下面接着研究。。。



5. 研究解决特殊字符乱码问题

   引起乱码的原因估计就是在字符集中找不到特定的字符，比如这个㎡。是不是GB2312这个字符集不够全面呢？带着这个疑问去查阅相关的资料关于中文的几个编码：

* GB2312
GB 2312 或 GB 2312-80 是中国国家标准简体中文字符集，全称《信息交换用汉字编码字符集·基本集》，又称 GB 0，由中国国家标准总局发布，1981 年 5 月 1 日实施。GB 2312 编码通行于中国大陆；新加坡等地也采用此编码。中国大陆几乎所有的中文系统和国际化的软件都支持 GB 2312。GB 2312 标准共收录 6763 个汉字，其中一级汉字 3755 个，二级汉字 3008 个；同时收录了包括拉丁字母、希腊字母、日文平假名及片假名字母、俄语西里尔字母在内的 682 个字符。GB 2312 的出现，基本满足了汉字的计算机处理需要，它所收录的汉字已经覆盖中国大陆99.75% 的使用频率。对于人名、古汉语等方面出现的罕用字，GB 2312 不能处理，这导致了后来 GBK 及 GB 18030 汉字字符集的出现。


* GBK
GBK 即汉字内码扩展规范，K 为汉语拼音 Kuo Zhan（扩展）中“扩”字的声母。英文全称 Chinese Internal Code Specification。GBK 共收入 21886 个汉字和图形符号，包括：GB 2312 中的全部汉字、非汉字符号。BIG5 中的全部汉字。与 ISO 10646 相应的国家标准 GB 13000 中的其它 CJK 汉字，以上合计 20902 个汉字。其它汉字、部首、符号，共计 984 个。GBK 向下与 GB 2312 完全兼容，向上支持 ISO 10646 国际标准，在前者向后者过渡过程中起到的承上启下的作用。GBK 采用双字节表示，总体编码范围为 8140-FEFE 之间，首字节在 81-FE 之间，尾字节在 40-FE 之间，剔除 XX7F 一条线


* GB18030
GB 18030，全称：国家标准 GB 18030-2005《信息技术中文编码字符集》，是×××现时最新的内码字集，是 GB 18030-2000《信息技术信息交换用汉字编码字符集基本集的扩充》的修订版。GB 18030 与 GB 2312-1980 和 GBK 兼容，共收录汉字70244个。与 UTF-8 相同，采用多字节编码，每个字可以由 1 个、2 个或 4 个字节组成。编码空间庞大，最多可定义 161 万个字符。支持中国国内少数民族的文字，不需要动用造字区。汉字收录范围包含繁体汉字以及日韩汉字

从上文中可以简要的得出GB 2312 过时标准、GBK 微软标准、GB 18030 国家标准，字符个数方面：GB 18030 > GBK > GB2312


所以我们决定采用 GB18030来解码我们的数据，代码改动如下：

```python
rep.encoding = "gb18030"
```

### 完整的代码

```python
from bs4 import BeautifulSoup
import requests

rep = requests.get("http://sh.fang.com/")
rep.encoding = "gb18030"
html = rep.text
soup = BeautifulSoup(html, 'html.parser')

#获取顶层 新盘推荐 的整个div
div = soup.find('div',attrs={'id':'ti011'})

#获取四个楼盘的div，更具他们的class = "tenrtd"
for house in div.find_all('div', attrs={'class': 'tenrtd'}):
    # 根据class="text1"获取存储楼盘标题的div
    titleDiv = house.find('div', attrs={'class': 'text1'})
    title = titleDiv.find('a').text
    # 根据class="text2"获取存储楼盘价格的div
    priceDiv = house.find('div', attrs={'class': 'text2'})
    price = priceDiv.find('b').text
    print(title, " ", price)
```

输出结果：

 中金海棠湾     48000元/㎡ 
 华谊逸品澜湾     43057元/㎡ 
 中国归谷创客SOHO     价格待定 
 新西塘孔雀城     14000元/㎡


问题解决了，关键知识点总结：

可以采用requests库来获取网页html

采用Beautiful soup基于html构建一个soup文档，然后用find 或者 find_all方法查询自己需要的html节点

根据目标网页的内容来更改response.encoding从而解决乱码问题

