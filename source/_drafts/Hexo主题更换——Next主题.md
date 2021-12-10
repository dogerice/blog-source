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







