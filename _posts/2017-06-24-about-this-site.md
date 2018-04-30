---
layout: post
title:  "搭建本站的一些知识 GitHub Pages 和 jekyll"
date:   2017-06-24
categories: Web
tags: Blog GitHub jekyll
author: 6xian
---

* content
{:toc}

## 1 说明

这是一个关于本站的测试页面，也算我的博客的第一篇文章，简单记录下自己是怎样学习搭建这个站点的。

无意中读到 「[Blogging Like a Hacker](http://tom.preston-werner.com/2008/11/17/blogging-like-a-hacker.html)」 这篇文章，突然就冒出来一个想法，要搭建一个属于自己的博客站点。因为没有时间也没有必要去折腾自己的域名和空间，也不愿再去申请各类免费博客站点的账号，所以就选择了上文中作者提到的 GitHub Pages 和 jekyll 来搭建博客站点，这个组合非常优雅，自己只要负责写文章，并一键同步到 GitHub 远程仓库，然后就可以在世界上的任何地方阅读文章了。

## 2 学习 GitHub Pages 和 jekyll

GitHub Pages 是托管站点的仓库，jekyll 是生成静态站点的框架，并且 GitHub 对 jekyll 做了深度整合，就是说，只要把 Blog 文档(文本格式，如 Markdown)放在 GitHub 的仓库 (username.github.io) 上，GitHub 会借助 jekyll 生成 HTML 页面，并自动部署网站。当然，我们也可以先通过自己本地安装的 jekyll 生成离线的站点，再通过 Git 上传到 GitHub 上。这种方式可能更好，因为可以对自己的文章预览修改，满意后再放到 GitHub 上去。

网络上介绍 GitHub Pages 和 jekyll 的文章非常多，我也是从知道这种搭建站点的思路后，一点一点学，从没有任何经验到成功把站点建起来，虽然跌跌撞撞，了解的知识也是零零碎碎，但终归也是一个学习的过程，我很希望也很乐意多学点。学习的过程中主要参考了 Jekyll 官网的文档，还参考了以下网络上的文章。

* 「[搭建一个免费的，无限流量的Blog----github Pages和Jekyll入门](http://www.ruanyifeng.com/blog/2012/08/blogging_with_jekyll.html)」  阮一峰
* 「[Using Jekyll as a static site generator with GitHub Pages](https://help.github.com/articles/using-jekyll-as-a-static-site-generator-with-github-pages/)」 和 「[Setting up your GitHub Pages site locally with Jekyll](https://help.github.com/articles/setting-up-your-github-pages-site-locally-with-jekyll/)」  GitHub 上的 help
* 「[HyG](https://github.com/Gaohaoyang/gaohaoyang.github.io)」 本站点主题的作者，我在他的主题上做了些修改
*  「[mojombo](http://tom.preston-werner.com/)」  「Blogging Like a Hacker」 文章的作者，非常喜欢他 Blog 的样式

有了这些资源的帮助，还要知道些 Web 前端的知识，为此我花了些时间来了解 HTML, CSS, JavaScript, 并在修改本站点主题的过程中，借助 「[w3schools.com](https://www.w3schools.com)」 加深对 Web 知识的理解。要说的是，非常感谢 [HyG](https://github.com/Gaohaoyang/gaohaoyang.github.io) 和 [mojombo](http://tom.preston-werner.com/) 两个作者，他们的 Jekyll 主题非常好，因为他们分享的源码，让我更快的掌握了一些 HTML 和 CSS 方面的知识，也让我建一个站点这个想法现在就实现了。

## 3 搭建站点的步骤和发布文章

这里简单记录下搭建站点的步骤，以及我平常写文章和发布文章的一些常用操作，目的就是把自己实践过的事情总结下，下次再做类似事情的时候，不用全盘去 google 了。至于具体的细节就不再赘述，知道要做什么，怎么做就相对好办了。

我平常都是在 Win7 下工作学习和娱乐，所以列举的步骤可能更适合 Windows 系统，但 Linux 和 Mac 系统应该会更方便，毕竟像 jekyll 所描述的 *「Jekyll is not officially supported for Windows.」* 能用 MacBook pro 工作的人是很幸福的。

### 3.1 搭建站点的步骤

首先安装运行环境，GitHub 需要 Git, jekyll 需要 Ruby. 
* 安装 Git [https://git-scm.com/download/win](https://git-scm.com/download/win), 新建本地仓库，熟悉 `git init`, `git add`, `git commit`,`git checkout`, `git rm`, `git branch`, `git merge` 等命令。
* 注册 GitHub 账号，设置用户名和邮箱，创建 SSH key. 
  * 试着在 GitHub 上建一个仓库，完成本地项目到远程仓库的关联，或者克隆远程仓库。  
    `git clone https://github.com/6xian/6xian.github.io.git 6xian.github.io`   
    `git remote add origin git@github.com:6xian/6xian.github.io.git`  
    `git push -u origin master`  
  * 熟悉 `git pull origin master`, `git add --all`, `git commit -m 'something'`, `git push origin master` 等操作。
* 安装 Ruby [https://rubyinstaller.org/downloads/](https://rubyinstaller.org/downloads/)，以及 rubygems, Bundler. 用来安装，管理 Ruby 程序包。
* 安装 jekyll. `gem install jekyll` 安装完后，运行 `jekyll -v`,  如果有些错误，可能的原因是很多包没有装上。根据出错的信息把各种包补上，我共安装了以下几个包。
    - `$ gem install minima`
    - `$ gem install  jekyll-sitemap`
    - `$ gem install  pygments.rb`
    - `$ gem install jekyll-feed`
    - `$ gem install tzinfo-data x64-mingw32`  

  之后运行 `$ jekyll -v`, 查看 jekyll 版本。截止到现在，最新的版本应该是 jekyll 3.5.0.

运行环境安装完后，就可以搭建站点了，一种方式是先在本地用 jekyll 生成站点，修改预览本地站点，然后用 Git 初始化，关联到远程 GitHub 的 username.github.io 仓库。一种方式是先在 GitHub 新建 username.github.io 远程仓库，然后 clone 到本地，在本地按照 jekyll 的目录结构修改配置和页面，然后再用 Git push 到远程仓库。

第一种方式，先用 jekyll 生成本地站点。
```
$ jekyll new 6xian.github.io  // 生成站点，jekyll 会按照默认框架生成站点所需的目录和文件
  ......
New jekyll site installed in D:/jekyllWorkspace/6xian.github.io.

$ cd 6xian.github.io

$ bundle exec jekyll serve  // 或者 (jekyll serve) 启动服务，之后就可以在本地访问站点了。
Configuration file: D:/jekyllWorkspace/6xian.github.io/_config.yml
            Source: D:/jekyllWorkspace/6xian.github.io
       Destination: D:/jekyllWorkspace/6xian.github.io/_site
       ...
 Auto-regeneration: enabled for 'D:/jekyllWorkspace/6xian.github.io'
    Server address: http://127.0.0.1:4000/
  Server running... press ctrl-c to stop.
```

本地站点生成后，通过访问 [http://127.0.0.1:4000/](http://127.0.0.1:4000/) 进行预览，修改再预览，之后可以关联推送自己的项目到 GitHub 远程仓库。完成之后就可以通过 [https://6xian.github.io/](https://6xian.github.io/) 访问自己的博客了。
```
$ cd 6xian.github.io
$ git init 
$ git remote add origin git@github.com:6xian/6xian.github.io
$ git push -u origin master
```

第二种方式，先在 GitHub 上 创建 username.github.io 远程仓库，参考 [https://pages.github.com/](https://pages.github.com/). 之后把远程仓库 clone 到本地，尝试在本地目录下新建一些 HTML 页面。然后在本地项目下执行 add, commit, push 等 Git 命令，这样就有一个简单的基于 GitHub Pages 的站点了。
```
$ git clone https://github.com/username/username.github.io
$ cd username.github.io
$ echo "Hello World" > index.html
$ git add --all
$ git commit -m "Initial commit"
$ git push -u origin master
```

有了这个简单的站点和页面后，我们在 GitHub 上去找一些自己喜欢的站点主题，也就是其他人的 GitHub Pages (username.github.io), 把他们的项目 clone 或下载到我们自己的站点目录下，按照 jekyll 文件和目录框架的要求，修改配置，修改页面。完成后，就可以开始写文章了。

### 3.2 发布文章

搭建完站点后，就可以写文章，发布文章了。jekyll 支持将文本文档自动生成静态 HTML 页面，就是说我们可以用自己喜欢的编辑器把文章写好，存储为 Markdown 或者 Textile 文件就行，有个规则就是在文档开头要有 YAML 头信息，两行三个破折号之间的数据为 YAML 头信息。示例如下： 
```
---
layout: post
title: Blogging Like a Hacker
---
```

再比如这篇文章的 YAML 头信息如下：
```
---
layout: post
title:  "搭建本站的一些知识 GitHub Pages 和 jekyll"
date:   2017-06-24
categories: Web
tags: Blog GitHub jekyll
author: 6xian
---
```

另外，建议所有博客文档都统一放在 `_posts` 目录下，方便管理。jekyll 会把所有存在 YAML 头信息的文件生成静态页面，所以可以在根目录下放一个 index.md 文件，但是在根目录下不能同时有 index.md 和 index.html. 不然 jekyll 不会解析，站点主页也就没有了。

我觉得最好的写文档方式就是用 Markdown 了，比如在 Windows 下用 Sublime Text 写文档和代码, 预览的话，给 Chrome 装一个 Markdown 插件就好了。 文章写好了后，在站点目录下运行 

`jekyll serve` 

或者 

`jekyll serve --watch ( --wath 参数表示 jekyll 会监测文档的变化，自动生成页面)`

生成静态页面并在本地预览。

如果要加入图片的话，可以把图片集中放在一个地方，在文档中用下面这段代码来引用图片。

```
<figure>
<a><img src="站点域名或项目名称/attaches/2017-06-24-about-this-site/1525073375257.png"></a>
</figure>
```

站点域名或项目名称, 即把 `site.url` 用两个花括号括起来。

下面这张图是我的测试。图片在本地的位置为 

`6xian.github.io/attaches/2017-06-24-about-this-site/1525073375257.png`

我在 `6xian.github.io` 项目下，新建了一个 `/attaches` 文件夹，集中存放所有文章的图片，每篇文章的图片单独放在以该文章名命名的文件夹下。

因为本地文件提交到 GitHub 后，我们的域名也是 `6xian.github.io`，和我们的图片本地位置是一样的，所以用这种方式，既可以在本地预览时引用图片，文章发布后，在远程也能引用图片。

<!-- ![1525073375257](../attaches/2017-06-24-about-this-site/1525073375257.png) -->

<figure>
<a><img src="{{site.url}}/attaches/2017-06-24-about-this-site/1525073375257.png"></a>
</figure>

最后是文章的发布，运行下面的命令之后，任何位置都可以访问你更新的文章了。

```
$ git add --all
$ git commit -m "some commit"
$ git push -u origin master
```

## 4 主题修改

本站的主题参考了 [HyG](https://github.com/Gaohaoyang/gaohaoyang.github.io) 和 [mojombo](http://tom.preston-werner.com/). 若您想使用这个 jekyll 博客主题，可以去他们的 GitHub 下载，或者参考 [6xian](https://github.com/6xian/6xian.github.io) 我现在用的这些样式。

下面对这个主题做些简要说明，也方便自己将来维护和修改这个站点。

### 4.1 目录结构

站点基本的目录结构可参考 jekyll 官网的文档 [https://jekyllrb.com/docs/structure/](https://jekyllrb.com/docs/structure/). 本站点的主要目录和文件如下。

``` shell
.6xian.github.io
├── _config.yml    # 主配置文件
├── _Gemfile       # Gemfile 和 Gemfile.lock 文件，用于 Bundler 管理(跟踪和编译) jekyll 需要的软件包
├── index.html     # 主页文件，也可是是符合 YAML 格式的 'index.md' 文件
├── readme.md      # 站点的说明文档，也是 Github 项目仓库的说明文档
├── .gitignore     # 该文件里列举的文件和目录，不会进入 Git 的管理范围。
├── _posts         # 放置 Blog 文章的目录
|   ├── 2017-06-24-about-this-site.md    # Blog 文章
|   └── 2017-xx-xx-xx-xx.mm
├── _drafts                              # 未发布的 BLog 文件， 这些文件的格式中都没有 title.MARKUP 数据
|   ├── 2017-xx-xx-xxxx-xxxx-xxxx.md
├── _includes          # 可以加载这些「包含部分」到你的布局或者文章中以方便重用
|   ├── footer.html    # 页面底部的 HTMl 代码
|   └── head.html      # 页面 <head> 标记的代码，这里会加载 CSS("/css/main.css"), JavaScript, HTML 头部等信息
|   └── header.html    # 页面顶部导航栏的代码
|   └── category.html  # 文章分类的显示页面，只对引用 post 模板的页面
|   └── tag.html       # 标签分类的显示页面，只对引用 post 模板的页面
├── _layouts           # layouts（布局）是包裹在文章外部的模板。布局可以在 YAML 头信息中根据不同文章进行选择
|   ├── default.html   # 默认布局文件，所有页面都会引用这个布局
|   ├── page.html      # 非 Blog 文章会引用这个模板布局，如 page 目录下的很多页面
|   └── post.html      # Blog 文章会引用这个模板布局
├── _sass              # 局部的 CSS 样式文件目录，用于导入 main.css 文件，main.css 定义了整个站点的 CSS 样式。.scss 文件的注释为 //
|   ├── _footer.scss   # 页面底部的样式
|   └── _header.scss   # 页面导航栏的样式
|   └── _index.scss    # 站点主页的样式
|   └── _layout.scss   # 默认 layouts（布局）的样式
|   └── _page.scss     # 默认 page.html 的样式
|   └── _post.scss     # 默认 post.html 的样式
|   └── _syntax-highlighting   # 语法高亮的样式
├── _page                  # 导航栏包含的页面
|   └── 0archives.html     # 文章归档的页面
|   └── 1category.html     # 文章分类的页面
|   └── 2tag.html          # 文章标签的页面
|   └── 3collections.md    # 一个相对独立的页面，充当收集的功能
|   └── 4about.md          # 关于作者的页面
├── _site              # Jekyll 完成转换后生成的页面放在这里（默认）。最好将这个目录放进 .gitignore 文件中，不同步到 GitHub 上
```
有了上面的目录和文件的说明，对整个站点的结构就比较清晰了。

### 4.2 主要配置

引用一段话来说明 _config.yml 文件的重要性。
> 
 Welcome to Jekyll!  
 This config file is meant for settings that affect your whole blog, values which you are expected to set up once and rarely need to edit after that. For technical reasons, this file is *NOT* reloaded automatically when you use 'jekyll serve'. If you change this file, please restart the server process.

_config.yml 文件里包括站点名称、地址，常用链接，分页参数，Markdown 样式等信息的设置。这些信息都是以全局变量的形式存在，jekyll 通过 YAML 和 liquid 来调用。可以很方便的对整个站点的信息进行配置和应用。可以参考本站点的 [_config.yml](https://github.com/6xian/6xian.github.io/blob/master/_config.yml) 文件。这里简单说下 YAML 和 liquid. 有助于理解怎么使用这些配置信息。

>YAML is a human friendly data serialization standard for all programming languages.    
>Liquid is a template engine which was crafted for very specific requirements.  
>YAML is a format that relies on white spacing to separate out the various elements of content. Jekyll lets you use Liquid with YAML as a way to parse through the data.

在 jekyll 里，我了解的应用包括：
* YAML 头信息，用两行 `---` 作为文件的开头，两行 `---` 中间的数据就是头信息，这里的键值可以被整个页面通过 liquid 标记来调用。另外 _layouts 布局目录里的页面可以通过 YAML 头在整个站点进行调用。如:
```
  ---
  layout: post
  ---
```
```
  ---
  layout: default
  ---
```
* Jekyll 使用 Liquid 模板语言，支持所有标准的 Liquid 标签和过滤器。Jekyll 甚至增加了几个过滤器和标签，方便使用。参考 [https://help.shopify.com/themes/liquid/basics](https://help.shopify.com/themes/liquid/basics).  
  - Objects. Objects contain attributes that are used to display dynamic content on a page. 
  - Filters. Filters are used to modify the output of strings, numbers, variables, and objects.
  - Tags. Tags make up the programming logic that tells templates what to do.


### 4.3 CSS 样式

整个站点呈现出来的外观主要就在 CSS 样式了，而 CSS 样式是基于站点的 HTML 代码。所以要想配置好 CSS 样式，先要理解好站点 HTML 代码是如何组织的。本站点页面的 HTML 代码主要分为四部分，分别是导航栏、正文左侧内容、正文右侧内容和页面底部。CSS 样式也是按照这四部分内容来组织。

首先说下整个站点 CSS 文件的组织和调用，_sass 目录下的 .scss 文件是站点模板和某些独立页面的 CSS 配置文件，但这里的文件没有被直接引用进页面，只是描述 CSS 的内容，真正被引用进页面的 CSS 文件是在 js 目录下的 main.scss 文件。在 main.scss 文件里导入了 _sass 目录下面的所有配置。
```
@charset  "utf-8";
@import "reset", "header", "post", "page", "syntax-highlighting", "index", "demo", "footer", "scrollbar", "backToTop";
```

_includes 目录下的 head.html 文件，调用了 main.scss 和 其他两个外部的 css 文件。
```
---
---
<link rel="stylesheet" href="https://cdn.bootcss.com/font-awesome/4.7.0/css/font-awesome.min.css">    
<link rel="stylesheet" href="https://at.alicdn.com/t/font_8v3czwksspqlg14i.css">
<link rel="stylesheet" href="{{ "/css/main.css " | prepend: site.baseurl }}">   
```
这里的两行三个破折号，以及为什么是 "/css/main.css " 而不是 "/css/main.scss " 文件，详细解释请参考 [https://jekyllrb.com/docs/assets/](https://jekyllrb.com/docs/assets/)

本站的 CSS 样式是基于 [HyG](https://github.com/Gaohaoyang/gaohaoyang.github.io) 进行的修改，并参照了 [mojombo](http://tom.preston-werner.com/) 的字体和颜色。具体的可以从他们的 GitHub 项目上进行学习，我简要说下自己的修改。

* 去除了文章标题带的各类标记和分类，采用更简洁的方式列出标题。（这只是修改 HTML 代码）
* 标题前的时间参考了 [mojombo](http://tom.preston-werner.com/) 的字体和颜色
* 修改了 _page.scss 的链接和行高，和主页保持一致，还是让页面更简洁
* 修改 _post.scss 相关文章的链接，和主页保持一致

### 4.4 语法高亮

博客文章肯定少不了代码，而代码的语法高亮能让文章更加美观，在 Markdown 的语法里面，代码用 两个 「\`\`\`」 包裹起来，要指定特定的代码语言的话，在第一个 「\`\`\`」 后面加上语言名称，比如  「\`\`\` python」, 如果不加语言名称的话都是以黑底白字显示，也是非常好看。下面是一些语法高亮的例子。

Python codes
``` python
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

shell 字符 
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

html codes
``` html
<html>
<body>
<p>Welcome!</p>
<a href="https://6xian.github.io/">Welcome to 6xian's site.</a>
</body>
</html>
```


这篇文章就讲完了。最后希望能有朋友关注和喜欢我这个站点，一起讨论和交流，能和大家一起学习进步更是我的荣幸。
