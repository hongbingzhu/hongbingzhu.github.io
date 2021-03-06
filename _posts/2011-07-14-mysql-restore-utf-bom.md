---
layout: post
title:  "MySQL恢复和UTF文件BOM标志读取问题"
date:   2011-07-14 14:23:26
categories: desktop database
tags: mysql utf encoding
---

* content
{:toc}

前天，客户来电说GPS巡线系统的后台数据库挂了，原因是原来分配给图片存储磁盘的空间不够了，他调整了一下分区。我让他把整个MySQL目录的文件备份下来（后来发现这是多么重要），然后重装一下数据库试试。重装以后发现不行。于是给我让他发了MySQL下面的data目录文件给我，还有今年二月份用后台软件备份下来的数据备份也发了过来。

网上查询发现MySQL的数据存储在data目录下的ibdata1文件中，使用UE打开该文件，发现全是0，显然不能包含数据，对比公司内部测试使用的MySQL服务器，确认该文件用于存储数据库数据。看来只能是使用二月份的备份数据了。

打开二月份备份数据，发现数据库的字符集用的是binary，这是当时不太清楚字符集该怎么设，从别人那里继承来的。试图重装一个MySQL，设定为binary，恢复还是有错误。在MySQL命令行中手动执行恢复操作，提示是数据库字段宽度不够，手动调整数据库字段大小，成功走完恢复过程，但是恢复出来的数据显示是乱码。

用UE打开备份文件，发现前面是FEFF，UTF的编码标记，后面是有规律的一个字节数据，一个字节的零，手动提取字节数据，恢复出来时可读的，于是想利用程序提取出来非零数据，然后恢复。结果，发现程序死活不能读到标记字节FEFF。想可能VC2008+Win7可能自动给你处理了FEFF，于是上VMWare+DOS622+TC2，结果还是一样，查到网上有一篇文章 [How NOT to skip BOM info (FF FE) when using fread or ifstream?](http://stackoverflow.com/questions/2475170/how-not-to-skip-bom-info-ff-fe-when-using-fread-or-ifstream) 这个现象一摸一样。就是没有解答。

早晨又想到一个方法，利用文件传输，这下该啥码都出来了吧，于是上串口互联，一个串口打开串口调试软件，另一个串口copy，发现接收出来数据不对，MODE命令设之，发现收到的数据和UE的还是不一样啊。对比利用EmEditor二进制打开的文件，却发现串口接收的和EmEditor的一样。对比EmEditor的字节数和文件属性字节数，一致。验证C程序，一致。用EmEditor给文件加上UTF编码标记，C程序读取，可以读取到BOM数据。最后验证UE中间做了手脚，天啊。哎，太相信UE了。

MySQL最终恢复却都没用到这些，呵呵。今天客户又打电话过来，问我怎么样了，我交流以后发现，他还是按我说的备份了全部的MySQL安装目录，还有，他提到其他盘有一个ibdata1文件，时间太长了，我都忘了，应该是当时我为了数据安全，安装的时候将数据文件放到别的分区中了。这下好了，分析客户发过来的全部资料，清楚了原来安装MySQL的原始结构，并且发现原来的MySQL配置文件的MySQL自动备份存在MySQL目录中。于是就超级简单了。停止MySQL服务，移掉MySQL相关目录，解压客户发过来的数据到MySQL安装目录中，修改配置文件，在相应的路径中放置ibdata1，重启服务，OK，一切正常。

想想，Linux系的软件还是好啊，恢复起来赶紧利落。