---
title: hexo主题及扩展
date: 2023-01-30 09:01:38
tags: "hexo"
---

#### Butterfly主题配置

##### 安装步骤

- 移动到博客所在根目录

- 下载主题代码到themes中

  ```
  # 方式1：git方式安装
  git clone -b master https://github.com/jerryc127/hexo-theme-butterfly.git themes/butterfly
  
  # 方式2：npm方式安装
  npm i hexo-themebutterfly
  ```

- 修改hexo的_config.yml文件，将主题改为butterfly

  ```
  theme: butterfly
  ```

  若重启、打开页面报如下错误，因没有**pug**以及 **stylus** 的渲染器，需要安装

  ```
  extends includes/layout.pug block content include ./includes/mixins/post-ui.pug #recent-posts.recent-posts +postUI include includes/pagination.pug
  ```

- 安装pub & stylus

  ```
  npm install hexo-renderer-pug hexo-renderer-stylus --save
  ```

- 最后重启hexo即可

##### 修改主题配置

主题的配置在 **/www/wwwroot/blog/themes/butterfly/_config.yml** 文件中

##### 标签外挂

