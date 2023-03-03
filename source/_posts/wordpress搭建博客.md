---
title: wordpress搭建博客
date: 2023-03-03 11:00:27
tags: "wordpress"
categories: "wordpress"
---

### 部署LNMP环境

#### 手动部署

[参考阿里云手动部署LNMP环境手册](https://help.aliyun.com/document_detail/171940.html)

#### 自动部署

- 下载lnmp集合包，下载地址：`http://soft.vpser.net/lnmp/lnmp1.6-full.tar.gz`

- 解压安装：

  ```
  tar -zxvf lnmp1.6-full.tar.gz
  cd lnmp1.6-full
  ./install.sh lnmp
  ```

- 开放mysql远程连接

  `grant all privileges on *.* to 'root'@'%' identified by 'root' with grant option;`

- 安装wordpress

  1. 安装wordpress安装包并解压到`/home/wwwroot`

     `wget https://cn.wordpress.org/latest-zh_CN.zip && unzip latest-zh_CN.zip -d /home/wwwroot`

  2. 登录mysql创建`wordpress`表

  3. 修改nginx配置文件

     ```
     find / -name nginx.conf		#查找nginx配置文件位置
     vim /usr/local/nginx/conf/nginx.conf 将root根目录位置修改为：/home/wwwroot/wordpress;
     nginx -t	#验证nginx是否有配置错误
     nginx -s reload		#重新加载nginx
     cd /home/wwwroot && chown -R www wordpress/ && chgrp -R www wordpress/		#修改wordpress目录权限
     ```

### 登录后台

- 用浏览器打开`http://ip/wp-admin/setup-config.php`，填写mysql信息

- 复制网页中的内容，在`/home/wwwroot/wordpress`中创建`wp-config.php`文件并填写
- 创建完毕后输入ip信息即可访问网站