---
title: Handler 机制再了解
date: 2017-09-11 11:27
tags:
	- Android
---


> 这里主要是先了解整个消息传递的过程，知道这样做的好处和必要性。而不是直接介绍里面的几个关键类，然后介绍这个机制，这样容易头晕。而且网络上已经有很多这样的文章了，那些作者所站的高度对于我这种初学者来说有点高，我理解起来是比较稀里糊涂的，所以这里从一个问题出发，一步一步跟踪代码，这里只是搞清楚 handler 是怎么跨线程收发消息的，具体实现细节还是参考网上的那些大神的 Blog 比较权威。
> PS. 本来是想分章节书写，谁知道这一套军体拳打下来收不住了，所以下面基本是以一种很流畅的过程解释而不是很跳跃，细心看应该会对理解 Handler 机制有所收获。

<!-- more -->

Q1: 假如有一个耗时的数据处理，而且数据处理的结果是对 UI 更新影响的，而 Android 中 UI 更新不是线程安全的，所以规定只能在主线程中更新。

下面我们有两种选择：

```java
主线程版本：

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";
    private Button btnTest;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.layout_test);

        init();
    }

    private void init() {
        btnTest = (Button) findViewById(R.id.btn_test);
        btnTest.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {

                // 假装数据处理
                int i = 0;
                for (i = 0; i < 10; i++) {
                    try {
                        Thread.sleep(1000);
                    } catch (InterruptedException e) {
                        e.printStackTrace();
                    }
                }
                
                // 假装更新 UI
                Log.d(TAG, "Handle it！" + i);
            }
        });
    }
}
```

