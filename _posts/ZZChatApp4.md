---
title: 实现一个类似QQ的社交聊天工具-4
date: 2017-04-14 14:00
tags:
    - Android
    - ZZChat
---

[GitHub](https://github.com/mk43/ZZChat)

## 实现一个类似QQ的社交聊天工具-4

### 准备

做好【ZZChatApp3】中的内容并且下载了[实现一个类似QQ的社交聊天工具-1](http://fitzeng.org/2017/04/14/ZZChatApp1/)中的资料就可以开始下面的了
环境自己尝试是否能配好，下面只是给出一些提示和建议。
Xampp + Java EE

### 实现目标

![GIF](Demo.gif)
这里已经实现基本的通讯了。由于最近特别忙，所以打算写两篇的网络编程总结成一篇，细节应该都会提到，只是不会太详细。希望大家多利用身边的网络资源。基本在网上都有现成的答案。

### 实现过程

<!-- more -->

![](show.png)
这就是大体的方向，所以目前目标很简单，为了不打断以后的编码思路，前期工作要做好。

#### 环境配置

![](apache.png) ![](window.png) ![](jfram.png)
下载Xampp 开启 Apache 和 mySql, 在浏览器中输入 127.0.0.1:  看看有 Xampp 界面就成功，点击 admin 出现数据库就说明数据库可以访问了。

![](mysqljar.png)
至于 Java EE 自身是不携带 WindowBuilder 也就是你无法使用 JFrame 的图形界面进行设计。怎么安装动手搜搜就知道了。
之后就只要添加访问数据库的一个库到工程里面。在资料4已经给出，导入 buildpath 就可以了。


#### 数据库创建

![](dbdesign.png) ![](dbcreate.png) ![](dbmsg.png)
这里只是效果图，数据库的内容我们通过代码来创建，可以保证每一次的测试环境一样。


#### 通讯协议

这一部分的目的是数据解析要用到的。可以这样想，我发送一个登录消息和聊天消息服务器能区分吗？如果能够区分，那是怎么区分的？这就是设计通讯协议的最原始原因。其实如果学过网络通讯就知道，TCP/IP协议有个头，这个头就存在这某些信息，代表着自己身份，之后的信息就按这身份的协议去解析。
这里就用一个很简单的 [Action]:[info, info, ... , info] 来作为对象传输。[Action]就是一个头，后面连着消息，至于消息如何解析，就要看你自己协议的具体约定了。这里我提供一个很简陋的在资源4中，也就是 Version lab版 所遵守的协议。

#### 服务端

![](showlogininout.png)
这里我们使用 socket 来编写。如果之前你没有接触过这方面，可能现在想知道手机怎么和电脑通讯？别急，先看看这个例子在电脑上怎么访问自己写的服务器。

![](telnet.png)

演示很简单，就是一个简单的界面来开启服务，如果你是测试,可以先不编写界面，直接运行就开启服务 serverListener.start(); 之后在cmd telnet 中通过 127.0.0.1 这个 IP 访问端口 27777 。可以看到控制台输出 “haha”， 如果你的电脑没有 telnet 可以开启或者直接在浏览器中访问 127.0.0.1:27777. 效果一样。这是就代表有一个和服务器建立的连接。

从这里可以猜想，手机如果连接的是电脑的无线，也就是处于同一个局域网内的话，是不是也可以访问到浏览器。
可以试试，手机连接电脑无线，那么通过什么访问端口呢？ 127.0.0.1？仔细思考就知道应该是行不通的，手机和电脑建立的连接走的是哪条线路呢？我们连的是无线，所以通过无线适配器和电脑建立连接，所以我们要访问电脑上的端口肯定是要经过这个适配器的。所以找到无线网络适配器的 IPv4, 就可以 通过 IPv4:27777 访问了。

下面解释一下代码。

```java
public class ServerListener extends Thread{
    private ServerSocket serverSocket;
    @Override
    public void run() {
        try {
            serverSocket = new ServerSocket(27777, 27);
            while (true) {
                Socket socket = serverSocket.accept();
                System.out.println("haha");
                ChatSocket chatSocket = new ChatSocket(socket);
                chatSocket.start();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

可以看到，这是一个死循环。只要不发生异常和手动关闭，这段代码是会一直运行下去的。
其中

```java
Socket socket = serverSocket.accept();
```

这一行是关键，代表的是没有 socket 接入的话，程序会一直阻塞在这句代码处，一旦有 socket 接入，则返回一个 socket ，由于一个服务器必然是有多个客户端连接的。所以我们给每个 socket 连接都分配一个线程并且开启线程，接着循环进入阻塞，直到有下一个连接建立，重复上述步骤。这就是这段代码的运行状态。


现在假设有线程已经开启了。
怎么接收数据？ socket 是以流的形式传输数据。所以只要获取流，再将这个流进行相关操作就行了。
下面解释一下我们的主要代码。

```java
public class ChatSocket extends Thread{

    private Socket socket;
    private String message = null;
    private BufferedReader bufferedReader;
    private BufferedWriter bufferedWriter;
    
    public ChatSocket(Socket s) {
        this.socket = s;
        try {
            this.bufferedReader = new BufferedReader(new InputStreamReader(s.getInputStream(), "UTF-8"));
            this.bufferedWriter = new BufferedWriter(new OutputStreamWriter(s.getOutputStream(), "UTF-8"));
        } catch (IOException e) {
            e.printStackTrace();
        }

    }
    
    @Override
    public void run() {
        try {
            String line = null;
            while ((line = bufferedReader.readLine()) != null) {
                if (!line.equals("-1")) {   
                    message += line;
                } else {
                    delMessage(message);
                    line = null;
                    message = null;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                MainWindow.getMainWindow().setShowMsg(this.username + " login out !");
                MainWindow.getMainWindow().removeOfflineUsers(this.username);
                ChatManager.getChatManager().remove(socketMsg);
                bufferedWriter.close();
                bufferedReader.close();
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }
}
```

首先定义一个 BufferedReader 和 BufferedWriter ，作用是接受客户端的数据和向客户端发送数据。很好记，Reader 对于自己来说就是读取别处的数据，Writer 是向目标对象写数据。所以 BufferedReader 是接受客户端的数据，BufferedWriter 是向客户端发送数据。

那么如何获取呢？

在构造函数中可以看到这两句

```java
this.bufferedReader = new BufferedReader(new InputStreamReader(s.getInputStream(), "UTF-8"));
this.bufferedWriter = new BufferedWriter(new OutputStreamWriter(s.getOutputStream(), "UTF-8"));
```

可以一起写也可以分开。如下：

```java
// 从 socket 获取输入流
InputStream inputStream = s.getInputStream();
// 将位流通过 “UTF-8” 的格式读取为字符流
InputStreamReader inputStreamReader = new InputStreamReader(inputStream, "UTF-8");
// 将字符流放入 Buffer。方便使用
BufferedReader bufferedReader = new BufferedReader(inputStreamReader);
```

多想想整个数据的传输过程就很明朗了。有点像 OSI 的那个七层模型，一层一层封装与解封装。

上面只是获取到了流，并没有进行数据读写操作。开始下面介绍读取流中的数据。

```java
String line = null;
while ((line = bufferedReader.readLine()) != null) {
    if (!line.equals("-1")) {   
        message += line;
    } else {
        delMessage(message);
        line = null;
        message = null;
    }
}
```

看着有点乱。。再看一下下面的吧。

```java
public void sendMsg(String msg) {
    try {
        while (socket == null) ;
        if (bufferedWriter != null) {
            System.out.println("send :" + msg);
            bufferedWriter.write(msg + "\n");
            bufferedWriter.flush();
            bufferedWriter.write("-1\n");
            bufferedWriter.flush();
        }
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

可以清楚的看到，发送消息之后后面会加一个 “-1” 作为消息的终止符，也就是接接收方在接收到 -1 时，说明前面的数据是作为一整个信息的，将数据传出进行处理。接着清空继续接收接下来的消息，循环往复。可以看到 bufferedReader 这个流只要不关闭，这个会永远循环下去。

好了消息的发送和接收都基本算是处理好了，接下来就是数据处理，数据处理必然是要依赖协议的。所以依赖协议自己实现如下的函数。

```java
public void delMessage(String msg) {
    if (msg != null) {
        String action = getAction(msg);
        switch(action) {
            case "LOGIN": { dealLogin(msg); break; }
            case "REGISTER": { dealRegister(msg); break; }
            case "DRESSUP": { dealDressUp(msg); break; }
            case "GETDRESSUP": { dealGetDressUp(msg); break; }
            case "PROFILE": { dealProfile(msg); break; }
            case "GETPROFILE": { dealGetProfile(msg); break; }
            case "GETFRIENDLIST": { dealGetFriendList(msg); break; }
            case "GETGROUPLIST": { dealGetGroupList(msg); break; }
            case "GETFRIENDPROFILE": { dealGetFriendProfile(msg); break; }
            case "STATE": { dealState(msg); break; }
            case "CHATMSG": { dealChatMsg(msg); break; }
            case "USERLIST": { dealUserList(msg); break; }
            case "ADDFRIEND": { dealAddFriend(msg); break; }
            case "GROUPMEMBERLIST": { dealGroupMemberList(msg); break; }
            case "ADDGROUP": { dealAddGroup(msg); break; }
            case "GETALLGROUPLIST": { dealGetAllGroupList(msg); break;}
            default : dealError(); break;
        }
    }
}
```

服务端的线程是随连接数的增加而增加，所以创建一个线程管理的类（ChatManager）就有必要了，这样我们可以轻松的对消息进行跨进程转发（聊天）。

```java
public class ChatManager {
    private ChatManager(){};
    List<SocketMsg> socketList = new ArrayList<>();
    private static final ChatManager chatManager = new ChatManager();
    public static ChatManager getChatManager() {
        return chatManager;
    }   
    public void add(SocketMsg cs) {
        socketList.add(cs);
    }
    public void remove(SocketMsg cs) {
        socketList.remove(cs);
    }    
}

public class SocketMsg {
    private ChatSocket chatSocket;
    private String username;
    public SocketMsg(ChatSocket chatSocket, String username) {
        this.chatSocket = chatSocket;
        this.username = username;
    }
    public String getUsername() {
        return username;
    }
    public void setUsername(String username) {
        this.username = username;
    }
    public ChatSocket getChatSocket() {
        return chatSocket;
    }
    public void setChatSocket(ChatSocket chatSocket) {
        this.chatSocket = chatSocket;
    }
}
```

就是这么简单。


服务端基本就讲到这里。细心的人会发现我们的接收消息和发送消息是不在同一个线程中的，稍微思考思考对客户端的代码编写有好处。

#### 客户端

在不考虑信息处理的情况下客户端仿照服务端很容易写出接受发送流的。

```java
server/ServerManager.java

public class ServerManager extends Thread {
    private static final String IP = "192.168.191.1";
    private Socket socket;
    private String message = null;
    private BufferedReader bufferedReader;
    private BufferedWriter bufferedWriter;
    public static ServerManager getServerManager() { return serverManager; }
    private ServerManager() { }
    public void run() {
        try {
            socket = new Socket(IP, 27777);
            bufferedReader =  new BufferedReader(new InputStreamReader(socket.getInputStream(), "UTF-8"));
            bufferedWriter = new BufferedWriter(new OutputStreamWriter(socket.getOutputStream(), "UTF-8"));

            String m = null;
            String line;
            while ((line = bufferedReader.readLine()) != null) {
                if (!line.equals("-1")) {
                    m += line;
                } else {
                    Log.d("TAG", "receive : " + m);
                    if (ParaseData.getAction(m).equals("GETCHATMSG")) {
                        receiveChatMsg.delChatMsg(m);
                    } else {
                        message = m;
                    }
                    m = null;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            try {
                bufferedWriter.close();
                bufferedReader.close();
                socket.close();
            } catch (IOException e) {
                e.printStackTrace();
            }
        }
    }

    public void sendMessage(Context context, String msg) {
        try {
            while (socket == null) ;
            if (bufferedWriter != null) {
                bufferedWriter.write(msg + "\n");
                bufferedWriter.flush();
                bufferedWriter.write("-1\n");
                bufferedWriter.flush();
            }
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```

代码是不是很熟悉？基本和服务端没什么变化，因为毕竟这是要和服务端建立连接的。

可以看到这是一个线程，所以从哪里开启这个线程呢？
我是在这里：

```java
AtyLoginOrRegister.java

public class AtyLoginOrRegister extends AppCompatActivity implements View.OnClickListener {
    private ServerManager serverManager = ServerManager.getServerManager();
    private void initViews() {
        serverManager.start();
    }
}
```

下面就是进行数据传输处理了，我举两个例子。

#### 数据传输

- 登录

```java
public class AtyLoginOrRegister extends AppCompatActivity implements View.OnClickListener {
    private void initViews() {
        btnLogin.setOnClickListener(this);
        btnRegister.setOnClickListener(this);
        serverManager.start();
    }

    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.btn_login: {
                String username = etLoginUsername.getText().toString();
                String password = etLoginPassword.getText().toString();
                if (login(username, password)) {
                    serverManager.setUsername(username);
                    Intent intent = new Intent(this, AtyMain.class);
                    startActivity(intent);
                    finish();
                } else {
                    etLoginUsername.setText("");
                    etLoginPassword.setText("");
                }
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

    private boolean login(String username, String password) {
        // check username and password whether legal
        if (username == null || username.length() > 10 || password.length() > 20) {
            return false;
        }
        // send msg to servers
        String msg = "[LOGIN]:[" + username + ", " + password + "]";
        serverManager.sendMessage(this, msg);
        // get msg from servers return
        String ack = serverManager.getMessage();
        // deal msg
        if (ack == null) {
            return false;
        }
        serverManager.setMessage(null);
        String p = "\\[ACKLOGIN\\]:\\[(.*)\\]";
        Pattern pattern = Pattern.compile(p);
        Matcher matcher = pattern.matcher(ack);
        return matcher.find() && matcher.group(1).equals("1");
    }
}
```

既然是登录，那么事件是发生在按钮的点击之后。下面我们来理一下登录过程：

点击登录按钮->获取登录信息->封装信息传输->服务端接收->解析信息->从服务器获取信息进行登录验证->返回验证结果->客户端获取数据->解析信息->进行登录状态调整

这就是一整个流程。下面就具体介绍。

  - 点击登录按钮

  ```java
  case R.id.btn_login: {
        String username = etLoginUsername.getText().toString();
        String password = etLoginPassword.getText().toString();
  }
  ```

  - 获取登录信息

  ```java
  String username = etLoginUsername.getText().toString();
  String password = etLoginPassword.getText().toString();
  ```

  - 封装信息传输

```java
  if (username == null || username.length() > 10 || password.length() > 20) {
      return false;
  }
  String msg = "[LOGIN]:[" + username + ", " + password + "]";
  serverManager.sendMessage(this, msg);
```

  - 服务端接收

```java
  String line = null;
  while ((line = bufferedReader.readLine()) != null) {
      if (!line.equals("-1")) {   
          message += line;
      } else {
          delMessage(message);
          System.out.println("receive : " + message);
          line = null;
          message = null;
      }
  }
```

  - 解析信息

```java
  String iusername = null;
  String iPassword = null;
  String p = "\\[LOGIN\\]:\\[(.*), (.*)\\]";
  Pattern pattern = Pattern.compile(p);
  Matcher matcher = pattern.matcher(msg);
  if (matcher.find()) {
      iusername = matcher.group(1);
      iPassword = matcher.group(2);
  }
```

  - 从服务器获取信息进行登录验证 && 返回验证结果

```java
  String sql = "SELECT password FROM USERS WHERE username = '" + iusername + "';";
  try {
      Statement statement = connection.createStatement();
      ResultSet resultSet = statement.executeQuery(sql);
      if (resultSet.next() && iPassword.equals(resultSet.getString(1)) ) {
          sendMsg("[ACKLOGIN]:[1]");
          this.username = iusername;
          MainWindow.getMainWindow().setShowMsg(this.username + " login in!");
          MainWindow.getMainWindow().addOnlineUsers(this.username);
          socketMsg = new SocketMsg(this,  this.username);
          ChatManager.getChatManager().add(socketMsg);
          return ;
      }
  } catch (SQLException e) {
      e.printStackTrace();
  }
  sendMsg("[ACKLOGIN]:[0]");
```

  - 客户端获取数据

```java
  String ack = serverManager.getMessage();
```

  - 解析信息

```java
  if (ack == null) {
      return false;
  }
  serverManager.setMessage(null);
  String p = "\\[ACKLOGIN\\]:\\[(.*)\\]";
  Pattern pattern = Pattern.compile(p);
  Matcher matcher = pattern.matcher(ack);
  return matcher.find() && matcher.group(1).equals("1");
```
  - 进行登录状态调整

```java
  if (login(username, password)) {
      serverManager.setUsername(username);
      Intent intent = new Intent(this, AtyMain.class);
      startActivity(intent);
      finish();
  } else {
      etLoginUsername.setText("");
      etLoginPassword.setText("");
  }
```


- 发送信息

流程一样只是处理消息的时候复杂一点，因为我们只为聊天设置一个协议。所以要提高信息的区分度而对数据进行的繁复操作是必须的。


发送：

```java
btnSend.setOnClickListener(new View.OnClickListener() {
    @Override
    public void onClick(View v) {
        String content = myMsg.getText().toString();
        if (!content.isEmpty()) {
            ChatMsg msg = new ChatMsg();
            msg.setContent(content);
            msg.setUsername(ServerManager.getServerManager().getUsername());
            msg.setIconID(ServerManager.getServerManager().getIconID());
            msg.setMyInfo(true);
            msg.setChatObj(chatObj);
            msg.setGroup(group.equals("0") ? chatObj : " ");
            if (sendToChatObj(msg.getContent())) {
                ChatMsg.chatMsgList.add(msg);
                chatMsgList.add(msg);
                myMsg.setText("");
            } else {
                Toast.makeText(AtyChatRoom.this, "send failed", Toast.LENGTH_SHORT).show();
            }
        }
    }
});

private boolean sendToChatObj(String content) {
    String msg = "[CHATMSG]:[" + chatObj + ", " + content + ", " + ServerManager.getServerManager().getIconID() +", Text]";
    ServerManager serverManager = ServerManager.getServerManager();
    serverManager.sendMessage(this, msg);
    try {
        Thread.sleep(300);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    String ack = serverManager.getMessage();
    if (ack == null) {
        return false;
    }
    String p = "\\[ACKCHATMSG\\]:\\[(.*)\\]";
    Pattern pattern = Pattern.compile(p);
    Matcher matcher = pattern.matcher(ack);
    return matcher.find() && matcher.group(1).equals("1");
}
```

接收：

由于日后扩展可能不只是文本信息，所以在对聊天信息的处理上我们直接再建一个类。

```java
class ReceiveChatMsg {
    void delChatMsg(String msg) {
        String sendName = null;
        String content = null;
        String avatarID = null;
        String fileType = null;
        String group = null;
        ServerManager.getServerManager().setMessage(null);
        String p = "\\[GETCHATMSG\\]:\\[(.*), (.*), (.*), (.*), (.*)\\]";
        Pattern pattern = Pattern.compile(p);
        Matcher matcher = pattern.matcher(msg);
        if (matcher.find()) {
            sendName = matcher.group(1);
            content = matcher.group(2);
            avatarID = matcher.group(3);
            fileType = matcher.group(4);
            group = matcher.group(5);

            ChatMsg chatMsg = new ChatMsg();
            chatMsg.setMyInfo(false);
            chatMsg.setContent(content);
            chatMsg.setChatObj(sendName);
            chatMsg.setUsername(ServerManager.getServerManager().getUsername());
            chatMsg.setGroup(group);

            chatMsg.setIconID(Integer.parseInt(avatarID));

            AtyChatRoom.chatMsgList.add(chatMsg);
            ChatMsg.chatMsgList.add(chatMsg);
        }
    }
}
```

细节就不多说了，注意区分同一个对象发信息给你是通过群还是私人。

还有就是当聊天窗口关闭后在再次打开聊天信息怎么加载？大家自己想想。不同可以看代码或者留言。

下面看下服务器的数据转发代码

```java
private void dealChatMsg(String msg) {
    String chatObj = null;
    String content = null;
    String avatarID = null;
    String msgType = null;
    String p = "\\[CHATMSG\\]:\\[(.*), (.*), (.*), (.*)\\]";
    Pattern pattern = Pattern.compile(p);
    Matcher matcher = pattern.matcher(msg);
    if (matcher.find()) {
        chatObj = matcher.group(1);
        content = matcher.group(2);
        avatarID = matcher.group(3);
        msgType = matcher.group(4);
    } else {
        return;
    }
    String out = null;
    String sqlGroup = "SELECT * FROM GROUPS WHERE groupName = '" + chatObj+ "';";
    try {
        Statement statement = connection.createStatement();
        ResultSet resultSet = statement.executeQuery(sqlGroup);
        // gruop chat
        if (resultSet.next()) {
            // find all group members to send msg
            String sql = "SELECT groupMemberName FROM GROUPINFO WHERE groupName = '" + chatObj + "';";
            resultSet = statement.executeQuery(sql);
            while (resultSet.next()) {                  
                // if user is online , then send.
                for (SocketMsg SocketMsg : ChatManager.getChatManager().socketList) {
                    if (SocketMsg.getUsername().equals(resultSet.getString(1)) && !SocketMsg.getUsername().equals(username)) {
                        out = "[GETCHATMSG]:[" + username + ", " + content + ", " + avatarID + ", Text, " + chatObj + "]";
                        SocketMsg.getChatSocket().sendMsg(out);
                    }
                }
            }
        // private chat
        } else {
            for (SocketMsg socketManager : ChatManager.getChatManager().socketList) {
                if (socketManager.getUsername().equals(chatObj)) {
                    out = "[GETCHATMSG]:[" + username + ", " + content + ", " + avatarID + ", Text,  ]";
                    socketManager.getChatSocket().sendMsg(out);
                }
            }
        }
        out = "[ACKCHATMSG]:[1]";
        sendMsg(out);
    } catch (SQLException e) {
        out = "[ACKCHATMSG]:[0]";
        sendMsg(out);
        e.printStackTrace();
    }
}
```

![](chatmsglog.png)

其实逻辑不是很复杂，慢慢看很容易懂的。其他的实现自己慢慢摸索可以实现。不想再拖长篇幅了，看代码可能比我讲的效率更高，理解更加深刻。在这个过程中可能你会遇到一个小 Bug 卡了一天，但是每个人都是这么成长过来的，一起加油!


到这里目前的就算完了，这个系列 Blog 可能不会更新了。但是 GitHub []() 上会对代码继续更新的。此外这个 Lab 版还有很多 Bug 欢迎大家提建议，很希望和大家一起交流学习。

多谢阅读。