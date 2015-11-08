title: 一些小工具
date: 2015-10-23 09:15:09
tags: [matlab,python]
categories: 学习
---
一些常用的脚本总结：


----------

 - 将多个txt文件合并为一个文件
   打开Windows的cmd窗口，输入以下命令
```cmd
copy *.txt NewName.txt
```
<!--more--> 

> 注意，合并后的txt文件，结尾带有一个sub字符串，需要手动删除，不然会出现意料不去的问题


----------


- 对图像进行分类移动到指定文件夹
 - matlab程序如下：
```matlab
% getAllFiles.m
function fileList = getAllFiles(dirName)
% 功能：指定文件路径，获取该路径所包含的所有文件
%       由于采用了嵌套操作，因此可以获取子文件夹的文件
dirData = dir(dirName);      %# Get the data for the current directory
dirIndex = [dirData.isdir];  %# Find the index for directories
fileList = {dirData(~dirIndex).name}';  %'# Get a list of the files
if ~isempty(fileList)
    fileList = cellfun(@(x) fullfile(dirName,x),...  %# Prepend path to files
        fileList,'UniformOutput',false);
end
subDirs = {dirData(dirIndex).name};  %# Get a list of the subdirectories
validIndex = ~ismember(subDirs,{'.','..'});  %# Find index of subdirectories
%#   that are not '.' or '..'
for iDir = find(validIndex)                  %# Loop over valid subdirectories
    nextDir = fullfile(dirName,subDirs{iDir});    %# Get the subdirectory path
    fileList = [fileList; getAllFiles(nextDir)];  %# Recursively call getAllFiles
end
end
```
```matlab
% classifyImage
dir_name = 'F:\迅雷下载\新建文件夹\test\full';

file_List = getAllFiles(dir_name);
if isempty(file_List);
    error('设定的文件夹内没有任何视频，请重新检查...')
end
save_name = '';
cd('F:\迅雷下载\新建文件夹\test\full');

 for i = 1 : length(file_List)
     %% 读取图片
     image_path = file_List{i};
     image = imread(image_path);
     [pathstr, name, ext] = fileparts(image_path); 
     clip_name = name(8 : 12);%取名字中的1到4位
     
     if(isempty(save_name))
         mkdir(strcat(pathstr, '\', clip_name));
         save_name = clip_name;
     end
     if(strcmp(save_name, clip_name))
         file_name = strcat(name, ext);
 		movefile(file_name, clip_name);
     end
     if(strcmp(save_name, clip_name) == 0)
          mkdir(strcat(pathstr, '\', clip_name));
          file_name = strcat(name, ext);
 		 movefile(file_name, clip_name);
          save_name = clip_name;
     end
 end
```


----------


 - matlab提取图像特征
    ```
    path_imgDB = './database/';
    addpath(path_imgDB);
    imgFiles = dir(path_imgDB);
    imgNamList = {imgFiles(~[imgFiles.isdir]).name}; %get the namelist of the pics
    clear imgFiles;
    imgNamList = imgNamList';

     numImg = length(imgNamList); %the num of the pics
     feat = [];  %the feature database
     %%%%%
    for i = 1:numImg
       oriImg = imread(imgNamList{i, 1}); 
       im_ = single(oriImg) ; % note: 255 range
       featVec=vl_hog(im_, cellSize, 'verbose') ;
       featVec = featVec(:);
       feat = [feat; featVec'];
       fprintf('extract %d image\n\n', i);
     end
    ```


----------

- 最后贴一个spider,方便获取图片
 - 新建一个a.conf文件，内容如下
 ```
 [spider]
 url_list_file: ./urls
 output_directory: ./output
 max_depth: 1
 crawl_interval: 1
 crawl_timeout: 1
 target_url: .*.(gif|png|jpg|bmp)$
 thread_count: 3

 ```
  - 建一个urls文件，里面是你要爬取图片的网址，如下所示
  ```
  http://www.sina.com.cn/
  http://www.baidu.com
  http://www.bing.com
  ```
  - fetcher.py文件

