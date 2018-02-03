---
title: 实现一个类似QQ的社交聊天工具-2
date: 2017-04-14 12:00
tags:
    - Android
    - ZZChat
---

[GitHub](https://github.com/mk43/ZZChat)

## 实现一个类似QQ的社交聊天工具-2

### 准备

做好【ZZChatApp1】中的内容并且下载了[实现一个类似QQ的社交聊天工具-1](http://fitzeng.org/2017/04/14/ZZChatApp1/)中的资料就可以开始下面的了

### 实现目标

![GIF](Demo.gif)
前面只是实现一个基本的框架，这次我们要往主界面里面添加内容，代码较多，最好自己看这篇文章时也参考我提供的第二阶段源码。
主要练习的是自定义控件

### 实现过程

<!-- more -->

![](showfile1.png) ![](showfile2.png)
这次的文件比上次多了，所以文件管理和控件ID的命名不能太随意，这是要注意的一点。代码难度不高，只要命名好了，不写注释其实也能看懂的。

#### TitleBar

在开始写三个页面和一个侧滑视图之前，我们先做个自定义控件。其实这此实验总共用到了两个，我在这只叙述复杂的那个，剩下的可以参考源码自己实现。在学习之前推荐观看[慕课](http://www.imooc.com/video/5079)，时间不是很长，但是对理清思路视频讲解更直接。

- 设计属性

其实就是两个button加一个textView。属性设计如下

```xml
values/atts.xml

<declare-styleable name="TitleBar">
    <attr name="titleText" format="string" />
    <attr name="titleTextSize" format="dimension" />
    <attr name="titleTextColor" format="color" />
    <attr name="titleBackground" format="reference|color" />

    <attr name="leftText" format="string" />
    <attr name="leftTextSize" format="dimension" />
    <attr name="leftTextColor" format="color" />
    <attr name="leftBackground" format="reference|color" />

    <attr name="rightText" format="string" />
    <attr name="rightTextSize" format="dimension" />
    <attr name="rightTextColor" format="color" />
    <attr name="rightBackground" format="reference|color" />
</declare-styleable>
```

- 绑定属性

可以根据前面设计的属性来判断需要哪些变量

```java
private Button btnLeft;
private Button btnRight;
private TextView tvTitle;

private String leftText;
private float leftTextSize;
private int leftTextColor;
private Drawable leftBackground;

private String rightText;
private float rightTextSize;
private int rightTextColor;
private Drawable rightBackground;

private String titleText;
private float titleTextSize;
private int titleTextColor;
private Drawable titleBackground;

/**
 * @param attrs this is titlebar's attribute set for binding with widgets
 */
private void findAttrs(AttributeSet attrs) {
    TypedArray typedArray = context.obtainStyledAttributes(attrs, R.styleable.TitleBar);

    leftText = typedArray.getString(R.styleable.TitleBar_leftText);
    leftTextSize = typedArray.getDimension(R.styleable.TitleBar_leftTextSize, 24);
    leftTextColor = typedArray.getColor(R.styleable.TitleBar_leftTextColor, 0);
    leftBackground = typedArray.getDrawable(R.styleable.TitleBar_leftBackground);

    titleText = typedArray.getString(R.styleable.TitleBar_titleText);
    titleTextSize = typedArray.getDimension(R.styleable.TitleBar_titleTextSize, 24);
    titleTextColor = typedArray.getColor(R.styleable.TitleBar_titleTextColor, 0);
    titleBackground = typedArray.getDrawable(R.styleable.TitleBar_titleBackground);

    rightText = typedArray.getString(R.styleable.TitleBar_rightText);
    rightTextSize = typedArray.getDimension(R.styleable.TitleBar_rightTextSize, 24);
    rightTextColor = typedArray.getColor(R.styleable.TitleBar_rightTextColor, 0);
    rightBackground = typedArray.getDrawable(R.styleable.TitleBar_rightBackground);

    typedArray.recycle();
}

/**
 * Init titlebar's widgets
 */
private void initViews() {
    btnLeft = new Button(context);
    btnRight = new Button(context);
    tvTitle = new TextView(context);

    btnLeft.setText(leftText);
    btnLeft.setTextSize(leftTextSize);
    btnLeft.setTextColor(leftTextColor);
    btnLeft.setBackground(leftBackground);

    btnRight.setText(rightText);
    btnRight.setTextSize(rightTextSize);
    btnRight.setTextColor(rightTextColor);
    btnRight.setBackground(rightBackground);

    tvTitle.setText(titleText);
    tvTitle.setTextSize(titleTextSize);
    tvTitle.setTextColor(titleTextColor);
    tvTitle.setBackground(titleBackground);

    tvTitle.setGravity(Gravity.CENTER);

    setBackgroundColor(0xFF01AAFF);
}

/**
 * Setting titlebar's layout
 */
private void setTitleBarLayoutParams() {

    btnLeft.setAllCaps(false);
    btnRight.setAllCaps(false);

    LayoutParams btnLeftLayoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
    btnLeftLayoutParams.addRule(RelativeLayout.ALIGN_PARENT_LEFT, TRUE);
    addView(btnLeft, btnLeftLayoutParams);

    LayoutParams btnRightLayoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.WRAP_CONTENT);
    btnRightLayoutParams.addRule(RelativeLayout.ALIGN_PARENT_RIGHT, TRUE);
    addView(btnRight, btnRightLayoutParams);

    LayoutParams titleLayoutParams = new LayoutParams(ViewGroup.LayoutParams.WRAP_CONTENT, ViewGroup.LayoutParams.MATCH_PARENT);
    titleLayoutParams.addRule(RelativeLayout.CENTER_IN_PARENT, TRUE);
    addView(tvTitle, titleLayoutParams);
}
```

其实从使用角度来说，可以理出一条我们为什么这么做的线。
因为在 XML 中我们使用一般都是 android : layout_width = "match_parent"
很明显 layout_width 可以看做一个 Key ，而 match_parent 是一个 Value.
所以 findAttrs(AttributeSet attrs) 就是把之前设计属性是所能识别的 Key 的值取出来，也就是 "XXX" 中的 XXX 数据，但是仅仅取出数据并没有什么用，最终还是要把数据赋值到控件上去，数据才能显示我们想要的效果。所以 initViews() 便是做这件事的。接下来的 setTitleBarLayoutParams() 只是对里面控件的属性的一个约束。

对于 Key-Value 的理解，其实可以这样看，如果声明了一个 app:no_name="12" 在 xml 中，而实际你并没有声明这个属性在 atts.xml 中，所以编译器知道的是找不到 Key ，而不是判定 Value 的对错。所以对于属性文件 atts.xml 就自然而然有了存在的意义。我理解为就是规范作用，而之后的值传递给 TextView 或者其他已有控件，就是之后的绑定属性值的操作就顺理成章了。

文笔不好，不知道讲清了没。。。大家还是好好看看前面推荐的视频和自己写一个小控件实现一下加深理解。

目前这个控件只是静态的，不能响应点击事件。可以参考之前的 Button 怎么实现的。一般 Button.setOnClickListener()
所以可以在控件中加一个 setTitleBarClickListetner() 方法。接下来就是有点难度的了。Button 自己的点击事件响应其实是由自己在 onClick() 方法中实现自己的逻辑的。 所以此时自定义控件就要对外提供一个接口，让实现者自己定义自己的点击事件。讲到这估计差不多了，看看代码就理解了。

```java
view/TitleBar.java

private titleBarClickListener listener;

/**
 * implement click events
 */
public interface titleBarClickListener {
    // 这两个方法相当于 Button 的 onCLick()
    void leftButtonClick();
    void rightButtonClick();
}

// 类似于 Button 的setOnClickListener();
public void setTitleBarClickListetner(titleBarClickListener listetner) {
    this.listener = listetner;
}

// 内部设置按钮点击监听
private void setButtonClickListener() {
    btnLeft.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            listener.leftButtonClick();
        }
    });

    btnRight.setOnClickListener(new OnClickListener() {
        @Override
        public void onClick(View v) {
            listener.rightButtonClick();
        }
    });
}
```

下面剩下的一个 PicAndTextBtn 大家可以自己实现，就是作为左侧滑动视图中的小控件，一个 ImageView 和 TextView组成的。

#### 侧滑界面


![](layoutslide.png)
如果前面都写好了下面就使用上面的控件来写滑动界面，布局很简单。

```xml
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android">

    <RelativeLayout>

        <LinearLayout>

            <LinearLayout>
                <ImageView/> Avatar
                <TextView/> Username
            </LinearLayout>

            <TextView/> Sign

        </LinearLayout>

    </RelativeLayout>

    <LinearLayout>
        <org.fitzeng.zzchat.view.PicAndTextBtn>Dress Up</org.fitzeng.zzchat.view.PicAndTextBtn>
        <org.fitzeng.zzchat.view.PicAndTextBtn>Profile</org.fitzeng.zzchat.view.PicAndTextBtn>
    </LinearLayout>

    <LinearLayout>
        <org.fitzeng.zzchat.view.PicAndTextBtn>setting</org.fitzeng.zzchat.view.PicAndTextBtn>
        <org.fitzeng.zzchat.view.PicAndTextBtn>night</org.fitzeng.zzchat.view.PicAndTextBtn>
    </LinearLayout>

</LinearLayout>
```

接下来就是实现其逻辑了，这次主要实现以下setting，dressup和profile下次实验实现。
实现控件的点击，虽然这个点击事件是你们自己写的，为了后面的实验一致，推荐命名最好和我一致。。。虽然我的命名也很烂。。。

```java
view/LayoutSlide.java

private void initViews() {
    this.addView(LayoutInflater.from(context).inflate(R.layout.layout_slide, null));

    dressUp = (PicAndTextBtn) findViewById(R.id.patb_dressup);
    profile = (PicAndTextBtn) findViewById(R.id.patb_profile);
    setting = (PicAndTextBtn) findViewById(R.id.patb_setting);
    night = (PicAndTextBtn) findViewById(R.id.patb_night);

    dressUp.setOnClickListener(new PicAndTextBtn.picAndTextBtnClickListener() {
        @Override
        public void onClick(View view) {
            Intent intent = new Intent(context, AtyDressUp.class);
            context.startActivity(intent);
        }
    });

    profile.setOnClickListener(new PicAndTextBtn.picAndTextBtnClickListener() {
        @Override
        public void onClick(View view) {
            Intent intent = new Intent(context, AtyProfile.class);
            context.startActivity(intent);
        }
    });

    setting.setOnClickListener(new PicAndTextBtn.picAndTextBtnClickListener() {
        @Override
        public void onClick(View view) {
            Intent intent = new Intent(context, AtySetting.class);
            context.startActivity(intent);
        }
    });

    night.setOnClickListener(new PicAndTextBtn.picAndTextBtnClickListener() {
        @Override
        public void onClick(View view) {
            if (nightMode) {
                findViewById(R.id.layout_slide).setBackgroundColor(0xff878787);
                nightMode = false;
            } else {
                findViewById(R.id.layout_slide).setBackgroundColor(0xffe9e9e9);
                nightMode = true;
            }
        }
    });
}
```

setting 界面的布局就不贴代码了，很简单。只分析 java 代码，其实目前也很水，也就实现了一个 Guide View 是否播放的功能。但是在之前的 Welcome 页面中还要做一点小小的修改。大家可以试验不该会怎么样。

```java
AtySetting.java

private void initViews() {
    titleBar = (TitleBar) findViewById(R.id.tb_setting);
    guide = (ImageView) findViewById(R.id.iv_setting_guide);
    password = (ImageView) findViewById(R.id.iv_setting_password);
    offline = (ImageView) findViewById(R.id.iv_setting_offline);

    guideMode = getSharedPreferences("zzchat", MODE_PRIVATE).getBoolean("guide", true);
    guide.setImageResource(guideMode ? R.drawable.btnselected : R.drawable.btnunselected);
    password.setImageResource(passwordMode ? R.drawable.btnselected : R.drawable.btnunselected);
    offline.setImageResource(offlineMode ? R.drawable.btnselected : R.drawable.btnunselected);

    guide.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            if (guideMode) {
                guide.setImageResource(R.drawable.btnunselected);
                guideMode = false;
            } else {
                guide.setImageResource(R.drawable.btnselected);
                guideMode = true;
            }
        }
    });

    password.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            if (passwordMode) {
                password.setImageResource(R.drawable.btnunselected);
                passwordMode = false;
            } else {
                password.setImageResource(R.drawable.btnselected);
                passwordMode = true;
            }
        }
    });

    offline.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            if (offlineMode) {
                offline.setImageResource(R.drawable.btnunselected);
                offlineMode = false;
            } else {
                offline.setImageResource(R.drawable.btnselected);
                offlineMode = true;
            }
        }
    });

    titleBar.setTitleBarClickListetner(new TitleBar.titleBarClickListener() {
        @Override
        public void leftButtonClick() {
            SharedPreferences sharedPreferences = getSharedPreferences("zzchat", MODE_PRIVATE);
            SharedPreferences.Editor editor = sharedPreferences.edit();
            editor.putBoolean("guide", guideMode);
            editor.apply();
            finish();
        }

        @Override
        public void rightButtonClick() {

        }
    });
}
```

night 功能也基本没有实现，留给大家完善了。。。

#### chats

![](layoutchats.png)
下面开始 Chat 页面的设计。同理后面的 Contact 和 Moment 也是和这个类似，大家参考源码可以把代码写了，当做练习。有些坑自己不踩不知道多深。

把 titleBar 引进来是很简单的。直接在主界面中加就是了。

实现 Chats 这个列表功能我们使用更加易用的 RecyclerView 下次实验实现的聊天界面我们可以试试 ListView。好了，都决定好了就开始写代码吧。

和前面一样，一个 RecyclerView 怎么加载数据？答案就是通过 Adapter ，问题又来了，那么加载的怎样的数据呢？所以现在目的很明确，就是设计数据 Item

布局是一个头像，一个Username, 一个签名

```xml
item_user.xml

<LinearLayout>
    <ImageView android:id="@+id/iv_item_avatar" />
    <LinearLayout>
        <TextView android:id="@+id/tv_item_username" />
        <TextView android:id="@+id/tv_item_sign" />
    </LinearLayout>
</LinearLayout>
```

有了布局，接下来就是自己构造适配器加载布局。


```java
adapter/AdapterUserItem.java

public class AdapterUserItem extends RecyclerView.Adapter<AdapterUserItem.BaseViewHolder> {

    private Context context;
    private List<UserItemMsg> userItemMsgList;

    public AdapterUserItem(Context context, List<UserItemMsg> userItemMsgList) {
        this.context = context;
        this.userItemMsgList = userItemMsgList;
    }

    @Override
    public BaseViewHolder onCreateViewHolder(ViewGroup parent, int viewType) {
        return new BaseViewHolder(LayoutInflater.from(context).inflate(R.layout.item_user, parent, false));
    }

    @Override
    public void onBindViewHolder(BaseViewHolder holder, int position) {
        holder.ivAvatar.setImageResource(userItemMsgList.get(position).getIconID());
        holder.tvUsername.setText(userItemMsgList.get(position).getUsername());
        holder.tvSign.setText(userItemMsgList.get(position).getSign());
    }

    @Override
    public int getItemCount() {
        return (userItemMsgList == null ? 0 : userItemMsgList.size());
    }

    class BaseViewHolder extends RecyclerView.ViewHolder{

        private ImageView ivAvatar;
        private TextView tvUsername;
        private TextView tvSign;

        BaseViewHolder(View itemView) {
            super(itemView);
            ivAvatar = (ImageView) itemView.findViewById(R.id.iv_item_avatar);
            tvUsername = (TextView) itemView.findViewById(R.id.tv_item_username);
            tvSign = (TextView) itemView.findViewById(R.id.tv_item_sign);

            itemView.setOnClickListener(new View.OnClickListener() {
                @Override
                public void onClick(View v) {
                    Toast.makeText(context, tvUsername.getText().toString(), Toast.LENGTH_SHORT).show();
                }
            });
        }
    }
}
```

我把代码全贴上来了，其实这算是一个经常用的模板了吧。RecyclerView 内部自己实现了 ViewHolder 类，所以说使用方便。

下面就是比较有代表性的 recyclerView 代码实现，可以注意注意 setLayoutManager() 函数。

```java
private void initViews() {
    context = getContext();
    recyclerView = (RecyclerView) rootView.findViewById(R.id.chatsRecycleView);

    loadData();

    adapterUserItem = new AdapterUserItem(context, userItemMsgList);

    recyclerView.setLayoutManager(new LinearLayoutManager(context));
    recyclerView.setAdapter(adapterUserItem);
}
```

怕篇幅过多，细的知识点不会太陈述，主要是对整个一个 App 实现过程中思考的一个介绍，不会觉得自己无从下手。

。。。。。。。。好像篇幅有点长了，还没有涉及服务端就这么多了。。。剩下的效果大家可以按动图显示的自己实现，方法其实全概括了。

![](layoutcontacts.png) ![](layoutmoments.png)

最后说下 TabHost 的一点东西

```java
tabLayout.addOnTabSelectedListener(new TabLayout.OnTabSelectedListener() {
    @Override
    public void onTabSelected(TabLayout.Tab tab) {
        tabList.get(tab.getPosition()).setIcon(ImageManager.imageID[tab.getPosition() + 3]);
        tabLayout.setTabTextColors(
                ContextCompat.getColor(AtyMain.this, R.color.colorBlack),
                ContextCompat.getColor(AtyMain.this, R.color.colorBlue)
        );
    }

    @Override
    public void onTabUnselected(TabLayout.Tab tab) {
        tabList.get(tab.getPosition()).setIcon(ImageManager.imageID[tab.getPosition()]);
    }

    @Override
    public void onTabReselected(TabLayout.Tab tab) {

    }
});
```

为什么要把下面的代码放在监听中执行，而不是在外面。大家可以做做实验，点击Tab和滑动ViewPager就能发现异同。

```java
tabLayout.setTabTextColors(
        ContextCompat.getColor(AtyMain.this, R.color.colorBlack),
        ContextCompat.getColor(AtyMain.this, R.color.colorBlue)
);
```

还有前面遗留的登录界面 Tab 上面的字母是大写，怎么解决？大家可以自己查资料，再看看我提供的源码。这次就这样，下次实现剩下的 DressUp，Profile 和聊天界面。再下次就是网络编程啦！