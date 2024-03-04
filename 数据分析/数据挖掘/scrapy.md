# Python爬虫基础

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
//input[@name="password" and @id="password"] #同时通过两个标签筛选
//input[@id="username" or @id="password"]
//element[text()="特定文本"] #对文本进行匹配
//element[contains(text(), "特定文本")] #包含特定文本
//span[@id="pe100_page_allnews"]//a[last()-1] #倒数第二个元素
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
sudo apt --fix-broken install #解决安装依赖问题
google-chrome --version #检查版本
下载浏览器驱动程序(对应指定版本):https://googlechromelabs.github.io/chrome-for-testing/
pip3 install selenium
```

#### 实战

- 方法列表driver:
```txt
发起请求: get(url)
标签交互: send_keys('xxx')
执行js程序: execute_script('jscode')
前进,后退: forward(),back()
关闭浏览器: quit()
driver.close():关闭当前页面,如果是唯一的标签页就退出
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

#### selenium配置项

```python
options = ChromeOptions()
options.add_argument('--headless')#无头浏览
options.add_argument('--disable-gpu')#不显示
options.add_argument('excludeSwitches=["enable-automation"]')#允许自动化(不显示头部)
options.add_argument(r'user-data-dir=C:/homecity/Cpp/Users')#指定用户配置目录
driver = Chrome(options=options)
driver.implicitly_wait(6)#等待6秒加载所有内容,如果加载出来就直接继续,否则超时就显示异常(防止还没加载出来就开始解析)
```

#### undetected_chromedriver

- undetected_chromedriver可以绕过cloud_flare5秒盾
- 解析网页内容的时候语法不太一样(使用By)

```python
from selenium.webdriver.common.by import By
from undetected_chromedriver import Chrome, ChromeOptions
options = ChromeOptions()
options.add_argument('excludeSwitches=["enable-automation"]')#允许自动化(不显示头部)
options.add_argument(r'user-data-dir=C:/homecity/Cpp/Users')#指定用户配置目录
driver = Chrome(options=options)
driver.implicitly_wait(6)
...
Before = driver.find_elements(By.XPATH,'//*[@id="info"]/h1/*[@class="before"]')[0].text
```

- 不用指定driver

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

