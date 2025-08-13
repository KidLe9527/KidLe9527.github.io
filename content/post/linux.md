---
title: 'linux学习过程心得体会'

data: 2025-08-11T10:25:00-08:00
lastmod: 2025-08-10T22:02:00-08:00
categories: ['笔记']
tags: ['linux']
cover: https://kidle9527.github.io/images/44.png
---

# LINUX 学习笔记

## Linux命令基础格式

```
command [-options] [parameter]
```

* command：命令本身
* -options：[可选，非必填]，命令的一些选项，可以通过选项控制命令的行为细节
* parameter：[可选，非必填]，命令的参数，多数用于命令的指向目标

### ls命令

1. ls 命令的参数作用，可以指定要查看的文件夹（目录）的内容，如果不给定参数，默认当前工作目录的内容
2. ls 的命令选项：
   * -a：可以展示出隐藏的内容
     * 以`.`开头的文件或文件夹默认被隐藏，需要-a才能显示出来
   * -l：以列表的形式展示内容，并展示更多细节
   * -h：需要和-l 选项搭配使用，以更加人性化的方式显示文件的大小单位
3. 命令的选项是可以组合使用的，比如：`ls -lah = ls -a -l -h`

### cd命令

1. cd命令的作用

   * cd命令来自英文：Change Directory.

   * cd命令可以切换当前工作目录,语法是：
     * 没有选项，只有参数，表示目标路径
     * 使用参数，切换工作目录到指定路径（默认切换到当前用户的HOME）

```
cd [linux路径]	
```

2. pwd 命令的作用
   * pwd命令来自英文：Print Work Directory
   * pwd命令，没有选项，没有参数，直接使用即可
   * 作用：输出当前所在的工作目录

### 相对路径、绝对路径和特殊路径符

1. 绝对路径：以根目录作为起点，路径必须要以 / 开头
2. 相对路径：以当前目录作为起点，描述路径的方式，路径不需要以 / 开头
3. 特殊路径符
   * `.`：表示当前目录，eg`cd ./Desktop` 表示切换到当前目录下的Desktop 目录内
   * `..`：表示上一级目录，eg `cd ../..` 表示切换到上二级目录
   * `~`：表示HOME目录， eg `cd ~/Desktop = cd ./Desktop` 切换到切换到HOME下的Desktop目录

### mkdir命令

1. mkdir命令来自英文：Make Directory
2. mkdir命令可以创建新的目录和文件夹
3. 语法：`mkdir [-p] linux路径`
   * 参数必填，表示linux路径，即要创建的文件夹路径
   * -p 选项可选，表示自动创建不存在的父目录，适用于创建连续多层级的目录
4. 注意：创建文件夹需要修改权限，初学请确保在HOME目录下操作

* tips: Mobaxterm的命令行可以用指令`clear`清空

### 文件操作命令

1. touch 命令创建文件
   * 语法：`touch linux语法`
   * touch 命令无选项，参数必填，表示要创建的文件路径
   * eg ：` touch ./wjl/test.txt` 表示在HOME目录下的wjl文件夹创建test.txt 文件
2. cat 命令查看文件内容
