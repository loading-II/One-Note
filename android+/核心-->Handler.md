核心：Handler

[toc]

###### 1、handler、MessageQueue、Looper、Message三者的关系

```
Message：消息的载体，存储发送的内容，里面有个关键的参数target;
handler：发送和处理Message；
MessageQueue：用于存储Message，内部实现依靠单链表的数据结构进行存储，便于插入和删除进行重排序操作；
Looper：消息蹦，不断从MessageQueue内部提取消息体，然后进行消息分发给对应的target(handler)进行处理;
```

###### 2、Looper和ThreadLocalde关系，为什么如此设计

###### 3、MessageQueue： enqueue 和 next 的代码详解

```
enqueue: 消息入队操作，将发送的Message根据时间插入到单链表中，这里面有通过加锁确保线程安全
next: 消息提取，内部实现无限的For循环，不断的从MessageQueue单链表中提取消息，当消息还没到执行时间时，会通过native函数实现线程阻塞，释放CPU时间，通过设置超时时间，或者有消息加入才会唤醒循环继续执行，添加一些必要的代码：
发送异步消息--同步屏障：
private int postSyncBarrier(long when) {
        // Enqueue a new sync barrier token.
        //登记一个新的同步屏障令牌。
        // We don't need to wake the queue because the purpose of a barrier is to stall it.
        // 我们不需要唤醒队列，因为屏障的目的是阻止它。
        synchronized (this) {
            final int token = mNextBarrierToken++;
            //1、屏障消息和普通消息的区别是屏障消息没有tartget。
            final Message msg = Message.obtain();
            msg.markInUse();
            msg.when = when;
            msg.arg1 = token;
            //注意看这里未进行target配置，意味着 target == null

            Message prev = null;
            Message p = mMessages;
            if (when != 0) {
                while (p != null && p.when <= when) {
                    prev = p;
                    p = p.next;
                }
            }
            if (prev != null) { // invariant: p == prev.next
                msg.next = p;
                prev.next = msg;
            } else {
                msg.next = p;
                mMessages = msg;
            }
            return token;//token 意味着令牌，可用去取消同步屏障
        }
    }
    
    
    Message next() {//消息执行顺序是：同步屏障(异步消息)、同步消息、闲时机制
        final long ptr = mPtr;
        if (ptr == 0) {
            return null;
        }

        int pendingIdleHandlerCount = -1; // -1 only during first iteration
        int nextPollTimeoutMillis = 0;
        for (;;) {
            if (nextPollTimeoutMillis != 0) {
                Binder.flushPendingCommands();
            }
            //1、如果有消息被插入到消息队列或者超时时间到，就被唤醒，否则阻塞在这。
            nativePollOnce(ptr, nextPollTimeoutMillis);

            synchronized (this) {
                // Try to retrieve the next message.  Return if found.
                final long now = SystemClock.uptimeMillis();
                Message prevMsg = null;
                Message msg = mMessages;
                if (msg != null && msg.target == null) {
                    // target == null 遇到了同步屏障，开始查找最近的一条异步消息
                    // Stalled by a barrier.  Find the next asynchronous message in the queue.
                    do {
                        prevMsg = msg;
                        msg = msg.next;
                    } while (msg != null && !msg.isAsynchronous());
                }
                if (msg != null) {
                    if (now < msg.when) {
                        // Next message is not ready.  Set a timeout to wake up when it is ready.
                        // 消息为准备好，设置超时时间去唤醒阻塞
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
                    //没有任何消息，一直休眠，等待被唤醒
                    nextPollTimeoutMillis = -1;
                }

                // Process the quit message now that all pending messages have been handled.
                if (mQuitting) {
                    dispose();
                    return null;
                }

                // If first time idle, then get the number of idlers to run.
                // Idle handles only run if the queue is empty or if the first message
                // in the queue (possibly a barrier) is due to be handled in the future.
                if (pendingIdleHandlerCount < 0
                        && (mMessages == null || now < mMessages.when)) {
                    pendingIdleHandlerCount = mIdleHandlers.size();
                }
                if (pendingIdleHandlerCount <= 0) {
                    // No idle handlers to run.  Loop and wait some more.
                    mBlocked = true;
                    continue;
                }

                if (mPendingIdleHandlers == null) {
                    mPendingIdleHandlers = new IdleHandler[Math.max(pendingIdleHandlerCount, 4)];
                }
                mPendingIdleHandlers = mIdleHandlers.toArray(mPendingIdleHandlers);
            }

            // Run the idle handlers.
            // We only ever reach this code block during the first iteration.
            for (int i = 0; i < pendingIdleHandlerCount; i++) {
                final IdleHandler idler = mPendingIdleHandlers[i];
                mPendingIdleHandlers[i] = null; // release the reference to the handler

                boolean keep = false;
                try {
                    keep = idler.queueIdle();
                } catch (Throwable t) {
                    Log.wtf(TAG, "IdleHandler threw exception", t);
                }

                if (!keep) {
                    synchronized (this) {
                        mIdleHandlers.remove(idler);
                    }
                }
            }

            // Reset the idle handler count to 0 so we do not run them again.
            pendingIdleHandlerCount = 0;

            // While calling an idle handler, a new message could have been delivered
            // so go back and look again for a pending message without waiting.
            nextPollTimeoutMillis = 0;
        }
    }
```

