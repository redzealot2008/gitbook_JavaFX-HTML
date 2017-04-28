[原文链接](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/js-javafx.htm)

#JavaScript向上调用JavaFX

现在你知道如何从JavaFX调用JavaScript。在这一章，你可以探索相反的功能——从web内容调用JavaFX。

通常的概念是在JavaFX应用程序中创建一个接口对象，通过调用JSObject.setMember()方法使JavaScript得知。之后，你可以调用公共方法并从JavaScript访问这个对象的公共字段。

##使用JavaScript命令退出JavaFX应用程序

首先，添加一行到help.html文件：``<p><a href="about:blank" onclick="app.exit()">Exit the Application</a></p>``。通过点击help.html文件中的退出应用程序链接，用户退出WebViewSample应用程序。修改应用程序，如[例6-1](#例6-1)所示实现此功能。

<span id="例6-1">例6-1</span> 通过使用JavaScript关闭JavaFX应用程序
<pre>
import javafx.application.Application;
import javafx.application.Platform;
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
import netscape.javascript.JSObject;
 
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
    final Button toggleHelpTopics = new Button("Toggle Help Topics");
    private boolean needDocumentationButton = false;
    
    
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
        toggleHelpTopics.setOnAction((ActionEvent t) -> {
            webEngine.executeScript("toggle_visibility('help_topics')");
        });
 
         // process page loading
        webEngine.getLoadWorker().stateProperty().addListener(
            (ObservableValue<? extends State> ov, State oldState, 
                State newState) -> {
                    toolBar.getChildren().remove(toggleHelpTopics);
                    if (newState == State.SUCCEEDED) {
                        <b>JSObject win= (JSObject) webEngine.executeScript("window");
						win.setMember("app", new JavaApp());</b>
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
    
    <b>// JavaScript interface object
	public class JavaApp {

		public void exit() {
			Platform.exit();
		}
	}</b>
 
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
</pre>

检查[例6-1](#例6-1)中的粗线。JavaApp接口exit()的方法是公开的；因此，它可以被外部访问。当调用此方法时，它会导致JavaFX应用程序终止。

[例6-1](#例6-1)中的JavaApp接口被设置为JSObject实例的成员，以便JavaScript识别到这个接口。在window.app或app下可以被JavaScript知道，它唯一的方法如app.exit()可以从JavaScript调用。

当你编译、运行WebViewSample应用程序，并点击帮助图标，退出应用程序链接出现在页面的底部，如[图6-1](#图6-1)所示。

<span id="">图6-1</span> 退出应用程序链接
![](http://ontjktddg.bkt.clouddn.com/image/webview-help-exit.png)

[“图6-1 退出应用程序链接”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-help-exit.htm)

检查文件内容，然后点击退出应用程序链接，如[图6-1](#图6-1)所示，关闭WebViewSample应用程序。