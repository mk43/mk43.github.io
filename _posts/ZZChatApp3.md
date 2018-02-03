---
title: 实现一个类似QQ的社交聊天工具-3
date: 2017-04-14 13:00
tags:
    - Android
    - ZZChat
---

[GitHub](https://github.com/mk43/ZZChat)

## 实现一个类似QQ的社交聊天工具-3

### 准备

做好【ZZChatApp2】中的内容并且下载了[实现一个类似QQ的社交聊天工具-1](http://fitzeng.org/2017/04/14/ZZChatApp1/)中的资料就可以开始下面的了

### 实现目标

![GIF](Demo.gif)
看演示效果就知道，这次主要的任务是实现两个界面，一个是 Dressup 选图片的，另一个是聊天界面。如果你对 RecyclerView 有一定认识了，可以自己自行编写 Dressup 界面。我们的聊天信息使用的是 listview ，尝试尽可能多的控件。好消息是这次 lab 之后基本的界面工作就完成了，下次 lab 我们开始进行网络编程。

### 实现过程

<!-- more -->

![](fileshow1.png) ![](fileshow2.png)
Profile 界面就不讲了，自己参照源码或者自己写，这里为了方便就直接把 username 作为 id 和 nickname 主要是为了后期数据库的建立不太麻烦，毕竟是个小练习。数据处理不是我们关注的重点，重点是整个开发流程。

#### Dressup实现

![](layoutprofile.png) ![](layoutdressup.png) 

我的习惯是先写界面

```xml
aty_dress_up.xml

<LinearLayout  >

    <org.fitzeng.zzchat.view.TitleBar android:id="@+id/tb_dress_up" >
    </org.fitzeng.zzchat.view.TitleBar>

    <TextView android:text="Choose an avatar"  />

    <android.support.v7.widget.RecyclerView
        android:id="@+id/rv_avatar" >
    </android.support.v7.widget.RecyclerView>

    <TextView android:text="Choose a background" />

    <android.support.v7.widget.RecyclerView
        android:id="@+id/rv_background" >
    </android.support.v7.widget.RecyclerView>

    <Button android:id="@+id/btn_save" />
</LinearLayout>
```

布局很简单，接下来就是建一个Aty去加载布局。布局好了，这是应该是加载资源，而加载资源又不得不用到我们的适配器。所以先写适配器。

下面以加载 Avatar 为例。

```java
adapter/AdapterAvatar.java

public class AdapterAvatar extends RecyclerView.Adapter<AdapterAvatar.BaseViewHoder>{

    private List<ImageMsg> imageViews;
    private Context context;
    private LayoutInflater inflater;
    private static int selectedImageAvatar = 0;
    private List<RelativeLayout> imageContainer = new ArrayList<>();
    private Drawable bgImageDrawable;

    @RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
    public AdapterAvatar(Context context, List<ImageMsg> imageViews) {
        this.context = context;
        this.imageViews = imageViews;
        this.inflater = LayoutInflater.from(context);
        selectedImageAvatar = 0;
        bgImageDrawable = context.getResources().getDrawable(R.drawable.bgimage, null);
    }

    @Override
    public BaseViewHoder onCreateViewHolder(ViewGroup parent, int viewType) {
        View view = inflater.inflate(R.layout.choose_image, parent, false);
        return new BaseViewHoder(view);
    }

    @Override
    public void onBindViewHolder(final BaseViewHoder holder, final int position) {
        holder.imageView.setImageResource(imageViews.get(position).getImageID());
        imageContainer.get(selectedImageAvatar).setBackground(bgImageDrawable);

        holder.imageView.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                if (position != selectedImageAvatar) {
                    imageContainer.get(position).setBackground(bgImageDrawable);
                    imageContainer.get(selectedImageAvatar).setBackgroundColor(0);
                    selectedImageAvatar = position;
                }
            }
        });
    }

    @Override
    public int getItemCount() {
        return imageViews == null ? 0 : imageViews.size();
    }

    class BaseViewHoder extends RecyclerView.ViewHolder {
        ImageView imageView;

        BaseViewHoder(View itemView) {
            super(itemView);
            imageView = (ImageView) itemView.findViewById(R.id.image);
            RelativeLayout layout =  (RelativeLayout) itemView.findViewById(R.id.imageContainer);
            imageContainer.add(layout);
        }
    }
}
```

代码很简洁，都是套路。唯一注意的一点是点击的逻辑处理。还有就是把图片资源（util/ImageManager）用一个类封装一下，和创建一个对象（util/ImageMsg）用来存储加载数据的信息。用起来就比较方便了。适配器好了，下面利用适配器把数据加载进 RecyclerView 。思路还是比较清晰的。

具体实现看代码吧。

```java
@RequiresApi(api = Build.VERSION_CODES.LOLLIPOP)
private void initViews() {
    titleBar = (TitleBar) findViewById(R.id.tb_dress_up);
    rvAvatar = (RecyclerView) findViewById(R.id.rv_avatar);
    rvBackground = (RecyclerView) findViewById(R.id.rv_background);
    btnSave = (Button) findViewById(R.id.btn_save);

    addAvatarView();

    titleBar.setTitleBarClickListetner(new TitleBar.titleBarClickListener() {
        @Override
        public void leftButtonClick() {
            finish();
        }

        @Override
        public void rightButtonClick() {
        }
    });

    btnSave.setOnClickListener(new View.OnClickListener() {
        @Override
        public void onClick(View v) {
            Toast.makeText(AtyDressUp.this, "saved", Toast.LENGTH_SHORT).show();
            finish();
        }
    });
}

private void addAvatarView() {
    List<ImageMsg> imageMsgs = new ArrayList<>();
    for (int anImagesAvatar : ImageManager.imagesAvatar) {
        ImageMsg imagemsg = new ImageMsg();
        imagemsg.setImageID(anImagesAvatar);
        imageMsgs.add(imagemsg);
    }

    AdapterAvatar avatarAdapter = new AdapterAvatar(this, imageMsgs);
    LinearLayoutManager layoutManager = new LinearLayoutManager(this, LinearLayoutManager.HORIZONTAL, false);
    rvAvatar.setLayoutManager(layoutManager);
    rvAvatar.setAdapter(avatarAdapter);
}
```

之后加载 Background 的代码自己写吧,这一部分就算完成了，不懂的可以参照源码。

#### 聊天界面

![](layoutchatroom.png)
看过第一行代码的应该有点了解，大家可以先上网搜搜其他资源。

主要问题是怎么让消息自适应气泡。有一种图片时.9.png格式，至于如何制作，很简单。选中图片右键，在最下面的选项中有一个Create 9-Patch file...点击之后就会弹出界面，自己把要缩放的区域集中在黑线的交汇区域。
![](ninepic.png)

聊天界面布局如下，这里有一个Bug，下面会陈述。

```xml
<LinearLayout  >
    <org.fitzeng.zzchat.view.TitleBar
        android:id="@+id/tb_chat_room" >
    </org.fitzeng.zzchat.view.TitleBar>
    <ScrollView >
        <LinearLayout >
            <ListView android:id="@+id/lv_chat_room" > </ListView>
            <LinearLayout >
                <Button android:text="emoji" />
                <Button android:text="draw" />
                <Button android:text="file" />
                <Button android:text="call" />
            </LinearLayout>
            <LinearLayout>
                <EditText android:id="@+id/myMsg"/>
                <Button android:id="@+id/btnSend"/>
            </LinearLayout>
        </LinearLayout>
    </ScrollView>
</LinearLayout>
```

接下来就是消息界面的设计，也就是适配器加载的布局。一个头像，一个 username 一个内容。OK。但是要区分是自己发送的消息还是别人发送的消息。所以有两个布局。

这时我们要建立一个聊天信息的Msg

```java
private boolean myInfo;
private int iconID;
private String username;
private String content;
private String chatObj;
```

数据什么的都贮备好了，下面进行是适配器的编写。由于 ListView 没有实现 ViewHolder 所以要自己实现。主要是为了视图缓存，减小视图加载时的资源消耗。

```java
adaptar/AdapterChatMsg.java

public class AdapterChatMsg extends ArrayAdapter<ChatMsg> {
    private LayoutInflater inflater;
    private List<ChatMsg> chatMsgs;
    public AdapterChatMsg(@NonNull Context context, @LayoutRes int resource, List<ChatMsg> chatMsgs) {
        super(context, resource);
        this.inflater = LayoutInflater.from(context);
        this.chatMsgs = chatMsgs;
    }

    @Override
    public int getCount() {
        return chatMsgs.size();
    }

    @Nullable
    @Override
    public ChatMsg getItem(int position) {
        return chatMsgs.get(position);
    }

    @NonNull
    @Override
    public View getView(int position, @Nullable View convertView, @NonNull ViewGroup parent) {

        ChatMsg msg = getItem(position);
        View view;
        ViewHolder viewHolder;

        if (convertView == null) {
            assert msg != null;
            if (msg.isMyInfo()) {
                view = inflater.inflate(R.layout.chat_me, parent, false);
            } else {
                view = inflater.inflate(R.layout.chat_other, parent, false);
            }
            viewHolder = new ViewHolder();
            viewHolder.icon = (ImageView) view.findViewById(R.id.icon);
            viewHolder.username = (TextView) view.findViewById(R.id.username);
            viewHolder.content = (TextView) view.findViewById(R.id.content);

            view.setTag(viewHolder);
        } else {
            view = convertView;
            viewHolder = (ViewHolder) view.getTag();
        }
        viewHolder.icon.setImageResource(chatMsgs.get(position).getIconID());
        viewHolder.username.setText(chatMsgs.get(position).getUsername());
        viewHolder.content.setText(chatMsgs.get(position).getContent());
        return view;
    }

    private class ViewHolder {
        ImageView icon;
        TextView username;
        TextView content;
    }
}
```

其中通过

```java
if (msg.isMyInfo()) {
    view = inflater.inflate(R.layout.chat_me, parent, false);
} else {
    view = inflater.inflate(R.layout.chat_other, parent, false);
}
```

来加载聊天左右视图。

```java
AtyChatRoom.java

public class AtyChatRoom extends AppCompatActivity{

    private TitleBar titleBar;
    private ListView listView;
    private EditText myMsg;
    private Button btnSend;
    private List<ChatMsg> chatMsgList;
    private AdapterChatMsg adapterChatMsgList;
    private String chatObj;


    @Override
    protected void onCreate(@Nullable Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        getSupportActionBar().hide();
//        getWindow().setFlags(WindowManager.LayoutParams.FLAG_FULLSCREEN, WindowManager.LayoutParams.FLAG_FULLSCREEN);
        setContentView(R.layout.aty_chat_room);
        getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_RESIZE |
                WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN);
//        AndroidBug5497Workaround.assistActivity(this);
        initViews();
    }

    private void initViews() {

        titleBar = (TitleBar) findViewById(R.id.tb_chat_room);
        listView = (ListView) findViewById(R.id.lv_chat_room);
        myMsg = (EditText) findViewById(R.id.myMsg);
        btnSend = (Button) findViewById(R.id.btnSend);
        chatMsgList = new ArrayList<>();

        chatObj = getIntent().getStringExtra("username");
        titleBar.setTitleText(chatObj);

        adapterChatMsgList = new AdapterChatMsg(AtyChatRoom.this, R.layout.chat_other, chatMsgList);

        listView.setAdapter(adapterChatMsgList);

        btnSend.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                String content = myMsg.getText().toString();
                if (!content.isEmpty()) {
                    ChatMsg msg = new ChatMsg();
                    msg.setContent(content);
                    msg.setUsername("hello");
                    msg.setIconID(R.drawable.avasterwe);
                    msg.setMyInfo(true);
                    msg.setChatObj(chatObj);
                    chatMsgList.add(msg);
                    myMsg.setText("");
                }
            }
        });

        titleBar.setTitleBarClickListetner(new TitleBar.titleBarClickListener() {
            @Override
            public void leftButtonClick() {
                finish();
            }
            @Override
            public void rightButtonClick() { }
        });
    }
}
```

这次我把整个Aty都贴出来了，主要是为了说明一个Bug。点击输入框软键盘弹出时 TitleBar 会被顶上去。为了解决这个问题，在onCreate函数中，加 SOFT_INPUT_ADJUST_RESIZE ，但是在全屏下同时设置 SOFT_INPUT_ADJUST_RESIZE 这个属性，TitleBar 又会被顶上去。所以，在 Stack Overflow 有大神给出了解决方案。就是 util 多出的那个 AndroidBug5497Workaround.java 文件。讲道理在 setContentView() 之后添加一句 AndroidBug5497Workaround.assistActivity(this); 就可以解决。。。。。。但是。。。可能是我的手机太渣，还是没能实现效果。据说这个是适合市面上大部分手机的。。。希望在你的手机上能行，所以我就保留了这些文件。为了取舍，我只能不实现全屏了。。。日后找到好的解决方案会在github更新。

目前好像。。完了。。。。看着少，其实里面的有些东西是值得深究的。至于如何实现 Moments 和 Contacts 滑动的留给大家自己探索了。


感谢大家的耐心阅读和支持，再下次开始之前，希望你已经搭建好了本地服务器。