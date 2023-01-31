---
title: hexo文档
date: 2023-01-30 09:49:00
tags: "hexo"
categories: "hexo"
---

### 配置

#### 网站

| 参数          | 描述                              |
| :------------ | :-------------------------------- |
| `title`       | 网站标题                          |
| `subtitle`    | 网站副标题                        |
| `description` | 网站描述                          |
| `keywords`    | 网站的关键词，支持多个关键词      |
| `author`      | 作者                              |
| `language`    | 网站使用的语言                    |
| `timezone`    | 网站时区，Hexo 默认使用电脑的时区 |

#### 指令

##### new

```
$ hexo new [layout] <title>
```

新建一篇文章，如果没有设置`layout`，则默认使用`_config.yml`中的`default_layout`参数代替

| 参数              | 描述                                          |
| :---------------- | :-------------------------------------------- |
| `-p`, `--path`    | 自定义新文章的路径                            |
| `-r`, `--replace` | 如果存在同名文章，将其替换                    |
| `-s`, `--slug`    | 文章的 Slug，作为新文章的文件名和发布后的 URL |

默认情况下，Hexo 会使用文章的标题决定文章文件的路径。对于独立页面，Hexo 会创建一个以标题为名字的目录，并在目录中放置一个 `index.md` 文件。可以使用 `--path` 参数来覆盖上述行为、自行决定文件的目录：

```
hexo new page --path about/me "About me"
```

###### 布局（layout）

Hexo 有三种默认布局：`post`、`page` 和 `draft`

| 布局    | 路径             |
| :------ | :--------------- |
| `post`  | `source/_posts`  |
| `page`  | `source`         |
| `draft` | `source/_drafts` |

设置为草稿布局时可以通过`publish`命令将草稿移动到`source/_post`文件夹中，草稿默认不会显示在页面中

```
hexo publish [layout] <title>
```

###### 模版（Scaffold）

新建文章时，Hexo 会根据 `scaffolds` 文件夹内对应的文件来建立，例如：

```
$ hexo new photo "My Gallery"
```

执行该指令时，Hexo 会尝试在 `scaffolds` 文件夹中寻找 `photo.md`，并根据其内容建立文章

##### generate

生成静态文件，该命令可以简写为`hexo g`

##### publish

```
hexo publish [layout] <filename>	#发表草稿
```

##### server

```
hexo server
```

启动服务器，默认情况下，访问网址为： `http://localhost:4000/`

| 选项             | 描述                           |
| :--------------- | :----------------------------- |
| `-p`, `--port`   | 重设端口                       |
| `-s`, `--static` | 只使用静态文件                 |
| `-l`, `--log`    | 启动日记记录，使用覆盖记录格式 |

##### deploy

```
hexo deploy
```

部署网站，该命令可以简写为`hexo d`

| 参数               | 描述                     |
| :----------------- | :----------------------- |
| `-g`, `--generate` | 部署之前预先生成静态文件 |

##### clean

```
hexo clean
```

清除缓存文件（db.json）和已生成的静态文件（public）

在某些情况（尤其是更换主题后），如果发现对站点的更改无论如何也不生效，可能需要运行该命令

##### list

```
$ hexo list <type>
```

列出网站资料

##### version

```
$ hexo version
```

显示 Hexo 版本

### 基本操作

#### Front-matter

`Front-matter `是文件最上方以 `---` 分隔的区域，用于指定个别文件的变量

##### Page Front-matter

| 参数          | 描述                                             |
| :------------ | :----------------------------------------------- |
| `title`       | 页面标题【必需】                                 |
| `date`        | 页面建立日期【必需】                             |
| type          | 标签、分类和友链三个页面需要配置【必需】         |
| `updated`     | 页面更新日期                                     |
| `description` | 页面描述                                         |
| keywords      | 页面关键字                                       |
| `comments`    | 评论模块【默认为true】                           |
| `top_img`     | 页面顶部图片                                     |
| `aside`       | 显示侧边栏【默认为true】                         |
| `aplayer`     | 在需要的页面加载aplayer的js和css，参考`音乐`配置 |
| `lang`        | 设置语言用于覆盖自动检测                         |

##### Post Front-matter

部分参数参照page配置

| 参数         | 描述                                                         |
| ------------ | ------------------------------------------------------------ |
| `tags`       | 文章标签                                                     |
| `categories` | 文章分类                                                     |
| `top_img`    | 文章顶部图片                                                 |
| `cover`      | 文章缩略图（若未设置top_img，文章页顶部将显示缩略图，可设为false/图片地址/留空） |
| `sticky`     | 文章置顶赋值，数值越大则置顶的优先级越大                     |
| `copyright`  | 转载文章不需要显示版权设置为`false`                          |

##### 分类和标签

只有文章支持分类和标签，可以在 Front-matter 中设置。在 Hexo 中两者有着明显的差别：分类具有顺序性和层次性，也就是说 `Foo, Bar` 不等于 `Bar, Foo`；而标签没有顺序和层次

```
categories:
- Diary
- Life
tags:
- PS3
- Games
```

上述分类方法会使分类 `Life` 成为 `Diary` 的子分类，而不是并列分类

如果需要为文章添加多个分类，可以尝试以下 list 中的方法：

```
categories:
- [Diary, PlayStation]
- [Diary, Games]
- [Life]
```

此时同时包括三个分类： `PlayStation` 和 `Games` 分别都是父分类 `Diary` 的子分类， `Life` 是一个没有子分类的分类

##### JSON Front-matter

除了 YAML 外，也可以使用 JSON 来编写 Front-matter，只要将 `---` 代换成 `;;;` 即可

