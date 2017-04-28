[原文链接](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/add-browser.htm)

#添加WebView组件到应用程序场景

本章介绍了WebViewSample应用程序和解释了如何添加WebView组件到JavaFX应用程序场景。

WebViewSample应用程序创建了Browser类封装了WebView对象和UI控件工具栏。WebViewSample类创建场景和添加Browser对象到场景。

##创建一个嵌入式浏览器

[例4-1](#例4-1)展示了如何添加WebView组件到应用程序场景。

<span id="例4-1">例4-1</span> 使用WebView和WebEngine类创建一个浏览器
```
import javafx.application.Application;
import javafx.geometry.HPos;
import javafx.geometry.VPos;
import javafx.scene.Node;
import javafx.scene.Scene;
import javafx.scene.layout.HBox;
import javafx.scene.layout.Priority;
import javafx.scene.layout.Region;
import javafx.scene.paint.Color;
import javafx.scene.web.WebEngine;
import javafx.scene.web.WebView;
import javafx.stage.Stage;
 
 
public class WebViewSample extends Application {
    private Scene scene;
    @Override public void start(Stage stage) {
        // create the scene
        stage.setTitle("Web View");
        scene = new Scene(new Browser(),900,600, Color.web("#666970"));
        stage.setScene(scene);
        scene.getStylesheets().add("webviewsample/BrowserToolbar.css");        
        stage.show();
    }
 
    public static void main(String[] args){
        launch(args);
    }
}
class Browser extends Region {
 
    final WebView browser = new WebView();final WebEngine webEngine = browser.getEngine();
     
    public Browser() {
        //apply the styles
        getStyleClass().add("browser");
        // load the web page
        webEngine.load("http://www.oracle.com/products/index.html");
        //add the web view to the scene
        getChildren().add(browser);
 
    }
    private Node createSpacer() {
        Region spacer = new Region();
        HBox.setHgrow(spacer, Priority.ALWAYS);
        return spacer;
    }
 
    @Override protected void layoutChildren() {
        double w = getWidth();
        double h = getHeight();
        layoutInArea(browser,0,0,w,h,0, HPos.CENTER, VPos.CENTER);
    }
 
    @Override protected double computePrefWidth(double height) {
        return 900;
    }
 
    @Override protected double computePrefHeight(double width) {
        return 600;
    }
}
```
在这段代码中,web引擎加载指向甲骨文公司网站的URL。通过使用getChildren和add方法把包含web引擎的WebView对象添加到应用程序场景。

当您添加、编译和运行这个代码片段,它生成应用程序窗口如[图4-1](#图4-1)所示。

<span id="图4-1">图4-1</span> 在应用程序场景中的WebView对象
![](http://ontjktddg.bkt.clouddn.com/image/webview-add.png)

[“图4-1 在应用程序场景中的WebView对象”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-add.htm)

##创建一个应用程序工具栏

添加一个带有四个Hyperlink对象的工具栏为了在不同的Oracle web资源间进行切换。在[例4-2](#例4-2)中学习修改后的Browser类的代码。它添加了可替代的web资源URL，包括甲骨文产品、博客、Java文档、和合作伙伴网络。代码片段也创建了一个工具栏并添加超链接。

<span id="例4-2">例4-2</span> 创建工具栏
```
import javafx.application.Application;
import javafx.event.ActionEvent;
import javafx.event.EventHandler;
import javafx.geometry.HPos;
import javafx.geometry.VPos;
import javafx.scene.Node;
import javafx.scene.Scene;
import javafx.scene.control.Hyperlink;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.layout.HBox;
import javafx.scene.layout.Priority;
import javafx.scene.layout.Region;
import javafx.scene.paint.Color;
import javafx.scene.web.WebEngine;
import javafx.scene.web.WebView;
import javafx.stage.Stage;
 
public class WebViewSample extends Application {
 
    private Scene scene;
 
    @Override public void start(Stage stage) {
        // create scene
        stage.setTitle("Web View");
        scene = new Scene(new Browser(),900,600, Color.web("#666970"));
        stage.setScene(scene);
        scene.getStylesheets().add("webviewsample/BrowserToolbar.css");
        // show stage
        stage.show();
    }
 
    public static void main(String[] args){
        launch(args);
    }
}
class Browser extends Region {
   private HBox toolBar;
 
   final private static String[] imageFiles = new String[]{
        "product.png",
        "blog.png",
        "documentation.png",
        "partners.png"
    };
    final private static String[] captions = new String[]{
        "Products",
        "Blogs",
        "Documentation",
        "Partners"
    };
 
    final private static String[] urls = new String[]{
        "http://www.oracle.com/products/index.html",
        "http://blogs.oracle.com/",
        "http://docs.oracle.com/javase/index.html",
        "http://www.oracle.com/partners/index.html"
    };
 
    final ImageView selectedImage = new ImageView();
    final Hyperlink[] hpls = new Hyperlink[captions.length];
    final Image[] images = new Image[imageFiles.length];
    final WebView browser = new WebView();
    final WebEngine webEngine = browser.getEngine();
 
    public Browser() {       
        //apply the styles
        getStyleClass().add("browser");
        
        <b>for (int i = 0; i < captions.length; i++) {
			final Hyperlink hpl = hpls[i] = new Hyperlink(captions[i]);
			Image image = images[i] =
				new Image(getClass().getResourceAsStream(imageFiles[i]));
			hpl.setGraphic(new ImageView (image));
			final String url = urls[i];

			hpl.setOnAction(new EventHandler<ActionEvent>() {
				@Override
				public void handle(ActionEvent e) {
					webEngine.load(url);
				}
			});
		}</b>
 
// load the home page        
        webEngine.load("http://www.oracle.com/products/index.html");
        
// create the toolbar
        toolBar = new HBox();toolBar.getStyleClass().add("browser-toolbar");toolBar.getChildren().addAll(hpls);        
    
        //add components
        getChildren().add(toolBar);
        getChildren().add(browser); 
    }
 
    private Node createSpacer() {
        Region spacer = new Region();
        HBox.setHgrow(spacer, Priority.ALWAYS);
        return spacer;
    }
 
    @Override protected void layoutChildren() {
        double w = getWidth();
        double h = getHeight();
        double tbHeight = toolBar.prefHeight(w);
        layoutInArea(browser,0,0,w,h-tbHeight,0, HPos.CENTER, VPos.CENTER);
        layoutInArea(toolBar,0,h-tbHeight,w,tbHeight,0,HPos.CENTER,VPos.CENTER);
    }
 
    @Override protected double computePrefWidth(double height) {
        return 900;
    }
 
    @Override protected double computePrefHeight(double width) {
        return 600;
    }
}
```

这段代码使用一个for循环创建超链接。setOnAction方法定义超链接行为。当用户点击一个链接,相应的URL值传递给webEngine的load方法。当您编译并运行修改后的应用程序时,应用程序窗口变化如[图4-2](#图4-2)所示。

<span i="图4-2">图4-2</span> 带有导航工具栏的嵌入式浏览器
![](http://ontjktddg.bkt.clouddn.com/image/webview-partner.png)

[“图4-2 带有导航工具栏的嵌入式浏览器”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-partner.htm)