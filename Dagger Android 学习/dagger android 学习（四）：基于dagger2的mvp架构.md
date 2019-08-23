

本系列博客主要用于自己学习积累，顺便帮助新手入门，如有问题，多多包涵。更详细的一些使用可以看看其他更加详细的博客。

[dagger android 学习（一）：dagger基础使用](https://juejin.im/post/5cc71fb7e51d456e6479b4fe)  
[dagger android 学习（二）：AndroidInjector的使用](https://juejin.im/post/5cc7202fe51d456e31164a6c)  
[dagger android 学习（三）：ContributesAndroidInjector的进一步优化](https://juejin.im/post/5cc72061e51d456e6154b4bc)  
[dagger android 学习（四）：基于dagger2的mvp架构](https://juejin.im/post/5cc72088e51d456e6d13351d)  


dagger2的基本使用已经介绍完了，那么接下来就介绍一下，dagger2遇上mvp架构会擦出怎么样的火花。  
mvp的架构想必就不用再谈了。model-view-presenter，每次我们都需要创建这三个必有的类，并且我们还需要在presenter中依赖view，view中创建presenter对象，而model进行数据处理这个则相对独立。那么dagger2能不能帮我们省去一些代码呢？答案是肯定的。

### 开始改造
首先这个整体模块的构造和上篇文章一样，所有重复的东西就不再论述，这里只论述关于model-view-presenter如何去处理，而@model @component具体怎么写就不阐述了。

#### View
创建接口IBaseView，以后所有的View实现这个接口即可。

    public interface IBaseView {
    }
    
#### Presenter
创建BasePresenter，这个presenter提供了View的获取，这样我们就可以在presenter处理完逻辑后返回给view去刷新ui。

    public class BasePresenter<T extends IBaseView> {
    
        private T view;
    
    
        public T getView() {
            return view;
        }
    
        public void attachView(T view){
            this.view = view;
        }
    }
    
#### DaggerMvpActivity
创建daggerMvpActivity，这个类是以后所有Activity的基类，通过实现类，就自动对presenter、view实现了依赖注入，这样我们就可以很方便的获取presenter和view，无需再进行new Presenter以及setView的操作。

    open class DaggerMvpActivity<T: BasePresenter<K>, K: IBaseView>: AppCompatActivity(), IBaseView {
    
    
        @Inject lateinit var presenter: T
    
        override fun onCreate(savedInstanceState: Bundle?) {
            AndroidInjection.inject(this)
            super.onCreate(savedInstanceState)
            presenter.attachView(this as K)
        }
    }
    
#### ActivityComponet
ActivityComponet与之前并无太大区别，只需要加入将之前的DaggerMvpActivity即可。

    @Subcomponent(modules = [AndroidInjectionModule::class])
    interface ActivityComponet: AndroidInjector<DaggerMvpActivity<BasePresenter<IBaseView>, IBaseView>>{
    
        //每一个继承BaseActivity的Activity，都共享同一个SubComponent
        @Subcomponent.Builder
        abstract class Builder: AndroidInjector.Builder<DaggerMvpActivity<BasePresenter<IBaseView>, IBaseView>>()
    }
    
### 使用
接下来，我们就可以按照之前的方法创建并使用Activity了。

    public class MainActivity extends DaggerMvpActivity<MainPresenter, MainView> implements MainView {
    
        @Inject
        A2 a2;
    
        @Inject
        SharedPreferences sp;
    
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
            sp.toString();
            presenter.doPresenter();
        }
    
        @Override
        public void showToast() {
            Log.e("MainActivity", "showToast");
        }
    }
    
    public class MainPresenter extends BasePresenter<MainView> {
    
        @Inject
        public MainPresenter(){
    
        }
    
    
        public void doPresenter(){
            Log.e("MainPresenter", "doPresenter");
            getView().showToast();
        }
    }
    
上面的Activity与Presenter使用过程中我们就再也见不到new Presenter(view)这种操作了。
dagger2的用法还有许多，这里先不详细介绍了，这几篇模块主要还是积累使用。如有问题，多多包涵。

demo地址：[app-version3-mvp]模块(https://github.com/JavaNoober/DaggerAndroid)