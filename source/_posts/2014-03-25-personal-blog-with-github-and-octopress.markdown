---
layout: post
title: "基于Github托管的Octopress个人博客"
date: 2014-03-25 07:33:30 +0800
comments: true
categories: Octopress
---
   Github Pages 一直是自己喜爱的博客形式，之前托管了一个简单的英文博客，直接将博客的源代码放在username.github.io项目下，
代码管理简单，但是需要github在访问时将Jeklly转换为静态的网页，未免会带来网站性能的损失。Octopress 提供了更好的管理博客的方法，并集成了更多可以使用的插件和自动化的功能，这个中文博客就是基于Octopress，仍然托管在Github上，不过不是以个人网页的形式，而是以project pages的形式托管.

  在部署octopress的时候，会遇到不少问题，这里主要来记录一下，以备以后使用.

####fatal：remote origin already exists ####
我的配置环境为reby 1.9.3， 在根据Octopress的[官方文档](http://octopress.org/docs/setup/)安装主题之后，并在github上建立好新的repository之后，执行rake setup\_github_pages会出错，提示

	remote origin already exists
解决方法：将Octopress/Rakefile中357行

	git remote add origin #{repo_url} 改为 git remote set-url origin #{repo_url}

####修改相应的配置文件####
由于现在的中文博客是放在github的一个project上，目的是为了实现如下方式访问：username.github.io/project,所以我们需要修改一下Octopress的配置文件，来指明目录结构。

* _config.yml:修改url,root,destination
* config.rb:修改http_path,http_image_path,http_fonts_path,css_dir

具体的改变可以参考[这里](https://github.com/Charlesjean/cn)相对应的文件。

####UTF-8或者GB2312编码问题####
在执行rake generate命令之后，如果遇到 invalid byte sequence UTF-8 等错误，说明在你的文件当中有非法的编码字符，这些字符可能是由于我们使用的编辑器默认添加的一些字符，这里我们首先推荐使用nodepad++或者markdown专用的编辑器。

解决方法：

* 修改Ruby193\lib\ruby\gems\1.9.1\gems\jekyll-0.12.1\lib\jekyll\convertible.rb， 改变如下

{% codeblock %}
self.content = File.read(File.join(base, name),:encoding=>"utf-8")
{% endcodeblock %}

* 利用nodepad++找出非法字符，并将文档编码设置为utf-8 without BOM 

上面基本上列出了我在搭建过程中遇到的问题，喜欢用github搭建博客的都起码有一些编程能力，相信大家应该都可以很好地解决这些基本的问题，也欢迎大家反映自己遇到的问题，我们一起讨论解决。

再一次感谢Github提供了如此简单，可靠，关键还是免费的搭建博客的支持:)