---
layout: post
title: github pages page build failed
categories: [github]
description: github pages page build failed
keywords: github
---

## 缘由
最近用github page 构建的blog 发现构建不成功，github构建失败得到的邮件只有一句

```shell
The page build failed with the following error: page build failed
```

参考

```shell
http://blog.xiayf.cn/2013/01/08/fix-github-pages-builds-failed/
```

## trouble shouting

本地配置 jekyll 环境

```shell
下载 ruby
https://rubyinstaller.org/downloads/
下载类似  Ruby+Devkit 2.6.5-1 (x64) 的

安装后 提示安装 msys2 toolchain 选择 3回车即可

安装完成后  安装 bundler
gem install bundler

在blog的根目录执行
bundle install
bundle update
bundle exec jekyll serve

在最好一个命令执行的时候会报响应的错误

 Incremental build: disabled. Enable with --incremental
      Generating...
       Jekyll Feed: Generating feed for posts
  Conversion error: Jekyll::Converters::Markdown encountered an error while converting '_posts/2018-07-21-TestNG-SpringBootTest.md':
                    Rouge::Guesser::Ambiguous
jekyll 3.8.5 | Error:  Ambiguous guess: can't decide between ["xml", "mason"]

```

![依赖注入方式](/images/post/201910/2.png)

其实此时就定位到了哪个文件存在的问题了 
我这里是代码块没有标记什么语言造成的