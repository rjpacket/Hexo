---
title: 实现外部url定位app某一activity进行跳转
date: 2016-09-13 09:19:11
tags: Hexo博客
---

## 为什么实现url跳转

实际是为了方便`h5`页面浏览器打开的时候进行跳转，或者推送消息打开指定的页面。有一个缺陷就是打开的页面是独立的，点击返回就直接退出`app`了，如果想点击返回回到`app`首页，就需要两次跳转，读者要是有什么好的方法，可以在下面留言告诉我。

### 1.导入依赖库，配置文件

在`project`下的`build.gradle`中添加
```gradle
    buildscript {
        dependencies {
            classpath 'com.neenbedankt.gradle.plugins:android-apt:1.7'
        }
    }
```

在`app`下的`build.gradle`中添加依赖
```gradle
   dependencies {
       apt 'com.github.mzule.activityrouter:compiler:1.1.1'
       compile 'com.github.mzule.activityrouter:activityrouter:1.1.1'
   }
```
`rebuild`,完成导入。

在`AndroidManifest.xml`里注册`ActivityRouter`，
```xml
    <activity
        android:name="com.github.mzule.activityrouter.router.RouterActivity"
        android:theme="@android:style/Theme.NoDisplay">
        <intent-filter>
            <action android:name="android.intent.action.VIEW" />
            <category android:name="android.intent.category.DEFAULT" />
            <category android:name="android.intent.category.BROWSABLE" />
            <data android:scheme="这个地方写自己app的协议名，类似http，比如：abc" />
        </intent-filter>
    </activity>
```

### 2.修改需要绑定的`Activity`
比如常见的`HomeActivity`，
```java
    @Router("main")
    public class MainActivity extends Activity {

    }
```
现在你可以一句话进行跳转了，
```java
    Routers.open(context,"abc://main");
```
是不是很神奇。

但有时候我们需要带参数过去，比如查看订单的时候，需要订单`id`，没事`url`支持带参数跳转，
```java
    @Router("order_detail/:orderId")
    public class OrderDetailActivity extends Activity {
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            String userId = getIntent().getStringExtra("orderId");
        }
    }
```
一句话，
```java
    Routers.open(context,"abc://order_detail/7788");  //这个7788就是订单号7788
```
当然还有指定参数的类型，不介绍了，自己去看吧。

## 结语

感谢你看到这里，是不是没听懂，没事，有库的地址，这个库名字是 [ActivityRouter](https://github.com/mzule/ActivityRouter)。喜欢就打赏吧。