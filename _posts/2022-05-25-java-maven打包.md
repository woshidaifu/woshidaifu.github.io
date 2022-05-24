---
layout: archive
title: "java : maven打包"
categories:
  - java
tags:
  - java
  - maven
---
## 问题描述：

* 使用maven打包，普通maven项目未使用任何框架，pom文件中build插件混乱。

## 原因排查：
1. 对几种打包插件了解不足。

## 问题解决方案：
### 1. 使用maven-jar-plugin和maven-dependency-plugin插件打包。
    此种打包方案中，需要将main函数所在类在maven-jar-plugin的设置中声明清楚。
    且此方案打包，得到的jar包第三方依赖包在独立lib目录。
### 2. 使用maven-assembly-plugin插件打包。
    此种打包方案，可以讲jar包与第三方lib直接合并打成一个jar。

### 3. 还有其他插件未使用。

## 一点想法不一定对
* 习以为常的东西也要了解。