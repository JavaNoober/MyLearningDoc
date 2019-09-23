最近一直在探索android flutter的混合开发，一路遇到了很多的坑，接下来便将这些记录下来，希望能帮助到大家的开发。  
## flutter常用指令

- **flutter help create**  
flutter创建项目的指令。我们可以指定创建module还是project，可以指定语言，如java、kotlin，同时可以指定是否生成androidx项目。

    
    -h, --help                     Print this usage information.
        --[no-]pub                 Whether to run "flutter pub get" after the project has been created.
                                   (defaults to on)
    
        --[no-]offline             When "flutter pub get" is run by the create command, this indicates whether to run it in offline mode or not. In offline mode, it will need to have all dependencies already available in the pub cache to succeed.
        --[no-]with-driver-test    Also add a flutter_driver dependency and generate a sample 'flutter drive' test.
    -t, --template=<type>          Specify the type of project to create.
    
              [app]                (default) Generate a Flutter application.
              [package]            Generate a shareable Flutter project containing modular Dart code.
              [plugin]             Generate a shareable Flutter project containing an API in Dart code with a platform-specific implementation for Android, for iOS code, or for both.
    
    -s, --sample=<id>              Specifies the Flutter code sample to use as the main.dart for an application. Implies --template=app. The value should be the sample ID of the desired sample from the API documentation website (http://docs.flutter.dev). An example can
                                   be found at https://master-api.flutter.dev/flutter/widgets/SingleChildScrollView-class.html
    
        --list-samples=<path>      Specifies a JSON output file for a listing of Flutter code samples that can created with --sample.
        --[no-]overwrite           When performing operations, overwrite existing files.
        --description              The description to use for your new Flutter project. This string ends up in the pubspec.yaml file.
                                   (defaults to "A new Flutter project.")
    
        --org                      The organization responsible for your new Flutter project, in reverse domain name notation. This string is used in Java package names and as prefix in the iOS bundle identifier.
                                   (defaults to "com.example")
    
        --project-name             The project name for this new Flutter project. This must be a valid dart package name.
    -i, --ios-language             [objc, swift (default)]
    -a, --android-language         [java, kotlin (default)]
        --[no-]androidx            Generate a project using the AndroidX support libraries


- **flutter channel**  

查看当前flutter分支

    Flutter channels:
      beta
      dev
      master
    * stable

- **flutter doctor**  

诊断当前flutter配置是否有问题

- **flutter doctor**  

诊断当前flutter配置是否有问题

- **flutter version --force 1.0.0**  

切换到指定flutter版本

- **flutter clean**  

clean当前工程

- **flutter run**  

运行当前工程

- **flutter analyze**

分析当前项目dart代码

- **flutter assemble**  

构建资源 

- **flutter attach**

attach当前项目，通常用于调试

- **flutter bash-completion**  

Output command line shell completion setup scripts.

- **flutter build**  

编译flutter项目

- **flutter config**  

配置flutter设置

- **flutter devices**  

查看设备列表

- **flutter drive**  

为当前项目运行驱动测试

- **flutter emulators**

查看模拟器列表以及运行

- **flutter format**

格式化dart代码

- **flutter install**

安装flutter项目

- **flutter logs**

查看运行日志 

- **flutter screenshot**

对设备进行截图

- **flutter test**

运行flutter单元测试

- **flutter upgrade**

更新flutter 版本

- **flutter version**  

查看flutter 版本

## flutter常见问题的解决

- flutter工程无法进行依赖

可以查看一下.android/Flutter/build.gradle的buildTypes中是否包含app的build.gradle中所需的buildTypes

- 缺少libflutter.so

flutter项目默认支持的是armeabi-v7a,所以需要在app的gradle中加入如下配置：  

        ndk {
            abiFilters "armeabi-v7a"
        }  
        
如果还是提示这个错误，则需要在gradle.properties中华加入如下配置：  

    target-platform=android-arm
    
- 编译apk成功，但是运行flutter失败,例如：VM snapshot must be valid. /Check failed: vm. Must be able to initialize the VM.

可以尝试flutter clean, 然后flutter run一下项目，再去运行主工程

- flutter混淆配置


    -keep class io.flutter.app.** {*;}
    -keep class io.flutter.plugin.** {*;}
    -keep class io.flutter.util.** {*;}
    -keep class io.flutter.view.** {*;}
    -keep class io.flutter.** {*;}
    -keep class io.flutter.plugins.** {*;}
    -dontwarn io.flutter.**
    
- 跳转flutter所在activity黑屏


        val flutterView = Flutter.createView(this, lifecycle, Gson().toJson(map))
        val layout = findViewById<FrameLayout>(R.id.flutter_container)
        layout.visibility = View.INVISIBLE
        layout.addView(flutterView)
        val listeners = arrayOfNulls<FlutterView.FirstFrameListener>(1)
        listeners[0] = FlutterView.FirstFrameListener {
            layout.visibility = View.VISIBLE
            loadingDialog.dismiss()
        }
        flutterView.addFirstFrameListener(listeners[0])

或者设置当前主题

- flutter的activity进入加载较慢

debug包这种情况比较明显，但是release加载很快，可以仿照一下闲鱼，在进入FlutterActivity的时候提供一个加载loading

## Activity中加载flutter几种方式

### 直接加载flutter的main.dart
GeneratedPluginRegistrant.registerWith即可

    public class MainActivity extends FlutterActivity {
      @Override
      protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        GeneratedPluginRegistrant.registerWith(this);
      }
    }

