---
title: Maven 简介
date: 2017-08-23 14:28:20
tags: 
    - Maven
---

## Maven 是什么？
一个软件项目管理和理解的工具

是能够用于构建系统的自动化工具

## Maven 有哪些功能？
* 构建：生成 class、jar、war、ear 等文件
* 生成文档：生成 javadoc 、网站文档 等
* 生成报告：生成 junit 测试报告 等
* 生成依赖类库：生成文档，说明项目对其他软件的依赖
* SCM：SCM（Software Configuration Management），软件配置管理，如版本控制、bug 管理 等
* 发布：生成供发布的分发包，如 Struts2 的分发包
* 部署：比如可将 web 应用程序自动部署到指定的服务器上
* 其他：由于有各种插件的支持，Maven 的功能可不断扩展。

## Maven 的结构
* 插件（Plugin）式架构
* 所有插件本身也是一个 Maven 架构，由仓库管理
* 每个插件提供多个目标（Goal）
 * 调用目标的格式 `mvn <Plugin>:<Goal>`

## 如何查看一个 Maven 插件的帮助文档
查看插件的帮助信息：`mvn <Plugin>:help`，比如：`mvn dependency:help` 或 `mvn ant:help` 等等。

例：查看 dependency 插件的帮助文档，使用命令
``` bash
$ mvn dependeny:help
```

>……
[INFO] org.apache.maven.plugins:maven-dependency-plugin:2.1
>
Maven Dependency Plugin 2.1
Provides utility goals to work with dependencies like copying, unpacking, analyzing, resolving and many more.
>
This plugin has 18 goals:
>
dependency: analyze
Analyzes the dependencies of this project and determines which are: used and declared; used and undeclared; unused and declared. This goal is intended to be used standalone, thus it always executes the test-compile phase - use the dependency:analyze-only goal instead when participating in the build lifecycle.
……

PS：鉴于网络问题可先将源改为开源中国的镜像库。