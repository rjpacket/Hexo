---
title: 揭开Android热更新的神秘面纱——AndFix篇
date: 2016-09-20 15:10:04
tags: 热更新
---

## 简介
上一篇讲了`RoocoFix`的用法，有一个痛点就是项目必须混淆才能使用，[AndFix](https://github.com/alibaba/AndFix)就完全不受混淆的限制，下面介绍一下用法。

<!-- more -->

### 1.新建一个项目
名字随意，博客里用`AndFix`，然后首页一个`TextView`，图片如下：

![bug页面](/img/05/05-01.jpeg)

可以看到显示的文字为“Heelo word!”，两个单词都拼错了，如果这是已经上线的版本，紧急下架不划算，所以就有了补丁更新。
### 2.导入依赖
在`app.gradle`下加入
```gradle
    dependencies {
        compile 'com.alipay.euler:andfix:0.5.0@aar'
    }
```
在`application`初始化：
```java
    public class App extends Application {

        @Override
        public void onCreate() {
            super.onCreate();

            //初始化
            patchManager = new PatchManager(getApplicationContext());
            patchManager.init(getVersionCode() + "");//current version
            patchManager.loadPatch();
            try {
                patchManager.addPatch(Environment.getExternalStorageDirectory() + File.separator + "andfix" + File.separator + "patch.apatch");//path of the patch file that was downloaded
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
```
这里需要注意的是下载的文件放在哪里一定要能找得到，我放的是在`sdcard`下`/andfix/patch.apatch`。

### 3.build
做完这些工作，接下来就是运行了，也就是`build`，需要打出签名`debug`或者`release`包，`build`运行完，项目`app/build/outputs/apk`下会多出一个文件`app-debug.apk`，截图：

![](/img/05/05-02.png)

这是初始版本`old.apk`，也就是你的线上版本，接下来修改完`bug`，也就是将“heelo word！”改成“Hello world!”，这里注意千万不要改appVersionCode，保持和之前一致就行，
继续`build`，这时候在`app/build/outputs/apk`文件夹下多出来第二个文件`app-debug.apk`，覆盖了之前的，这就是补丁版本new.apk:

![](/img/05/05-02.png)

### 4.制作补丁包
这里需要下载一个工具：[apkpatch](https://github.com/alibaba/AndFix/raw/master/tools/apkpatch-1.0.3.zip)
![](/img/05/05-04.png)

在你解压的位置打开命令行窗口：
```
    apkpatch -f new.apk -t old.apk -o D:/Android/HotFix/out -k andfix.jks -p 123456 -a andfix -e 123456
```
敲完会在`D:/Android/HotFix/out`文件夹下生成补丁包：
![](/img/05/05-03.png)

### 5.上传补丁包，下载
这个过程需要后台的配合，但是可以将`patch.jar`放在`sdcard`文件夹下（没有支持从`assets`文件夹读取的api，只好手动）模拟下载好之后的过程，
我放在了`sdcard/andfix/patch.apatch`：

![](/img/05/05-06.jpg)

### 6.测试
将项目改回线上版本，在`Application`下初始化并加载`patch.apatch`包：
```java
    patchManager.addPatch(Environment.getExternalStorageDirectory() + File.separator + "andfix" + File.separator + "patch.apatch");
```
测试结果：
![](/img/05/05-05.jpg)

### 注意的要点
1.打断点确定是否读取到了补丁文件

2.一定要打签名包，敲命令行时需要签名文件信息

3.`application`注册到`manifest`文件里

4.不支持布局、资源文件的修改（好失望）

暂时这么多，有问题留言。

本篇demo源码已上传 [#github#](https://github.com/rjpacket/AndFixDemo.git)
