---
title: hugo部署
link: hugo部署
categories:
- daily
date: 2024-04-15 20:40:02 +0800
date_modified: 2024-04-15 20:40:02 +0800
---

# hugo部署

## 初始化

https://github.com/judddd/factory011.github.io



## theme bug

```shell
hugo/themes/even/layouts/partials/head.html

然后移除这行代码：

{{- template "_internal/google_news.html" . -}}
```





## MarsEdit api扩展

```shell
https://github.com/elliotekj/orbit

ruby app/orbit.rb -s ~/blog -u "cd ~/blog && hugo"

首先，在终端运行您的 Ruby 程序，并按下 Ctrl + Z 将其暂停（停止）。
然后，运行 bg 命令将程序放在后台运行：
bg
disown
```





## 定制化

https://blog.csdn.net/qq_37908043/article/details/93350094

## 参考文档
https://stack.jimmycai.com/config/menu

## 最终
https://gitee.com/judddd/xuhao-hugo