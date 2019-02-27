---
title: "解决使用 vagrant 时 virtualbox 未安装 guest additions 的问题"
date: 2016-01-18T22:38:06+08:00
draft: false
categories: ["linux"]
tags: ["linux", "vm", "vagrant"]
---
最近在使用`vagrant`过程中，初始化一个`box`启动时会卡很久，最后报错：

```sh
No guest additions were detected on the base box for this VM!
Guest additions are required for forwarded ports, shared folders, host only networking, and more.
If SSH fails on this machine, please install the guest additions and repackage the box to continue.
```

```sh
Failed to mount folders in Linux guest. This is usually because
the "vboxsf" file system is not available. Please verify that
the guest additions are properly installed in the guest and
can work properly. The command attempted was:

mount -t vboxsf -o uid=`id -u vagrant`,gid=`getent group vagrant | cut -d: -f3` vagrant /vagrant
mount -t vboxsf -o uid=`id -u vagrant`,gid=`id -g vagrant` vagrant /vagrant

The error output from the last command was:

stdin: is not a tty
mount: unknown filesystem type 'vboxsf'
```

根据启动过程中第一段信息的提示，基本确定为`guest addtions`未正确安装引起的问题。

<!--more-->

## 安装 guest additions
* 首先，使用`vagrant halt`命令关闭虚拟主机。
* 然后在 virtualbox 中打开虚拟主机，进入系统。
* 在虚拟机菜单栏选中`Devices`，`Insert Guest Additions CD image`挂载增加功能文件。

```sh
// 先安装linux头文件，否则下面步骤因为依赖头文件可能会失败。
sudo apt-get install linux-headers-$(uname -r)
sudo mount /dev/cdrom /media/cdrom
cd /media/cdrom
sudo ./VBoxLinuxAddtions.run
```

之后就能正常使用`vagrant`了。
