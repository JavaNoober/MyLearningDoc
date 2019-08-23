

本系列博客主要用于自己学习积累，顺便帮助新手入门，如有问题，多多包涵。更详细的一些使用可以看看其他更加详细的博客。

[dagger android 学习（一）：dagger基础使用](https://juejin.im/post/5cc71fb7e51d456e6479b4fe)  
[dagger android 学习（二）：AndroidInjector的使用](https://juejin.im/post/5cc7202fe51d456e31164a6c)  
[dagger android 学习（三）：ContributesAndroidInjector的进一步优化](https://juejin.im/post/5cc72061e51d456e6154b4bc)  
[dagger android 学习（四）：基于dagger2的mvp架构](https://juejin.im/post/5cc72088e51d456e6d13351d)  


### 添加依赖

    implementation 'com.google.dagger:dagger:2.21'
    annotationProcessor 'com.google.dagger:dagger-compiler:2.21'
### 基本注解介绍
#### @Inject
用来标记构造函数，以及标记实例，这样使用被标记的对象的时候，会自动调用所标记的构造函数来创建对象。
#### @Module
用于标记提供对象的类，这个类负责提供一些我们需要的对象，比如SharedPreferences是通过构造函数无法直接提供的，这时候就可以通过一个有@Module的类，这个类负责提供内容
#### @Provides
Module标记了提供对象的类，而Provides则标记了这个类里面提供对象的方法，例如：

    @Module
    public class ABParamModule {
        A a;
        B b;
    
        public ABParamModule(A a, B b) {
            this.a = a;
            this.b = b;
        }
    
        @Provides
        public A provideA2() {
            return a;
        }
    
        @Provides
        public B provideB2() {
            return b;
        }
    
        @Provides
        public C provideC() {
            return new C(a,b);
        }
    }
    
#### @Component
Component用于标记接口，这个接口把提供依赖的类，和使用依赖的类串联起来

#### @Scope、@Singleton
用于标记类，在同时被标记的类的范围里，对象创建回是单例的

### 基本使用方式
Bean类：

    public class A {

        @Inject
        public A(){
    
        }
    
        public void someThingA(){
            Log.e("DAGGER", "someThingA");
        }
    }
    
    public class B {

        @Inject
        public B(){
    
        }
    
        public void someThingB(){
            Log.e("DAGGER", "someThingB");
        }
    }
    
    
    public class C {
        private A a;
    
        private B b;
    
        @Inject
        public C(A a, B b) {
            this.a = a;
            this.b = b;
        }
    
    
        public void doSomethingC(){
            a.someThingA();
            b.someThingB();
        }
    }
    
    
    @Singleton
    public class UserInfo {

        @Inject
        public UserInfo(){
    
        }
    
        private String userName;
    
        private String age;
    
        private int id;
    
    
        public int getId() {
            return id;
        }
    
        public String getAge() {
            return age;
        }
    
        public String getUserName() {
            return userName;
        }
    
        public void setAge(String age) {
            this.age = age;
        }
    
        public void setId(int id) {
            this.id = id;
        }
    
        public void setUserName(String userName) {
            this.userName = userName;
        }
    }
Module类：

    @Module
    public class ABParamModule {
        A a;
        B b;
    
        public ABParamModule(A a, B b) {
            this.a = a;
            this.b = b;
        }
    
        @Provides
        public A provideA2() {
            return a;
        }
    
        @Provides
        public B provideB2() {
            return b;
        }
    
        @Provides
        public C provideC() {
            return new C(a,b);
        }
    }
    
Component类：

    @Singleton
    @Component
    public interface MyAppComponent {
        void inject(MainActivity activity);
    }
    
Activity使用：

    public class MainActivity extends AppCompatActivity {
    
        @Inject
        A a;
    
        @Inject
        C c;
    
    
        @Inject
        UserInfo userInfo;
    
    
        @Inject
        UserInfo userInfo2;
    
        @Override
        protected void onCreate(Bundle savedInstanceState) {
            super.onCreate(savedInstanceState);
            setContentView(R.layout.activity_main);
            DaggerMyAppComponent.builder().build().inject(this);
            a.someThingA();
            c.doSomethingC();
            userInfo.setAge("111");
            Log.e("DAGGER", userInfo.toString());
            Log.e("DAGGER", userInfo2.toString());
        }
    }

打印结果：
因为UserInfo被Singleton标记了，所以这是一个单例。这样最简单的一个dagger2使用就完成了。


    9-04-19 17:37:42.986 23362-23362/? E/DAGGER: someThingA
    2019-04-19 17:37:42.986 23362-23362/? E/DAGGER: someThingA
    2019-04-19 17:37:42.986 23362-23362/? E/DAGGER: someThingB
    2019-04-19 17:37:42.986 23362-23362/? E/DAGGER: com.xiaoqi.daggerandroid.UserInfo@9c4fd37
    2019-04-19 17:37:42.986 23362-23362/? E/DAGGER: com.xiaoqi.daggerandroid.UserInfo@9c4fd37
    
    
demo地址:查看项目中的[app-simple](https://github.com/JavaNoober/DaggerAndroid)