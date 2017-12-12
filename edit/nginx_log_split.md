---
categories: linux 

tags: 
  - Nginx
  - Linux

title: Nginx 日志切割

date: 2017-
---

# 前言

Nginx 是一个非常轻量的 Web 服务器，有体积小、性能高、速度快等诸多优点，但Nginx 产生的访问和错误日志默认写到access.log 和error.log 中，不利于日志分析，而且时间一长，日志文件会非常大。因此我们要对Nginx 日志进行手动切割。

