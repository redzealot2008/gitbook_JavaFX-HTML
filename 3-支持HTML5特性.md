[原文链接](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/html-five.htm)

#支持HTML5特性

本章描述了JavaFX web组件支持的HTML5特性范围。大多数支持的功能是WebEngine类和WebView类的实现的一部分，这些功能没有公共API。

JavaFX web组件的当前实现提供以下HTML5特性支持:

- 画布和SVG（可缩放矢量图形Scalable Vector Graphics) 

- 媒体播放

- 表单控件

- 历史维护

- 交互元素标签

- DOM（文档对象模型Document Object Model）

- Web workers

- Web sockets

- Web字体

##画布和SVG

支持canvas和svg元素标签允许基本的图形功能包括图形渲染，使用可缩放矢量图形（SVG）语法构建形状，应用颜色设置，视觉效果和动画。[例3-1](#例3-1)提供了一个简单的测试，使用canvas和svg标签来渲染web组件。

<span id="例3-1">例3-1</span> 使用画布和SVG元素
```
<!DOCTYPE HTML>
<html>
    <head>
        <title>Canvas and SVG</title>
        <canvas style="border:3px solid darkseagreen;" width="200" height="100">          
        </canvas> 
        <svg>
            <circle cx="100" cy="100" r="50" stroke="black" 
                stroke-width="2" fill="red"/>
        </svg> 
    </body>
</html>
```
当你使用例3-1的HTML代码加载到WebViewSample应用程序中,它渲染图形如图3-1所示。

图3-1 渲染图形
![](http://ontjktddg.bkt.clouddn.com/image/webview-canvas.png)

[“图3-1 渲染图形”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-canvas.htm)

##媒体播放

WebView组件使您能够在HTML页面中播放视频和音频内容。编解码器支持如下:

- 音频:AIFF、WAV(PCM)、MP3、AAC

- 视频:VP6、H264

- 媒体容器:FLV、FXM、MP4和MpegTS(HLS)

关于嵌入媒体内容的更多信息,请参见HTML5规范[video](http://www.w3.org/TR/html5/embedded-content-0.html#the-video-element)和[audio](http://www.w3.org/TR/html5/embedded-content-0.html#the-audio-element)标签。

##表单控件

JavaFX web组件能够渲染表单和处理数据输入。支持的表单控件包括文本字段、按钮、复选框和其他可用的输入控件。[例3-2](#例3-2)提供了一组简单的控件,使您能够输入一个问题摘要并指定其优先级。

<span id="例3-2">例3-2</span> 表单输入控件
```
<!DOCTYPE HTML>
<html>
<form>
 <p><label>Login: <input></label></p>
 <fieldset>
  <legend> Priority </legend>
  <p><label> <input type=radio name=size> High </label></p>
  <p><label> <input type=radio name=size> Medium </label></p>
  <p><label> <input type=radio name=size> Low </label></p>
 </fieldset>
</form>
</html>
```
当[例3-2](#例3-2)的HTML内容载入WebView组件，产生的输出如[图3-2](#图3-2)所示。

<span id="图3-2">图3-2</span> 渲染表单元素
![](http://ontjktddg.bkt.clouddn.com/image/webview-form.png)

[“图3-2 渲染表单元素”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-form.htm)

关于用户如何使用表单控件提交数据和处理数据的更多信息,请参见[HTML5规范](http://www.w3.org/TR/html5/forms.html#forms)。

##历史维护

你可以通过使用javafx.scene.web包中的WebHistory类获得访问过的页面列表。WebHistory类代表关联WebEngine对象的会话历史。

WebViewSample应用程序中启用了此功能,您将学会使用JavaFX web组件的功能。参见[管理Web历史记录](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/history.htm#CBAGIBCH)章节的实现细节。

##支持交互式元素标记

WebView组件提供支持交互式HTML5元素，比如details、summary、command、menu。[例3-3](#例3-3)展示了details和summary元素如何可以渲染在web组件中。示例还使用progress和meter控件元素。

<span id="例3-3">例3-3</span> 使用Details，Summary，Progress，和Meter元素
```
<!DOCTYPE HTML>
<html>
<h1>Download Statistics</h1>
 <details>
  <summary>Downloading... <progress max="100" value="25"></progress> 25%</summary>
  <ul>
   <li>Size: 1,7 MB</li>
   <li>Server: oracle.com</li>
   <li>Estimated time: 2 min</li>   
  </ul>  
 </details>
<h1>Hard Disk Availability</h1> 
  System (C:) <meter value=240 max=326></meter> </br>
  Data (D:) <meter value=101 max=130></meter>
</html>
```
当web组件加载这个页面时，WebViewSample应用程序如[图3-3](#图3-3)所示。

<span id="图3-3">图3-3</span> 渲染交互式HTML5元素
![](http://ontjktddg.bkt.clouddn.com/image/webview-interactive-tags.png)

[“图3-3 渲染交互式HTML5元素”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-interactive-tags.htm)

关于交互式元素属性的更多信息请参见[HTML5规范](http://www.w3.org/TR/html5/interactive-elements.html#interactive-elements)。

##文档对象模型

WebEngine对象,作为JavaFX web组件不可见的一部分,可以创建和提供访问web页面的文档对象模型(DOM)。文档模型的根元素可以通过使用WebEngine类的getDocument()的方法来访问。[例3-4](#例3-4)提供了一个代码片段来实现一些简单的任务:获取一个URI的web页面并显示在应用程序窗口的标题。

<span id="例3-4">例3-4</span> 从DOM派生出URI
```
WebView browser = new WebView();
WebEngine webEngine = browser.getEngine();
stage.setTitle(webEngine.getDocument().getDocumentURI());
```
此外,文档模型事件规范支持在JavaFX代码中定义事件处理程序。请参见WebEngine类规范中附加一个事件监听器到web页面元素的示例。

##Web Sockets

WebView组件支持WebSocket接口使JavaFX应用程序能够与服务器进程建立双向沟通。WebSocket接口详细描述在[WebSocket API规范](http://www.w3.org/TR/websockets/)中。[例3-5](#例3-5)展示了使用web sockets的常见模型。

<span id="例3-5">例3-5</span> 在HTML代码中使用网络套接字
```
<!DOCTYPE HTML>
<html>
<head>
<title>Web Worker</title>
</head>
<body>
<script>
    socketConnection = new WebSocket('ws://example.com:8001');    
    socketConnection.onopen = function () {
        // do some stuff
    };
</script>
</body>
</html>
```
##Web Workers

JavaFX web组件支持并行运行web worker脚本动态加载到web页面中。此功能允许长时间运行的脚本执行不需要等待用户交互。

[例3-6](#例3-6)展示了web页面使用myWorker脚本运行长时间任务。

<span id="例3-6">例3-6</span> 使用Web Worker脚本
```
<!DOCTYPE HTML>
<html>
<head>
<title>Web Worker</title>
</head>
<body>
<script>
   var worker = new Worker('myWorker.js');
   worker.onmessage = function (event) {
     document.getElementById('result').textContent = event.data;
   };
</script>
</body>
</html>
```
从[HTML5规范](http://www.w3.org/TR/workers/)中了解更多的web worker脚本相关信息。

##Web字体支持

JavaFX web组件支持web字体声明使用@font-face规则。这条规则允许需要时自动下载链接字体。根据HTML5规范,此功能提供能力选择字体匹配给定页面的设计目标。[例3-7](#例3-7)的HTML代码使用@font-face规则链接指定URL的字体。

<span id="例3-7">例3-7</span> 使用Web字体
```
<!DOCTYPE HTML>
<html>
  <head>
    <title>Web Font</title>
    <style>
      @font-face {
        font-family: "MyWebFont";
        src: url("http://example.com/fonts/MyWebFont.ttf")
      }
      h1 { font-family: "MyWebFont", serif;}
    </style>
  </head>
  <body>
    <h1> This is a H1 heading styled with MyWebFont</h1>
  </body>
</html>
```
当这段HTML代码加载到WebViewSample应用程序,渲染如[图3-4](#图3-4)所示。

<span id="图3-4">图3-4</span> 渲染Web字体
![](http://ontjktddg.bkt.clouddn.com/image/webview-fonts.png)

[“图3-4 渲染Web字体”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-fonts.htm)