直接在主线程中处理数据，接着直接根据处理结果更新 UI。我想弊端大家都看到了，小则 UI 卡顿，大则造成 [ANR](http://fitzeng.org/2017/04/07/RecurrentANR/)。


```java
子线程版本：

public class MainActivity extends AppCompatActivity {

    private static final String TAG = "MainActivity";
    private Button btnTest;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.layout_test);

        init();
    }

    private void init() {
        btnTest = (Button) findViewById(R.id.btn_test);
        btnTest.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                new Thread(new Runnable() {
                    @Override
                    public void run() {
                        // 假装数据处理
                        int i;
                        for (i = 0; i < 10; i++) {
                            try {
                                Thread.sleep(1000);
                            } catch (InterruptedException e) {
                                e.printStackTrace();
                            }
                        }
                        // 返回处理结果
                        handler.sendEmptyMessage(i);
                    }
                }).start();
            }
        });
    }

    Handler handler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            // 假装更新 UI
            Log.d(TAG, "Handle MSG = " + msg.what);
        }
    };
}
```

这是一种典型的处理方式，开一个子线程处理数据，通过 Android 中提供的 Handler 机制进行跨线程通讯，把处理结果返回给主线程，进而更新 UI。这里我们就是探讨 Handler 是如何把数据发送过去的。

![](subThread.png)

到这里，我们了解到的就是一个 Handler 的黑盒机制，子线程发送，主线程接收。接下来，我们不介绍什么 `ThreadLocal`、`Looper` 和 `MessageQueue`。而是直接从上面的代码引出它们的存在，从原理了解它们存在的必要性，然后在谈它们内部存在的细节。


一切罪恶源于 `handler.sendEmptyMessage();`，最终找到以下函数 `sendMessageAtTime(Message msg, long uptimeMillis)`：

```java
Handler.class
/**
 * Enqueue a message into the message queue after all pending messages
 * before the absolute time (in milliseconds) <var>uptimeMillis</var>.
 * <b>The time-base is {@link android.os.SystemClock#uptimeMillis}.</b>
 * Time spent in deep sleep will add an additional delay to execution.
 * You will receive it in {@link #handleMessage}, in the thread attached
 * to this handler.
 * 
 * @param uptimeMillis The absolute time at which the message should be
 *         delivered, using the
 *         {@link android.os.SystemClock#uptimeMillis} time-base.
 *         
 * @return Returns true if the message was successfully placed in to the 
 *         message queue.  Returns false on failure, usually because the
 *         looper processing the message queue is exiting.  Note that a
 *         result of true does not mean the message will be processed -- if
 *         the looper is quit before the delivery time of the message
 *         occurs then the message will be dropped.
 */
public boolean sendMessageAtTime(Message msg, long uptimeMillis) {
    MessageQueue queue = mQueue;
    if (queue == null) {
        RuntimeException e = new RuntimeException(
                this + " sendMessageAtTime() called with no mQueue");
        Log.w("Looper", e.getMessage(), e);
        return false;
    }
    return enqueueMessage(queue, msg, uptimeMillis);
}
```

`MessageQueue` 出来了，我们避免不了了。里面主要是 `Message next()` 和 `enqueueMessage(Message msg, long when)` 方法值得研究，但是现在还不是时候。

从 `MessageQueue queue = mQueue;` 中可以看出我们的 `handler` 对象里面包含一个 mQueue 对象。至于里面存的什么怎么初始化的现在也不用太关心。大概有个概念就是这是个消息队列，存的是消息就行，具体实现细节后面会慢慢水落石出。
后面的代码就是说如果 queue 为空则打印 log 返回 false；否则执行 `enqueueMessage(queue, msg, uptimeMillis);` 入队。那就好理解了，handler 发送信息其实是直接把信息封装进一个消息队列。

```java
Handler.class
private boolean enqueueMessage(MessageQueue queue, Message msg, long uptimeMillis) {
    msg.target = this;
    if (mAsynchronous) {
        msg.setAsynchronous(true);
    }
    return queue.enqueueMessage(msg, uptimeMillis);
}
```

这里涉及 Message，先说下这个类的三个成员变量：

```java
/*package*/ Handler target;
    
/*package*/ Runnable callback;
    
/*package*/ Message next;
```

所以 `msg.target = this;` 把当前 handler 传给了 msg。

中间的 if 代码先忽略，先走主线：执行了 `MessageQueue` 的 `enqueueMessage(msg, uptimeMillis);`方法。接着看源码

```java
MessageQueue.class
boolean enqueueMessage(Message msg, long when) {
    if (msg.target == null) {
        throw new IllegalArgumentException("Message must have a target.");
    }
    if (msg.isInUse()) {
        throw new IllegalStateException(msg + " This message is already in use.");
    }

    synchronized (this) {
        if (mQuitting) {
            IllegalStateException e = new IllegalStateException(
                    msg.target + " sending message to a Handler on a dead thread");
            Log.w(TAG, e.getMessage(), e);
            msg.recycle();
            return false;
        }

        msg.markInUse();
        msg.when = when;
        Message p = mMessages;
        boolean needWake;
        if (p == null || when == 0 || when < p.when) {
            // New head, wake up the event queue if blocked.
            msg.next = p;
            mMessages = msg;
            needWake = mBlocked;
        } else {
            // Inserted within the middle of the queue.  Usually we don't have to wake
            // up the event queue unless there is a barrier at the head of the queue
            // and the message is the earliest asynchronous message in the queue.
            needWake = mBlocked && p.target == null && msg.isAsynchronous();
            Message prev;
            for (;;) {
                prev = p;
                p = p.next;
                if (p == null || when < p.when) {
                    break;
                }
                if (needWake && p.isAsynchronous()) {
                    needWake = false;
                }
            }
            msg.next = p; // invariant: p == prev.next
            prev.next = msg;
        }

        // We can assume mPtr != 0 because mQuitting is false.
        if (needWake) {
            nativeWake(mPtr);
        }
    }
    return true;
}
```
代码有点长，不影响主线的小细节就不介绍了，那些也很容易看懂的，但是原理还是值得分析。
`if (mQuitting)...`，直接看看源码初始化赋值的函数是在 `void quit(boolean safe)` 函数里面，这里猜测可能是退出消息轮训，消息轮训的退出方式也是值得深究，不过这里不影响主线就不看了。 `msg.markInUse(); msg.when = when;` 标记消息在用而且继续填充  msg，下面就是看注释了。我们前面介绍的 Message 成员变量 next 就起作用了，把 msg 链在一起了。所以这里的核心就是把 msg 以一种链表形式插进去。似乎这一波分析结束了，在这里划张图总结下：
![](sendMsg.png)
推荐自己根据所观察到的变量赋值进行绘制图画，这样印象更加深刻。

OK，消息是存进去了，而且也是在 handler 所在的线程中。那么到底怎么取出信息呢？也就是前面小例子

```java
Handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        // 假装更新 UI
        Log.d(TAG, "Handle MSG = " + msg.what);
    }
};
```
`handleMessage()` 什么时候调用？这里基本断了线索。但是如果你之前哪怕看过类似的一篇文章应该都知道其实在 Android 启动时 main 函数就做了一些操作。这些操作是必要的，这也就是为什么我们不能直接在子线程中 `new Handler();`。

```java
public static void main(String[] args) {
	Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");
	SamplingProfilerIntegration.start();
	
	// CloseGuard defaults to true and can be quite spammy.  We
	// disable it here, but selectively enable it later (via
	// StrictMode) on debug builds, but using DropBox, not logs.
	CloseGuard.setEnabled(false);
	
	Environment.initForCurrentUser();
	
	// Set the reporter for event logging in libcore
	EventLogger.setReporter(new EventLoggingReporter());
	
	// Make sure TrustedCertificateStore looks in the right place for CA certificates
	final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
	TrustedCertificateStore.setDefaultUserDirectory(configDir);
	
	Process.setArgV0("<pre-initialized>");
	
	Looper.prepareMainLooper(); // -------1
	
	ActivityThread thread = new ActivityThread();
	thread.attach(false);
	
	if (sMainThreadHandler == null) {
	    sMainThreadHandler = thread.getHandler(); // -------2
	}
	
	if (false) {
	    Looper.myLooper().setMessageLogging(new
	            LogPrinter(Log.DEBUG, "ActivityThread"));
	}
	
	// End of event ActivityThreadMain.
	Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
	Looper.loop(); // -------3
	
	throw new RuntimeException("Main thread loop unexpectedly exited");
}
```
可以看出这里在获取 sMainThreadHandler 之前进行了 `Looper.prepareMainLooper();` 操作，之后进行了 `Looper.loop();` 操作。

下面开始分析：

```java
Loopr.class
 /** Initialize the current thread as a looper.
  * This gives you a chance to create handlers that then reference
  * this looper, before actually starting the loop. Be sure to call
  * {@link #loop()} after calling this method, and end it by calling
  * {@link #quit()}.
  */
public static void prepare() {
    prepare(true);
}

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}

/**
 * Initialize the current thread as a looper, marking it as an
 * application's main looper. The main looper for your application
 * is created by the Android environment, so you should never need
 * to call this function yourself.  See also: {@link #prepare()}
 */
public static void prepareMainLooper() {
    prepare(false);
    synchronized (Looper.class) {
        if (sMainLooper != null) {
            throw new IllegalStateException("The main Looper has already been prepared.");
        }
        sMainLooper = myLooper();
    }
}

/**
 * Return the Looper object associated with the current thread.  Returns
 * null if the calling thread is not associated with a Looper.
 */
public static @Nullable Looper myLooper() {
    return sThreadLocal.get();
}
```
前两个方法是在自己创建 Looper 的时候用，第三个是主线程自己用的。由于这里消息传递以主线程为线索。`prepare(false);`说明了这是主线程，在 `sThreadLocal.set(new Looper(quitAllowed));` 中的 `quitAllowed` 为 false 则说明主线程的 MessageQueue 轮训不能 quit。这句代码里还有 ThreadLocal 的 set() 方法。先不深究实现，容易晕，这里需要知道的就是把一个 Looper 对象“放进”了 ThreadLocal，换句话说，通过 ThreadLocal 可以获取不同的 Looper。
最后的 `sThreadLocal.get();` 展示了 get 方法。说明到这时 Looper 已经存在啦。
现在看看 Looper 类的成员变量吧！

```java
Looper.class
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();
private static Looper sMainLooper;  // guarded by Looper.class

final MessageQueue mQueue;
final Thread mThread;
```

在这里先介绍一下 ThreadLocal 的上帝视角吧。直接源码，可以猜测这是通过一个 `ThreadLocalMap` 的内部类对线程进行一种 map。传进来的泛型 T 正是我们的 looper。所以 ThreadLocal 可以根据当前线程查找该线程的 Looper，具体怎么查找推荐看源码，这里就不介绍了。
```java
/**
 * Returns the value in the current thread's copy of this
 * thread-local variable.  If the variable has no value for the
 * current thread, it is first initialized to the value returned
 * by an invocation of the {@link #initialValue} method.
 *
 * @return the current thread's value of this thread-local
 */
public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null)
            return (T)e.value;
    }
    return setInitialValue();
}

 * Sets the current thread's copy of this thread-local variable
 * to the specified value.  Most subclasses will have no need to
 * override this method, relying solely on the {@link #initialValue}
 * method to set the values of thread-locals.
 *
 * @param value the value to be stored in the current thread's copy of
 *        this thread-local.
 */
public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}
```

分析到这里，handler 和 looper 都有了，但是消息还是没有取出来？
这是看第三句 `Looper.loop();`。

```java
Looper.class
/**
 * Run the message queue in this thread. Be sure to call
 * {@link #quit()} to end the loop.
 */
public static void loop() {
    final Looper me = myLooper();
    if (me == null) {
        throw new RuntimeException("No Looper; Looper.prepare() wasn't called on this thread.");
    }
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }

        // This must be in a local variable, in case a UI event sets the logger
        final Printer logging = me.mLogging;
        if (logging != null) {
            logging.println(">>>>> Dispatching to " + msg.target + " " +
                    msg.callback + ": " + msg.what);
        }

        final long traceTag = me.mTraceTag;
        if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
            Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
        }
        try {
            msg.target.dispatchMessage(msg);
        } finally {
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
        }

        if (logging != null) {
            logging.println("<<<<< Finished to " + msg.target + " " + msg.callback);
        }

        // Make sure that during the course of dispatching the
        // identity of the thread wasn't corrupted.
        final long newIdent = Binder.clearCallingIdentity();
        if (ident != newIdent) {
            Log.wtf(TAG, "Thread identity changed from 0x"
                    + Long.toHexString(ident) + " to 0x"
                    + Long.toHexString(newIdent) + " while dispatching to "
                    + msg.target.getClass().getName() + " "
                    + msg.callback + " what=" + msg.what);
        }

        msg.recycleUnchecked();
    }
}
```

一开始也是获取 Looper，但是那么多 Looper 怎么知道这是哪个 Looper 呢？这先放着待会马上解释。把 loop() 函数主要功能搞懂再说。
接下来就是获取 Looper 中的 MessageQueue了，等等，这里提出一个疑问，前面说了 Handler 中也存在 MessageQueue，那这之间存在什么关系吗？（最后你会发现其实是同一个）
先往下看，一个死循环，也就是轮训消息喽，中间有一句 `msg.target.dispatchMessage(msg);` 而前面介绍 msg.target 是 handler 型参数。所以和 handler 联系上了。

```java
Handler.class
/**
 * Handle system messages here.
 */
public void dispatchMessage(Message msg) {
    if (msg.callback != null) {
        handleCallback(msg);
    } else {
        if (mCallback != null) {
            if (mCallback.handleMessage(msg)) {
                return;
            }
        }
        handleMessage(msg);
    }
}
```
逻辑很简单，总之就是调动了我们重写的 handleMessage() 方法。

Step 1：`Looper.prepare();`
> 在 Looper 中有一个静态变量 sThreadLocal，把创建的 looper “存在” 里面，创建 looper 的同时创建 MessageQueue，并且和当前线程挂钩。
 
Step 2：`new Handler();` 
> 通过上帝 ThreadLocal，并根据当前线程，可获取 looper，进而获取 MessageQueue，Callback之类的。 
```java
Handler.class
/**
 * Use the {@link Looper} for the current thread with the specified callback interface
 * and set whether the handler should be asynchronous.
 *
 * Handlers are synchronous by default unless this constructor is used to make
 * one that is strictly asynchronous.
 *
 * Asynchronous messages represent interrupts or events that do not require global ordering
 * with respect to synchronous messages.  Asynchronous messages are not subject to
 * the synchronization barriers introduced by {@link MessageQueue#enqueueSyncBarrier(long)}.
 *
 * @param callback The callback interface in which to handle messages, or null.
 * @param async If true, the handler calls {@link Message#setAsynchronous(boolean)} for
 * each {@link Message} that is sent to it or {@link Runnable} that is posted to it.
 *
 * @hide
 */
public Handler(Callback callback, boolean async) {
    if (FIND_POTENTIAL_LEAKS) {
        final Class<? extends Handler> klass = getClass();
        if ((klass.isAnonymousClass() || klass.isMemberClass() || klass.isLocalClass()) &&
                (klass.getModifiers() & Modifier.STATIC) == 0) {
            Log.w(TAG, "The following Handler class should be static or leaks might occur: " +
                klass.getCanonicalName());
        }
    }

    mLooper = Looper.myLooper();
    if (mLooper == null) {
        throw new RuntimeException(
            "Can't create handler inside thread that has not called Looper.prepare()");
    }
    mQueue = mLooper.mQueue; // 前面的两个 MessageQueue 联系起来了，疑问已解答。
    mCallback = callback;
    mAsynchronous = async;
}
```
这个函数可以说明在 new Handler() 之前该线程必需有 looper，所以要在这之前调用 `Looper.prepare();`。

Step 3：`Looper.loop();`
> 进行消息循环。

基本到这里整个过程应该是清楚了，这里我画下我的理解。
![](threadLocal.png)

那么我们现在来看一下 handler 是怎么准确发送信息和处理信息的。注意在 handler 发送信息之前，1、2、3 步已经完成。所以该获取的线程已经获取，直接往该线程所在的 MessageQueue 里面塞信息就行了，反正该信息会在该 handler 所在线程的 looper 中循环，最终会通过消息的 target 参数调用 dispatchMessage()，而在 dispatchMessage() 中会调用我们重写的 handleMessage() 函数。

