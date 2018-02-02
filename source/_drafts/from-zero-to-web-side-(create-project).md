---
title:
- 从零开始做论坛之创建工程
date:
- 2016-12-07 23:08:18
tags:
- Java
- Web
- Spring
- SpringMVC
- Maven
- Mybatis
---

## 0x00 准备弄一个论坛
准备弄一个前后分离的BBS论坛，后端用常规JavaWeb来写写
SpringMVC + Mybatis + Redis
前端暂时没定，预计是Angular2 或者Vue。
开源，第一阶段可能会比较慢。主要用来设计网站，和数据库。现在还毫无头绪，也没想好具体怎么捯饬。

## 0x01 开坑，创建项目
尽量稳一些，不使用最新的改装框架，只用最新的原始框架。
先创建Maven工程，引入Spring支持，和Mybatis持久层框架，