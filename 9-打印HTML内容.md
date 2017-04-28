[原文链接](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/printing.htm#)

#打印HTML内容

本章教你如何打印WebView组件中加载的网页。

使用JavaFX 8中的打印API，您可以打印JavaFX应用程序的图形内容。相应的类和枚举位于 javafx.print包中。

##使用打印API

在JavaFX应用程序启用打印功能，你必须使用PrinterJob类。这个类代表关联着系统默认打印机的一个打印机作业。使用Printer类来为特定作业改变打印机。对于每一个打印作业,您可以通过使用JobSettings类的属性指定的作业设置，比如collation、copies、pageLayout、pageRanges、paperSource、printColor、printResolution、printQuality、printSides。

你可以打印场景图的任何节点包括根节点。您还可以打印没添加到场景中的节点。使用 printPage方法初始化对特定节点的打印作业：job.printPage(node)。有关打印功能的更多信息，请参见JavaFX 8 API规范。

使用JavaFX web组件时,您通常需要打印加载到浏览器的HTML页面而不是应用程序界面本身。这就是为什么print方法被添加到WebEngine类。该方法面向打印关联web引擎的HTML页面。

##添加上下文菜单启用打印

通常，您添加打印命令到应用程序菜单或指定的工具栏印刷按钮。在WebViewSample应用程序中，工具栏带有过多的控件，这就是为什么你添加打印命令到上下文菜单右键单击启用。[例9-1](#例9-1)展示了一个代码片段，它添加了带有打印命令的上下文菜单到应用程序工具栏。

<span id="例9-1">例9-1</span> 创建工具栏上下文菜单
```
//adding context menu
final ContextMenu cm = new ContextMenu();
MenuItem cmItem1 = new MenuItem("Print");
cm.getItems().add(cmItem1);
toolBar.addEventHandler(MouseEvent.MOUSE_CLICKED, (MouseEvent e) -> {
    if (e.getButton() == MouseButton.SECONDARY) {
        cm.show(toolBar, e.getScreenX(), e.getScreenY());
    }
});
```
当你添加[例9-1](#例9-1)的代码片段到WebViewSample应用程序，运行它，右击工具栏出现打印上下文菜单，如[图9-1](#图9-1)所示。

<span id="图9-1">图9-1</span> 打印上下文菜单
![](http://ontjktddg.bkt.clouddn.com/image/webview-print.png)

[“图9-1 打印上下文菜单”的描述](http://docs.oracle.com/javase/8/javafx/embedded-browser-tutorial/img_text/webview-print.htm)

##处理打印作业

在打印上下文菜单添加到应用程序界面之后，您可以定义打印操作。首先，您必须创建一个 PrinterJob对象。然后，调用WebEngine.print方法把这个打印作业作为参数传进来，如[例9-2](#例9-2)所示。

<span id="例9-2">例9-2</span> 调用WebEngine.print方法
```
//processing print job
cmItem1.setOnAction((ActionEvent e) -> {
    PrinterJob job = PrinterJob.createPrinterJob();
    if (job != null) {
        webEngine.print(job);
        job.endJob();
    }
});
```
检查打印机作业不为空是很重要的，因为如果系统中没有可用的打印机createPrinterJob方法返回null。

研究[例9-3](#例9-3)为了评估WebViewSample应用程序启用打印功能的完整代码。

<span id="例9-3">例9-3</span> WebViewSample启用打印功能
<pre>
import javafx.application.Application;
import javafx.application.Platform;
import javafx.beans.value.ObservableValue;
import javafx.collections.ListChangeListener.Change;
import javafx.concurrent.Worker.State;
import javafx.event.ActionEvent;
import javafx.event.Event;
import javafx.geometry.HPos;
import javafx.geometry.Pos;
import javafx.geometry.VPos;
import javafx.print.PrinterJob;
import javafx.scene.Node;
import javafx.scene.Scene;
import javafx.scene.control.Button;
import javafx.scene.control.ComboBox;
import javafx.scene.control.ContextMenu;
import javafx.scene.control.Hyperlink;
import javafx.scene.control.MenuItem;
import javafx.scene.image.Image;
import javafx.scene.image.ImageView;
import javafx.scene.input.MouseButton;
import javafx.scene.input.MouseEvent;
import javafx.scene.layout.HBox;
import javafx.scene.layout.Priority;
import javafx.scene.layout.Region;
import javafx.scene.paint.Color;
import javafx.scene.web.PopupFeatures;
import javafx.scene.web.WebEngine;
import javafx.scene.web.WebHistory;
import javafx.scene.web.WebHistory.Entry;
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
    final WebView smallView = new WebView();
    final ComboBox comboBox = new ComboBox();
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
 
 
        comboBox.setPrefWidth(60);
 
        // create the toolbar
        toolBar = new HBox();
        toolBar.setAlignment(Pos.CENTER);
        toolBar.getStyleClass().add("browser-toolbar");
        toolBar.getChildren().add(comboBox);
        toolBar.getChildren().addAll(hpls);
        toolBar.getChildren().add(createSpacer());
 
        //set action for the button
        toggleHelpTopics.setOnAction((ActionEvent t) -> {
            webEngine.executeScript("toggle_visibility('help_topics')");
        });
 
        smallView.setPrefSize(120, 80);
 
        //handle popup windows
        webEngine.setCreatePopupHandler(
                (PopupFeatures config) -> {
                    smallView.setFontScale(0.8);
                    if (!toolBar.getChildren().contains(smallView)) {
                        toolBar.getChildren().add(smallView);
                    }
                    return smallView.getEngine();
        });
 
        //process history
        final WebHistory history = webEngine.getHistory();
        history.getEntries().addListener(
            (Change<? extends Entry> c) -> {
                c.next();
                c.getRemoved().stream().forEach((e) -> {
                comboBox.getItems().remove(e.getUrl());
            });
                c.getAddedSubList().stream().forEach((e) -> {
                comboBox.getItems().add(e.getUrl());
            });
        });
 
        //set the behavior for the history combobox               
        comboBox.setOnAction((Event ev) -> {
            int offset
                    = comboBox.getSelectionModel().getSelectedIndex()
                    - history.getCurrentIndex();
            history.go(offset);
        });
 
        // process page loading
        webEngine.getLoadWorker().stateProperty().addListener(
            (ObservableValue<? extends State> ov, State oldState, 
                State newState) -> {
                    toolBar.getChildren().remove(toggleHelpTopics);
                    if (newState == State.SUCCEEDED) {
                        JSObject win
                                = (JSObject) webEngine.executeScript("window");
                        win.setMember("app", new JavaApp());
                        if (needDocumentationButton) {
                            toolBar.getChildren().add(toggleHelpTopics);
                        }
                    }
        });
        <b>//adding context menu
		final ContextMenu cm = new ContextMenu();
		MenuItem cmItem1 = new MenuItem("Print");
		cm.getItems().add(cmItem1);
		toolBar.addEventHandler(MouseEvent.MOUSE_CLICKED, (MouseEvent e) -> {
			if (e.getButton() == MouseButton.SECONDARY) {
				cm.show(toolBar, e.getScreenX(), e.getScreenY());
			}
		});

		//processing print job
		cmItem1.setOnAction((ActionEvent e) -> {
			PrinterJob job = PrinterJob.createPrinterJob();
			if (job != null) {
				webEngine.print(job);job.endJob();
			}
		});</b>
 
        // load the home page        
        webEngine.load("http://www.oracle.com/products/index.html");
 
        //add components
        getChildren().add(toolBar);
        getChildren().add(browser);
    }
 
    // JavaScript interface object
    public class JavaApp {
 
        public void exit() {
            Platform.exit();
        }
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
</pre>

为了扩展WebViewSample应用程序的打印功能，使用javafx.print包中可用的类。

在JavaFX应用程序中，您可以通过使用TabPane类实现浏览器标签，当用户添加了一个新标签时创建一个新的WebView对象。

为了进一步加强这个应用程序，您可以应用效果、变换、和动画过渡。你也可以添加更多 WebView实例到应用程序场景。

请参见JavaFX API文档和JavaFX CSS规范获取关于可用特性的更多信息。你也可以学习[JavaFX in SWing](http://docs.oracle.com/javase/8/javafx/interoperability-tutorial/fx_swing.htm#JFXIP561)教程学习如何添加WebView组件到你现有的Swing应用程序。

相关的API文档

- WebView

- WebEngine

- WebHistory

- Region

- Hyperlink

- Worker