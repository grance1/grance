######crawler.py：  SpiderMain作为爬虫的总调用程序，顾名思义是程序入口。
首先创建入口url，命令如下：
root_url = 'http://baike.baidu.com/view/21087.htm'

#####启动爬虫的命令：
obj_spider.craw(root_url)

########SpiderMain函数中会用到url管理器，HTML下载器，HTML解析器以及htmloutput输出器模块
用到构造函数将其初始化，代码如下：
 def __init__(self):
      self.urls = UrlManger()
      self.downloader = HtmlDownloader()
      self.parser = HtmlParser()
      self.outputer = HtmlOutputer()

########craw方法是爬虫的调用程序，将入口url添加到url管理器，代码如下：
 def craw(self, root_url):
     count = 1
     self.urls.add_new_url(root_url)
    while self.urls.has_new_url():#如果有url的时候
       try:
       new_url = self.urls.get_new_url()
       print 'craw %d : %s' % (count, new_url)
       html_cont = self.downloader.download(new_url)#下载对应的数据
       new_urls, new_data = self.parser.parse(new_url, html_cont)#解析器
       self.urls.add_new_urls(new_urls)
       self.outputer.collect_data(new_data)#收集数据
    if count == 1000:#爬虫爬1000条
       break

       count += 1

       except:
       print 'craw failed'

############数据输出为html对象，代码如下：
 self.outputer.output_html()#输出数据
 self.outputer.mysql_data()

#########UrlManger作为url管理器
用到了四个方法分别为add_new_url，add_new_urls， has_new_url，get_new_url
add_new_url代码如下：#向管理器中添加一个url
 if url is None:
      return
if url not in self.new_urls and url not in self.old_urls:
      self.new_urls.add(url)

########add_new_urls代码如下：#向管理器中添加批量的url
   if urls is None or len(urls) == 0:
       return
   for url in urls:
       self.add_new_url(url)

#######has_new_url代码如下：#判断管理器中是否有新的待爬去的url
 return len(self.new_urls) != 0

#######get_new_url代码如下：#从管理器中获取待爬去的url
 new_url = self.new_urls.pop()
      self.old_urls.add(new_url)
      return new_url

#######HtmlDownloader作为HTML下载器
 def download(self, url):
    if url is None:
       return None
    response = urllib2.urlopen(url)
    if response.getcode() != 200:
      return None
     return response.read()

######HtmlParser作为HTML解析器
 def get_new_urls(self, soup):
     new_urls = set()
     links = soup.find_all('a', href=re.compile(r'/view/\d+\.htm'))#主要用到正则表达式
     for link in links:
      new_url = link['href']
     page_url = 'http://baike.baidu.com'
     new_full_url = page_url + new_url
     new_urls.add(new_full_url)
     return new_urls

######## HtmlOutputer作为将其爬出来的数据在页面上显示出来，可以运行output.html即可在浏览器中查询，也可以直接打开文件，看到相关的数据             
 def mysql_data(self):#和数据库的连接
      con = MySQLdb.connect(host='127.0.0.1', user='root', passwd='123', db='test')#数据库是test，表是test2
      cur = con.cursor()
     for d in self.datas:
       print 'insert'
       cur.execute("INSERT INTO test2(url,title,summary) VALUES (%s, %s, %s)",
      (d['url'].encode('utf-8'), d['summary'].encode('utf-8'), d['title'].encode('utf-8')))
       print 'succed'
      cur.close()
      con.commit()
      con.close()


