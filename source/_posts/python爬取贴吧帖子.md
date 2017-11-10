---
title: python爬取贴吧帖子
date: 2017-10-20 15:03:39
tags: [技术分享,python]
---
#### 一、介绍
> 我们常遇到一些很长的贴吧连载帖子想存到本地再看
此文就是运用python爬取指定百度贴吧的帖子并存到本地满足需求
环境:python2.7
目标网页:[【长篇连载】剑网3的正史和野史——从头开始讲剧情故事](https://tieba.baidu.com/p/2196794546)
源码存放:[源码github](https://github.com/gtntangll/python-webcrawler/tree/master/%E7%99%BE%E5%BA%A6%E8%B4%B4%E5%90%A7%E5%B8%96%E5%AD%90)
本文参考:[静觅](http://cuiqingcai.com/)博客python实战系列


------
#### 二、页面的抓取
>目标网页网址为https://tieba.baidu.com/p/2196794546
满足可以选择是否只看楼主的抓取我们点一下 **只看楼主** 并点一下后页的链接来获取只看楼主和翻页的参数
![](http://upload-images.jianshu.io/upload_images/2660278-4276c194fc060ba0?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
这时候可以看到只看楼主多出的参数 ***see_lz*** 和当前页的参数 ***pn***
![](http://upload-images.jianshu.io/upload_images/2660278-a60efb75075a8526.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

接下来定义一个DEMO类开始获取整个网页
```python
# -*- coding:utf-8 -*-
import urllib
import urllib2
import re

class DEMO:

    def __init__(self,baseUrl):
        self.baseURL = baseUrl
 
    def getPage(self):
        url = baseURL
        request = urllib2.Request(url)
        response = urllib2.urlopen(request)
        print response.read()

baseURL = 'https://tieba.baidu.com/p/2196794546?see_lz=1&pn=3'
demo = DEMO(baseURL)
demo.getPage()
```
运行结果:
> ![](http://upload-images.jianshu.io/upload_images/2660278-c117103345a21c02.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

爬取网页代码成功后我们完善一下代码,将只看楼主和页码提为参数

```python

def __init__(self,baseUrl,seeLZ):
        
        self.baseURL = baseUrl
        self.seeLZ = '?see_lz=' +str(seeLZ)
        self.tool = Tool()

    def getPage(self,pageNum):
        
        try:
            url = self.baseURL+self.seeLZ + '&pn=' + str(pageNum)
            request = urllib2.Request(url)
            response = urllib2.urlopen(request)
            return response.read().decode('utf-8')
        except urllib2.URLError, e:
            if hasattr(e,"reson"):
                print u"链接失败,原因:",e.reason
                return None
```
#### 三、提取出想要的信息
> 打开目标网页审查元素（F12）
首先找到帖子标题的代码段:
![](http://upload-images.jianshu.io/upload_images/2660278-9e676b34298adde7.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

```html
<h3 class="core_title_txt pull-left text-overflow  " title="【长篇连载】剑网3的正史和野史——从头开始讲剧情故事" style="width: 416px">【长篇连载】剑网3的正史和野史——从头开始讲剧情故事</h3>
```
我们将提取此h3里的文本则正则表达式为:
```python
<h3 class="core_title_txt.*?>(.*?)</h3>
```
接下来我们就可以写一个提取标题的方法:
```python
def getTitle(self,title):
        
        page = self.getPage(1)
        pattern = re.compile('<h3 class="core_title_txt.*?>(.*?)</h3>',re.S)
        result = re.search(pattern,page)
        if result:
            return result.group(1).strip()
        else:
            return None
```
同理找到帖子页数的代码段,可得出提取页数的正则:
```python
<li class="l_reply_num".*?</span>.*?<span.*?>(.*?)</span>
```
提取页数的方法:
```python
def getPageNum(self,page):
        
        page = self.getPage(1)
        pattern = re.compile('<li class="l_reply_num".*?</span>.*?<span.*?>(.*?)</span>',re.S)
        result = re.search(pattern,page)
        if result:
            return result.group(1).strip()
        else:
            return None
```
接下来是我们主要目的,提取正文,正则为:
```python
<div id="post_content_.*?>(.*?)</div>
```
这个正则提取出来的只是div里面的文本内容,它还会包括了图片标签，换行符，段落符等的标签。所以我们还需处理一下。
这时候添加一个类利用正则将这些标签都替换掉
```python
class Tool:
    removeImg = re.compile('<img.*?>| {7}|')
    removeAddr = re.compile('<a.*?>|</a>')
    replaceLine = re.compile('<tr>|<div>|</div>|</p>')
    replaceTD = re.compile('<td>')
    replacePara = re.compile('<p.*?>')
    replaceBR = re.compile('<p.*?>')
    replaceBR = re.compile('<br><br>|<br>')
    removeExtraTag = re.compile('<.*?')
    def replace(self,x):
        x = re.sub(self.removeImg,"",x)
        x = re.sub(self.removeAddr,"",x)
        x = re.sub(self.replaceLine,"\n",x)
        x = re.sub(self.replaceTD,"\t",x)
        x = re.sub(self.replacePara,"\n     ",x)
        x = re.sub(self.replaceBR,"\n",x)
        x = re.sub(self.removeExtraTag,"",x)
        
        return x.strip()
```
初始化类后,我们可以写提取正文的方法:
```python
def getContent(self,page):
        
        pattern = re.compile('<div id="post_content_.*?>(.*?)</div>',re.S)
        items = re.findall(pattern,page)
        
        contents = []
        for item in items:
            content = "\n" + self.tool.replace(item) + '\n'
            contents.append(content.encode('utf-8'))
        return contents
```
#### 四、写入文件,保存为txt
```python
def setFileTitle(self,title):
        
        if title is not None:
            self.file = open(title + ".txt","w+")    
        else:
            self.file = open(self.defaultTitle + ".txt","w+")
        
    def writeData(self,contents):
        
        for item in contents:
            self.file.write(item) 
```
#### 五、完善代码
有了以上的方法作为基础，我们来写运行的方法:
```python
def start(self):
        indexPage = self.getPage(1)
        pageNum = self.getPageNum(indexPage)
        title = self.getTitle(indexPage)
        self.setFileTitle(title)
        if pageNum == None:
            print "URL已失效,请重试"
            return None
        try:
            print "该帖子共有" + str(pageNum) + "页"
            for i in range(1,int(pageNum)+1):
                print "正在写入第" + str(i) + "页数据"
                page = self.getPage(i)
                contents = self.getContent(page)
                self.writeData(contents)
        
        except IOError,e:
            print "写入异常,原因" + e.message

        finally:
            print "写入任务完成"
```

此时完成的代码:
```python
# -*- coding:utf-8 -*-
import urllib
import urllib2
import re

class Tool:
    removeImg = re.compile('<img.*?>| {7}|')
    removeAddr = re.compile('<a.*?>|</a>')
    replaceLine = re.compile('<tr>|<div>|</div>|</p>')
    replaceTD = re.compile('<td>')
    replacePara = re.compile('<p.*?>')
    replaceBR = re.compile('<p.*?>')
    replaceBR = re.compile('<br><br>|<br>')
    removeExtraTag = re.compile('<.*?')
    def replace(self,x):
        x = re.sub(self.removeImg,"",x)
        x = re.sub(self.removeAddr,"",x)
        x = re.sub(self.replaceLine,"\n",x)
        x = re.sub(self.replaceTD,"\t",x)
        x = re.sub(self.replacePara,"\n     ",x)
        x = re.sub(self.replaceBR,"\n",x)
        x = re.sub(self.removeExtraTag,"",x)
        
        return x.strip()

class DEMO:
    
    def __init__(self,baseUrl,seeLZ):
        
        self.baseURL = baseUrl
        self.seeLZ = '?see_lz=' +str(seeLZ)
        self.tool = Tool()

    def getPage(self,pageNum):
        
        try:
            url = self.baseURL+self.seeLZ + '&pn=' + str(pageNum)
            request = urllib2.Request(url)
            response = urllib2.urlopen(request)
            return response.read().decode('utf-8')
        except urllib2.URLError, e:
            if hasattr(e,"reson"):
                print u"链接失败,原因:",e.reason
                return None

    def getTitle(self,title):
        
        page = self.getPage(1)
        pattern = re.compile('<h3 class="core_title_txt.*?>(.*?)</h3>',re.S)
        result = re.search(pattern,page)
        if result:
            return result.group(1).strip()
        else:
            return None

    def getPageNum(self,page):
        
        page = self.getPage(1)
        pattern = re.compile('<li class="l_reply_num".*?</span>.*?<span.*?>(.*?)</span>',re.S)
        result = re.search(pattern,page)
        if result:
            return result.group(1).strip()
        else:
            return None

    def getContent(self,page):
        
        pattern = re.compile('<div id="post_content_.*?>(.*?)</div>',re.S)
        items = re.findall(pattern,page)
        
        contents = []
        for item in items:
            content = "\n" + self.tool.replace(item) + '\n'
            contents.append(content.encode('utf-8'))
        return contents

    def setFileTitle(self,title):
        
        if title is not None:
            self.file = open(title + ".txt","w+")    
        else:
            self.file = open(self.defaultTitle + ".txt","w+")
        
    def writeData(self,contents):
        
        for item in contents:
            self.file.write(item) 

    def start(self):
        indexPage = self.getPage(1)
        pageNum = self.getPageNum(indexPage)
        title = self.getTitle(indexPage)
        self.setFileTitle(title)
        if pageNum == None:
            print "URL已失效,请重试"
            return None
        try:
            print "该帖子共有" + str(pageNum) + "页"
            for i in range(1,int(pageNum)+1):
                print "正在写入第" + str(i) + "页数据"
                page = self.getPage(i)
                contents = self.getContent(page)
                self.writeData(contents)
        
        except IOError,e:
            print "写入异常,原因" + e.message

        finally:
            print "写入任务完成"

print u"请输入帖子代号"
baseURL = 'http://tieba.baidu.com/p/' + str(raw_input(u'http://tieba.baidu.com/p/'))
seeLZ = raw_input("是否只获取楼主发言,是输入1,否输入0\n")
demo = DEMO(baseURL,seeLZ)
demo.start()
```

好我们来运行一下试试:
![](http://upload-images.jianshu.io/upload_images/2660278-b5266d9072abe99a.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
到这里输入目标网页的代号:
![](http://upload-images.jianshu.io/upload_images/2660278-b634bf06d3e62f5c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
整个运行过程:
![](http://upload-images.jianshu.io/upload_images/2660278-6e4d5baf2831bfa5.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)
好,这时候该目录下多了这个txt文件,打开看看
![](http://upload-images.jianshu.io/upload_images/2660278-4d97bffcd6ef45ec.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)


文本已经成功存到本地,到这里就成功了。
