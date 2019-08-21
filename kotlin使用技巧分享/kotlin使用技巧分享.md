## Delegates

### Delegates.notNull()

如果某些变量在定义的时候，不能直接进行初始化操作，但是从代码逻辑上看，这个变量一定是会被初始化的，不过kotlin会强制要求定义的时候携带"?"，这样之后需要使用的时候都需要添加？，进行判空。  
**Delegates.notNull()**属性可以帮助我们在初始化的时候不再需要携带"?"
以如下代码为例：

    class MyApplication : Application() {
    
        companion object{
            var context: Application by Delegates.notNull() // 可以不再携带"？"
            var context2: Application? = null // 必须给定一个null的初始值
        }
    
        override fun onCreate() {
            super.onCreate()
            context = this
            context2 = this
        }
    }

### Delegates.observable

该方法可以帮助直接定义观察者，在值改变的时候进行回调。  

    var observerValue by Delegates.observable("1"){
            property: KProperty<*>, s: String, s1: String ->
        println("$s -> $s1")
    }

    @Test
    fun changeValue(){
        observerValue = "222" // 打印 1 -> 222
    }

## let、with、run、apply、also

### let

    public inline fun <T, R> T.let(block: (T) -> R): R {
        return block(this)
    }
    
使用：

    object.let{
       it.todo()//在函数体内使用it替代object对象去访问其公有的属性和方法
       ...
    }
    
    //另一种用途 判断object为null的操作
    object?.let{//表示object不为null的条件下，才会去执行let函数体
       it.todo()
    }

#### let使用前后对比

使用前：

    mVideoPlayer?.setVideoView(activity.course_video_view)
    	mVideoPlayer?.setControllerView(activity.course_video_controller_view)
    	mVideoPlayer?.setCurtainView(activity.course_video_curtain_view)

使用后：
    
    mVideoPlayer?.let {
    	   it.setVideoView(activity.course_video_view)
    	   it.setControllerView(activity.course_video_controller_view)
    	   it.setCurtainView(activity.course_video_curtain_view)
    }

### with

    public inline fun <T, R> with(receiver: T, block: T.() -> R): R = receiver.block()
    
使用：
 
     val result = with(user) {
            println("my name is $name, I am $age years old, my phone number is $phoneNum")
            1000
        }
        
#### with使用前后对比
使用前：

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
    
       ArticleSnippet item = getItem(position);
    		if (item == null) {
    			return;
    		}
    		holder.tvNewsTitle.setText(StringUtils.trimToEmpty(item.titleEn));
    		holder.tvNewsSummary.setText(StringUtils.trimToEmpty(item.summary));
    		String gradeInfo = "难度：" + item.gradeInfo;
    		String wordCount = "单词数：" + item.length;
    		String reviewNum = "读后感：" + item.numReviews;
    		String extraInfo = gradeInfo + " | " + wordCount + " | " + reviewNum;
    		holder.tvExtraInfo.setText(extraInfo);
    		...
    }
    
使用后：

    override fun onBindViewHolder(holder: ViewHolder, position: Int){
       val item = getItem(position)?: return
       with(item){
          holder.tvNewsTitle.text = StringUtils.trimToEmpty(titleEn)
    	   holder.tvNewsSummary.text = StringUtils.trimToEmpty(summary)
    	   holder.tvExtraInf.text = "难度：$gradeInfo | 单词数：$length | 读后感: $numReviews"
           ...   
       }
    
    }
    
### run

    public inline fun <T, R> T.run(block: T.() -> R): R = block()
    
使用：

    object.run{
    //todo 相对于let不再需要it
    }
    
#### run使用前后对比
使用前：

    @Override
    public void onBindViewHolder(ViewHolder holder, int position) {
    
       ArticleSnippet item = getItem(position);
    		if (item == null) {
    			return;
    		}
    		holder.tvNewsTitle.setText(StringUtils.trimToEmpty(item.titleEn));
    		holder.tvNewsSummary.setText(StringUtils.trimToEmpty(item.summary));
    		String gradeInfo = "难度：" + item.gradeInfo;
    		String wordCount = "单词数：" + item.length;
    		String reviewNum = "读后感：" + item.numReviews;
    		String extraInfo = gradeInfo + " | " + wordCount + " | " + reviewNum;
    		holder.tvExtraInfo.setText(extraInfo);
    		...
    }
    
