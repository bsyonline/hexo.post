---
title: Jade 语法实践
date: 2016-07-18 14:29:39
tags:
 - Jade
 - Hexo
category: 
 - Wiki
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

jade 是一个模板引擎，可以通过 node.js 来使用。本来对前端技术了解不多，接触 jade 源于搭建 hexo 博客系统。maupassant 主题模板是基于 jade 的，所以想按自己的想法来定制 maupassant 主题简单了解了一下 jade 。

hexo 的 maupassant 主题是一个极简主题模板，安装也很简单，用起来不错，但是有些小地方还是想按照自己的想法改一改：
1. 默认只有归档，没有 tags 和 categories ，而且添加了 tags 和 categories 后不能显示 tags 和 categories 的列表；
2. 每篇文章 tags 标签在文章末尾，希望 categories 一样在文章开头显示；

### 增加 tags 列表
安装 hexo 和 maupassant 主题后在 themes/maupassant/_ config.yml 中添加 tags 和 categories 两个 menu，重启服务界面能看到 tags 和 categories 的菜单导航。但是点击会报找不到页面错误，所以还需要生成 tags/index和categories/index 。执行
```
hexo new page tags
hexo new page categories
```
会生成 tags/index.md 和 categories/index.md 文件，重启后不会再报找不到页面的错误。
但 index.md 默认是空白页面，内容需要自己写，所以想参考 archive 来写一些内容。想显示的内容也很简单，首先就是列出所有 tags 或 categories ，然后能显示每个 tag 或category 下的文章数量。
网站的 tags 和 categories 不是固定的，所以在 index.md 中写死是不行的，需要利用模板来实现。看了看 maupassant 主题的 layout 结构，首先先参考 archive.jade 创建了一个 tags.jade 文件,只保留大的结构。
```
extends base

block title
  title= page.tag + ' | ' + config.title
block content

  include _partial/paginator.jade
```
参考 archive.jade 代码，对网站所有 tag 进行分组，然后遍历显示。
```
each tags in _.groupBy(site.tags.toArray(), function(p){return page.tag})
  ul.listing
    for tag in tags
      li
        h4
          a(href=url_for(tag.path))= _.split(tag.path,'\/')[1]
```
最后一行代码等号后边是 tag 的名字，原本以为 tag.name 就可以了，但是重建之后发现不能正常显示。不明白原因，查看了 hexo 的文档，在 hexo 的变量声明中 tag 的名字是page.tag，猜测是在 post 页面中才能引用 tag 属性。查了文档也没找到正确的打开方式，忽然发现 tag.path 可以正常显示 tag 的路径，所以是否可以在 path 中将 tag 的 name 提取出来呢？于是查了一下[https://lodash.com/docs](https://lodash.com/docs)( archive.jade 里注释中有让看的，就看看喽)，还真有 spilt 的函数。改完重新生成，显示正常了(别忘了把 tag/index.md 的 layout 改成 tags 哈)。


### 改变文章中 tags 的显示位置
默认界面 tags 在文章结束坐下位置，但我期望在界面文章开始位置增加 tags 的显示，即在点击量统计的后边再增加显示 tags 内容。结合模板代码和界面的显示效果，可以大致定位到具体代码位置。文章正文默认使用的 post 模板，位于 themes/maupassant/layout/post.jade 。在文件15-19行是点击量统计的代码，所以在后面可添加 tags 显示代码。
```
if page.tags.length > 0
  span= ' | '
  span.tag
    for tag in page.tags.toArray()
      i(class=['fa','fa-paperclip']) &nbsp;
      a(href=url_for(tag.path))= tag.name + ''

```
jade 模板的语法有严格缩进要求，所以和上边的代码位置一致即可。
代码看起来很简单，但是改的时候还是费了一些周折。
```
page.tags.toArray()
```
的写法参考上文
```
page.categories.toArray()
```
很容易写出来。
```
i(class=['fa','fa-paperclip']) &nbsp;
```
是一个 i 标签，用于显示图标，渲染成 html 是
```
<i class="fa fa-paperclip">&nbsp;</i>
```
如果不看 jade 的语法，只根据 post.jade 文件中的代码可以大致推测 jade 的语法规则比如
```
span.category
```
渲染成 html 后是```
<span class="category"></span>
```
但是单 class 属性图标无法正常显示，也试过
```
i.fa.fa-paperclip
或
i.fa i.fa-paperclip
```
等等写法，均不能达到效果，只好去看 jade 的文档。在[http://jade-lang.com/reference/attributes/](http://jade-lang.com/reference/attributes/)中 **Class Attributes** 找到了多个 class 的语法说明，问题至此得以解决。

```