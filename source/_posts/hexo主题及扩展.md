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

##### 鼠标样式更改

首先准备想要生成鼠标样式的图片，调整对应的像素大小，将图片类型转换为.cur（或直接在[鼠标指针 - 光标 - 电脑鼠标指针下载 - 致美化 - 漫锋网 (zhutix.com)](https://zhutix.com/tag/cursors/)中找想要的样式），在主题路径source的css文件夹中新建一个mouse文件夹用于存放这些图片

###### 代码引入

在butterfly主题的`_config.yml`文件中修改如下：

![](https://s1.ax1x.com/2023/01/28/pSUjNCV.png)

在主题路径source中的css文件夹新建一个`mouse.css`文件，加入类似下列内容：

```css
body,
html {
    cursor: url('./mouse/alternate.cur'), auto !important;
}

/* 悬停图片时的鼠标指针 */
/* 选择链接标签时的鼠标指针 */
/* 选中输入框时的鼠标指针 */
/* 悬停按钮时的鼠标指针 */
/* 悬停列表标签时的鼠标指针 */
/* 悬停页脚链接标签（例如页脚徽标）时的鼠标指针 */
/* 悬停页码时的鼠标指针 */
/* 悬停菜单栏时的鼠标指针 */
img,
a:hover,
input:hover,
button:hover,
i:hover,
#footer-wrap a:hover,
#pagination .page-number:hover,
#nav .site-page:hover {
    cursor: url('./mouse/link.cur'), auto !important;
}
```

重新刷新即可看到鼠标样式已经生效

##### 搜索功能

###### 安装插件

```
npm install hexo-generator-search --save
```

###### 更改配置

在`主配置文件`中插入类似下列代码：

```yaml
search:
  path: search.xml
  field: all # post:文章范围、page:页面范围、all:覆盖所有
  content: true # 内容是否包含每一篇文章的全部内容
  template:  # ./search.xml 指定定制的XML模板
```

将butterfly配置文件中`local_search`的`enable`参数设为true表示开启本地搜索

##### 主题图片的更改

<font color=red>图片的大小最好不要超过3M，否则会影响文章的打开速度</font>

###### 网站logo

将图片的名称更改为`favicon.png`替换掉source下的img中的原logo即可

###### 头像修改

将图片的名称更改为`avatar.png`加入source下的img中，并将对应参数修改：

```yaml
# Avatar (头像)
avatar:
  img: /img/avatar.png
  #是否一直旋转
  effect: false
```

##### 主图修改

将图片加入source下的img中，在配置文件中修改`index_img`参数的值为该路径

- `default_top_img:`后面是当图片显示错误时的代替图片
- `index_img:`后是进入网站时显示的图像
- `archive_img:`是点击头像下面的文章进入的页面的顶部图
- `tag_img:`是点击头像下面的标签进入的页面的顶部图
- `category_img:`是点击头像下面的分类进入的页面的顶部图

###### 随机图片的匹配

此时设置了每一个分类，每一个标签页的图片，但是tags主页与categories主页都是原图，需要调整cover属性：

```yaml
cover:
  # display the cover or not
  #首页是否显示文章封面
  index_enable: true
  #侧边栏是否显示文章封面
  aside_enable: true
  #归档页是否显示文章封面
  archives_enable: true
  # the position of cover in home page
  # left/right/both
  position: both
  #当没有设置封面的时候，使用以下的照片
  default_cover:
     - 图片地址1
     - 图片地址2
     .....
```

###### 错误页面的匹配

```yaml
# Replace Broken Images
#在这里可以自定义所有图片不能显示的情况下显示的图片
error_img:
  flink: /img/404.jpg
  post_page: /img/404.jpg

# A simple 404 page
error_404:
  enable: true
  subtitle: '页面丢失，联系博主进行解决吧'
  #图片的背景
  background: 链接地址
```

##### 字数统计

```
npm install hexo-wordcount --save
```

在butterfly的配置文件中将wordcount的enable参数设置为true

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

找到想要的图标加入到购物车，添加至项目，将项目下载至本地，在主题的source/css文件下将红色标注的移动过去

![](https://moonshuo.cn//images/202211011545280.png)

原css目录下的mouse.css（先前添加的鼠标样式）修改为style.css（将hexo配置文件中的inject参数也做对应修改）；将该iconfont.css中的内容添加进去：

![](https://s1.ax1x.com/2023/01/30/pSwYKbV.png)

调整标题栏格式，重新部署即可：

![](https://s1.ax1x.com/2023/01/30/pSwYlUU.png)

