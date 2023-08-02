---
layout: post
title: Jekyll使用和Liquid语法
date:   2021-04-18 00:00:00 +0800
categories: [Unclassfy]
---
* toc
{:toc}
Jekyll很多功能使用 Liquid 绑定
jekyll与Liquid关联是在HTML页面添加front-mater

{% raw %}
```html
---
some yaml： 告诉给liquid的配置内容
---
下面是从liquid runtime 访问的内容

{{ object.var }} : for 简单数据访问

{% if  %} {% endif %} : for 简单逻辑执行

{{ object.var | filter_name }} : filter 内置的函数处理，“|”是管道的类似概念，传入变量，后面是函数名字

```

Liquid主要有：Object Tag Filter 3个组件

### Layout : 重复的通用页面内容

`_layout` 默认文件夹， 就是内容模板 

关注： `{{content}}` content 对象是Liquid提供的填充功能，将front-mater 的layout 指向为当前文件的 文件内容填充 到 content

content 是填充文件内容对象

### Include : pages 页面之间的navigate

`_include` 默认文件夹

每个页面都有 导航，添加到layout 是个合适的位置

`_include/navigation.html` 创建内容

```html
<nav>
<a href="home"> Home </a>
<a href="/about.html"> About </a>
</nav>
```

一个导航模板便可以了

添加到 layout 以应用到所有页面

`_layout/default.html`

```html
...
<body>
{% include navigation.html %}
...
{{content}}
</body>
...

```

使用了`include` Tag，文本导入

当前页高亮

```html
<nav>
<a href="/home" {% if page.url == "/home" %} style="color:red;" {% endif %} >
 Home </a>
<a href="/about.html" {% if page.url == "/about.html" %} style="color:red;"{% endif%}
</nav>

```

关注 page.url 对象

### Data Files

Jekyll supports loading data from YAML, JSON, and CSV files located in a `_data` directory

支持从半结构性语言文件中读取数据：YAML JSON CSV

比如对于读取yaml文件内容来显示导航：
`_data/navigation.yml`

```yml
- name: Home
  link: /
- name: About
  link: /about.html
```
更新: `_includes/navigation.html`

```html
<nav>
{% for item in site.data.navigation %}

<a href="{{item.link}}" {% if page.url == item.link %} style="color:red;" {%endif%}  >
{{ item.name }}
</a>

{% endfor %}
</nav>

```
注意： site.data 对象对应`_data`文件夹

### Assets 图片等资源

使用CSS JS Image 以及别的资源在jekyll也可以很直接

默认文件夹`assets` 分别有：`css` `images` `js`

#### SASS

inline style 不是最佳实践，应该是css文件中定义 class

给navigation.html 的`<a>`标签添加 `class="current"` 的 class id

创建`assets/css/styles.scss`
```
---
---
@import "main";
```
空 front matter告诉Jekyll这个文件要被处理

import语句告诉Sass寻找 `_sass/main.scss` ，默认位置的文件夹为`_sass`

创建 `_sass/main.scss`文件：
```css
.current {
  color: green;
}
```

最后，要在使用这个样式的html文件的 `<head>`内 `<link rel="stylesheet" href="/assets/css/styles.css">` 显然这个文件是自动生成

### Blogging

`_post`文件夹下文件命名时间标题的文件 会被转化为静态网页

注意：一般有如下配置：

```ruby
---
layout: post
author: Hpbbs # 是自定义的变量
---
```

#### List posts

`/blog.html`

```html
---
layout: default
title: Blog
---
<h1>Latest Posts</h1>

<ul>
{% for post in site.posts %}

<li>
<h2> 
<a href="{{ post.url}}"> {{ post.title }}</a>
{{ post.excerpt }}
</h2>
</li>

{% endfor %}
</ul>
```

`post.url` 自动生成 path of post

`post.title`：文章标题

`post.excerpt`：文章的第一段

