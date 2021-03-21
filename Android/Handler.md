# Handler

## 一个小型合作的流水线

当我们遇到多线程的问题，考虑到线程间消息传递的时候，首先想到的肯定是 `Handler`。虽然写这篇文章的初衷并不是想探究 Handler 的机制，但我们还是先从这个被说烂了的话题开始。

### Handler 的工作原理

首先，在了解 `Handler` 之前，我们需要了解有四个关键的类是组成 `Handler`  的基础。它们分别是

- Handler 负责协调安排未来某个时间点的消息或可运行状态，以及对不同线程的运行机制进行合理的排队
- Looper 主要作用如其名，一个循环的机制，为线程运行消息循环分发
- MessageQueue 一个链式队列数据结构，将消息实体串联成链
- Message 消息实体，存储我们需要传递的消息的内容和信息等

#### Looper 和 MessageQueue——Handler 的流水线

在 `ActivityThread` 类中，作为入口方法的 `main()` 方法中，通过调用 `Looper` 的 `loop()` 方法，启动 `Looper` 的循环机制（这里我们注意到，在方法的最后，抛出了一个主线程循环意外退出的异常，说明 Android 的主流程都是通过 Handler 来驱动的）。

```java
/**
 * ActivityThread
 */
public static void main(String[] args) {
    
    // ...
    Looper.prepareMainLooper();
	// ...
    Looper.loop();

    throw new RuntimeException("Main thread loop unexpectedly exited");
}
```

进入 `loop()` 方法，这里我们可以看到一个死循环，传说中的死循环这么快就跟我们见面了吗？其实不然，我们平时面试时更关注的死循环并不是这个，或者说它只是其中的一部分。废话先不说，这段代码精简后的大致作用可以归纳为：从 `MessageQueue` 的对象队列里取出一个未处理的消息，即 `Message` 实例，然后获取 `Message` 对象的 `target` 属性，它是一个 `Handler` 对象，然后通过 `dispatchMessage()` 方法来将消息进行分发。

```java
/**
 * Looper
 */
public static void loop() {
    final MessageQueue queue = me.mQueue;

    // Make sure the identity of this thread is that of the local process,
    // and keep track of what that identity token actually is.
    // 我最开始读到这段源码的时候，很困惑这个方法为什么调用了两遍，后来经过思索想明白了原因，这里稍作记录。
    // 这个方法调用的是 native 的代码，源码如下：
    // int64_t IPCThreadState::clearCallingIdentity()
	// {
    //     int64_t token = ((int64_t)mCallingUid<<32) | mCallingPid;
    //     clearCaller();
    //     return token;
	// }
	// void IPCThreadState::clearCaller()
	// {
	//     mCallingPid = getpid(); //当前进程pid赋值给mCallingPid
	//     mCallingUid = getuid(); //当前进程uid赋值给mCallingUid
	// }
    // 具体作用可以网上自行搜索，这个方法的作用，简而言之，就是将（可能是）其他进程的 pid 和 uid 清除，更换为自己的，
    // 而 token 是用来存储原来进程的 pid 和 uid 的64位整型，所以第一遍调用时返回的是之前进程的 pid 和 uid 信息，
    // 再次调用时，返回的才是当前进程的，而被我精简掉的源码里需要通过这个 token 来判断进程是否切换过，所以这个方法在这里会调用两遍
    Binder.clearCallingIdentity();
    final long ident = Binder.clearCallingIdentity();

    for (;;) {
        Message msg = queue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return;
        }
        try {
            msg.target.dispatchMessage(msg);
            
        } catch (Exception exception) {
            throw exception;
        }

        msg.recycleUnchecked();
    }
}
```

因为 `dispatchMessage()` 方法比较简单，所以我们先越过过程看结果，看看这个方法的实现。