```
import urllib
import socket
import re
import logging as log
import MyHtmlParser
import chardet
from threading import Timer
import urllib2
import StringIO
import gzip

class Fetcher:
    """
    Implement fetching functions of single thread
    """
    def __init__(self, url, output, timeout):
        self.url = url
        self.output_dir = output
        self.timeout = timeout

    def check_url(self, url):
        """
        check whether a url is valid
        :param url: given page url
        :return: True(valid) / False(invalid)
        """
        url_format = '(http|ftp|https):\/\/[\w\-_]+(\.[\w\-_]+)+([\w\-\.,@?^=%&amp;:/~\+#]*[\w\-\@?^=%&amp;/~\+#])?'
        url_pattern = re.compile(url_format)
        if url_pattern.match(url):
            return True
        return False

    def enc_dec(self, content):
        """
        decode the given content into utf-8
        :param content: the given content
        :return: the decoded content in utf-8
        """
        try:
            encoding_dict = chardet.detect(content)
            web_encoding = encoding_dict['encoding']
            if web_encoding is not None:
                if web_encoding.lower() == 'utf-8':
                    return content
                else:
                    return content.decode('web_encoding', 'ignore').encode('utf-8')
            else:
                return content
        except Exception as e:
            log.warn('encode detection error %s' % e)
            return content

    def check_url(self, url):
        """
        check whether a url is valid
        :param url: given page url
        :return: True(valid) / False(invalid)
        """
        url_format = '(http|ftp|https):\/\/[\w\-_]+(\.[\w\-_]+)+([\w\-\.,@?^=%&amp;:/~\+#]*[\w\-\@?^=%&amp;/~\+#])?'
        url_pattern = re.compile(url_format)
        if url_pattern.match(url):
            return True
        return False

    def read_content(self, thread_id):
        """
        get the content of a page
        :return: content (string)
        """
        log.info(str(thread_id) + " - getting url: %s" % self.url)
        content = ''

        try:
            response = urllib2.urlopen(self.url, timeout = self.timeout)
            if response.info().get('Content-Encoding',"") == 'gzip':  #e.g www.sina.com.cn
                buf = StringIO.StringIO(response.read())
                f = gzip.GzipFile(fileobj=buf)
                content = f.read()
            else:
                content = response.read()
            content = self.enc_dec(content)
            return content
        except socket.timeout:
            log.warn("Timeout in fetching %s" % self.url)
        except urllib2.HTTPError as e:
            log.warn("error in fetching content of %s: %s" % (self.url, e))
            return content
        except urllib2.URLError as e:
            log.warn("error in fetching content of %s: %s" % (self.url, e))
            return content
        except socket.gaierror as e:
            log.warn("error in fetching content of %s: %s" % (self.url, e))
            return content

    def get_sub_urls(self, content):
        """
        return the sub-page urls given content of current page
        :param content: content of current page
        :return: a list containing all sub-page urls
        """

        if not self.check_url(self.url):
            return []
        sub_urls = []
        html_parser = MyHtmlParser.MyHTMLParser()
        if content is not None:
            try:
                html_parser.feed(content)
                for item in html_parser.get_urls():
                    if self.check_url(item):
                        sub_urls.append(item)
            except Exception as e:
                log.warn("Error in getting subpage of %s: %s" % (self.url, e))
                return sub_urls

            return sub_urls
```

  

 - MyHtmlParser.py
```


import urllib2
from HTMLParser import HTMLParser
import urlparse

class MyHTMLParser(HTMLParser):
    """
    encapsulate HTMLParser, for fetching specific content
    """
    def __init__(self):
        HTMLParser.__init__(self)
        self.links = []
        self.figures = []
        self.href = 0

    def append_link(self, url):
        if url.startswith('http://'):
            self.links.append(url)

    def handle_starttag(self, tag, attrs):
        if tag == 'a' or tag == 'link':
            for name, value in attrs:
                if name == "href":
                    self.append_link(value)
        if tag == 'img' or tag == 'script':
            for name, value in attrs:
                if name == "src":
                    self.append_link(value)

    def get_urls(self):
        """
        :return: links of sub-pages
        """
        return self.links

    def get_figures(self):
        """
        :return: figures src current page
        """

```
 - spider.py
