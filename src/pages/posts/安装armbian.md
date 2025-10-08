---
layout: ../../layouts/MarkdownPostLayout.astro
title: '安装armbian'
pubDate: 2024-04-15
description: '在玩客云上安装armbian'
author: 'FlyLoongZ'
tags: ["armbian", "linux"]
---

## 准备工作
### 下载Amlogic USB Burning Tool
我是在[AndroidMTK](https://androidmtk.com/download-amlogic-usb-burning-tool)下载的。
>可是是体质问题，下载了几个版本（v3.2.8~v2.1.9）都刷不进去（卡1%或97%）。根据[GitHub issues](https://github.com/hzyitc/armbian-onecloud/issues/20#issuecomment-1278685190)里的建议，下载V2.0.5.15 一次成功。
### 下载armbian
打开`https://github.com/hzyitc/armbian-onecloud/releases`下载，这里我选择[Armbian-unofficial_24.5.0-trunk_Onecloud_bookworm_edge_6.7.9.burn.img.xz](https://github.com/hzyitc/armbian-onecloud/releases/download/ci-20240311-162146-UTC/Armbian-unofficial_24.5.0-trunk_Onecloud_bookworm_edge_6.7.9.burn.img.xz)
>要选择带burn的才能用Amlogic USB Burning Tool刷入
## 刷机
- 拆机，可以参考网上视频
- 打开USB_Burning_Tool软件
- 左上角文件>导入烧录包，选择下载的固件，点击开始
- 双公头USB数据线，一头插靠近HDMI接口USB，一头电脑USB
- 短接刷机点通电到3%再松手，直到刷机完成
短接图：
![v1.1短接图](images/Pasted%20image%2020240415190906.png)
v1.1短接图（sd 卡槽空白）
![v1.3短接图](images/Pasted%20image%2020240415190930.png)
v1.3短接图（卡槽带版本）
![v1.3短接图](images/Pasted%20image%2020240415190958.png)
放大图
>参考[玩客云刷Debian小白保姆级教程AllinOne](https://www.right.com.cn/forum/thread-8344722-1-1.html)，[玩客云刷机短接图](https://www.bilibili.com/read/cv21738633/)
## 配置
### 基本配置
- SSH连接armbian, 默认用户root，密码1234
`ssh root@玩客云的ip地址`
- 设置语言
`sudo nano /etc/default/locale`
修改文件内容为:
```
LANG=zh_CN.UTF-8  
LANGUAGE=zh_CN.UTF-8`
```
删除`/etc/profile.d/armbian-lang.sh`
`sudo rm /etc/profile.d/armbian-lang.sh`
- 更换国内镜像源
打开`/etc/apt/sources.list`
`sudo nano /etc/apt/sources.list`
替换内容为:
```
# 默认注释了源码镜像以提高 apt update 速度，如有需要可自行取消注释
deb https://mirrors.cernet.edu.cn/debian/ bookworm main contrib non-free non-free-firmware
# deb-src https://mirrors.cernet.edu.cn/debian/ bookworm main contrib non-free non-free-firmware

deb https://mirrors.cernet.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware
# deb-src https://mirrors.cernet.edu.cn/debian/ bookworm-updates main contrib non-free non-free-firmware

deb https://mirrors.cernet.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware
# deb-src https://mirrors.cernet.edu.cn/debian/ bookworm-backports main contrib non-free non-free-firmware

deb https://mirrors.cernet.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware
# deb-src https://mirrors.cernet.edu.cn/debian-security bookworm-security main contrib non-free non-free-firmware

# deb https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
# # deb-src https://security.debian.org/debian-security bookworm-security main contrib non-free non-free-firmware
```
替换armbian 软件源:
```
sudo sed -i.bak 's#http://apt.armbian.com#https://mirrors.cernet.edu.cn/armbian#g' /etc/apt/sources.list.d/armbian.list
```
### cpu设置
- 打开`/etc/default/cpufrequtils`
`sudo nano /etc/default/cpufrequtils`
将`GOVERNOR=`设置为`schedutil`
> cpufreq-info 查看cpu信息