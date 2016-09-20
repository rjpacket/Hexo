---
title: 揭开Android热更新的神秘面纱——RoocoFix篇
date: 2016-09-18 14:38:56
tags: 热更新
---

## 简介
[RoocoFix](https://github.com/dodola/RocooFix)是基于`HotFix`的一个框架，作者说`HotFix`过于简单，一年以前比较流行的有阿里的`dexposed`、大众点评的`Nuwa`,但是`Nuwa`限制比较多，5.0以后不支持，`HotFix`也已经过时了，今天介绍一下[RoocoFix](https://github.com/dodola/RocooFix)。

<!-- more -->

### 1.新建一个项目
名字随意，博客里用`RoocoFix`，然后首页一个`TextView`，图片如下：

![bug页面](/img/04/04-01.jpeg)

可以看到显示的文字为“Heelo word!”，两个单词都拼错了，如果这是已经上线的版本，紧急下架不划算，所以就有了补丁更新。
### 2.导入依赖
在根`build.gradle`下加入
```gradle
    dependencies {
        classpath 'com.dodola:rocoofix:1.2.6'
    }
```
在`app.gradle`下加入
```gradle
    apply plugin: 'com.dodola.rocoofix'

    repositories {
        jcenter()
    }

    rocoo_fix {
        includePackage = ['com/taovo/rjp/roocofix']//指定将来可能需要制作补丁的package(就是指定插庄的范围)
        excludeClass = ['App.class']//将不需要加到patch里的类写在这里(不需要插庄的类)

    //    preVersionPath = '1'//注意：此项属性只在需要制作补丁的时候才需开启！！如果不需要制作补丁则需要去掉此项

        enable = true//注意：关掉此项会无法生成Hash.txt文件

    //    scanref = true//默认为 false，开启这个选项会将与补丁 class 相引用的 class 都打入包中来解决 ART 虚拟机崩溃问题，功能 Beta 中
    }

    dependencies {
        compile 'com.dodola:rocoo:1.1'
    }
```
在`application`初始化：
```java
    public class App extends Application {

        @Override
        public void onCreate() {
            super.onCreate();

            //初始化
            RocooFix.init(this);
            RocooFix.initPathFromAssets(getApplicationContext(), "patch.jar");
        }
    }
```
上面的跟项目已有的叠加在一起。

### 3.build
做完这些工作，接下来就是运行了，也就是`build`，这里非常的诡异，直接点绿色的三角形是没有用的，照步骤来，首先开启混淆，增加混淆语句：
```java
    -keep class com.dodola.rocoofix.** {*;}
    -keep class com.lody.legend.** {*;}
    -keepclassmembers class com.dodola.rocoosample.** {
      public <init>();
    }
```
然后点击右边的`Gradle`下的`build`，如下截图所示：

![](/img/04/04-02.png)

`build`运行完，项目`app`下会多出一个文件夹`rocoofix`，截图：

![](/img/04/04-03.png)

这是初始版本`version1`，也就是你的线上版本，接下来修改完`bug`，也就是将“heelo word！”改成“Hello world!”，然后将你的`appVersionCode`增加，保持和之前不一致就行，再打开`app.gradle`里的preVersionPath = '1'，
也就是声明这是一个补丁包，“1”是声明前一个版本，同理，继续`build`，这时候在`rocoofix`文件夹下多出来第二个子文件夹`version2`，并且多一个`patch.jar`，这就是补丁包，自动化生成的：

![](/img/04/04-05.png)

### 4.上传补丁包，下载
这个过程需要后台的配合，但是可以将`patch.jar`放在`assets`文件夹下模拟下载好之后的过程：

![](/img/04/04-04.png)

### 5.测试
将项目改回线上版本，在Application下初始化并加载patch.jar包：
```
    RocooFix.initPathFromAssets(getApplicationContext(), "patch.jar");
```
测试结果：
![](/img/04/04-06.jpeg)

### 注意的要点
1.build.gradle版本最好2.1.2以上

2.一定要开启混淆

3.application注册到manifest文件里

暂时这么多，有问题留言。下一篇介绍阿里系的[AndFix](https://github.com/alibaba/AndFix)。

本篇demo源码已上传 [#github#](https://github.com/rjpacket/RoocoFixDemo)





