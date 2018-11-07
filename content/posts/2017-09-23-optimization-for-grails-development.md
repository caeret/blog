---
title: "Grails 开发优化小记"
date: 2017-09-23T21:31:06+08:00
categories: ["development"]
tags: ["go", "grails"]
---

13 年入了人生第一个 RMBP 放在家用，从此过上了使用 8G 内存和 256G SSD 的幸福生活。

16 年底入职新公司，从入职第一天我就把自己的笔记本带到了公司服役，大部分时间都在和 PHP 和 Golang 打交道，偶尔也会跑跑其他程序，基本没有遇到过卡顿很厉害的情况。

最近两个月，部门从隔壁部门接手了一个服务，基于 java6 使用 grails 2.x 版本开发的 web 服务。为了方便开发调试，将 java、groovy、grails 等各种依赖做了一个 docker 镜像。从此，每次要启动开发环境和打包服务，只需要启动 docker 镜像就可以了。

但是可能由于只分配给 docker 服务的资源太少，服务要加载的依赖项太多，每次 docker 里的服务启动起来或者打包完都需要好几分钟。另外每次修改了源文件，服务热更新变动后的文件也需要10至30秒，真不是一般得慢。感觉我的时间都浪费在了漫长的等待中……

忍不了，于是想把开发环境搬到部门的开发机上。但是环境在开发机上，每次更新了源码后如果想要看效果，就需要把源码同步到开发机上。而我又懒得手动同步，我就想做一个自动触发器，监控到文件有变动，就自动把变动的文件同步到开发机。比如，监控一个目录下，如果有文件修改（新增、删除、写入、移动等）就执行 `rsync` 命令把目录同步到远程服务器。于是，就有了 `gwatch` [(https://github.com/caeret/gwatch)](https://github.com/caeret/gwatch)，指定监控目录，如果目录下文件有变动，执行命令。可以指定变动不触发命令的路径，触发命令的延迟等。

这样每次在本地修改了源码，就会自动同步到开发机。终于没有漫长的等待了。
