---
title: 实现一个类似QQ的社交聊天工具-1
date: 2017-04-14 11:00
tags:
    - Android
    - ZZChat
---

[GitHub](https://github.com/mk43/ZZChat)

## 实现一个类似QQ的社交聊天工具-1

### 准备
- AndroidStudio
- 模拟器

[资料](https://pan.baidu.com/s/1c1SabD2) 密码: jme4
大家将图片复制到drawable供接下来的使用，部分图片源于网络，不做商业用途应该不算侵权吧。如果有，我会删除资源的。

### 实现目标

![](Demo.gif)
按照最先开始的计划，我们只实现一个静态的ZZChat界面，考验的就是Android控件的基本知识。如果碰到没见到过的控件可以去Google看开发文档。

### 实现过程

<!-- more -->

![](showfile1.png)
在看到实现的设计下，我们最先想到的是有四的Activity（欢迎界面，引导页，登录注册，主界面），同时对应四个布局

#### 修改Manifest

```xml
AndroidManifest.xml

<application
    android:allowBackup="true"
    android:icon="@drawable/icon"
    android:label="@string/app_name"
    android:supportsRtl="true"
    android:theme="@style/AppTheme">
    <activity android:name=".aty.AtyWelcome">
        <intent-filter>
            <action android:name="android.intent.action.MAIN" />

            <category android:name="android.intent.category.LAUNCHER" />
        </intent-filter>
    </activity>
    <activity android:name=".aty.AtyGuide" />
    <activity android:name=".aty.AtyLoginOrRegister" />
    <activity android:name=".aty.AtyMain" />
</application>
```

#### 欢迎界面

- 全屏

```java
onCreate()

getSupportActionBar().hide();
getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
```

- 引导界面只在首次开启时显示

```java
AtyWelcome.java

private void initLoad() {
    SharedPreferences sharedPreferences = getSharedPreferences("zzchat", MODE_PRIVATE);
    boolean welcome = sharedPreferences.getBoolean("welcome", true);
    if (!welcome) {
        handler.sendEmptyMessageDelayed(GO_HOME, DELAY);

    } else {
        handler.sendEmptyMessageDelayed(GO_GUIDE, DELAY);
        SharedPreferences.Editor editor = sharedPreferences.edit();
        editor.putBoolean("welcome", false);
        editor.apply();
    }
}

Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        switch (msg.what) {
            case GO_GUIDE: {
                goGuide();
                break;
            }
            case GO_HOME: {
                goHome();
                break;
            }
            default:
                break;
        }
    }
};
```

#### 引导页

引导页我们使用一个ViewPager实现，如果之前不熟悉的可以看我的另一个[利用ViewPager做的轮播图](http://fitzeng.org/2017/04/07/SlideShow/)。

- 布局

相信看了前面动图的效果对布局实现应该是有底了

```xml
aty_guide.xml

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <android.support.v4.view.ViewPager
        android:id="@+id/vp_guide"
        android:layout_width="match_parent"
        android:layout_height="match_parent">
    </android.support.v4.view.ViewPager>

    <LinearLayout
        android:orientation="horizontal"
        android:layout_centerHorizontal="true"
        android:layout_alignParentBottom="true"
        android:layout_marginBottom="20dp"
        android:layout_width="wrap_content"
        android:layout_height="wrap_content">
        <ImageView
            android:id="@+id/iv_indicator_dot1"
            android:src="@drawable/selected"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="10dp"
            android:layout_width="10dp"
            android:layout_height="10dp" />

        <ImageView
            android:id="@+id/iv_indicator_dot2"
            android:src="@drawable/unselected"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="10dp"
            android:layout_width="10dp"
            android:layout_height="10dp" />

        <ImageView
            android:id="@+id/iv_indicator_dot3"
            android:src="@drawable/unselected"
            android:layout_marginLeft="10dp"
            android:layout_marginRight="10dp"
            android:layout_width="10dp"
            android:layout_height="10dp" />
    </LinearLayout>

</RelativeLayout>
```

- 适配布局

到这了，如何实现ViewPager加载布局就是我们现在应该想的事了。
目前可以最先想到和做到的是实现三个加载的布局，为了方便我们只使用一个ImageView来实现，同理其他三个页面也是类似，第三个多加一个Enter入口进入主页。

```xml
guide_page1.xml

<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
    <ImageView
        android:src="@drawable/shot2"
        android:scaleType="centerCrop"
        android:layout_width="match_parent"
        android:layout_height="match_parent" />
</RelativeLayout>
```

- 适配器

现在的情况是有了布局和资源页面，怎么把资源页面加载进布局，这时就要用到Adapter了，也就是适配器。
新建一个adapter资源包
新建一个AdapterGuideViewPager类继承PagerAdapter

```java
adapter/AdapterGuideViewPager.java

public class public class AdapterGuideViewPager extends PagerAdapter{

    private Context context;
    private List<View> viewList;

    public AdapterGuideViewPager(Context context, List<View> viewList) {
        this.context = context;
        this.viewList = viewList;
    }

    @Override
    public int getCount() {
        return viewList.size();
    }

    @Override
    public boolean isViewFromObject(View view, Object object) {
        return (view == object);
    }

    @Override
    public void destroyItem(ViewGroup container, int position, Object object) {
        container.removeView(viewList.get(position));
    }

    @Override
    public Object instantiateItem(ViewGroup container, int position) {
        container.addView(viewList.get(position));
        return viewList.get(position);
    }
}
```

一定要注意getCount()和isViewFromObject()函数的实现。

有了适配器，只要给adapter添加之前的guide视图作为资源，再给viewPager设置资源适配器。基本效果就实现了。

```java
AtyGuide.java

private void initViews() {
    // load view
    final LayoutInflater inflater = LayoutInflater.from(this);

    viewList = new ArrayList<>();
    viewList.add(inflater.inflate(R.layout.guide_page1, null));
    viewList.add(inflater.inflate(R.layout.guide_page2, null));
    viewList.add(inflater.inflate(R.layout.guide_page3, null));

    // bind Id with imageView
    for (int i = 0; i < indicatorDotIds.length; i++) {
        imageViews[i] = (ImageView) findViewById(indicatorDotIds[i]);
    }

    adapterGuideViewPager = new AdapterGuideViewPager(this, viewList);

    viewPager = (ViewPager) findViewById(R.id.vp_guide);
    viewPager.setAdapter(adapterGuideViewPager);
    viewPager.addOnPageChangeListener(this);

    btnToMain = (Button) (viewList.get(2)).findViewById(R.id.btn_to_main);
    btnToMain.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Intent intent = new Intent(AtyGuide.this, AtyLoginOrRegister.class);
            startActivity(intent);
        }
    });
}
```

- 指示器实现

当前页面是第几页，要给用户一个比较明显的提示，可以利用两个不同颜色的小圆点。但是要想知道移动的改变就要实现监听事件

实现onPageSelected()方法就可以了。

```java
AtyGuide.java

public void onPageSelected(int position) {
    for (int i = 0; i < indicatorDotIds.length; i++) {
        if (i != position) {
            imageViews[i].setImageResource(R.drawable.unselected);
        } else {
            imageViews[i].setImageResource(R.drawable.selected);
        }
    }
}
```


##### 登录注册

- 界面

这里可以自己设计，我使用TabHost实现，学习使用不同控件，不过布局值得主页的是ID的设置，自己可以试试如果不这样会出现什么效果。

```xml
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:background="@drawable/shot1"
    android:layout_width="match_parent"
    android:layout_height="match_parent">

    <TabHost android:id="@+id/tabHost">
        
        <TabWidget android:id="@android:id/tabs"> 
        </TabWidget>

        <FrameLayout android:id="@android:id/tabcontent">
            <LinearLayout> Login Layout </LinearLayout>
            <LinearLayout> Register Layout </LinearLayout>
        </FrameLayout>
    </TabHost>
</RelativeLayout>
```

- 跳转

目前还没进行数据处理，所以我们直接跳转进入界面

```java
private void initViews() {
    tabHost = (TabHost) findViewById(R.id.tabHost);

    btnLogin = (Button) findViewById(R.id.btn_login);
    etLoginUsername = (EditText) findViewById(R.id.et_login_username);
    etLoginPassword = (EditText) findViewById(R.id.et_login_password);

    btnRegister = (Button) findViewById(R.id.btn_register);
    etRegisterUsername = (EditText) findViewById(R.id.et_register_username);
    etRegisterPassword = (EditText) findViewById(R.id.et_register_password);
    etInsurePassword = (EditText) findViewById(R.id.et_insure_password);

    tabHost.setup();
    tabHost.addTab(tabHost.newTabSpec("Login").setIndicator("Login").setContent(R.id.layout_login));
    tabHost.addTab(tabHost.newTabSpec("Register").setIndicator("Register").setContent(R.id.layout_register));

    btnLogin.setOnClickListener(this);
    btnRegister.setOnClickListener(this);
}

@Override
public void onClick(View v) {
    switch (v.getId()) {
        case R.id.btn_login: {
            Intent intent = new Intent(this, AtyMain.class);
            startActivity(intent);
            finish();
            break;
        }
        case R.id.btn_register: {
            Intent intent = new Intent(this, AtyMain.class);
            startActivity(intent);
            finish();
            break;
        }
        default:
            break;
    }
}
```

- 添加依赖库

![](packageerror.png)
如果你遇到上面的bug，说明到现在我们的TabHost是无法工作的，因为缺少一个依赖库。
compile 'com.android.support:design:2x.x.x'

添加方式是在File->Project Structure 在弹出的窗口中选择app,之后操作看图
![](addpackage.png)
此时需要重新Gradle, 这时可能一个错误在build.gradle。按Alt + Enter, 选择忽略就好。

```java
dependencies {
    compile fileTree(include: ['*.jar'], dir: 'libs')
    androidTestCompile('com.android.support.test.espresso:espresso-core:2.2.2', {
        exclude group: 'com.android.support', module: 'support-annotations'
    })
    compile 'com.android.support:appcompat-v7:25.3.1'
    compile 'com.android.support.constraint:constraint-layout:1.0.1'
    testCompile 'junit:junit:4.12'
    compile 'com.android.support:design:26.0.0-alpha1'
}
```
#### 主界面

之前的页面基本实现，那么主界面如何实现，参考QQ，为了避免控件上的使用难度，我们直接用google提供的DrawerLayout
写代码之前先理清思路，这个主界面明显是包含三个页面加一个侧换页面，也就是四个。
新建一个view资源文件，创建四个视图类
接着新建四个布局供类加载

- 布局

这个布局有要主页的地方
侧滑视图要设置android:layout_gravity="start" 属性。
DrawerLayout最好为根容器
推荐如下布局
最外层就是DrawerLayout，中间只有一个主内容和一个侧滑布局。你要添加的内容全部在主内容中实现。

这里采用ViewPager + TabLayout 来实现，不熟悉的点[这里](http://fitzeng.org/2017/04/07/TabLayout/)

```xml
aty_main.xml

<android.support.v4.widget.DrawerLayout >

    <LinearLayout >
        <android.support.v4.view.ViewPager > </android.support.v4.view.ViewPager>
        <android.support.design.widget.TabLayout > </android.support.design.widget.TabLayout>
    </LinearLayout>

    <org.fitzeng.zzchat.view.SlideLayout android:layout_gravity="start">侧换视图</org.fitzeng.zzchat.view.SlideLayout>

</android.support.v4.widget.DrawerLayout>
```

- 加载页面

以Chats为例

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
    android:background="#09868f"
    android:orientation="vertical"
    android:layout_width="match_parent"
    android:layout_height="match_parent">
</LinearLayout>
```

```java
public class LayoutChats extends Fragment {

    private View rootView;

    @Nullable
    @Override
    public View onCreateView(LayoutInflater inflater, @Nullable ViewGroup container, @Nullable Bundle savedInstanceState) {
        rootView = inflater.inflate(R.layout.layout_chats, container, false);
        return rootView;
    }
}
```

十分简单，看代码就能懂

之前已经使用了侧滑视图，接下来的三个视图分别对应加载进了三个类。如何将这些视图加载进主界面，前面已经说过如何加载经viewPager了，这里也是一样的。实现一个适配器。

- 适配器

和前面不一样的是前面直接inflate一个布局进资源列表，而这里是把布局加载进类中了。所以实现方式稍微有点不一样, 这样对布局的内容控制性个人认为较好,因为布局中的子控件逻辑可以在各自的类中实现。看代码就懂了。

```java
adapter/AdapterMainViewPager.java

public class AdapterMainViewPager extends FragmentPagerAdapter {

    private List<Fragment> fragmentList = new ArrayList<>();

    public AdapterMainViewPager(FragmentManager fragmentManager) {
        super(fragmentManager);
    }

    public void addFragment(Fragment fragment) {
        fragmentList.add(fragment);
    }

    @Override
    public Fragment getItem(int position) {
        return fragmentList.get(position);
    }

    @Override
    public int getCount() {
        return fragmentList.size();
    }
}
```

```java
private void initViews() {
    drawable = (DrawerLayout) findViewById(R.id.dl_main);
    viewPager = (ViewPager) findViewById(R.id.vp_main);
    tabLayout = (TabLayout) findViewById(R.id.tl_main);

    tabList = new ArrayList<>();

    AdapterMainViewPager adapter = new AdapterMainViewPager(getSupportFragmentManager());

    adapter.addFragment(new LayoutChats());
    adapter.addFragment(new LayoutContacts());
    adapter.addFragment(new LayoutMoments());

    viewPager.setAdapter(adapter);

    tabLayout.setupWithViewPager(viewPager);

    tabList.add(tabLayout.getTabAt(0));
    tabList.add(tabLayout.getTabAt(1));
    tabList.add(tabLayout.getTabAt(2));
    tabList.get(0).setIcon(R.drawable.icon).setText("Chats");
    tabList.get(1).setIcon(R.drawable.icon).setText("Contacts");
    tabList.get(2).setIcon(R.drawable.icon).setText("Moments");
}
```

代码比较简洁，不知道意思的可以按ctrl点击类名或方法名看源码注释。

![](showfile2.png)
到这里基本的效果实现了，不清楚了可以参考阶段性源码。