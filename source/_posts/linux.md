title: ubuntu常用命令及常见问题解决方案
date: 2015-10-23 20:34:24
tags: [linux,ubuntu]
categories: 学习
---
- ubuntu如何获取root权限：
```
sudo -i //输入密码即可切换到root权限
```
<!--more-->
- vmware 如何给ubuntu安装vmtools
 -  复制 vmwaretools.tar.gz并解压
 -  右键，属性，权限，文件夹访问的权即改为<font color=blue face=华文新魏>**`创建和删除文件`**</font>
 - 打开终端，cd到<font color=red>vmware-tools-distrib</font>，输入ls显示所有文件
 - 输入命令
 ```
 sudo ./vmware-install.pl
 ```
 - 接下来N多的enter，N多的YES，直到你看到---the vmware team就可关闭窗口，然后重新启动就可以焕然一新了。
 
- 当使用 `sudo apt-get install` 命令时，ubuntu提示“E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系”的解决办法
 - 从昨天晚上开始，开始尝试在ubuntu下编译caffe，先是安装了一大堆的库，一开始都非常顺利，虽然有一点小小的问题，但是很快解决了，但当我用
 `sudo apt-get install libboost-all-dev` 
 就出错了，提示“E: 无法修正错误，因为您要求某些软件包保持现状，就是它们破坏了软件包间的依赖关系”，从昨天晚上，到今天上午都没有解决，百度的一大堆办法，都没有很好的解决。
 下午看到了这样的一条解决方案：http://packages.ubuntu.com/wily/amd64/libboost-all-dev/download
 使用 aptitude 或者 synaptic 来安装，首先运行
 ```
 sudo apt-get install aptitude
 sudo aptitude install libboost-all-dev
 ```
 我记得昨天晚上好像也用过这条命令，但是没有成功，现在想想可能是因为（Y,n,q）这个输的有误吧。
 好了，就这么多！
 2015/10/29日于lab