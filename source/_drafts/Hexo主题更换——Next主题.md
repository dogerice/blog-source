---
title: Hexo主题安装——Next主题
tags:
- Hexo
---



 

在Hexo中有两个主要的配置文件，其名称都为 `_config.yml` 其中一个是位于站点根目录下的站点配置文件，一个是位于主题目录 `themes`下的主题配置文件



```
.
├── _config.yml  <-站点配置文件
├── package.json
├── scaffolds
├── source
|   ├── _drafts
|   └── _posts
└── themes
    └── next  
    	└── _config.yml  <-主题配置文件
```

Hexo安装主题只需要将主题文件拷贝到 `themes` 目录下，然后修改下配置文件即可，步骤如下

### 安装主题

如果使用的是Hexo5.0或更新版本，最简单的安装方式是通过npm安装`npm install hexo-theme-next`

或者git克隆 [next](https://github.com/next-theme/hexo-theme-next) 仓库放到theme目录下`git clone https://github.com/next-theme/hexo-theme-next themes/next` 

### 启用主题

打开站点配置文件，找到theme字段修改为next

```
theme: next
```

### 配置主题

将主题配置文件拷贝到根目录并命名为` _config.next.yml`

```
# 通过 npm 方式安装的
cp node_modules/hexo-theme-next/_config.yml _config.next.yml
# 通过 Git 方式安装的
cp themes/next/_config.yml _config.next.yml
```

至此Next主题基本安装完成，接下来就可以进行页面样式的调整了

#### 设置Scheme

next可选的页面布局有四种，默认为Muse

```yml
# Schemes
#scheme: Muse
#scheme: Mist
#scheme: Pisces
scheme: Gemini
```

我选择的是`Gemini` 大致风格如下

![image-20211217201702638](image-20211217201702638.png)

#### 添加菜单页面——分类、标签等

找到`menu`相关配置，解开需要的菜单项的注释

```
menu:
  home: / || fa fa-home
  #about: /about/ || fa fa-user
  archives: /archives/ || fa fa-archive
  tags: /tags/ || fa fa-tags
  categories: /categories/ || fa fa-th
  #schedule: /schedule/ || fa fa-calendar
  #sitemap: /sitemap.xml || fa fa-sitemap
  #commonweal: /404/ || fa fa-heartbeat
```

此时`/tags` 和 `/categories` 还不能正常访问，需要用如下命令创建page

```
hexo new page "tags"
hexo new page "categories"

```

创建page后会在source目录下新建对应的文件夹，并在每个文件夹下创建一个index.md，编辑index.md增加type

```
---
title: tags
date: 2021-12-17 20:26:23
type: "tags"
---

---
title: categories
date: 2021-12-17 20:26:42
type: "categories"
---
```

写文章时在头部加上分类和标签即可，分类只能由一个，标签可有多个，格式如下

```
---
title: Chrome浏览器播放RTSP视频流解决方案
date: 2021-01-03 01:19:19
tags:
- FFmpeg
- RTSP
- RTMP
categories:
- tools
---
```









