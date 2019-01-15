---
title: Maven基础应用
date: 2019-01-14 21:26:16
tags: 工具
---

maven-model-builder-3.3.9.jar/org/apache/maven/model中有超级pom

mirror标签配置私服

proxy标签配置代理

pluginGroups插件

servers配置私服用户名密码等

profiles，activeProfile配置不同环境



packaging，jar、war、pom、maven-plugin

dependencyManagement：只能出现在父pom工程，不是给子类直接用的，需要使用还是需要自己引用，类似抽象类，可以对版本进行控制

scope：

​	compile 编译（默认）

​	test 测试

​	provided 编译，Servlet，打包时不会打进去

​	runtime 运行时有效，编译时可以不用，jdbc驱动实现

​	system 本地一些jar加上systemPath

mvn dependency:tree > d.txt

依赖仲裁：最短路径，pom书写顺序，exclusions排除包

 生命周期：

​	clean:pre-clean,clean,post-clean

​	default:compile,package,install,deploy....

​	site:pre-site,site,post-site,site-deploy



mvn clean package -U 强制从版本库拉一次

常用命令：

​	clean:删除target

​	compile:编译

​	test:运行项目中所有的测试用例

​	install:	将模块安装到本地仓库

​	package:打包

​	deploy:把本地jar发布到远程仓库中

mojo，findbugs，versions，source