```java
/**
 * Handler
 */
public void dispatchMessage(@NonNull Message msg) {
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

这里就直接调用了 `Handler` 对象的 `handleMessage()` 方法，并传递 `Message` 的实例，所以我们在使用 `Handler` 时在这个方法中就可以接收到我们需要的消息实体（callback 默认不实现，实现后变更为调用相应的方法）。

好，结果我们已经知道了，那现在我们回过头来，研究一下上面 `Looper` 类的 `loop()` 方法中调用的 `queue.next()` 方法是如何拿到消息实体的（后面的注释已经提醒我们这个方法可能会阻塞）。

```java
/**
 * MessageQueue
 */
Message next() {
    // Return here if the message loop has already quit and been disposed.
    // This can happen if the application tries to restart a looper after quit
    // which is not supported.
    final long ptr = mPtr;
    if (ptr == 0) {
        return null;
    }

    int pendingIdleHandlerCount = -1; // -1 only during first iteration
    // 这个变量作为 nativePollOnce 方法的参数表示休眠的时间
    // 当值为 -1 时，表示无限休眠，直到有线程唤醒
    // 当值为 0 时，表示立即唤醒
    int nextPollTimeoutMillis = 0;
    for (;;) {
        if (nextPollTimeoutMillis != 0) {
            Binder.flushPendingCommands();
        }
		
        // 根据 nextPollTimeoutMillis 变量的值进行休眠
        nativePollOnce(ptr, nextPollTimeoutMillis);

        synchronized (this) {
            // Try to retrieve the next message.  Return if found.
            final long now = SystemClock.uptimeMillis();
            Message prevMsg = null;
            Message msg = mMessages;
            // 如果 Message 的 target 为 null，则说明它是 Looper synchronization barrier 的临界点
            if (msg != null && msg.target == null) {
                // Stalled by a barrier.  Find the next asynchronous message in the queue.
                do {
                    prevMsg = msg;
                    msg = msg.next;and the message is the earliest asynchronous message in the queue
                } while (msg != null && !msg.isAsynchronous());
            }
            // 经过上面的循环后，到达这里的 Message 要么是 null，要么是 isAsynchronous() 方法返回 true
            if (msg != null) {
                if (now < msg.when) {
                    // Next message is not ready.  Set a timeout to wake up when it is ready.
                    // 消息的发送时间未到，此时的 nextPollTimeoutMillis 为距离 msg 的发送时间的时间间隔，
                    // 那 nativePollOnce() 方法休眠相应的时间后，msg 即到了它该发送的时间
                    nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
                } else {
                    // Got a message.
                    mBlocked = false;
                    if (prevMsg != null) {
                        prevMsg.next = msg.next;
                    } else {
                        mMessages = msg.next;
                    }
                    msg.next = null;
                    if (DEBUG) Log.v(TAG, "Returning message: " + msg);
                    msg.markInUse();
                    return msg;
                }
            } else {
                // No more messages.
                // 没有更多的消息，此时 nextPollTimeoutMillis 赋值为 -1，
                // 那 nativePollOnce() 方法将导致线程永久休眠，直到有其他线程将其唤醒
                nextPollTimeoutMillis = -1;
            }

            // Process the quit message now that all pending messages have been handled.
            if (mQuitting) {
                dispose();
                return null;
            }
        }
    }
}
```

`next()` 方法看起来很长，但是它的主要工作只有一件事，就是找到符合要求的 Message 实例并返回。但是这个方法又特别重要，有一个常问的重要的面试考点。我们上面已经提到了，`Looper` 的 `loop()` 方法中有一个死循环，作用是源源不断地从 `MessageQueue` 中「打捞」 `Message` 实体，而「打捞」的动作正是通过 `next()` 方法完成的。在 `next()` 方法中，也有一个死循环，完成上面的「打捞」工作。具体的细节我在代码中作了部分注释，可以帮助理解。其中提到了一个概念——「Looper synchronization barrier」，关于它的介绍我们放在[下面的内容](#barrier)里。

好了，介绍完了 Handler 机制中的死循环，它是死循环双重嵌套的形式，那么面试问题来了：请问 Handler 机制中的死循环是如何做到不阻塞主线程的呢？网上搜索到的答案通常是死循环也未必会阻塞主线程，只要不在 `onCreate()` 或 `onStart()` 等生命周期中阻塞就不会导致界面的卡死，其次在 `MessageQueue` 中没有 `Message` 实体时，线程会进入到一个休眠的状态，在有新消息来临时线程才会被唤醒，balabala小魔仙……我们看到 `next()` 方法的死循环的一开始有一句代码 `nativePollOnce()`，它是一个 native 的方法，通过执行 Linux 中的 epoll 机制来是线程休眠和运行，它和 `nativeWake()` 方法配对使用，在类文件的开头均有声明。所以每次在执行完一遍 `next()` 方法后，都会根据 `nextPollTimeoutMillis` 变量的值来决定休眠的时间。如果没有可被「打捞」的消息，那么线程将被永久休眠，等待被唤醒。那么在哪里唤醒的呢，我们暂时不管，在这里先记住线程休眠，主线程被阻塞，等待一个白马王子将其唤醒。至于白马王子何时到来，我们静待。

#### Handler——消息操作台

现在，我们再从消息发送的源头追溯——通过 `Handler` 的一系列 `sendMessage()` 方法，将消息发送出去。

我们以 `sendEmptyMessage()` 方法为例，经过一系列的调用后，最终会执行 `enqueueMessage()` 方法，该方法又会调用 `MessageQueue` 的 `enqueueMessage()` 方法，该方法代码如下：

```java
/**
 * MessageQueue
 */
