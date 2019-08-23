

本系列博客主要用于自己学习积累，顺便帮助新手入门，如有问题，多多包涵。更详细的一些使用可以看看其他更加详细的博客。

[dagger android 学习（一）：dagger基础使用](https://juejin.im/post/5cc71fb7e51d456e6479b4fe)  
[dagger android 学习（二）：AndroidInjector的使用](https://juejin.im/post/5cc7202fe51d456e31164a6c)  
[dagger android 学习（三）：ContributesAndroidInjector的进一步优化](https://juejin.im/post/5cc72061e51d456e6154b4bc)  
[dagger android 学习（四）：基于dagger2的mvp架构](https://juejin.im/post/5cc72088e51d456e6d13351d)  


在安卓实际开发中，我们需要用到许多的activity以及fragment，这样我们在进行依赖注入的之后都要进行一些重复的操作，比如在comonent中加入void inject(XXXActivity activity)，以及在Activity中加入DaggerMyAppComponent.builder().build().inject(this)，好在dagger2也提供了解决方法，让我们不需要每次去写inject方法。

### 添加依赖

    implementation 'com.google.dagger:dagger:2.21'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.21'
    implementation 'com.google.dagger:dagger-android:2.21'
    implementation 'com.google.dagger:dagger-android-support:2.21'
    annotationProcessor 'com.google.dagger:dagger-android-processor:2.21'
    
### 开始改造

#### AndroidInjector
创建一个接口继承AndroidInjector

    @Subcomponent(modules = {
        AndroidInjectionModule.class,
    })
    public interface MainActivitySubcomponent extends AndroidInjector<MainActivity> {

        @Subcomponent.Builder
        abstract class Builder extends AndroidInjector.Builder<MainActivity> {
        }
    }
    
#### ActivityModule
实现用于绑定activity的Module

    @Module(subcomponents = MainActivitySubcomponent.class)
    public abstract class MainActivityModule {
    
        @Binds
        @IntoMap
        @ClassKey(MainActivity.class)
        abstract AndroidInjector.Factory<?> bindMainActivityInjectorFactory(MainActivitySubcomponent.Builder builder);
    }
    
#### AppComponent

    @Component(modules = {
            AndroidInjectionModule.class,
            AndroidSupportInjectionModule.class,
            MainActivityModule.class,
            Main2ActivityModule.class,
    })
    public interface MyAppComponent {
        void inject(MyApplication application);
    
        @Component.Builder
        interface Builder{
            MyAppComponent build();
        }
    }
    
    
#### Application

application继承HasActivityInjector，实现inject方法，返回DispatchingAndroidInjector对象即可

    public class MyApplication extends Application implements HasActivityInjector {
    
        @Inject
        DispatchingAndroidInjector<Activity> dispatchingAndroidInjector;
    
    
        @Override
        public void onCreate() {
            super.onCreate();
            DaggerMyAppComponent.builder().build().inject(this);
        }
    
        @Override
        public AndroidInjector<Activity> activityInjector() {
            return dispatchingAndroidInjector;
        }
    }

#### BaseActivity
baseActivity在super.onCreate之前调用AndroidInjection.inject(this)，这样之后的activity就只需要继承它，直接可以使用了，不需要再单独加入inject方法

    open class BaseActivity: AppCompatActivity() {
    
        override fun onCreate(savedInstanceState: Bundle?) {
            AndroidInjection.inject(this)
            super.onCreate(savedInstanceState)
        }
    }
    
#### 使用

使用的时候直接@inject就行，无需任何处理

    public class MainActivity extends BaseActivity {
    
        @Inject
        A2 a2;
    
        @Inject
        C c;
    
    
        @Inject
        UserInfo userInfo;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            findViewById(R.id.tv).setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    startActivity(new Intent(MainActivity.this, Main2Activity.class));
                }
            });

            a2.someThingA();
            c.doSomethingC();
    
            userInfo.setAge("111");
            Log.e("DAGGER", userInfo.toString());
        }
    }
    
### 总结
dagger-android-support解决了每次新建Activity都需要在Activity中添加格子的inject方法，所有的inject都在BaseActivity中，这让代码简洁了很多。但是即使如此，还是有较多一部分的模板代码，如例子中的ActivityModule和ActivitySubcomponent。每次新建一个Activity都需要创建这两个类，这样看起来还是很麻烦，好在dagger2还提供了@ContributesAndroidInjector注解解决了这个问题。  
demo地址:查看项目中的[app模块](https://github.com/JavaNoober/DaggerAndroid)