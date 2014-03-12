---
layout: post
title: "Build My Personal Site with Jekyll"
tagline: "Build My Personal Site With Jekyll"
description: ""
tags: ["Jekyll"]
---
{% include JB/setup %}

Jekyll  /d3i:kel/

*   静态站点生成器，它会根据网页源码生成静态文件，它提供了模板、变量、插件等功能。
*   _site目录就是实际网站的内容
*   任何以下划线开头的文件和目录都不会成为网站的一部分, 如_layouts
*   任何不以下划线开头的文件和目录都会被复制到生成的网站中(默认为_site)
*   有YAML front matter的任何文件(不以下划线开头)将会先被Jekyll处理然后才被放进_site目录中
*   如果不以YFM开头，将会直接被放进_site目录

Install on Ubuntu

    sudo apt-get install ruby1.9.1 rubygems1.8 libyaml-ruby libopenssl-ruby
    sudo apt-get install python-pygments
    sudo gem install json
    sudo gem install rdoc
    sudo gem install jekyll

安装Jekyll bootstrap

    $ git clone https://github.com/plusjade/jekyll-bootstrap.git USERNAME.github.com
    $ cd USERNAME.github.com
    $ git remote set-url origin git@github.com:USERNAME/USERNAME.github.com.git
    $ git push origin master

本地预览Blog

    $ git clone https://github.com/plusjade/jekyll-bootstrap.git
    $ cd jekyll-bootstrap
    $ jekyll server

在浏览器中预览[http://localhost:4000]

更换[themes](http://themes.jekyllbootstrap.com/preview/hooligan/)

    rake theme:install git="https://github.com/dhulihan/hooligan.git"

Create a Post and Page
具体内容要看Rakefile文件，例如:

    rake post title="Hello Jekyll"
    rake page name="about.md" or "pages/about.md" or "pages/about"

Basic Usage

    $ jekyll build
    # => The current folder will be generated into ./_site

    $ jekyll build --destination <destination>
    # => The current folder will be generated into <destination>
    
    $ jekyll build --source <source> --destination <destination>
    # => The <source> folder will be generated into <destination>

    $ jekyll build --watch
    # => The current folder will be generated into ./_site,
    #    watched for changes, and regenerated automatically.
 
使用别人的模板建立自己的Blog

*   git clone 别人的模板
*   删除掉.git
*   修改_config.yml中的个人信息为自己的
*   修改_include/themes/default.html中的个人信息为自己的，也可以删除掉一些自己不需要的内容
*   git post tagline="hello world"
*   本地查看jekyll build, jekyll server

我使用模板的是(http://blog.evercoding.net/), [More](https://github.com/mojombo/jekyll/wiki/Sites)


Reference

*   [Markdown语法说明](http://wowubuntu.com/markdown/#overview)
*   [JekyllQuickStart](http://jekyllbootstrap.com/usage/jekyll-quick-start.html)
*   [Jekyll Introduction](http://jekyllbootstrap.com/lessons/jekyll-introduction.html)
*   [用Jekyll构建静态网站](http://yanping.me/cn/blog/2011/12/15/building-static-sites-with-jekyll)
