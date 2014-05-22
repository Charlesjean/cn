---
layout: post
title: "解决Android SDK Manager无法下载SDK的问题"
date: 2014-05-20 22:31:33 +0800
comments: true
categories: Android
---
计算机应用可以说是少数国内与国外技术差别比较小的领域，主要原因是网络的帮助，使我们很方便的看到国外的技术文档与很多优秀的文章，但是很多时候国内的环境确实令人非常气愤，居然Android SDK网站都会被墙，以至于Android SDK Manager根本无法下载。解决方法是安装并打开goagent，并将SDK Manager的Proxy设置为`127.0.0.1:8087`。如果不知道goagent的话，那就好好搜索一下吧，国内的环境，基本上这个是必备的了。