### 继承FlutterActivity

    class TestFlutterActivity : FlutterActivity() {
    
        private lateinit var flutterFragment: FlutterFragment
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
    //        setContentView(R.layout.activity_flutter_test)
            GeneratedPluginRegistrant.registerWith(this)
        }
    
    
        override fun createFlutterView(context: Context?): FlutterView {
            val matchParent = WindowManager.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT)
            val nativeView = this.createFlutterNativeView()
            val flutterView = FlutterView(this, null, nativeView)
            flutterView.setInitialRoute("web_view_page")
            flutterView.layoutParams = matchParent
    //        flutterView.enableTransparentBackground()
            val layout = findViewById<FrameLayout>(R.id.flutter_container)
            val listeners = arrayOfNulls<FlutterView.FirstFrameListener>(1)
            listeners[0] = FlutterView.FirstFrameListener {
                layout.visibility = View.VISIBLE
            }
            flutterView.addFirstFrameListener(listeners[0])
            this.addContentView(flutterView, matchParent)
            return flutterView
        }
    
        override fun onBackPressed() {
            if (flutterView != null) {
                flutterView.popRoute()
            } else {
                super.onBackPressed()
            }
        }
    }

### addContentView直接加载FlutterView

    class MainFlutterActivity : FlutterFragmentActivity() {
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_main_flutter)
            val mFlutterView: View = Flutter.createView(this, lifecycle, "main_flutter")
            val mParams = FrameLayout.LayoutParams(FrameLayout.LayoutParams.MATCH_PARENT,
                    FrameLayout.LayoutParams.MATCH_PARENT)
            addContentView(mFlutterView, mParams)
        }
    }
    
### 通过FlutterFragment

    class FlutterActivity : AppCompatActivity() {
    
        private lateinit var flutterFragment: FlutterFragment
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_helper_center)
            flutterFragment = Flutter.createFragment("web_view_page")
            supportFragmentManager.beginTransaction().replace(R.id.flutter_container, flutterFragment).commitAllowingStateLoss()
        }
    }
    
## Android原生与Flutter间的通信、传值
可以通过flutter_boost，但是目前有适配问题，不支持flutter1.7、1.9

### Flutter传值->Android
有点类似于Eventbus的方式：

flutter.dart中发送事件：

      static const platform = const MethodChannel("webview");
    
      void _finish() {
        platform.invokeMethod("finish");
      }

android接收并处理：

        MethodChannel(flutterView, "webview").setMethodCallHandler { methodCall, result ->
            if(methodCall.method == "finish"){
                finish()
            }
        }
        
### Android传值->Flutter
android跳转flutter的时候可以通过路由名字直接传入值。  

#### dart:

    import 'dart:ui' as ui;
    
    void main() => runApp(MyApp());
    
    class MyApp extends StatelessWidget {
      static const String HELP_URL =
          'https://xxxx';
    
      @override
      Widget build(BuildContext context) {
        return MaterialApp(
          title: 'Flutter Demo',
          theme: ThemeData(
            // This is the theme of your application.
            //
            // Try running your application with "flutter run". You'll see the
            // application has a blue toolbar. Then, without quitting the app, try
            // changing the primarySwatch below to Colors.green and then invoke
            // "hot reload" (press "r" in the console where you ran "flutter run",
            // or press Run > Flutter Hot Reload in a Flutter IDE). Notice that the
            // counter didn't reset back to zero; the application is not restarted.
            primarySwatch: Colors.blue,
          ),
          home: _widgetRouter(ui.window.defaultRouteName)
        );
      }
    }
    
    Widget _widgetRouter(String json) {
      return RouterManager.getInstance().getPageByRouter(json);
    }
    

    class RouterManager {
      static RouterManager mInstance = new RouterManager();
    
      static RouterManager getInstance() {
        return mInstance;
      }
    
      /*
        根据传入的route名  找到对应跳转的路径
       */
      StatefulWidget getPageByRouter(String jsonStr) {
        String path = "";
        String param = "";
        if (jsonStr != null && jsonStr.isNotEmpty && jsonStr != "/") {
          var jsonResponse = jsonDecode(jsonStr);
          path = jsonResponse["path"];
          print("==== router === path = $path");
          param = jsonResponse["param"];
          print("==== router === param = $param");
          path = path != null && path.isNotEmpty ? path : RouterPath.HOME_PAGE;
          param = param != null && param.isNotEmpty ? param : "[]";
        }
        Map<String, dynamic> map = json.decode(param);
        Widget widget = _getWidgetPage(path, map);
    
        if (widget == null) {
          debugPrint('==== 找不到widget =====');
        }
        return widget;
      }
    
      Widget _getWidgetPage(String path, Map<String, dynamic> map) {
        Widget widget;
        switch (path) {
          case RouterPath.HOME_PAGE:
            widget = MyHomePage(title: 'Flutter Demo Home Page');
            break;
          case RouterPath.WEB_VIEW_PAGE:
            widget = WebViewPage(map["title"], map["url"]);
            break;
        }
        return widget;
      }
    }
    
    class RouterPath {
      static const String HOME_PAGE = 'home_page';
      static const String WEB_VIEW_PAGE = 'web_view_page';
    }

