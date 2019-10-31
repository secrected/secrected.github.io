---
title: Linux文本查找常用命令：grep | less
date: 2019-05-02 16:53:49
categories: [Java,Tools]
tags: grep
top: 0
---

> grep全称: Global search Regular Expression and Print out the line
> 可理解为：利用正则表达式进行全局搜索的文本处理工具

查日志最常用日志不过  `tail -f   redis.conf` 或者 `less redis.conf`一类的，但通常数据量大还需借助些精确查找等命令

<!--more-->

#### grep

| function  | terminal                                | description                                |
| --------- | --------------------------------------- | ------------------------------------------ |
| 不区分大小写    | grep -i  -n  "start"  stdout.log        | 不加-i默认区分大小写，-n显示行号                         |
| 显示前后几行    | grep -a3  "ScanCode.java:36" stdout.log | 前后三行，等于 -C3；-A3  、 -B5 前5后3行（after，before） |
| 精确匹配      | grep  -w  "ScanCode" stdout.log         |                                            |
| 不包含的行     | -v                                      |                                            |
| 支持扩展正则表达式 | -E                                      | 不加grep支持基本正则表达式                            |
| 符合条件的行数   | grep  -c  "ScanCode"  stdout.log        | 统计                                         |

最常可用的，别的不啰嗦，前后三行效果如图：

![](/pho/article/grep.png)



#### less命令查找

linux正统查看文件内容的工具，更简单，less redis.conf后操作：

| description                            | terminal      | remak        |
| -------------------------------------- | ------------- | ------------ |
| g            跳到第一行；G            跳到最后一行 | g   \|    G   |              |
| 查找字符串                                  | /no  \|   ?no | 回车后控制n向上和N向下 |
| 向前移动一屏                                 | ctrl + F      |              |
| 向后移动一屏                                 | ctrl + B      |              |

按 F 可以实现类似 tail -f 的效果。ctrl+c退出
