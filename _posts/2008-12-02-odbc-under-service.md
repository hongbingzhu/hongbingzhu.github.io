---
layout: post
title:  "服务运行状态下的ODBC数据库访问问题"
date:   2008-12-02 16:55:00
categories: desktop database
tags: odbc service
---

* content
{:toc}

将一个在调试下正常运行的ALT服务程序以服务的方式运行，发现程序出错。由于之前发现过VC8的服务部分的程序有点问题，所以认为可能是VC8服务部分程序问题，使用调试打印调试程序运行，经过好几小时的跟踪打印，最后发现是数据库访问的问题，数据库连接失败。上网搜索文章，在微软的一篇文章中提到权限问题，更改服务运行的用户为当前用户，发现可以正常运行。打开ODBC查看，原来数据源在用户数据源下，导致服务启动用户没有访问权限，改到系统数据源下，OK。