使用后：

    override fun onBindViewHolder(holder: ViewHolder, position: Int){
      getItem(position)?.run{
          holder.tvNewsTitle.text = StringUtils.trimToEmpty(titleEn)
    	   holder.tvNewsSummary.text = StringUtils.trimToEmpty(summary)
    	   holder.tvExtraInf = "难度：$gradeInfo | 单词数：$length | 读后感: $numReviews"
           ...   
       }
    }
    
### apply

作用功能和run函数很像，唯一不同点就是它返回的值是对象本身，而run函数是一个闭包形式返回，返回的是最后一行的值。

    public inline fun <T> T.apply(block: T.() -> Unit): T { block(); return this }
    
使用：

    val result = user.apply {
        println("my name is $name, I am $age years old, my phone number is $phoneNum")
        1000
    }
    println("result: $result")
    

#### apply使用前后对比

使用前：

    mSheetDialogView = View.inflate(activity, R.layout.biz_exam_plan_layout_sheet_inner, null)
            mSheetDialogView.course_comment_tv_label.paint.isFakeBoldText = true
            mSheetDialogView.course_comment_tv_score.paint.isFakeBoldText = true
            mSheetDialogView.course_comment_tv_cancel.paint.isFakeBoldText = true
            mSheetDialogView.course_comment_tv_confirm.paint.isFakeBoldText = true
            mSheetDialogView.course_comment_seek_bar.max = 10
            mSheetDialogView.course_comment_seek_bar.progress = 0
            
使用后：

    mSheetDialogView = View.inflate(activity, R.layout.biz_exam_plan_layout_sheet_inner, null).apply{
       course_comment_tv_label.paint.isFakeBoldText = true
       course_comment_tv_score.paint.isFakeBoldText = true
       course_comment_tv_cancel.paint.isFakeBoldText = true
       course_comment_tv_confirm.paint.isFakeBoldText = true
       course_comment_seek_bar.max = 10
       course_comment_seek_bar.progress = 0
    
    }
    
### also
let是以闭包的形式返回，返回函数体内最后一行的值，如果最后一行为空就返回一个Unit类型的默认值。而also函数返回的则是**传入对象的本身**。

    public inline fun T.also(block: (T) -> Unit): T { block(this); return this }
    
使用：

    val result = "testAlso".also {
            println(it.length)
            1000
        }
    println(result)//打印testAlso
    
#### also使用前后对比
与let类似，返回对象不同而已

## takeIf、takeUnless

### takeIf
    public inline fun <T> T.takeIf(predicate: (T) -> Boolean): T? = if (predicate(this)) this else null
使用：

    val result = "Hello World".takeIf {
        it.length > 1
    }
    
    println(result)// print Hello World

#### takeIf使用例子

    val outDirFile = File(outputDir.path).takeIf { it.exists() } ?: return false
    // do something with existing outDirFile
    
    val index = input.indexOf(keyword).takeIf { it >= 0 } ?: error("keyword not found")
    // do something with index of keyword in input string, given that it's found
    
### takeUnless

    public inline fun <T> T.takeUnless(predicate: (T) -> Boolean): T? = if (!predicate(this)) this else null
    
定义上看，takeIf和takeUnless只是在方法体中的if的判断相反，其他都一样。

## lazy懒加载

之后调用member的时候都会去进行判断是否已经初始化，没有的话会自动进行初始化操作。

    val member by lazy { LazyMember() }
    
    fun lazyMemberTest(){
        member.doSomethings()
    }

## filter、map

### filter
过滤某些数据：

e.g.

    val riskList = result.data.riskList.filter {
                it.wxUid != myWxid
            }


### map
    public inline fun <K, V, R> Map<out K, V>.map(transform: (Map.Entry<K, V>) -> R): List<R> {
        return mapTo(ArrayList<R>(size), transform)
    }

使用：

    private fun convertWechatPerson(selectContacts: List<EncryptionContactPrevious>): ArrayList<WechatPerson> {
        val contacts = ArrayList<WechatPerson>()
        selectContacts.map {
            contacts.add(WechatPerson(it.username, it.alias, it.conRemark, it.domainList, it.nickName, it.pyInitial, it.quanPin, it.showHead, it.type, it.imgHeadPath, it.labels))
        }
        return contacts
    }
    
不同于foreach，map会返回一个List集合

    private fun convertWechatPerson(selectContacts: List<EncryptionContactPrevious>): ArrayList<WechatPerson> {
        return selectContacts.map {
            WechatPerson(it.username, it.alias, it.conRemark, it.domainList, it.nickName, it.pyInitial, it.quanPin, it.showHead, it.type, it.imgHeadPath, it.labels)
        }
    }