```
from __future__ import with_statement
from ConfigParser import ConfigParser
import os
import logging
import Queue
import fetcher
import logging as log
import hashlib
import time
import threading
import re
import urllib
import MyHtmlParser

class MiniSpider(object):
    """
    MiniSpider: the outmost class that crawl pages
    """
    def __init__(self, configure):
        self._url_list = configure.get("spider", "url_list_file")
        self._output_dir = configure.get("spider", "output_directory")
        self._max_depth = int(configure.get("spider", "max_depth"))
        self._interval = int(configure.get("spider", "crawl_interval"))
        self._timeout = int(configure.get("spider", "crawl_timeout"))
        self.fetch_pattern = configure.get("spider", "target_url")
        self.fig_pattern = re.compile(self.fetch_pattern)
        self._thread_count = int(configure.get("spider", "thread_count"))
        self._url_queue = Queue.Queue()
        self._url_visited = [] # need to be synchronized among threads
        self._lock = threading.Lock()
        self.init_urls()
        if not os.path.exists(self._output_dir):
            os.makedirs(self._output_dir)

        self._thread_list = []

    def init_urls(self):
        """
        Initialize the _url_queue and _url_visited
        :return:None
        """
        init_depth = 1
        with open(self._url_list) as url_file:
            for line in url_file:
                line = line.strip()
                try:
                    if line not in self._url_visited:
                        self._url_queue.put([line, init_depth], timeout=1)
                        self._url_visited.append(line)
                except Queue.Full as e:
                    log.warn(e)
                    pass

    def fetch(self, thread_id):
        """
        Fetch according to BFS with _url_queue
        :return: None
        """
        #log.info("Running thread %s" % threading.current_thread())
        #print "Running thread", threading.current_thread()
        log.info("Running thread %s" % thread_id)
        while True:
            time.sleep(self._interval)
            try:
                cur_url, cur_depth = self._url_queue.get(timeout=1)
                cur_url = cur_url.strip()
            except Queue.Empty as e:
                log.warn(e)
                continue

            # check whether this url is a figure
            if self.fig_pattern.match(cur_url):
                log.info(str(thread_id) + " - downloading: %s" % cur_url)
                fig_suffix = cur_url[cur_url.rindex("."):]
                file_name = "%s%s" % (hashlib.md5(cur_url).hexdigest(), fig_suffix)
                file_path = os.path.join(self._output_dir, file_name)
                urllib.urlretrieve(cur_url, file_path)
            elif cur_depth <= self._max_depth:
                # get content of this page
                fetch_tool = fetcher.Fetcher(cur_url,
                                             self._output_dir,
                                             self._timeout)
                content = fetch_tool.read_content(thread_id)
                if content is None or len(content) == 0:
                    continue

                # get sub-page urls
                sub_urls = fetch_tool.get_sub_urls(content)
                if sub_urls is None:
                    continue
                for item in sub_urls:
                    with self._lock:  # lock _url_visited, check
                        if item in self._url_visited:
                            continue
                    try:
                        with self._lock:  # lock _url_visited, add
                            self._url_visited.append(item)
                        self._url_queue.put([item, cur_depth + 1], timeout=1)
                    except Queue.Full as e:
                        log.warn(e)
                        break

            self._url_queue.task_done()

    def multi_thread(self):
        """
        Fetch pages by multi-thread
        :return:
        """
        for i in xrange(self._thread_count):
            single_thread = threading.Thread(target = self.fetch, args = (str(i)))
            self._thread_list.append(single_thread)
            single_thread.setDaemon(True)
            single_thread.start()

        # wait for all tasks until finished
        self._url_queue.join()

        ## stop all spiders
        #for t in self._thread_list:
        #    t.join(self._timeout)
        log.info("mini spider finished")

if __name__ == "__main__":
    config = ConfigParser()
    #config.readfp(sys.argv[1])
    config.readfp(open("a.conf", 'r'))

    # initialize logging
    LOG_LEVEL = logging.DEBUG
    logging.basicConfig(level=LOG_LEVEL,
            format="%(levelname)s:%(name)s:%(funcName)s->%(message)s",  #logging.BASIC_FORMAT,
            datefmt='%a, %d %b %Y %H:%M:%S', filename='spider.log', filemode='a')

    spider = MiniSpider(config)
    spider.multi_thread()
```
 - test.py
```
import unittest
import fetcher
import urllib2
import StringIO
import gzip


class TestFetcher(unittest.TestCase):
    """
    Unittest of Fetcher
    """
    def setUp(self):
        self.timeout = 1
        self.fetch_tool = fetcher.Fetcher("http://www.baidu.com", 'output', timeout=self.timeout)

    def test_check_url(self):
        url = "http:/www.baidu.com"
        self.assertEquals(self.fetch_tool.check_url(url), False)

        url = "http://ftp.baidu.com"
        self.assertEquals(self.fetch_tool.check_url(url), True)

        url = "http://https.baidu.com"
        self.assertEquals(self.fetch_tool.check_url(url), True)

    def test_enc_dec(self):
        url = "http://www.baidu.com"
        response = urllib2.urlopen(url, timeout = self.timeout)
        urllib2.urlopen
        if response.info().get('Content-Encoding',"") == 'gzip':  #e.g www.sina.com.cn
            buf = StringIO.StringIO(response.read())
            f = gzip.GzipFile(fileobj=buf)
            content = f.read()
        else:
            content = response.read()
        content_len = len(self.fetch_tool.enc_dec(content))
        self.assertNotEqual(content_len, 0)

if __name__ == '__main__':
    unittest.main()
```
好了，下次再加吧。

- cifar10图片提取
自己实验中要用到cifar10的图片，官网给出的mat文件，从中可以很方便提取出来图片文件，图片的命名方式为label+batch_no. ,代码如下：
```
load('test_batch.mat')
for i=1:10000
    label=labels(i,:);
    temp=data(i,:);
    temp=reshape(temp,32,32,3);
    switch label
        case 0
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 1
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 2
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 3
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 4
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 5
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 6
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 7
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 8
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
        case 9
            imwrite(temp,[num2str(label),'_batch','_',num2str(i),'.jpg']);
    end
end
```