![image-20240124100150990](https://raw.githubusercontent.com/Illusion61/note/main/others/markdown/2024-01-24-100207.png)

![2024-01-24-102103](https://raw.githubusercontent.com/Illusion61/note/main/others/markdown/2024-01-24-102103.png)

#### 需要自己手写的部分

- 爬虫:提供起始url,解析规则(parse以及自定义规则)
- 管道:保存数据
- 下载中间件:自定义下载配置,比如代理等
- 爬虫中间件:自定义爬虫配置,比如自定义请求request等

## Scrapy操作简介

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
LOG_LEVEL = 'ERROR'
USER_AGENT = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36"
ITEM_PIPELINES = {
   "myspider.pipelines.ImgPipe": 300,
} 
IMAGES_STORE = './imagePro'
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
```

- 运行爬虫:`scrapy crawl [爬虫名称] `

```bash
scrapy crawl homepage [--nolog]
```

## Scrapy编程

#### 爬虫部分

- response属性&方法

```txt
response.headers:响应头
response.url:响应url(不包括锚点)
response.status:响应码
response.body:响应二进制格式
response.text:响应文本格式
response.encoding:编码
response.meta:元数据,供请求时候传递item使用

response.request:请求对象
response.request.url:请求url(包括锚点#)
response.request.headers:请求头
```

```txt
response.xpath():使用xpath对结果数据进行解析,返回选择器对象列表
response.extract():从选择器对象列表或者单个选择器对象中提取数据
response.extract_first():从选择器对象列表中提取第一个数据,如果为空就返回null
```
- 构建scrapy.Request对象

```txt
yield scrapy.Request(url=url,callback=self.subparse,meta={'item':item})
```
- 钩子函数:start_requests
#### 持久化存储

- 终端指令进行持久化存储
- 管道进行持久化步骤

```txt
1. 在item类中定义相关的属性(items.py中)
    filename = scrapy.Field()
    url = scrapy.Field()
2. 数据解析(spider.py)
3. 将解析到的数据封装到item类型的对象中(spider.py)
	from myspider.items import MyspiderItem
	...
    item = MyspiderItem() 
    item['filename'] = xxx
    item['url'] = xxx
4. 将item类型的对象提交给管道进行持久化存储的操作(spider.py)
	yield item
5. 在管道类process_item中将其接收到的item进行持久化存储操作(pipelines.py)
open_spider
	filename = item['filename']
	url = item['url']
close_spider
6. 在配置文件中开启管道(settings.py)
	ITEM_PIPELINES = {
       "myspider.pipelines.MyspiderPipeline": 300,
       "myspider.pipelines.MyspiderPipelineMysql":301,
    } #数字越小,优先级越高
```
- **多个管道进行持久化存储**:`怎么将爬虫数据既保存到本地又上传数据库`

  - 管道优先级:高优先级的管道**在**低优先级管道的**前边**

  ```txt
  ITEM_PIPELINES = {
         "myspider.pipelines.MyspiderLocalPipeline": 299,
         "myspider.pipelines.MyspiderDBPipeline":300,
      } #数字越小,优先级越高
  ```

  - 在高优先级管道的process_item中,return item,**return的值就会被后面管道作为item参数接收**

  ```python
  class MyspiderLocalPipeline:#优先级299,在前面
      def process_item(self, item, spider):#1.接收到spiders的item对象
          xxx#2.本地存储item对象
          return item#3.返回item对象
  class MyspiderDBPipeline:#优先级300,在后面
      def process_item(self, item, spider):#4.item接收到前面过程3返回的item对象
          xxx#5.上传数据库存储item对象
          retumlrn item
  ```
  
- ImagesPipeline(需要运行环境有pillow):

  ```python
  #./pipelines.py
  import scrapy
  from scrapy.pipelines.images import ImagesPipeline
  class imgPipe(ImagesPipeline):
  	#根据图片地址对图片数据进行请求
  	def get_media_requests(self, item, info):
  		yield scrapy.Request(item['src'])
  	#指定图片存储的路径
  	def file_path(self, request, response=None, info=None,):
  		imgName = request.url.split('/')[-1]
  		return imgName
  	#将item传递给下一个管道类 
  	def item_completed(self, results, item, info):
  		return item
  ```

  ```python
  #指定图片存储的目录
  IMAGES_STORE = './img_bobo'
  #指定开启的管道
  ITEM_PIPELINES = {
     "myspider.pipelines.imgPipe": 300,
  }
  ```

#### 下载中间件



#### scrapy常见问题

- scrapy怎么确定数据是给调度器还是提交给管道?:**靠数据类型判断**

- 针对不同的网页解析,怎么定义不同的方法?:**调用不同的回调函数**,一般情况下**一类页面对应一个爬虫方法,分析页面中所有的有价值元素(url和持久化数据)进行yield**

```python
# ./spiders/myspider.py
class Myspider(scrapy.Spider):
	def parse(self, response):
        subpages = response.xpath(...@href).extract()
        for sub in subpages:
            yield scrapy.Request(sub,callback=self.subparse)#调用self.subparse
    def subparse(self,reponse):
```

#### Scrapy爬虫程序框架


```python
#./items.py
import scrapy
class MyspiderItem(scrapy.Item):
    filename = scrapy.Field()
    url = scrapy.Field()
```
```python
#./pipelines.py
class MyspiderPipeline:
	fp = None
	def open_spider(self,spider):
		print("开始爬虫......")
		self.fp = open('./images.txt','w+')
	def process_item(self, item, spider):
		self.fp.write(item["url"]+'\n')
		return item
	def close_spider(self,spider):
		print("结束爬虫")
		self.fp.close()
```
```python
# ./spiders/homepage.py
import scrapy
from myspider.items import MyspiderItem
class HomepageSpider(scrapy.Spider):
    name = "home"#爬虫程序名称,作为程序的"唯一标识"(scrapy crawl home)
    allowed_domains = ["aaa.com"]#过滤器根据这个进行过滤网址,可以为空
    start_urls = ["https://aaa.com","https://bbb.com"]#爬虫起始网址
    urlSet = set()#自定义一些哈希集合用来对网址进行去重
    #默认启动钩子函数
    def start_requests(self):
        for url in self.start_urls:
            yield scrapy.Request(url=url,callback=self.parse)
    #默认初始爬虫方法,一般用来爬取主页
    def parse(self,response):#response是从下载器接收到的响应对象
        nextpages = response.xpath('//span[@id="pe100_page_contentpage"]/div/a/@href').extract()
        for nextpage in nextpages:
            if nextpage not in self.urlSet:
                self.urlSet.add(nextpage)
                yield scrapy.Request(nextpage,callback=self.parse)
                print(nextpage)
        subpages = response.xpath('//div[@class="MidL_con"]/p/a/@href').extract()
        for sub in subpages:
            yield scrapy.Resuest(url=url,callback=self.subparse)
    #自定义爬虫方法,一般用来爬取更深层次的页面
    def subparse(self,response):
        images = response.xpath('//div[@class="MidL_con"]/p/a/@href').extract()
        for image in images:
            #构建item对象
            item = MyspiderItem()
            item["url"] = image
            item["filename"] = xxx
            yield item
```

```python
#./settings.py
ROBOTSTXT_OBEY = False
LOG_LEVEL = 'ERROR'
USER_AGENT = "Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/121.0.0.0 Safari/537.36"
ITEM_PIPELINES = {
   "myspider.pipelines.MyspiderPipeline": 300,
} 
```

## 下载中间件

#### UA&IP代理(发送请求)

#### selenium(处理请求)



