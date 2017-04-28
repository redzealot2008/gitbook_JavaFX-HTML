[原文链接](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/js-commands.htm)

#处理JavaScript命令

本章扩展了WebViewSampley应用程序并解释了如何从JavaFX代码调用JavaScript命令。

WebEngine类提供了API来执行在当前HTML页面上下文中的脚本。

##理解executeScript方法

WebEngine类的executeScript方法允许执行声明在加载的HTML页面中的任何JavaScript命令。使用以下字符串在web引擎中调用这个方法：webEngine.executeScript("``<function name>``");。

方法执行结果被转换为java.lang.Object实例通过使用下列规则：

- JavaScript Int32 被转换为 java.lang.Integer

- JavaScript numbers 被转换为 java.lang.Double

- JavaScript string值 被转换为 java.lang.String

- JavaScript boolean值 被转换为 java.lang.Boolean

关于转换结果的更多信息请参考WebEngine类的API文档。

##从JavaFX代码调用JavaScript命令

扩展WebViewSample应用程序引入一个帮助文件和执行JavaScript命令切换（显示或隐藏）帮助文件主题列表。创建工具栏帮助项指向help.html文件，用户可以预览Oracle web站点的参考资料。

添加如[例5-1](#例5-1)所示的help.html文件到WebViewSample应用程序。

<span id="例5-1">例5-1</span> help.html文件
```
<html lang="en">
    <head>
        <!-- Visibility toggle script -->
        <script type="text/javascript">
            <!--
            function toggle_visibility(id) {
                var e = document.getElementById(id);
                if (e.style.display == 'block')
                    e.style.display = 'none';
                else
                    e.style.display = 'block';
            }
//-->
        </script>
    </head>
    <body>
        <h1>Online Help</h1>
        <p class="boxtitle"><a href="#" onclick="toggle_visibility('help_topics');" 
  class="boxtitle">[+] Show/Hide Help Topics</a></p>    
        <ul id="help_topics" style='display:none;'>
            <li>Products - Extensive overview of Oracle hardware and software products, 
                and summary Oracle consulting, support, and educational services. </li>
            <li>Blogs - Oracle blogging community (use the Hide All and Show All buttons 
                to collapse and expand the list of topics).</li>
            <li>Documentation - Landing page to start learning Java. The page contains 
                links to the Java tutorials, developer guides, and API documentation.</li>
            <li>Partners - Oracle partner solutions and programs. Popular resources and 
                membership opportunities.</li>
        </ul>
    </body>
</html>
```
修改后的应用程序代码如[例5-2](#例5-2)所示，创建帮助工具栏项和一个额外的按钮用来隐藏和显示帮助主题。这个按钮只有在帮助页面被选中时添加到工具栏。

<span id="例5-2">例5-2</span> 添加切换帮助主题按钮
```
import javafx.application.Application;
import javafx.beans.value.ObservableValue;
import javafx.concurrent.Worker.State;
import javafx.event.ActionEvent;
import javafx.geometry.HPos;
import javafx.geometry.Pos;
import javafx.geometry.VPos;
import javafx.scene.Node;
import javafx.scene.Scene;
import javafx.scene.control.Button;
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
 
    @Override
    public void start(Stage stage) {
        // create scene
        stage.setTitle("Web View Sample");
        scene = new Scene(new Browser(stage), 900, 600, Color.web("#666970"));
        stage.setScene(scene);
        // apply CSS style
        scene.getStylesheets().add("webviewsample/BrowserToolbar.css");
        // show stage
        stage.show();
    }
 
    public static void main(String[] args) {
        launch(args);
    }
}
 
class Browser extends Region {
 
    private final HBox toolBar;
    final private static String[] imageFiles = new String[]{
        "product.png",
        "blog.png",
        "documentation.png",
        "partners.png",
        "help.png"
    };
    final private static String[] captions = new String[]{
        "Products",
        "Blogs",
        "Documentation",
        "Partners",
        "Help"
    };
    final private static String[] urls = new String[]{
        "http://www.oracle.com/products/index.html",
        "http://blogs.oracle.com/",
        "http://docs.oracle.com/javase/index.html",
        "http://www.oracle.com/partners/index.html",
        WebViewSample.class.getResource("help.html").toExternalForm()
    };
    final ImageView selectedImage = new ImageView();
    final Hyperlink[] hpls = new Hyperlink[captions.length];
    final Image[] images = new Image[imageFiles.length];
    final WebView browser = new WebView();
    final WebEngine webEngine = browser.getEngine();
    final Button toggleHelpTopics = new Button("Toggle Help Topics");private boolean needDocumentationButton = false;
    
    
    public Browser(final Stage stage) {
        //apply the styles
        getStyleClass().add("browser");
                
        for (int i = 0; i < captions.length; i++) {
            // create hyperlinks
            Hyperlink hpl = hpls[i] = new Hyperlink(captions[i]);
            Image image = images[i]
                    = new Image(getClass().getResourceAsStream(imageFiles[i]));
            hpl.setGraphic(new ImageView(image));
            final String url = urls[i];
            final boolean addButton = (hpl.getText().equals("Help"));  
            
            // process event 
            hpl.setOnAction((ActionEvent e) -> {
                needDocumentationButton = addButton;
                webEngine.load(url);
            });
                    
        }
 
        // create the toolbar
        toolBar = new HBox();
        toolBar.setAlignment(Pos.CENTER);
        toolBar.getStyleClass().add("browser-toolbar");
        toolBar.getChildren().addAll(hpls);
        toolBar.getChildren().add(createSpacer());
 
        //set action for the button
        toggleHelpTopics.setOnAction((ActionEvent t) -> {webEngine.executeScript("toggle_visibility('help_topics')");});
 
         // process page loading
        webEngine.getLoadWorker().stateProperty().addListener(
			(ObservableValue<? extends State> ov, State oldState,
				State newState) -> {
					toolBar.getChildren().remove(toggleHelpTopics);
					if (newState == State.SUCCEEDED) {
						if (needDocumentationButton) {
							toolBar.getChildren().add(toggleHelpTopics);
						}
					}
		});
 
        // load the home page        
        webEngine.load("http://www.oracle.com/products/index.html");
 
        //add components
        getChildren().add(toolBar);
        getChildren().add(browser);
    }
 
    private Node createSpacer() {
        Region spacer = new Region();
        HBox.setHgrow(spacer, Priority.ALWAYS);
        return spacer;
    }
 
    @Override
    protected void layoutChildren() {
        double w = getWidth();
        double h = getHeight();
        double tbHeight = toolBar.prefHeight(w);
        layoutInArea(browser,0,0,w,h-tbHeight,0,HPos.CENTER,VPos.CENTER);
        layoutInArea(toolBar,0,h-tbHeight,w,tbHeight,0,HPos.CENTER,VPos.CENTER);
    }
 
    @Override
    protected double computePrefWidth(double height) {
        return 900;
    }
 
    @Override
    protected double computePrefHeight(double width) {
        return 600;
    }
}
```
加载总是发生在后台线程。在调度后台工作之后立即返回初始化加载方法。getLoadWorker()方法提供了一个Worker接口实例跟踪加载进度。如果帮助页面的进度状态为SUCCEEDED，切换帮助主题按钮添加到工具栏，如[图5-1](#图5-1)所示。

<span id="图5-1">图5-1</span> 切换帮助主题按钮
![](http://ontjktddg.bkt.clouddn.com/image/webview-help-toggle.png)

[“图5-1 切换帮助主题按钮”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-help-toggle.htm)

[例5-3](#例5-3)所示的setOnAction方法定义了切换帮助主题按钮的行为。

<span id="例5-3">例5-3</span> 执行JavaScript命令
```
//set action for the button
toggleHelpTopics.setOnAction((ActionEvent t) -> {
    webEngine.executeScript("toggle_visibility('help_topics')");
});
```
当用户单击切换帮助主题按钮时，executeScript方法在help.html页面运行toggle_visibility JavaScript函数，帮助主题显示出来，如[图5-2](#图5-2)所示。当用户执行另一个点击，toggle_visibility函数隐藏帮助主题列表。

<span id="图5-2">图5-2</span> 显示帮助主题
![](http://ontjktddg.bkt.clouddn.com/image/webview-show-help.png)

[“图5-2 显示帮助主题”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-show-help.htm)