在_getWidgetPage内配置路由跳转的值及对应界面。  

#### android

与之前主要的区别在于参数通过hashmap转换成json形式放入路由名称中，然后把路由名连同参数一起传给main.dart

        val map = HashMap<String, String>()
        val params = HashMap<String, String>()
        params.put("title", "帮助中心")
        params.put("url", WebUrlConfig.HELP_URL)
        map.put("path", "web_view_page")
        map.put("param", Gson().toJson(params))
        val matchParent = WindowManager.LayoutParams(ViewGroup.LayoutParams.MATCH_PARENT, ViewGroup.LayoutParams.MATCH_PARENT)
        val nativeView = this.createFlutterNativeView()
        val flutterView = FlutterView(this, null, nativeView)
        flutterView.setInitialRoute(Gson().toJson(map))
        flutterView.layoutParams = matchParent
        val layout = findViewById<FrameLayout>(R.id.flutter_container)
        val listeners = arrayOfNulls<FlutterView.FirstFrameListener>(1)
        listeners[0] = FlutterView.FirstFrameListener {
            layout.visibility = View.VISIBLE
        }
        flutterView.addFirstFrameListener(listeners[0])
        this.addContentView(flutterView, matchParent)

#### android部分的封装
我这里对其android部分进行了封装，需要使用的话直接拷贝代码即可：

BaseFlutterActivity：

    abstract class BaseFlutterActivity : AppCompatActivity() {
    
        lateinit var flutterView: FlutterView
    
        private val loadingDialog: LoadingDialog by lazy {
            LoadingDialog(this, "加载中")
        }
    
        override fun onCreate(savedInstanceState: Bundle?) {
            super.onCreate(savedInstanceState)
            setContentView(R.layout.activity_flutter_activity)
            loadingDialog.show()
            val map = HashMap<String, String>()
            map["path"] = routeUrl()
            map["param"] = Gson().toJson(routeParams())
            flutterView = Flutter.createView(this, lifecycle, Gson().toJson(map))
            val layout = findViewById<FrameLayout>(R.id.flutter_container)
            layout.addView(flutterView)
            val listeners = arrayOfNulls<FlutterView.FirstFrameListener>(1)
            listeners[0] = FlutterView.FirstFrameListener {
                layout.visibility = View.VISIBLE
                loadingDialog.dismiss()
            }
            flutterView.addFirstFrameListener(listeners[0])
    
            if (setChannel().isNotEmpty()) {
                setFlutterMessageHandler()?.let {
                    MethodChannel(flutterView, setChannel()).setMethodCallHandler(it)
                }
            }
    
        }
    
        open fun setChannel(): String {
            return ""
        }
    
        open fun setFlutterMessageHandler(): MethodChannel.MethodCallHandler? {
            return null
        }
    
    
        abstract fun routeUrl(): String
    
        abstract fun routeParams(): HashMap<String, String>
    }

使用：  
下面代码是跳转一个webview并且传入url：

    class FlutterWebActivity : BaseAegisFlutterActivity() {
    
        private val title: String by lazy { intent.getStringExtra(TITLE) }
        private val url: String by lazy { intent.getStringExtra(URL) }
    
        companion object {
            const val TITLE = "title"
            const val URL = "url"
    
            fun buildIntent(context: Context?, title: String, url: String): Intent {
                val intent = Intent(context, FlutterWebActivity::class.java)
                intent.putExtra(TITLE, title)
                intent.putExtra(URL, url)
                return intent
            }
        }
    
        override fun routeUrl(): String = "web_view_page"
    
        override fun routeParams(): HashMap<String, String> {
            val params = HashMap<String, String>()
            params["title"] = title
            params["url"] = url
            return params
        }
    
        override fun setChannel(): String = "webview"
    
        override fun setFlutterMessageHandler(): MethodChannel.MethodCallHandler? {
            return MethodChannel.MethodCallHandler { methodCall, _ ->
                if (methodCall.method == "finish") {
                    finish()
                }
            }
        }
    }
    
## 总结
Android Flutter的混合开发基础先总结到这里，希望能帮助到大家。后续内容我会再进行补充。