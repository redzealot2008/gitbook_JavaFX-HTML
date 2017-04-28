[原文链接](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/overview.htm#CEGHBDHF)

#JavaFX WebView组件概述

本章介绍了JavaFX嵌入式浏览器,一个提供了web查看器以及通过其API提供了完整的浏览功能的用户界面组件。

嵌入式浏览器组件基于WebKit,一个开源的web浏览器引擎。它支持级联样式表(CSS)、JavaScript、文档对象模型(DOM)和HTML5。

嵌入式浏览器允许您在JavaFX应用程序中执行以下任务:

- 从本地和远程URL渲染HTML内容

- 获得网络历史记录

- 执行JavaScript命令

- 从JavaScript向上调用JavaFX

- 管理web弹出窗口

- 应用效果到嵌入式浏览器

嵌入式浏览器继承了Node类所有字段和方法,因此有Node类的所有功能。

这些在javafx.scene.web包中的类构成了嵌入式浏览器。图2-1显示了嵌入式浏览器的体系结构,以及如何与其它JavaFX类关联。

图2-1 嵌入式浏览器的体系结构
![WebView组件的体系结构](http://ontjktddg.bkt.clouddn.com/image/webview-structure.png)

[图2-1 嵌入式浏览器体系结构的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-structure.htm)

##WebEngine类

WebEngine类提供了基本的web页面功能。它支持用户交互，如导航链接和提交HTML表单,尽管它不直接与用户交互。WebEngine类一次处理一个web页面。它支持基本浏览功能，加载HTML内容、访问DOM以及执行JavaScript命令。

两个构造函数允许您创建一个WebEngine对象:一个空的构造方法和一个指定URL的构造函数。如果你实例化一个空的构造函数,可以通过load方法传递URL给WebEngine对象。

从JavaFX SDK 2.2开始,开发人员可以启用和禁用JavaScript调用web引擎和应用自定义样式表。用户样式表替换默认页面样式由用户定义的WebEngine实例渲染。

##WebView类

WebView类是Node类的扩展。它封装了一个WebEngine对象,包含HTML内容到应用程序场景,并提供属性和方法应用效果和转换。调用WebView对象的getEngine()方法返回与之关联的web引擎。

[例2-1](#例2-1)展示了在您的应用程序中创建WebView和WebEngine对象的典型方式。

<span id="例2-1">例2-1</span> 创建WebView和WebEngine对象
```
WebView browser = new WebView();
WebEngine webEngine = browser.getEngine();
webEngine.load("http://mySite.com");
```
##PopupFeatures类

PopupFeatures类描述按照JavaScript规范定义的网页弹出窗口功能。当你需要在你的应用程序中打开一个新的浏览器窗口,这个类的实例传递给弹出处理程序，这个处理程序通过setCreatePopupHandler方法注册到WebEngine对象，如[例2-2](#例2-2)所示。

<span id="例2-2">例2-2</spam> 创建一个弹出处理程序
```
webEngine.setCreatePopupHandler(new Callback<PopupFeatures, WebEngine>() {
    @Override 
	public WebEngine call(PopupFeatures config) {                
        // do something
        // return a web engine for the new browser window
    }
});
```
如果方法返回同一个WebView对象的web引擎,目标文档是在同一个浏览器窗口中打开。为了在另一个窗口打开目标文档,指定另一个WebView的WebEngine对象。当你需要阻止弹出窗口,返回null值。

##其他特性

在使用WebView组件时,您应该记住它默认缓存在内存中。这意味着一旦包含WebView组件的应用程序关闭缓存的内容会丢失。不过,开发人员可以使用java.net.ResponseCache类来实现应用程序级别的缓存。从WebKit的角度来说,持久化缓存是网络层属性，类似于连接和cookie处理程序。一旦它们被安装,WebView组件以透明的方式使用它们。