## Kotlin函数作为参数

kotlin除了支持字段类型做完参数，还支持函数方法作为参数：

    private var mOnItemClickListener: ((View,Int) -> Unit)? = null

    viewHolder.convertView.setOnClickListener { v ->
        mOnItemClickListener?.let {
            val position = viewHolder.adapterPosition
            it.invoke(v,position)
        }
    }

这样使用的时候就无需再定义一个接口

## kotlin支持扩展函数

Kotlin的扩展函数可以让你作为一个类成员进行调用的函数，但是是定义在这个类的外部。这样可以很方便的扩展一个已经存在的类，为它添加额外的方法。

rxjava线程切换：

    fun <T> Observable<T>.setThread(): Observable<T> {
        return subscribeOn(Schedulers.io())
             .observeOn(AndroidSchedulers.mainThread())
    }

findViewById：

    fun <V : View> Activity.bindView(id: Int): Lazy<V> {
        return lazy { findViewById<V>(id) }
    }

更加通用的自定义表达式：

    fun <T> T.checkExpression(expression: Boolean, function: T.() -> T): T {
        if (expression) {
            function.invoke(this)
        }
        return this
    }


### T.()->Unit 和 ()->Unit 的区别

定义两个函数：

    fun <T : View> T.afterMearsure(f: T.() -> Unit){
    }
    
    fun <T : View> T.afterMearsure2(f: () -> Unit){
    }
    
区别在于this的作用范围：
![image](https://raw.githubusercontent.com/JavaNoober/KotlinUsage/master/kotlin1.png)

根据这一特性可以构建DSL风格代码

## kotlin的lambda表达式规则

不用lambda表达式：

    // 这里举例一个Android中最常见的按钮点击事件的例子
    mBtn.setOnClickListener(object : View.OnClickListener{
            override fun onClick(v: View?) {
                Toast.makeText(this,"onClick",Toast.LENGTH_SHORT).show()
            }
        })
        
等价于：

    // 调用
    mBtn.setOnClickListener { v -> Toast.makeText(this,"onClick",Toast.LENGTH_SHORT).show() }
    
特性如下：

- 表达式总是被大括号括着
- 其参数(如果存在)在 -> 之前声明(参数类型可以省略)
- 函数体(如果存在)在 -> 后面

相比java8的lambda表达式，还有其他特性：

### 当方法中最后一个参数为函数类型，则可以放在大括号调用

    fun lambdaFun(boolean: Boolean, block: (Int) -> Unit){
        
    }
    
    fun lambdaFunTest(){
        lambdaFun(true, block = {
            
        })
        
        lambdaFun(true){
            
        }
    }

### it
it是在当一个高阶函数中Lambda表达式的参数只有一个的时候可以使用it来使用此参数。it可表示为单个参数的隐式名称，是Kotlin语言约定的。

    val arr = arrayOf(1,3,5,7,9)
    // 过滤掉数组中元素小于2的元素，取其第一个打印。这里的it就表示每一个元素。
    println(arr.filter { it < 5 })
    
### 下划线（_）

如果参数未使用，可以隐藏，如果是多个参数，某一个未使用可以用_来表示

## 协程

类似于线程，但是占用内存更低且更高效，同时在协程中执行代码，是顺序执行，而java 线程需要通过回调。

        // do first: main
        // do async: DefaultDispatcher-worker-5
        // do end: main
        GlobalScope.launch(Dispatchers.Main) {
            println("do first: ${Thread.currentThread().name}")
            GlobalScope.async(Dispatchers.Default) {
                println("do async: ${Thread.currentThread().name}")
            }.await()
            println("do end: ${Thread.currentThread().name}")
        }
        
## DSL

DSL（domain specific language），即领域专用语言：专门解决某一特定问题的计算机语言，比如大家耳熟能详的 SQL 和正则表达式。

    val yesterday = 1.days.ago
    
    val yesterday = 1 days ago
    
    val twoMonthsLater = 2 months fromNow
    
    activity!!.startActivity<ChatActivity, Any>(ChatActivity.CORP_ID to data.corpId,
        ChatActivity.PEER_STATUS to data.peerStatus, ChatActivity.IM_ID to data.imId,
        ChatActivity.USER_NAME to data.userName, ChatActivity.USER_HEAD to data.avatar)
        
## DSL风格的retrofit配合协程、lifecycle的网络请求