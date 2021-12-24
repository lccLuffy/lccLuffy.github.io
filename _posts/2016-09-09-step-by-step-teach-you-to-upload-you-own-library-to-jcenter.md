---
layout: post
permalink: /step-by-step-teach-you-to-upload-you-own-library-to-jcenter/
title: 一步一步教你上传自己的 Library 到 JCenter
date: 2016-09-09T23:16:00
lang: zh-CN
---

我们在Android开发过程中一定会用到别人的库，比如squareup公司的OKHttp：
```
compile 'com.squareup.okhttp3:okhttp:3.4.1'
```
这样我们版本更新的时候只需要更改一下版本号就行，而不用去下载jar包，给开发带来了极大的便利，但如果我们自己想上传lib供其他开发者使用呢？那么此教程会带着你一步一步发布自己的library。Let's go!


### 确定要上传的Library
如果你有Library可以忽略此步骤，没有的话添加library。在Android Studio中选择File->New->New Module,然后选择一个Library,新建一个Library。这里以新建DemoLibrary为例子。**（注意这里的Library需要后面的Package的名字一致）。**
![新建一个Library](https://static.lufficc.com/image/ad2a895fc8eca0f284fafab1c0ce01d8.png)
![新建一个Library](https://static.lufficc.com/image/a7e0fe6cd124b7e5d4fcf41613bef593.png)

现在项目的结构如下图，接下来就是添加必要的Jcenter的依赖，为上传做准备。
![项目的结构](https://static.lufficc.com/image/d481518bb518b985a5653280b7509ec7.png)



 ******************************************************************* 



### 注册账号
首先去[bintray官网](https://bintray.com/)注册账号，注册完成后验证邮箱，然后登陆进入首页点击View All，选择Maven仓库，新建一个Package，填写Package名字**（注意Package需要和你的Library的名字一致）**

![](https://static.lufficc.com/image/5537a5ba30dbdd49baab5e2b507ea1b7.png)


注意点击你的头像->Your Profile->Edit->Api key,这个先记下来，后面上传要用到。
![Api key](https://static.lufficc.com/image/286181bbe322d773d0c3e489ebf26fb9.png)

新建一个Package
![新建一个Package](https://static.lufficc.com/image/6c39acaf0cbf63f4b85a23f7eeb7eac2.png)
![新建一个Package](https://static.lufficc.com/image/cdbbf4669ea432e5dc75e0cef161a324.png)



 ******************************************************************* 



### 添加依赖
在整个工程的build.gradle文件中添加`classpath 'com.novoda:bintray-release:0.3.4'`,***注意是整个工程的build.gradle***。

![添加依赖](https://static.lufficc.com/image/eafea98913c122a593384370fdf8c5f0.png)

接着是在你自己Library（这里是DemoLibrary）的build.gradle的文件中配置自己的信息，复制下面的脚本，改成你自己的信息即可
``` gradle
apply plugin: 'com.android.library'
apply plugin: 'com.novoda.bintray-release'

publish {
    userOrg = 'lufficc' //你的用户名
    groupId = 'com.lufficc' //你的唯一的groupId，对应com.squareup.okhttp3:okhttp:3.4.1中的com.squareup.okhttp3
    artifactId = 'DemoLibrary' //你的library的名字，对应com.squareup.okhttp3:okhttp:3.4.1中的okhttp
    publishVersion = '0.0.1' //版本号
    desc = 'This is a demo library to teach how to publish you own library to jcenter with android studio.'
    website = 'http://lufficc.com/' //建议填写github地址，不过不影响，这里做演示填的自己的网址
 
    bintrayUser = 'lufficc' //你的用户名
    bintrayKey = 'Your api key' //在你的账户里面查找
}
```
经过上面的配置，上传成功后那么别人引用你的library的代码就为`compile 'com.lufficc:DemoLibrary:0.0.1'`。

![你自己Library的build.gradle的文件中配置自己的信息](https://static.lufficc.com/image/c864b7dae97021dea6660bb743dd0e1d.png)



 ******************************************************************* 


### 上传
经过上面的配置，现在就可以传了，上传之前记得Sync一下Project,然后打开命令行，输入,回车：
``` gradle
gradlew clean build bintrayUpload -PdryRun=false
```
然后等待几分钟，期间会联网下载依赖的库，最后如果没有问题，会显示BUILD SUCCESSFUL信息，然后去官网查看刚才建的Package，会发现多了你刚才上传的版本号。

![BUILD SUCCESSFUL](https://static.lufficc.com/image/74381d1a6a49480511a2106eea9ed822.png)
![上传成功的Package](https://static.lufficc.com/image/ce3209d9745e933b7a49e799a6d26ffe.png)

点进去可以看到有三种引用方式：
![](https://static.lufficc.com/image/cbd39db7be19a0930c6c37e95fe7b18e.png)

但是到这里还无法让别人也能引用，目前只是你自己的私人库。下面是添加到Jcenter,非常简单。



 ******************************************************************* 


### 添加到Jcenter
在上面的页面中点击Add To JCenter，然后随便填写一下comments，点击send，然后工作人员会审核和，你只需等待几个小时，然后会有站内消息提示你已经发布发到Jcenter，这样别人也可以引用你的Library，有没有很自豪的感觉！
![](https://static.lufficc.com/image/fe06887af70db608557a387b2e896e58.png)
![](https://static.lufficc.com/image/06321bb78d2bb15d8b10b1afa0c13f31.png)



 ******************************************************************* 


### 更新版本号
这个非常简单，当你的Libraryd代码更改后，只需要更改一下上面的配置里面的`publishVersion`，运行`gradlew clean build bintrayUpload -PdryRun=false`，就可以更新版本号了。这样，整个过程就结束了，遇到什么问题欢迎评论提出或者私信我。


 ******************************************************************* 

### 总结
> 其实上传没那么复杂

1. 注册账号
1. 为自己的Library项目添加依赖，配置信息
1. 上传，添加到Jcenter
1. 更新版本号



 ******************************************************************* 


### 常见问题
1. 如果你的Java doc含有中文导致上传失败，可以尝试在lib的build.gradle添加如下代码：
``` gradle
 allprojects {
    tasks.withType(Javadoc) {
        options{
            encoding "UTF-8"
            charSet 'UTF-8'
            links "http://docs.oracle.com/javase/7/docs/api"
        }
    }
}
```
1. 本教程是基于插件[novoda/bintray-release](https://github.com/novoda/bintray-release)的，更多问题可以查看[issues](https://github.com/novoda/bintray-release/issues)或者查看[Wiki](https://github.com/novoda/bintray-release/wiki)。
