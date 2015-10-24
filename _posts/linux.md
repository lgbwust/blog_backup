title: ubuntu权限获取及vm tools的安装
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