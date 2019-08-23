

本系列博客主要用于自己学习积累，顺便帮助新手入门，如有问题，多多包涵。更详细的一些使用可以看看其他更加详细的博客。

[dagger android 学习（一）：dagger基础使用](https://juejin.im/post/5cc71fb7e51d456e6479b4fe)  
[dagger android 学习（二）：AndroidInjector的使用](https://juejin.im/post/5cc7202fe51d456e31164a6c)  
[dagger android 学习（三）：ContributesAndroidInjector的进一步优化](https://juejin.im/post/5cc72061e51d456e6154b4bc)  
[dagger android 学习（四）：基于dagger2的mvp架构](https://juejin.im/post/5cc72088e51d456e6d13351d)  


上篇文章讲述了，如果在baseActivity中调用inject方法实现依赖注入，但是这种需要重复的创建ActivityModule和ActivitySubcomponent类，好在dagger2提供了@ContributesAndroidInjector注解解决了这个问题，下面介绍如何使用@ContributesAndroidInjector注解。  

首先所有的代码基于上篇文章中的demo来改，重复的东西就不在论述了。

## 改造方法
### 1、删除重复模板代码
删除之前demo中的MainActivitySubcomponent和MainActivityModule
### 2、创建BaseActivity类型的AndroidInjector实现接口

相比于之前的AndroidInjector，之前是基于具体的XXXActivity，这次则是BaseActivity

    @Subcomponent(modules = [AndroidInjectionModule::class])
    interface ActivityComponet: AndroidInjector<BaseActivity>{
    
        //每一个继承BaseActivity的Activity，都共享同一个SubComponent
        @Subcomponent.Builder
        abstract class Builder: AndroidInjector.Builder<BaseActivity>()
    }
    
### 3、创建ActivityModule，管理所有Activity
新建一个ActivityModule类，管理所有Activity，并且加上@ContributesAndroidInjector注解即可。

    @Module(subcomponents = [ActivityComponet::class])
    abstract class ActivityModule{
        @ContributesAndroidInjector(modules = [ABModule::class])
        abstract fun mainActivityInjector(): MainActivity
    
        @ContributesAndroidInjector(modules = [ABModule::class])
        abstract fun main2ActivityInjector(): Main2Activity
    }

### 3、修改MyAppComponent
 删除之前的XXXActivityModule，替换为新的ActivityModule
 
    @Component(modules = {
            AndroidInjectionModule.class,
            AndroidSupportInjectionModule.class,
            ActivityModule.class
    })
    public interface MyAppComponent {
    
    
        @Component.Builder
        interface Builder {
            MyAppComponent build();
        }
    
    
        void inject(MyApplication application);
    }
## 总结
@ContributesAndroidInjector注解的功能其实就是直接帮我们重建了重复的模板代码，我们可以执行make project，之后我们便可以找到dagger自动创建的一个类ActivityModule_MainActivityInjector：

    @Module(subcomponents = ActivityModule_MainActivityInjector.MainActivitySubcomponent.class)
    public abstract class ActivityModule_MainActivityInjector {
      private ActivityModule_MainActivityInjector() {}
    
      @Binds
      @IntoMap
      @ClassKey(MainActivity.class)
      abstract AndroidInjector.Factory<?> bindAndroidInjectorFactory(
          MainActivitySubcomponent.Builder builder);
    
      @Subcomponent(modules = ABModule.class)
      public interface MainActivitySubcomponent extends AndroidInjector<MainActivity> {
        @Subcomponent.Builder
        abstract class Builder extends AndroidInjector.Builder<MainActivity> {}
      }
    }
    
很显然这个类和我们之前删去的类是一样的，这也就是为什么@ContributesAndroidInjector注解可以帮我们节省很多代码。  
demo地址:查看项目中的[app-version2模块](https://github.com/JavaNoober/DaggerAndroid)