boolean enqueueMessage(Message msg, long when) {

    synchronized (this) {

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

好耶，「白马王子」来了！看到了吧，`nativeWake()` 方法显真身了，当有新的消息压入队列，消息需要被处理，此时就需要唤醒睡眠的线程。但是「白马王子」

的到来是需要条件的，即 `needWake`，那到底是怎样的条件呢？想想无非是判断当前的线程是否处于可能阻塞的状态，我们来看看。

在第一个条件 `p == null || when == 0 || when < p.when` 下，相比于罗列所有的满足条件的情况，更简单的方法是判断我们前面的线程被阻塞的情况是不是在这里被判定为 `needWake`， 因为在等待新的消息，所以 `mMessage` 值为 `null`，此时的 `needWake = mBlocked`，而 `mBlocked` 的线程被阻塞的情况下值是为 `true` 的，所以这里会被判定为需要被唤醒。而在 `else` 分支中，其实条件为`p != null && when != 0 && when >= p.when`，这说明消息队列中的消息并没有被取完，而是正在一个循环中，通常情况下是不需要再唤醒它，除非像注释中所说的 `there is a barrier at the head of the queue and the message is the earliest asynchronous message in the queue`。

到这里，Handler 的大概工作流程就可以串联起来了——循环队列相当于物流，消息相当于商品，物流无时无刻在运转，当你需要新的商品时，商品被商家发送至物流，然后分发到目标客户即你的手中。

### <span id="barrier">Looper synchronization barrier</span>

在看源码的时候，不止一次会接触到这个概念，而且在上面我们也已经率先使用了这个概念，那么这个概念到底是个什么？搞清楚这个问题，我们需要从它的特征入手，在 `MessageQueue` 的 `next()` 方法中，我们说如果 `mMessages.target == null`，那么它就是一个 barrier 的临界点，我们通过查找 `mMessage` 的写引用，最终定位到 `MessageQueue#postSyncBarrier()` 这个方法。我这里摘录它的注释，相信大家对这个概念就会有一个清晰的认识。

> Posts a synchronization barrier to the Looper's message queue.
>
> Message processing occurs as usual until the message queue encounters the synchronization barrier that has been posted.  When the barrier is encountered, later synchronous messages in the queue are stalled (prevented from being executed) until the barrier is released by calling {@link #removeSyncBarrier} and specifying the token that identifies the synchronization barrier.
>
> This method is used to immediately postpone execution of all subsequently posted synchronous messages until a condition is met that releases the barrier. Asynchronous messages (see {@link Message#isAsynchronous} are exempt from the barrier and continue to be processed as usual.
>
> This call must be always matched by a call to {@link #removeSyncBarrier} with the same token to ensure that the message queue resumes normal operation. Otherwise the application will probably hang!

在了解这个概念之前还需要知道一个属性的存在，那就是 `Message#isAsynchronous（）` 。

好了，总结一下就是 `Looper synchronization barrier` 是 `MessageQueue` 中那些 `target == null` 的 `Message`，它们不需要被发送，只作为一种队列状态的判断标识。当 `Message.isAsynchronous() == true` 时，遇到 `Looper synchronization barrier` 时，`Looper` 会被阻塞，直到 `removeSyncBarrier()` 方法（和 `postSyncBarrier()` 方法成对使用）移除这个标识。但是如果 `Message.isAsynchronous() == false` 时，则不会被 barrier 阻断，具体使用场景见上方注释。

太多的代码和解说赶不上一张图片更能让人形成概念，那我从网上找了一张图片稍作加工，希望可以比较形象地说明 `Handler` 机制中各个类之间的分工。

<video src="Handler.assets/Handler.MOV"></video>

### 映射关系

为了话题的自然过渡，这里我们思考一个问题，一个线程可以有多个 `Looper` 吗？一个 `Looper` 可以对应多个 `MessageQueue` 吗？从源码中看，一个线程是无法创建多个 `Looper` 和多个 `MessageQueue` 的，那么多个 `Looper` 和 `MessageQueue` 会导致什么问题呢？最主要的就是我们上面说的消息同步性的问题了，多个消息队列和循环体如何保证消息的次序限制以及同步分发就是一个很复杂的问题。那么系统又是如何保证每个线程的 `Looper` 的唯一性的呢？那就是使用 `ThreadLocal` 了。

### ThreadLocal

由于本篇内容旨在讨论 `Handler` 的相关机制，所以对于 `ThreadLocal` 的机制不做过多讨论。

`Looper#prepare()` 方法在 `Looper` 使用前必须调用，在这个方法里可以看到 `ThreadLocal` 的应用。

```java
/**
 * Looper
 */
static final ThreadLocal<Looper> sThreadLocal = new ThreadLocal<Looper>();

private static void prepare(boolean quitAllowed) {
    if (sThreadLocal.get() != null) {
        throw new RuntimeException("Only one Looper may be created per thread");
    }
    sThreadLocal.set(new Looper(quitAllowed));
}
```

`sThreadLocal` 对象是一个全局的静态对象，通过使用 `sThreadLocal#set()` 方法来存储 `Looper` 的实例，而 `ThreadLocal` 把真正的对象存储交给了它的静态内部类 `ThreadLocalMap`，这是一个自定义的 hash map，具体内部实现请自行阅读源码。

```java
/**
 * ThreadLocal
 */

public void set(T value) {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null)
        map.set(this, value);
    else
        createMap(t, value);
}

ThreadLocalMap getMap(Thread t) {
    return t.threadLocals;
}

public T get() {
    Thread t = Thread.currentThread();
    ThreadLocalMap map = getMap(t);
    if (map != null) {
        ThreadLocalMap.Entry e = map.getEntry(this);
        if (e != null) {
            @SuppressWarnings("unchecked")
            T result = (T)e.value;
            return result;
        }
    }
    return setInitialValue();
}
```

可以看到，`ThreadLocalMap` 又和 `Thread` 绑定，每个 `Thread` 对应一个唯一的 `ThreadLocalMap` 实例， `ThreadLocalMap` 的 `key` 的类型是 `ThreadLocal`，而在 `Looper` 中的 `sThreadLocal` 作为静态对象，进程内唯一，通过这样的关系，可以唯一对应到 `TreadLocalMap` 中的某个元素，实现读取。

