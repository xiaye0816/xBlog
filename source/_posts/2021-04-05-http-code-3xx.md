---
title: http 3xx重定向相关状态码
date: 2021-04-05 17:08:42
categories: http
tags: [http]
description: http code 301、302、303、307、308的区别
toc: true
---

## 综述
http状态码中，3xx相关的状态码，代表的是重定向。表示为了完成请求，需要客户端进行附加的请求操作。
本文将介绍一下301、302、303、307、308这几个http code的差异以及实现重定向时的最佳实践。

## 基础状态码

### 301 Moved Permanently

#### 设计目标
- 本次请求重定向到目标地址
- 浏览器缓存记录，下次直接请求新的目标地址
- 搜索引擎替换记录的url
- 重定向过程中，method和body保持不变

### 302 Found

#### 设计目标
- 本次请求重定向到目标地址
- 浏览器不需要缓存重定向记录，搜索引擎也不需要替换记录的url
- 重定向过程中，method和body保持不变

### 303 See Other

#### 设计目标
- 对于修改类请求（PUT、POST等）的资源，服务端不进行处理（可能是不符合条件，资源重复上传等），服务端将请求重定向到一个message页，客户端需要通过GET请求获取新的信息。

## 基础状态码总结
301和302一般代表的是永久跳转和临时跳转，永久跳转的话，client端就会缓存记录，下次遇到老的地址，直接本地就替换成新的地址了，不需要再发起两次请求。
例如，某些网站在开启全站https后，对于http的访问，会301到https域名上，例如：
首次访问http域名，返回301：
![](https://tva1.sinaimg.cn/large/008eGmZEly1gp8zxlfeo1j322g0todqk.jpg)
浏览器会转到新地址访问：
![](https://tva1.sinaimg.cn/large/008eGmZEly1gp8zxyn1vqj31su0t2qco.jpg)
下次再访问老地址时，本地就替换地址了，只会发起一次对新地址的请求：
![](https://tva1.sinaimg.cn/large/008eGmZEly1gp9006luimj323e0rodpi.jpg)

## 基础状态码的问题
虽然301和302已经满足了对于永久跳转和临时跳转的问题，但是在不同浏览器的实现里，对于设计目标中"重定向过程中，method保持不变"的部分，实现有所差异。
部分浏览器在发起重定向时，可能会把method改写成GET，哪怕首次请求的是PUT或者POST，这就导致了不确定性和潜在的问题。

## 解决方案
为了解决历史原因导致的不确定性问题，http1.1开始，新增了307和308两个状态码，作为302和301的替代。
这两个状态码强调了"重定向过程中，method和body保持不变"的语义。

### 307 Temporary Redirect
- 临时跳转，和302的设计目标相同
- 重点强调了，重定向过程中，method不变

### 308 Permanent Redirect
- 永久跳转，和301的设计目标相同
- 重点强调了，重定向过程中，method不变

## 最佳实践
- 如果是幂等类请求的重定向，如GET、HEAD等，使用301、302重定向
- 如果是非幂等类请求的重定向，如PUT、POST等，使用307、308重定向
- 明确需要将PUT、POST类请求降级为GET请求重定向的，使用303重定向

这样实现的优点是语义明确，可以防止不同浏览器之间实现导致的差异。