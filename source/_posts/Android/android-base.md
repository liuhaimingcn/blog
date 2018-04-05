title: Android 基础
date: 2016-5-28  
tags:
    - note  
categories:
    - Android
---

### Android 开发的基本环境  
* JDK (Java Development Kit)  
* IDE (Android Studio)  
* Android SDK (Android Software Development Kit)  
* ADT (Android Development Tools)  

<!-- more -->

### ADT 和 Android SDK的区别
* ADT(Android Development Tools)： 目前Android开发所用的开发工具是Eclipse，在Eclipse编译IDE环境中，安装ADT，为Android开发提供开发工具的升级或者变更，简单理解为在Eclipse下开发工具的升级下载工具。ADT只是一个Eclipse的插件，里面可以设置sdk路径。     
* SDK(Software Development Kit)： 一般是一些被软件工程师用于为特定的软件包、软件框架、硬件平台、操作系统等建立应用软件的开发工具的集合。在Android中，他为开发者提供了库文件以及其他开发所用到的工具。简单理解为开发工具包集合，是整体开发中所用到的工具包，如果你不用Eclipse作为你的开发工具，你就不需要下载ADT，只下载SDK即可开发。  

### JDK 和 JRE的区别
* JRE(Java Runtime Environment): 顾名思义是java运行时环境，包含了java虚拟机，java基础类库。是使用java语言编写的程序运行所需要的软件环境，是提供给想运行java程序的用户使用的。  
* JDK(Java Development Kit): 顾名思义是java开发工具包，是程序员使用java语言编写java程序所需的开发工具包，是提供给程序员使用的。
* JDK包含了JRE，同时还包含了编译java源码的编译器javac，还包含了很多java程序调试和分析的工具：jconsole，jvisualvm等工具软件，还包含了java程序编写所需的文档和demo例子程序。如果你需要运行java程序，只需安装JRE就可以了。如果你需要编写java程序，需要安装JDK。

### Android项目目录结构
* src	放java代码的目录
 * gen目录下的文件是编译器生成
* assets 资源目录，例如音频、图片、xml（不一定要打入apk包中）
 * bin存放编译后的.class .dex  .apk文件的目录，编译器生成
 * libs放第三方jar包
 * res资源目录 （都要打入apk包中）
 * drawable（根据名字存放不同分辨率的图片，Android系统为了适配移动设备会根据设备的DPI去对应的目录选择图片）
  * Drawable-hdpi 存放高分辨率图片；
  * Drawable-ldpi 存放低分辨率图片；
  * Drawable-mdpi 存放中分辨率图片；
  * Drawable-xhdpi 存放中高分辨率图片；
  * Drawable-xhdpi 存放特高分辨率图片。

 * layout 布局文件，Android系统为了使控制层和View层做分离，对一些静态的界面尽量写成xml文件的形式放在Layout文件夹下。
 * menu 存放菜单文件
 * values 存放文字信息配置
  * dimens.xml文件存放一些尺寸信息，为了适配屏幕用；
  * string.xml文件存放文本信息；
  * styles.xml 文件中定义了一些属性集，方便复用和修改。
 * AndroidManifest.xml 清单文件，这个文件列出了应用程序所提供的功能，需要什么权限，用到那些服务，当前应用的版本，最低支持android版本，应用的名称、图标和包名，有那些组件，每个组件的配置信息

<br>