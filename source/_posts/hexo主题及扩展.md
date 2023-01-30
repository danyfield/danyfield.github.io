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

##### 标题栏标签完善

hexo自带的标签包括分类，首页，标签等，且其默认配置了对应的目录：

![](https://s1.ax1x.com/2023/01/30/pSdTrqI.png)

在source目录执行下面命令，建立标签页，其他的categories等也是相同的操作

```
hexo new page "tags"
```

在生成路径的文件中增加type属性，内容与title相同并用双引号括住

```
title: tags
date: 2023-01-30 11:27:08
type: "tags"
```

打开主题路径下的配置文件配置对应的标签及相应图标：

![](https://s1.ax1x.com/2023/01/30/pSdTzLR.png)

图标可以在阿里图标官网选择：[iconfont-阿里巴巴矢量图标库](https://www.iconfont.cn/)

找到对应的图标，加入到购物车，添加至项目，将项目下载至本地，在butterfly文件的source/css文件下，将红色标注的全部移动过去

![](https://moonshuo.cn//images/202211011545280.png)

将原css目录下的mouse.css（先前添加的鼠标样式）修改为style.css（将hexo配置文件中的inject参数也做对应修改）；将该iconfont.css中的内容添加进去：

![](https://s1.ax1x.com/2023/01/30/pSwYKbV.png)

调整标题栏格式，重新部署即可：

![](https://s1.ax1x.com/2023/01/30/pSwYlUU.png)