###### 4、Message发送的几种方式和区别

```
实际上氛围post 和 send 系列api:
handler.post(runnbale)
handler.postAtTime(runnbale)
handler.postDelayed(runnbale)
handler.send(message)
handler.sendAtTime(message)
handler.sendDelayed(message)
post系列api发送的是runnable,而send系列发送的是message,不同点是，runnable最后成为了 Message.callback = runnable;
无论post还是send最后都通过MessageQueue.enqueue插入到链表中，
```

###### 5、同步消息、异步消息、同步屏障、闲时机制

```
同步消息：通过默认构造器实现的handler实例，默认mAsynchronous为false，也就是同步消息，同步发送的消息，在加入到链表中是通过when这个时间进行排序的，也就是按照时间进行插入到链表中；客户端最常用也就是这种消息；

异步消息： 和同步消息中的mAsynchronous为true，即为异步消息；异步消息的target = null；在同步消息中 target是用于查找处理消息的handler，而异步消息不需要target，直接执行

同步屏障：设置一个屏障 -> 为了挡住同步消息，保障异步消息优先处理的机制。多用于系统内部调用比如requestLayout，APP无法使用，会有红色报错；

闲时机制：当没有任何消息的时候，就会执行闲时任务，可以应用到冷启动加速中，对于不重要的任务可以放到闲时中去处理

以上代码，都需要在MessageQueue.next中进行解读
```

###### 6、分发机制

```
直接看handler的代码：
 public void dispatchMessage(@NonNull Message msg) {
        if (msg.callback != null) {
            //优先handler.post发送的runable回调
            handleCallback(msg);
        } else {
            if (mCallback != null) {
                //构造器中定义的callback执行
                if (mCallback.handleMessage(msg)) {
                    return;
                }
            }
            //最后才是根据返回给handler的handlerMessage处理
            handleMessage(msg);
        }
    }
```

###### 7、如何在子线程内实现Handler机制

```
主线程在ActivityThread已经实现了Handler机制，另外HandlerThread 在异步线程中也实现handler机制，我们直接看handlerThread即使，在HandlerThread.run中，看到：
//第一步prepare，创建MessageQueue，并将loop保存在ThreadLocal内
Looper.prepare();
//处理消息体内容
onLooperPrepared()
//第二步loop，启动消息循环提取消息处理
Looper.loop();
//创建handler，绑定looper
mHandler = new Handler(getLooper());
```

###### 8、MessageQueue中 next 内部实现的是一个for无限循环，为什么不会导致现成堵塞

```
待续
```



###### 9、线程阻塞是什么，如何唤起线程，MessageQueue.next为什么不会导致主线程堵塞

