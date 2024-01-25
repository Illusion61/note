# Python爬虫解析

## XPATH

#### XPATH解析原理

- 实例化一个etree对象,将被解析的源代码数据加载到对象中
- 调用etree对象中的xpath方法结合**xpath表达式**实现标签定位和内容捕获

```bash
pip3 install lxml
```

#### XPATH解析代码

- xpath获得的是一个**列表**
- xpath是按照先后顺序添加列表元素的,而不是按照层级顺序

```python
from lxml import etree
tree = etree.parse(filePath)#由本地html文件创建etree
tree = etree.HTML('page_text')#由源码字符串创建etree
res = tree.xpath(xpath表达式)
```

#### XPATH表达式(重点)

- 筛选标签

```python
/ 表示一个层级
// 表示不定数量的多个层级(1-n)
tag[@attrName="attrValue"] 根据属性进行筛选
tag[index] 根据第几个进行筛选(索引从1开始)
```

- 获得标签的文本和属性(也可以把文本和属性看作tree的一部分)

```python
/text()获得直系文本标签中的内容
//text()标签中非直系的文本内容
/@attrName取属性
```

```python
/html/head/title/text() #获得网页标题
/html/body//div[@class="image-grid"]//img/@src #获得网页图片地址
/html/body//div[@class="image-grid"]//img[3]/@src #获得网页图片地址
//input[@name="password" and @id="password"] #同时通过两个标签筛选
//input[@id="username" or @id="password"]
//span[@id="pe100_page_allnews"]//a[last()-1] #倒数第二个元素
```

## Selenium

#### 简介

- 是一款针对浏览器自动化的程序,速度比较慢但是能爬取一些受限制的网页内容
- 便捷地获取网站中动态加载的数据(由javascript生成内容的网页内容),因为借助浏览器渲染
  - 很多时候一些audio的**src**需要等到**点击按钮**之后**才会出现**
  - 动态加载的网页需要用户采取一些**行为**,之后**HTML才会被更改出现资源**

#### 安装部署

- 本地环境部署:

```bash
pip3 install selenium
进入chrome浏览器的help-->About Google Chrome查看浏览器版本
下载浏览器驱动程序(对应指定版本):https://googlechromelabs.github.io/chrome-for-testing/
```

- 无界面Linux服务器安装chrome&driver:

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo dpkg -i google-chrome*.deb # 如果出错，执行下一个命令
sudo apt-get install -f
```



#### 实战

- 部署环境

```bash

```
- 方法列表driver:
```txt
发起请求: get(url)
标签交互: send_keys('xxx')
执行js程序: execute_script('jscode')
前进,后退: forward(),back()
关闭浏览器: quit()
```

```txt
标签定位: find_element(s)_by...
	find_element_by_xpath()返回一个元素对象
	find_elements_by_xpath()返回元素对象列表
	不能使用xpath中.../@attrName取属性,必须返回元素.应该使用get_attribute方法
	获取属性: 元素.get_attribute('attrName')
	获取文本: 元素.text
```



- 代码示例
```python
from selenium import webdriver
import time
browser = webdriver.Chrome('./chromedriver.exe')
browser.implicitly_wait(5)
browser.get('https://acg.gamersky.com/pic/wallpaper/pc/')
num = int(browser.find_element_by_xpath('//*[@id="pe100_page_allnews"]/a[last()-1]').text)-1
web_list = []
for i in range(num):
	btn = browser.find_element_by_xpath('//*[@id="pe100_page_allnews"]/a[last()]')
	time.sleep(10)
	webpages = browser.find_elements_by_xpath('//div[@class="Mid_L"]//ul/li/a')
	btn.click()
	for web in webpages:
		web_list.append(web.get_attribute('href'))
webpages = browser.find_elements_by_xpath('//div[@class="Mid_L"]//ul/li/a')
for web in webpages:
	web_list.append(web.get_attribute('href'))
```



# Scrapy

## Scrapy基本概念

#### 爬虫一般步骤

- 初始url加入url队列
- request获取网页数据
- 解析网页数据(提交新的请求进入url队列)
- 保存获取的数据

#### 为什么要使用scrapy

- 节省时间:很多功能已经实现,不需要自己全部手写
- 性能出色:拥有比较出色的底层框架
- 个性化定制:可以使用很多自定义功能,比如随机代理切换,自定义cookie等,实现起来比较方便

#### scrapy架构

![image-20240124100150990](C:\homecity\Note\others\markdown\2024-01-24-100207.png)

![2024-01-24-102103](C:\homecity\Note\others\markdown\2024-01-24-102103.png)

#### 需要自己手写的部分

- 爬虫:提供起始url,解析规则
- 管道:保存数据
- 下载中间件:自定义下载配置,比如代理等
- 爬虫中间件:自定义爬虫配置,比如自定义请求request等

## Scrapy实际操作

#### scapry安装配置

- scrapy安装:

```bash
apt-get install scrapy
pip/pip3 install scrapy
scrapy version 验证是否成功安装scrapy
```

- scrapy配置(settings.py)

```python
ROBOTSTXT_OBEY = False #是否遵从robots.txt协议,如果为True那么会自动不request一些网页
```

#### scapry项目全流程

- 创建scrapy项目:`scrapy startproject [项目名称]`

```bash
scrapy startproject myspider
├── myspider
│   ├── __init__.py
│   ├── items.py 存储
│   ├── middlewares.py 中间件
│   ├── pipelines.py 管道存储
│   ├── settings.py 设置
│   └── spiders 爬虫文件夹,存储新建的爬虫
│       └── __init__.py
└── scrapy.cfg
```

- 创建爬虫:`scrapy genspider [爬虫名称] [允许的域名]`

```bash
scrapy genspider homepage sciform.one
```

```python
class HomepageSpider(scrapy.Spider):
    name = "homepage" #爬虫名称
    allowed_domains = ["sciform.one"] #存储允许爬虫的域名,用于过滤url
    start_urls = ["https://sciform.one"] #起始域名(首页)
	#起始url-->调度器-->scrapy引擎-->download中间件-->下载器-->spider解析
    def parse(self, response):#response就是获取到的网页数据
        print(response)
        pass
```

- 运行爬虫:`scrapy crawl [爬虫名称] `

```bash
scrapy crawl homepage
```

