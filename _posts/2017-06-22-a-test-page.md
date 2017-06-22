---
layout: post
title:  "a test page"
date:   2017-06-22
categories: testpage
tags: Python
author: 6xian
---

* content
{:toc}

这是一个测试页面。

跌跌撞撞，终于把这个站点搭建起来了。

无意中读到 「[Blogging Like a Hacker](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html)」 这篇文章，突然就冒出来一个想法，要搭建一个属于自己的博客站点。

知道了 jekyll 之后. 就去其官网学习 jekyll 的安装和构建站点，再学习如何利用 GitHub Page 部署站点，再去不断地找 jekyll 主题模板。同时，还找了本书来了解 HTML5 CSS3 JavaScript 相关知识，利用这些零碎的知识摸索着建一个个人博客站点。

非常感谢 [HyG](https://github.com/Gaohaoyang/gaohaoyang.github.io) 和 [mojombo](http://tom.preston-werner.com/) 两个作者，他们的 Jekyll 主题非常好，也是因为他们分享的源码，让我更快的掌握了一些 HTML 和 CSS 方面的知识，也让我建一个站点这个想法现在就实现了。

这也是一个学习的过程，我很希望也很乐意多学点。

## Python 代码

```python
# !/usr/bin/env python3
# -*- coding: utf-8 -*-

'''a test module'''

import sys

def test():
    args = sys.argv
    if len(args)==1:
        print('Hello, world!')
    elif len(args)==2:
        print('Hello, %s!' % args[1])
    else:
        print('Too many arguments!')

if __name__=='__main__':
    test()
```

## shell 字符 

```
d:\jekyllWorkspace\6xian.github.io (master)
λ jekyll serve --watch
Configuration file: d:/jekyllWorkspace/6xian.github.io/_config.yml
            Source: d:/jekyllWorkspace/6xian.github.io
       Destination: d:/jekyllWorkspace/6xian.github.io/_site
 Incremental build: disabled. Enable with --incremental
      Generating...
                    done in 3.462 seconds.
  Please add the following to your Gemfile to avoid polling for changes:
    gem 'wdm', '>= 0.1.0' if Gem.win_platform?
 Auto-regeneration: enabled for 'd:/jekyllWorkspace/6xian.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
      Regenerating: 1 file(s) changed at 2017-06-22 19:26:40 ...done in 3.725213 seconds.
      Regenerating: 1 file(s) changed at 2017-06-22 19:28:53 ...done in 3.276188 seconds.
      Regenerating: 1 file(s) changed at 2017-06-22 19:28:57 ...done in 3.836219 seconds.
```

