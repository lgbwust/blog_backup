title: hexo博客问题解决方案
date: 2015-10-24 21:20:40
tags: hexo
categories: 学习
---
#### hexo 博客一些常见问题及解决办法<持续更新中>


----------
在我的第一篇博客中曾说过我多次配过hexo博客都没有成功。每次都会遇到各种各样奇怪的问题。在这里我简单的作一个汇总。希望对大家有所帮助。<!--more-->

 - 先简单说一下hexo安装的方法。
  - 首先准备好[Github](github.com)、[Git](http://git-scm.com/)、[Node.js](https://nodejs.org)
  -  安装hexo，在任意位置右键
 ```
  git bash
  npm install -g hexo
  hexo init
  npm install
  hexo g
  hexo s
  ```
  完了之后，输入localhost:4000查看。
  - 生成 ssh keys
  ```
  ssh-keygen -t rsa -C "邮件地址@youremail.com"
  Generating public/private rsa key pair.
  Enter file in which to save the key      (/Users/your_user_directory/.ssh/id_rsa):<按回车键>
  Enter passphrase (empty for no passphrase):<输入加密串>
  Enter same passphrase again:<再次输入加密串>
  //注意密码是不会显示出来的
  ```
  - 测试
  ```
  ssh -T git@github.com
  ```
  - 配置用户名与密码
  ```
  git config --global user.name "xxx"//用户名
  git config --global user.email  "xxx@gmail.com"//填写自己的邮箱
  ```
  - clone theme
  ```
  git clone https://github.com/litten/hexo-theme-yilia.git themes/yilia
  ```
  - 配置_config.yml
  ```
  theme: yilia
  deploy:
  git: github
  repository: http://github.com/username/username.github.io.git
  branch: master
  ///注意：后面都有一个空格！！！
  ```
  - 上传到Github Pages
  ```
  hexo clean
  hexo g
  hexo d
  ```
  - 到此为止，已基本配置完毕，可以开始写博客了。输入即可
  `hexo n xxx.md`


----------

- 下面说一下过程中可能遇到的一些问题。
  - deploy的问题
   冒号后面没有加空格导致失败，它不会报任何错误。所以一定要记得加上，如果加了依然没有成功，本人就遇到了这个问题。通过网上搜索，大家基本都是成功的，也一些人出现这样的问题，但是没有说解决方案，这里我说一下我解决的方法，deploy内容这样改一下
   ```
   deploy:
   type: git
   repository: git@github.com:lgbwust/lgbwust.github.io.git
   branch: master
   ```
   OK，完美！
   - 多PC端的问题
   今天我尝试有自己的笔记上安装一下hexo，可以正常生成网页，输入`hexo s` 也可以正常，但是网页中输入localhost:4000，一直都没有任何反映，经查，可能由于4000端口被占用，需要这要改一下：
   ```
   .\hexo\node_modules\hexo-server\index.js--->port的数值改一下
   ```
   OK，完美！
   - SHH key的问题
   一切准备好，当我准备上传到github的时候，出现
   ```
   Error: Permission denied (publickey). fatal: Could not read from     remote repository.
   ```
   网上查找，都说是什么shh key没有配置，怕没弄好，我又重新配了一遍，输入`ssh -T git@github.com` 测试的是ok的啊，通过git将代码push到github也是OK的，但是为什么hexo就不行呢? 找了好久终于发现了问题。由于lab的电脑的git版本为2.6.0，而笔记本上版本为2.5.0 。于是我尝试更新一下git版本，结果成功了！
   OK，完美！！！

  
