---
layout: post
title: 'flutter project setup'
date: 2022-2-1 20:20:10
author: 'Youzi'
catalog: true
tags:
  - flutter
  - 项目配置
---

# flutter项目初始化配置

从`AS`打开项目后，遇到的各种各样的报错，记录一下debug的过程；

## gradle和JDK版本问题

官网兼容性配置查询：[gradle/JDK版本兼容性](https://docs.gradle.org/current/userguide/compatibility.html#compatibility)

在打开项目后，AS会自动拉取依赖，可能会报gradle版本和JDK版本存在不兼容的情况，参考上面的兼容性表格，在路径`gradle/wrapper/gradle-wrapper.properties`下，修改`distributionUrl`链接中，gradle的版本，然后点击`file/sync project with gradle files`，重新下载对应版本的gradle即可；

```
distributionUrl=https\://services.gradle.org/distributions/gradle-7.3-all.zip
```

## `flutter SDK`找不到

在`local.properties`配置中，添加一下flutter SDK的路径就好：

```
flutter.sdk=D:\\flutter
```

## `Failed to read key AndroidDebugKey from store`

报错信息：`Failed to read key AndroidDebugKey from store`，找到对应的路径，一般是`c:/users/xxx/.android/debug.keystore`，删除该文件，然后重新debug运行项目，此时AS会重新生成一次这个文件，就可以重新运行了。
