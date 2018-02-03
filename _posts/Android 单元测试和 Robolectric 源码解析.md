---
title: Android 单元测试和 Robolectric 源码解析
date: 2017-05-14
tags:
    - Android
    - Android Unit Test
---

#### 前言
作为一个 Android 开发的程序员，最痛苦的事情其实莫过于测试了，龟速的模拟器和麻烦的手机，基本上每测试一次都要浪费 1-2 min 去加载程序。而有了 Robolectric 这些就可以避免了。至于 Robolectric 的介绍推荐大家看看[官网](http://robolectric.org/) （其中的用户指导是很好的学习资料）。我们用他的很大一个原因是他不需要再去把程序加载到 Android 手机或者模拟器中运行，他有自己的实现，能够调用 Android 中的很多库，下面的源码分析会提及。可是 Android Studio 对 Robolectric 不是很友好，而且在 Google 的官方教程中的测试工具也不是这个。。。同时 Robolectric 对于 Android Studio 的教程介绍似乎还是停留在 1.x 阶段，但是在 2.x 中的使用略有不同。

讲了这么多，所以 Robolectric 和 Junit 4 有什么不同？不都是测试吗？
我的理解是 Junit 4 与 Robolectric 的关系和 Java 与 Android 的关系差不多。毕竟 Robolectric 是个第三方的测试库，其中很多还是要用到 Junit 的。

好了，基本的介绍完成了，下面开始使用，但是使用不是我们这次的重点，重点是源码分析。但是源码分析也是建立在知道使用的基础上，如果你之前没有使用过，推荐你看官方的 user guide。或者这两篇 Blog。

- [Android单元测试框架Robolectric3.0介绍(一)](http://www.jianshu.com/p/9d988a2f8ff7#)
- [Android单元测试框架Robolectric3.0介绍(二)](http://www.jianshu.com/p/3aa0e4efcfd3)

<!-- more -->

#### 准备
其实前面 Blog 的介绍可能会有点偏差，现在的 Android Studio 2.3.1 在建立工程时除了自己的源码包，还有两个，分别是 test 和 androidTest。经过我的尝试， test 包下的测试文件可以直接测试，而 androidTest下面的还是要 android 运行环境的。个人更倾向于一个用做单元测试一个用于集成测试这种理解。所以在导入依赖包时要注意了。

```java
1. testCompile "org.robolectric:robolectric:3.3.2"
2. androidTestCompile "org.robolectric:robolectric:3.3.2"
```
第一种对应的测试文件要放在 test 包下，而第二种就是放在 androidTest 包下，如果搞错，你的 Robolectric 中的测试方法是无法使用的。

```xml
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:24.2.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.2'
    testCompile 'junit:junit:4.12'
    testCompile 'org.robolectric:robolectric:3.3.2'
    sourceCompatibility = 1.8
    targetCompatibility = 1.8
}
```

至于运行的话，大家可以直接右键文件，title bar 点击运行或者在 cmd 运行都行。但是有一点要注意，一定要配置文件的 working directory，不然找不到 manifest.xml 等文件。


#### 测试
写一个如下布局

```xml
<?xml version="1.0" encoding="utf-8"?>
<ScrollView xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    tools:context="org.fitzeng.robolectrictest.MainActivity">
    <LinearLayout
        android:orientation="vertical"
        android:layout_width="match_parent"
        android:layout_height="wrap_content">
        <EditText
            android:id="@+id/et"
            android:text="@string/app_name"
            android:maxLines="1"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
        <Button
            android:id="@+id/btn_et"
            android:text="EditText Test"
            android:textAllCaps="false"
            android:textAppearance="?android:attr/textAppearanceLarge"
            android:layout_width="match_parent"
            android:layout_height="wrap_content" />
    </LinearLayout>
</ScrollView>
```
```java
public class MainActivity extends AppCompatActivity implements View.OnClickListener{

    private EditText editText;
    private Button btnEt;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        initViews();

    }

    private void initViews() {
        editText = (EditText) findViewById(R.id.et);
        btnEt = (Button) findViewById(R.id.btn_et);
        btnEt.setOnClickListener(this);
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn_et: {
                btnEt.setText(editText.getText().toString());
                break;
            }
            default:
                break;
        }
    }
}
```
可以看到，这里就是测试一下点击事件，看看点击之后是否能获取 EditText 中的文本内容。

下面开始写测试代码，注意之前如果配置的是 testCompile 的话一定要把测试文件建在 test 包下，不然无法导入 Robolectric 。

```java
@RunWith(RobolectricTestRunner.class)
@Config(constants = BuildConfig.class, sdk = 24)
public class MainActivityTest {

    private Button btnEt;
    private EditText editText;

    @Before
    public void setUp() {
        // get mainActivity obj
        MainActivity mainActivity = Robolectric.setupActivity(MainActivity.class);
        // get button
        btnEt = (Button) mainActivity.findViewById(R.id.btn_et);
        // get edittext
        editText = (EditText) mainActivity.findViewById(R.id.et);
    }

    @Test
    public void testMainActivity() {
        // simulate click event
        btnEt.performClick();
        // get button content
        String expectedContent = btnEt.getText().toString();
        // get edittext content
        String actualContent = editText.getText().toString();
        // compare expectedContent and actualContent
        Assert.assertEquals(expectedContent, actualContent);
    }
}
```
上面的代码我都加了注释，基本过程很清楚了，这里点击事件是只要点击，我们就把 EditText 中的文本替换 Button 的文本。
测试结果如下：
![](testresult.png)
可以看到花的 27s 完成了点击测试，很方便。大家可以试试测试失败会是什么情况。

这里并没有开启模拟器，但是却完成了整个点击事件并进行了检测。事件虽小，但是意义很大。意味着我们不需要开启模拟器也可以对 Android 程序进行测试了。


#### 源码分析

在开始这个流程的分析之前，如果你之前对 Junit 没有任何了解，可以看看 [Junit 的分析](https://www.ibm.com/developerworks/cn/java/j-lo-junit-src/)，这里仿照 Junit 对 Robolectric 进行类比分析。
可以找到 Junit 的 main 入口
![](junitmain.png)
下面一步一步分析就好了。

但是 Robolectric 代码量实在太大，去里面找东西实在太难，但是凭借程序员的直觉，相信大家最先找到的有价值的文件可能是下面几个：
![](robolectricfile.png)
其中看到测试生命周期是不是眼前一亮？
![](robolectricclass.png)
还有 Robolectric 类里面的函数，包含了 Fragment， AttributeSet， Activity， Service， IntentService， ContentProvider 等等熟悉的内容。可以肯定，这里就是“宇宙中心”。上面的测试代码其实也验证了这个观点。
再看看 RobolectricTestRunner 这个类，还是和 Junit 4 有一腿的。里面的方法无非就是做一些初始化的工作，注释说的很明确了，就是提供一个模拟的 Android 运行时环境（和前面说的加载 manifest文件有关），这也是为什么可以直接在没有模拟器的情况下进行一些模拟点击测试的原因。

```java
/**
 * Installs a {@link SandboxClassLoader} and {@link ResourceTable} in order to
 * provide a simulation of the Android runtime environment.
 */
// public class RobolectricTestRunner extends SandboxTestRunner
// public class SandboxTestRunner extends BlockJUnit4ClassRunner
// public class BlockJUnit4ClassRunner extends ParentRunner<FrameworkMethod>
// public abstract class ParentRunner<T> extends Runner implements Filterable,Sortable
// public abstract class Runner implements Describable
```

这样没有实例只看源码有点不知道往哪个方向解析，所以我们开始从源码角度看看前面测试过程怎么执行的来加深理解。

最开始是调用了 Robolectric.setupActivity(Class<T>) 那么我们就从这里入手。

##### step 1 : Robolectric.setupActivity(Class<T>)

```java
public static <T extends Activity> T setupActivity(Class<T> activityClass) {
    return buildActivity(activityClass).setup().get();
}
```
首先是获取 Activity 对象，正好 setUpActivity(Class<T>) 是返回一个 Activity 对象的。那么是如何获取到的呢？那就要看源码具体实现了。大概过程是建立一个 Activity 然后 setup 一下再获取对象。 

##### step 2.1 : Robolectric.buildActivity(Class<T>)
```java
public static <T extends Activity> ActivityController<T> buildActivity(Class<T> activityClass) {
    return buildActivity(activityClass, null);
}
```

##### step 2.2 : Robolectric.buildActivity(Class<T>, Intent)
```java
public static <T extends Activity> ActivityController<T> buildActivity(Class<T> activityClass, Intent intent) {
    return ActivityController.of(getShadowsAdapter(), ReflectionHelpers.callConstructor(activityClass), intent);
}
```
到这里就可以看到利用这个类名和 Intent （null），采用反射机制可以获取这个类的对象。有兴趣的还可以深究下去。

##### step 3.1 : Robolectric.buildActivity(activityClass).setup()
```java
/**
* Calls the same lifecycle methods on the Activity called by Android the first time the Activity is created.
*
* @return Activity controller instance.
*/
public ActivityController<T> setup() {
    return create().start().postCreate(null).resume().visible();
}
/**
 * Calls the same lifecycle methods on the Activity called by Android when an Activity is restored from previously saved state.
 *
 * @param savedInstanceState Saved instance state.
 * @return Activity controller instance.
 */
public ActivityController<T> setup(Bundle savedInstanceState) {
    return create(savedInstanceState)
        .start()
        .restoreInstanceState(savedInstanceState)
        .postCreate(savedInstanceState)
        .resume()
        .visible();
}
```

前面的两个函数的注释很清楚，第一个是 Activity 第一次调用时调用，第二个是 Activity 之前被调用过并且在 Bundle 对象中保存了实例状态，可以将 Bundle 中的数据作为参数直接调用函数加载。返回的是一个 Activity 控制器实例。


##### step 3.2 : setup() -> create().start().postCreate(null).resume().visible()
```java
ActivityController.create()
public ActivityController<T> create() {
    return create(null);
}
public ActivityController<T> create(final Bundle bundle) {
    shadowMainLooper.runPaused(new Runnable() {
        @Override
        public void run() {
            ReflectionHelpers.callInstanceMethod(Activity.class, component, "performCreate", from(Bundle.class, bundle));
        }
    });
    return this;
}

ActivityController.start()
public ActivityController<T> start() {
    invokeWhilePaused("performStart");
    return this;
}

ActivityController.postCreate(Bundle)
public ActivityController<T> postCreate(Bundle bundle) {
    invokeWhilePaused("onPostCreate", from(Bundle.class, bundle));
    return this;
}

ActivityController.resume()
public ActivityController<T> resume() {
    invokeWhilePaused("performResume");
    return this;
}

ActivityController.visible()
public ActivityController<T> visible() {
    shadowMainLooper.runPaused(new Runnable() {
        @Override
        public void run() {
            ReflectionHelpers.setField(component, "mDecor", component.getWindow().getDecorView());
            ReflectionHelpers.callInstanceMethod(component, "makeVisible");
        }
    });

    ViewRootImpl root = component.getWindow().getDecorView().getViewRootImpl();
    if (root != null) {
        // If a test pause thread before creating an activity, root will be null as runPaused is waiting
        // Related to issue #1582
        Display display = Shadow.newInstanceOf(Display.class);
        Rect frame = new Rect();
        display.getRectSize(frame);
        Rect insets = new Rect(0, 0, 0, 0);
        final RuntimeAdapter runtimeAdapter = RuntimeAdapterFactory.getInstance();
        runtimeAdapter.callViewRootImplDispatchResized(
                root, frame, insets, insets, insets, insets, insets, true, null);
    }
    return this;
}
```

这里看起来代码量有点多，但是大概意思应该就是为 Activity 提供一个 Android 运行环境的保障。从 Window ， DecorView 之类的就可以确认我们的想法。如果不熟悉的话可以再去了解 Android 的界面绘制过程，这里目前知道这层意思就可以了。

##### step 4 : Robolectric.buildActivity(activityClass).setup().get()
```java
public T get() {
    return component;
}
```

。。。我看到这里首先的感觉是一脸懵逼，怎么这么简单，我们的预期是返回一个 Activity 对象，也就是一个组件，从意思来看可以理解，但是这个 component 是怎么来的？
下面就来慢慢分析。

```java
public abstract class ComponentController<C extends ComponentController<C, T>, T> {
  protected final C myself;
  protected T component;
  protected final ShadowLooperAdapter shadowMainLooper;

  protected Intent intent;

  protected boolean attached;

  @SuppressWarnings("unchecked")
  public ComponentController(ShadowsAdapter shadowsAdapter, T component, Intent intent) {
    this(shadowsAdapter, component);
    this.intent = intent;
  }

  @SuppressWarnings("unchecked")
  public ComponentController(ShadowsAdapter shadowsAdapter, T component) {
    myself = (C) this;
    this.component = component;
    shadowMainLooper = shadowsAdapter.getMainLooper();
  }

  public T get() {
    return component;
  }
}
```

看到这里我们可以确认的是 component 是来自 ComponentController() 构造函数来进行初始化的，这是你会发祥一个很熟悉的参数 ShadowsAdapter ，看看这个类。看完之后你会发现这其实是一个接口。这时你要思考的是，这些参数怎么来的，不可能凭空产生，肯定是在你的类生成 Activity 组件过程中构造的。看看 step 2.2 中的函数你会发现，第一个就是 getShadowsAdapter()，第二个是 ReflectionHelpers.callConstructor(activityClass)， 是不是发现了点什么，就是从这里开始，埋下了种子。同时 step 3.2 中有 
component ，发现他就是抽象类 ComponentController 中的变量，而且是 protected 的。那么现在唯一的猜想就是 ActivityController 继承自 ComponentController ，从类的命名来说是很合理的。那么源码是这样的吗？

```java
// public class ActivityController<T extends Activity> extends org.robolectric.util.ActivityController<T> 
// abstract public class ActivityController<T extends Activity> extends ComponentController<org.robolectric.android.controller.ActivityController<T>, T> 
```

看到这说明我们的想法完全正确！
总算弄清楚 component 的身份，但是还是不知道他是怎么生成的？我们的预测是和 Activity 要有关联，目前的分析还看不出来，而且这里的 step 3.2 中的 component 直接作为参数传递了，说明在之前就已经被初始化了，也就是构造函数 ComponentController(ShadowsAdapter shadowsAdapter, T component) 已经被调用了。那么我们还是要回到 step 2.2 中的 ActivityController.of(getShadowsAdapter(), ReflectionHelpers.callConstructor(activityClass), intent) 因为那是 Activity 的消失和 component 出现的临界点。
开始看看 of 函数。

```java
public static <T extends Activity> ActivityController<T> of(ShadowsAdapter shadowsAdapter, T activity, Intent intent) {
    return new ActivityController<>(shadowsAdapter, activity, intent).attach();
}

private ActivityController(ShadowsAdapter shadowsAdapter, T activity, Intent intent) {
    super(shadowsAdapter, activity, intent);
    this.shadowsAdapter = shadowsAdapter;
    shadowReference = shadowsAdapter.getShadowActivityAdapter(this.component);
}
```
这时看到 activity 传给了super，应该有点警觉，前面不是验证了 ActivityController 继承自 ComponentController 吗？而 component 又是在 ComponentController 中的，难道。。。

接着 super

```java
protected ActivityController(ShadowsAdapter shadowsAdapter, T activity, Intent intent) {
    super(shadowsAdapter, activity, intent);
}
public ComponentController(ShadowsAdapter shadowsAdapter, T component, Intent intent) {
    this(shadowsAdapter, component);
    this.intent = intent;
}
public ComponentController(ShadowsAdapter shadowsAdapter, T component) {
    myself = (C) this;
    this.component = component;
    shadowMainLooper = shadowsAdapter.getMainLooper();
}
```
费了这么一大圈最终还是找到了，就是和之前的猜想一致。就是 Activity 经过一系列的操作 (这里操作是指 ReflectionHelpers.callConstructor(activityClass), intent) ) 最终直接传递给 component 。

所以获取 Activity 对象的分析就告一段落了。至于构建细节，怎么在 JVM 中绘制 View 的我也不怎么清楚，要想了解可以对 step 3.1 中的函数进一步深究。我大概可以确定应该是在那个过程中完成的。

###### 过程总结

看图
![](setupactivity.png)

###### 模拟点击 -> performClick()

接下来就是根据获取的 Activity 来获取布局中的控件进行测试。

这里分析的是一个点击事件：

```java
/**
 * Call this view's OnClickListener, if it is defined.  Performs all normal
 * actions associated with clicking: reporting accessibility event, playing
 * a sound, etc.
 *
 * @return True there was an assigned OnClickListener that was called, false
 *         otherwise is returned.
 */
public boolean performClick() {
    final boolean result;
    final ListenerInfo li = mListenerInfo;
    if (li != null && li.mOnClickListener != null) {
        playSoundEffect(SoundEffectConstants.CLICK);
        li.mOnClickListener.onClick(this);
        result = true;
    } else {
        result = false;
    }

    sendAccessibilityEvent(AccessibilityEvent.TYPE_VIEW_CLICKED);
    return result;
}
```

你会发现这里的 performClick() 中其实是调用了 onClick() 的。下面就是要验证确定会执行这个点击事件。
首先 li != null 这个条件是怎么成立的？

```java
static class ListenerInfo {
    /**
     * Listener used to dispatch focus change events.
     * This field should be made private, so it is hidden from the SDK.
     * {@hide}
     */
    ......
}
```

通过上述代码你会发现这是一个 View 的静态内部类。所以其实 mListenerInfo 应该是类中的一个变量。那么这个变量是在何时被初始化的呢？

```java
ListenerInfo mListenerInfo;

ListenerInfo getListenerInfo() {
    if (mListenerInfo != null) {
        return mListenerInfo;
    }
    mListenerInfo = new ListenerInfo();
    return mListenerInfo;
}
```

经过一番查找可以肯定的是，前面应该是调用了 getListenerInfo() 函数。而对于一个 Button 的 listenerInfo 凭直觉可以猜测下很大可能是 setOnClickListener() 中调用的。因为设置点击监听应该是要获取监听信息的吧。。。再就是前面除了对 Button 设置了点击事件就没做其他操作了，而且如果事件得到触发的话意味着 mListenerInfo 一定是被初始化了的。种种猜测都指向 setOnClickListener() ，那就看看源码吧。

```java
/**
 * Register a callback to be invoked when this view is clicked. If this view is not
 * clickable, it becomes clickable.
 *
 * @param l The callback that will run
 *
 * @see #setClickable(boolean)
 */
public void setOnClickListener(@Nullable OnClickListener l) {
    if (!isClickable()) {
        setClickable(true);
    }
    getListenerInfo().mOnClickListener = l;
}
```

确实是如此，如果你没有进行前面的猜测的话，可以直接搜这个函数看看在哪些地方调用了，这时你会发现一个共同点：所有和视图监听有关的函数都有调用 getListenerInfo() 。至于为什么这样？很简单，因为他们都是 Listener ，自然需要 ListenerInfo 来确定自己是什么监听，并且通过 ListenerInfo 进行管理，只要 ListenerInfo 中的某个变量是 null 就意味着这个 Listener 是未注册的。这点可以从 li.mOnClickListener != null 这个条件验证，也就是前面点击事件的触发条件。

```java
public void setOnScrollChangeListener(OnScrollChangeListener l) {
    getListenerInfo().mOnScrollChangeListener = l;
}

public void setOnFocusChangeListener(OnFocusChangeListener l) {
    getListenerInfo().mOnFocusChangeListener = l;
}

public void addOnLayoutChangeListener(OnLayoutChangeListener listener) {
    ListenerInfo li = getListenerInfo();
    if (li.mOnLayoutChangeListeners == null) {
        li.mOnLayoutChangeListeners = new ArrayList<OnLayoutChangeListener>();
    }
    if (!li.mOnLayoutChangeListeners.contains(listener)) {
        li.mOnLayoutChangeListeners.add(listener);
    }
}
```

好了，点击事件就分析到这，总结一下：最开始就是设置监听，设置监听过程中会在 View 中将点击事件用 ListenerInfo 记录。在模拟点击事件中调用 performClick() ，下面就是对事件是否注册来确定是否触发点击事件。

之后就是自己写逻辑了，对你的预期和模拟跑出来的结果是否一致进行测试。这里就是 Junit 内容了。

#### 总结

通过这次源码分析，发现只要细心，很多大牛的代码慢慢读也是可以读懂的。虽然代码量太大不可能一行不落地阅读，但是从一个小例子出发，慢慢分析就可以得出你自己理解，其实代码的逻辑就是常人的逻辑，其中值得学习的恰恰是这种常人逻辑之间的协调和对代码整个的宏观把控。


这是我第一次写关于源码分析的文章，其中肯定有很多不足，欢迎大家指正。
还有推荐下我的个人 [Blog](http://fitzeng.org)

最后
感谢阅读