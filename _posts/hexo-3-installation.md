---
title: Hexo 3 安装简单指北
date: 2017-01-16 20:05:36
tags:
 - Hexo
category: 
 - Wiki
thumbnail: 
author: bsyonline
lede: "没有摘要"
---

之前装过一次 hexo ，用了一段时间，后来重做系统之后，安装出现了一些问题，花了一些时间重新整理一下。

node.js 和 git 安装好后，开始安装 hexo 。

```shell
sudo npm install hexo -g
```

### 初始化

```shell
hexo init hexo
```
网上有教程说先创建目录，再执行
```
hexo init
```
也是可以的。

这步执行完就可以执行 clean ，generate ， server 等命令了。

### 部署
如果要使用 github 提供的网站服务，就需要将 hexo 部署到 github.io 上，所以需要先安装部署插件。

```shell
npm install hexo-deployer-git --save
```
配置 git
```
deploy:
  type: git
  repository: git@github.com:bsyonline/bsyonline.github.io.git
  branch: master
```

```shell
heox clean
hexo g
hexo deploy
```
至此，访问 bsyinline.github.io 就可以看到网页。

### primer 主题安装
1. primer 是一款 github 风格的主题，项目地址 [https://github.com/yumemor/hexo-theme-primer](https://github.com/yumemor/hexo-theme-primer) 。
   我使用的是 primer 2.0 ，按照说明文档配置即可 [https://github.com/yumemor/hexo-theme-primer/tree/2.0](https://github.com/yumemor/hexo-theme-primer/tree/2.0) 。

2. 添加 guitar 菜单

guitar 基本参考 open 布局，在 themes/primer/layout/ 下创建 guitar.ejs
```
<body>
	<%- partial('_partial/header') %>
	<%- partial('_partial/guitar', {post: page}) %>
```

在 themes/primer/layout/_partial/ 下创建 guitar.ejs
```
<section class="collection-head geopattern">
    <div style="height:30px;"><h1>&nbsp;</h1></div>
</section>
<section class="container">
    <div class="search" >
      <%- partial('_widget/guitar-search') %>
    </div>
    <div class="guitar-card-list">
      <% site.categories.each(function(category){ %>
        <% if(category.name == 'guitar'){ %>
          <% category.posts.each(function(post){ %>
            <li class="collection-card">
              <a href="{%=clone_url%}" class="card text-center" target="_blank">
                  <div class="thumbnail" style="margin-bottom:0px">
                      <div class="guitar-card-image geopattern" data-pattern-id="<%=category.name%>">
                          <div class="guitar-card-image-cell">
                              <h3 class="guitar-card-title">
                                <%- partial('post/title',{class_name:'posts-list-name',post:post,index:true}) %>
                              </h3>
                          </div>
                      </div>
                  </div>
              </a>
            </li>
          <% }) %>
        <%}%>
      <% }) %>
    </div>
</section>
```

在 themes/primer/source/_vendor/guitar/ 下创建 guitar.css
```
.guitar-card-list {
    padding-left: 0;
    position: relative;
    padding-top: 2em;
}
.guitar-card-image {
    display: table;
    height: 120px;
    width: 100%;
    border-radius: 4px;
}

.guitar-card-image .guitar-card-image-cell {
    display: table-cell;
    vertical-align: middle;
    padding-left: 1em;
}

.guitar-card-image h3 {
    margin: 0;
    font-size: 1.5em;
    color: white;
}

.guitar-card-image a {
    color: #fff;
}

.guitar-card-description {
    height: 2em;
    overflow: hidden;
}

.card .thumbnail:hover .guitar-card-title a{
    color:#000;
    text-decoration:none;
}

.guitar-card-text .meta-info {
    margin-right: 10px;
}
```

在 themes/primer/source/css/style.styl 中引入 guitar.css
```
@import "_vendor/guitar/guitar.css"
```

在 themes/primer/layout/_widget/ 下创建 guitar-search.ejs
```
<div id="site_search" style="margin-left:10px;height:40px;">
	<div style="float:left;height:40px;margin-right:10px;margin-bottom:10px;font-size:18pt;">
The Emerald Dream</div>
	<div style="float:right;height:40px;margin-bottom:10px;">
	<!-- Google -->
	<% if( theme.search.use == "google" ){ %>
		<form action="http://www.google.com/search?" data-site="<%=theme.url%>">
	    	<input type="text" id="search_box" name="q" placeholder="Search" style="width: 600px;">
	    	<button type="button" class="btn btn-default" id="site_search_do"><span class="octicon octicon-search"></span></button>
	    </form>
	<% } %>

	<!-- 本地搜索 -->
	<% if( theme.search.use == "local" ){ %>
		<!-- Local -->
		<form id="search-form" >
			<input type="text" id="search" placeholder="Search" style="width: 600px;background-color: #fff;
    background-position: right center;
    background-repeat: no-repeat;
    border: 1px solid #ccc;
    border-radius: 0px;
    box-shadow: 0 0px 0px rgba(0, 0, 0, 0.075) inset;
    color: #333;
    font-size: 13px;
		height:25px;
    min-height: 34px;
    outline: medium none;
    padding: 7px 8px;
    vertical-align: middle;">
		</form>

		<div id="local-search-result" style="width: 600px;"></div>

	<% } %>
</div>
</div>
<style>
#local-search-result{
	border-left: 1px solid #ccc;
	border-right: 1px solid #ccc;
	border-radius: 0px;
	box-shadow: 0 0px 0px rgba(0, 0, 0, 0.075) inset;
	font-size: 14px;
}
.search-result-list{
	width: 600px;

}
#local-search-result ul li{
	width: 598px;
	border-bottom: 1px solid #ccc;
	margin-top: -1px;
}
</style>
```