然后把 `/blog.html` 添加到 navigation 里面去：`_data/navigation.yml`

### Collections 聚合

`_config.yml` 中添加

```yaml
collections:
  authors:
```

Add authors:
`_authors/jill.md`

```ruby
---
short_name: jill
name: Jill Smith
position: Chief Editor
---
Jill is an avid fruit grower based in the south of France.
```

`_authors/ted.md`

```ruby
---
short_name: ted
name: Ted Doe
position: Writer
---
Ted has been eating fruit since he was bady.
```

让我们尝试访问collections的数据

`staff.html`

```html
---
layout: default
title: staff
---
<h1> Staff </h2>

<ul>
{% for author in site.authors %}
<li>
  <h2>{{ author.name }}</h2>
  <p> {{ author.position }} </p>
  <p> {{ author.content | markdownify }} </p>
</li>
{% endfor %}
</ul>
```

注意 `author.content` 使用了filer，文档是markdowm所以需要过滤
还有一点：只要是front-matter之外的就是content ，是一种文件头 文件体的分布

Ps:collections的文件是不会生成文档页面的，但是为了让每个职员都有各自的页面，需要修改
`_config.yml`

```
collections:
  authors:
    output: true
```
然后可以添加到一个页面内容里，通过 author.url 访问链接地址


同时，也需要创建一个layout ，`_layouts/author.html` 

```html
---
layout: default
---

<h1>{{ page.name }}</h1>
<h1>{{ page.position }} </h1>
{{ content }}

```
然后可以将对应页面的layout 指定为 author

#### Front-matter defaults

不需要在每个页面明确 对应的layout 配置
在`_config.yml` 统一配置默认的 layout并指明应用的scope

如：
```yaml

...
defaults:
  - scope:
      path: ""
      type: "authors"
    values:
      layout: "author"

  - scope:
      path: ""
      type: "posts"
    values: 
      layout: "post"

  - scope:
      path: ""
    values:
      layout: "default"
```
### List author’s posts 根据分类发文章

`_layouts/author.html` 迭代 output 作者的文章

```ruby
---
layout: defalut
---

<h1> {{ page.name }} </h1>
....

{% assign filtered_posts = site.posts | where: 'author', page.short_name %}
{% for post in filtered_posts %}
 .... post.url ...  post.title
{% endfor %}

```

### Deployment

网站有个Gemfile是很好的实践，确保jekyll version 和别的 gems依然正确，在不同的环境下

在根目录创建Gemfile

可以使用：   

`bundle init`

`bundle add jekyll`

`Gemfile`:

```
source "https://rubygems.org"

gem "jekyll"

```

bundler安装gems 并创建Gemfile.lock 锁定当前gem 版本for future `bundle instasll` 如果想升级， `bundle update`

如果要使用Gemfile 你应该运行 `jekyll serve`时携带前缀 `bundle exec`
这回限制你的Ruby environment 为当前Gemfile的

### Plugin 插件

插件允许你创建自定义生成内容，有很多插件，也可以写自己的

有三个官方插件：
- jekyll-sitemap : 创建一个sitemap file 帮助搜索引擎索引内容
- jekyll-feed : 创建RSS feed 为你的文章
- jekyll-seo-tag : Adds meta tags to help with SEO

使用前需要添加到Gemfile

```ruby
source '...rubygem..'

gem 'jekyll'

group :jekyll_plugins do
  gem 'plugin name'
  gem '...'
  gem '...'
end

```
然后还要添加到 `_config.yml`

```yaml
plugin:
 - jekyll-feed
 - jekyll-sitemap
 - jekyll-seo-tag
```

然后安装`bundle update`

`jekyll-sitemap` 不需要任何配置，会在build时自动创建

另外两个需要在 html 的 head 添加

head
{% feed_meta %}
{% seo %}
head

完成

### Environment 
命令行指定JEKYLL_ENV="product" 在liquid中 通过{{jekyll.environment}}访问
{% endraw %}