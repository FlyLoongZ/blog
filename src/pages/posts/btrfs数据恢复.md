---
layout: ../../layouts/MarkdownPostLayout.astro
title: 'btrfs数据恢复'
pubDate: 2025-10-08
description: '最近一块用于bt下载的hdd硬盘突然挂了，经过多次尝试终于恢复成功，于是记录一下数据恢复的过程'
author: 'FlyLoongZ'
tags: ["btrfs", "hdd", "linux"]
cover: /src/images/Btrfs_logo.svg
---

## 目录
- [目录](#目录)
- [故障出现](#故障出现)
- [尝试修复](#尝试修复)
- [尝试提取数据](#尝试提取数据)
- [备份分区](#备份分区)
- [在raw上提取数据](#在raw上提取数据)
- [参考](#参考)


## 故障出现
一天，我挂载webdav准备看番时突然发现视频怎么不见了，于是查看demsg日志发现btrfs出现io读取错误、硬盘出现无法恢复的读取错误。

## 尝试修复
首先使用`sudo btrfs check /dev/sda1`，修复失败
```bash
Opening filesystem to check...
Error reading 319275008, -1
Error reading 319275008, -1
bad tree block 319275008, bytenr mismatch, want=319275008, have=0
ERROR: could not find extent tree
ERROR: cannot open file system
```
然后尝试`sudo btrfs check -s1 --backup /dev/sda1`
```bash
using SB copy 1, bytenr 67108864
Opening filesystem to check...
No valid Btrfs found on /dev/sda1
ERROR: cannot open file system
```
`sudo btrfs check -s2 --backup /dev/sda1`，修复失败
```bash
using SB copy 2, bytenr 274877906944
Opening filesystem to check...
No valid Btrfs found on /dev/sda1
ERROR: cannot open file system
```

最后尝试`sudo btrfs rescue super-recover /dev/sda1`、`sudo btrfs rescue chunk-recover /dev/sda1`、`sudo btrfs rescue zero-log /dev/sda1`都修复失败。
```bash
checksum verify failed on 319275008 wanted 0x00000000 found 0xb6bde3e4
ERROR: could not find extent tree
Failed to recover bad superblocks
```

## 尝试提取数据
在root tree损坏的情况下无法修复分区,于是尝试提取数据

首先使用`sudo btrfs restore /dev/sda1 /mnt/raid/hdd_data`提取数据，提取失败
```bash
checksum verify failed on 319275008 wanted 0x00000000 found 0xb6bde3e4
checksum verify failed on 319275008 wanted 0x00000000 found 0xb6bde3e4
bad tree block 319275008, bytenr mismatch, want=319275008, have=0
ERROR: could not find extent tree
Could not open root, trying backup super
No valid Btrfs found on /dev/sda1
Could not open root, trying backup super
No valid Btrfs found on /dev/sda1
Could not open root, trying backup super
```

## 备份分区
经过上面的尝试，为了不进一步破坏硬盘，使用`sudo ddrescue -d /dev/sda1 /mnt/raid/hdd_bf/backup.img /mnt/raid/hdd_bf/backup.log`导出硬盘raw数据。

## 在raw上提取数据
首先使用`sudo btrfs-find-root -a /mnt/raid/hdd_bf/backup.img | tee tree.log`查找btrfs root tree。
```bash
Superblock thinks the generation is 1020854
Superblock thinks the level is 1
Well block 308641792(gen: 1020854 level: 1) seems good, and it matches superblock
Well block 393347072(gen: 1020852 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 393134080(gen: 1020852 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 392265728(gen: 1020852 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 391823360(gen: 1020852 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 380911616(gen: 1020851 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 342802432(gen: 1020850 level: 1) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 379961344(gen: 1020849 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 333774848(gen: 1020849 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 357154816(gen: 1020846 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 367886336(gen: 1020845 level: 1) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 356302848(gen: 1020844 level: 0) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 569491456(gen: 1020841 level: 1) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 563396608(gen: 1020840 level: 1) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
Well block 539213824(gen: 1020839 level: 1) seems good, but generation/level doesn't match, want gen: 1020854 level: 1
```
以Well block 393134080(gen: 1020852 level: 0) seems good为例使用393134080作为下面命令-t后的数字

使用`sudo btrfs restore -Di -t 308641792 /mnt/raid/hdd_bf/backup.img /mnt/raid/hdd_data`试运行
```bash
checksum verify failed on 319275008 wanted 0x00000000 found 0xb6bde3e4
checksum verify failed on 319275008 wanted 0x00000000 found 0xb6bde3e4
bad tree block 319275008, bytenr mismatch, want=319275008, have=0
ERROR: could not find extent tree
Could not open root, trying backup super
No valid Btrfs found on /mnt/raid/hdd_bf/backup.img
Could not open root, trying backup super
No valid Btrfs found on /mnt/raid/hdd_bf/backup.img
Could not open root, trying backup super
```
若出现`Could not open root, trying backup super`就使用下一个数字

我一直尝试到569491456才可以找到数据
`sudo btrfs restore -Di -t 569491456 /mnt/raid/hdd_bf/backup.img /mnt/raid/hdd_data`
```bash
parent transid verify failed on 569491456 wanted 1020854 found 1020841
parent transid verify failed on 569491456 wanted 1020854 found 1020841
Ignoring transid failure
This is a dry-run, no files are going to be restored
checksum verify failed on 363069440 wanted 0x00000000 found 0xb6bde3e4
checksum verify failed on 363069440 wanted 0x00000000 found 0xb6bde3e4
bad tree block 363069440, bytenr mismatch, want=363069440, have=0
ERROR: search for next directory entry failed: -5
ERROR: searching directory /mnt/raid/hdd_data/download/mikan/魔女与使魔 failed: -5
```
最后去掉-D导出数据`sudo btrfs restore -i -t 569491456 /mnt/raid/hdd_bf/backup.img /mnt/raid/hdd_data`

## 参考
- [btrfs man](https://btrfs.readthedocs.io/en/latest/man-index.html)
- [btrfs - archlinux wiki](https://wiki.archlinuxcn.org/wiki/Btrfs)
- [tg message](tg://resolve?domain=archlinuxcn_group&post=3487314)