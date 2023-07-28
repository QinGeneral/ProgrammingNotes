# Android Handler 机制

[TOC]

Android UI 操作限制在主线程中进行，如果在子线程中进行 UI 操作，会抛出异常。但是有时候我们需要在子线程中进行耗时操作，然后将结果返回给主线程，这时候就需要用到 Handler 机制。

如果可以多线程进行 UI 操作，为保证安全性就需要加锁，这样会导致卡顿。所以 Android 采用了单线程模型，同时 Android Activity 生命周期事件通知也是通过 Handler 机制完成的。

## Handler、Looper、MessageQueue

先来简单介绍下三者的作用和发送、处理消息的概要流程。

Handler 的使用需要当前线程中存在已经初始化的 Looper，在 ActivityThread 的 main 方法中会初始化主线程的 Looper，Looper 会通过 TreadLocal 和当前线程绑定，所以主线程可以直接使用 Handler，而子线程使用 Handler 需要先做初始化 Looper 的操作，HandlerThread 对此操作做了封装。

Looper 会创建一个 MessageQueue，MessageQueue 用来存放 Message，Handler 通过 Looper 获取 MessageQueue，然后通过 MessageQueue 进行消息的入队。

三者的作用如下：

- Handler：发送和处理消息；
- Looper：`looper.loop()` 方法中有一个 for 循环，不断的从 MessageQueue 中取出消息，只有在 `MessageQueue.next()` 返回 null 时，才会终止循环；
- MessageQueue：存放消息，链表结构。

Handler 发送消息最终会通过 `sendMessageAtTime` 方法调用 MessageQueue 的 `enqueueMessage` 方法，将消息入队。每个入队的 Message 会带有要执行的时间 when 字段，此时间是相对于系统启动时间的时间戳。入队时，会检查 when 字段，整个链表是按照 when 从小到大排序；

当 Looper 从 MessageQueue 中取出 Message 时，`MessageQueu.next()` 方法也有一个 for 循环，会判断当前时间是否大于 when，如果大于则取出，否则会计算等待时间，进入下一次循环。

当 `Looper.loop()` 方法中拿到消息后，会执行 `msg.target.dispatchMessage(msg)` 方法，这个方法会调用 `msg.callback` 或 `handler.handleMessage` 方法，这样就完成了消息的处理。

另外，由于消息加入链表和取出会加锁，所以消息的执行并不能严格按照 msg.when 的时间执行。

## Handler 初始化

```java
1. Looper.prepare();
2. Looper.loop();
3. mHandler = new Handler(getLooper());
```

## 发送消息流程

1. Handler 发送消息 `handler.postMessage\handler.sendMessage`；
2. 最终调用到 `handler.enqueueMessage`，在方法内部设置 `msg.target = this`，和 `setAsynchronous` 处理异步消息；
3. 最后调用 `messageQueue.enqueueMessage` 方法，将消息加入链表，以下为 `enqueueMessage` 方法的核心代码：
   ```java
   boolean enqueueMessage(Message msg, long when) {
         if (msg.target == null) {
             throw new IllegalArgumentException("Message must have a target.");
         }

         synchronized (this) {
             ...
             if (p == null || when == 0 || when < p.when) {
                 // 当前无消息，或非延迟消息，或延迟时间小于当前消息的延迟时间，直接插入到链表头部
                 msg.next = p;
                 mMessages = msg;
                 needWake = mBlocked;
             } else {
                 needWake = mBlocked && p.target == null && msg.isAsynchronous();
                 Message prev;
                 // 找到第一个延迟时间大于当前消息的延迟时间的消息，插入到该消息之前
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
                 msg.next = p;
                 prev.next = msg;
             }

             if (needWake) {
                 nativeWake(mPtr);
             }
         }
         return true;
     }
   ```

## 处理消息流程

1. Looper.loop() 开启循环；
   ```java
   for (;;) {
      if (!loopOnce(me, ident, thresholdOverride)) {
          return;
      }
   }
   ```
2. loop 方法中会调用 `MessageQueue.next()` 方法取出消息，进行处理；
   ```java
   private static boolean loopOnce(final Looper me, final long ident, final int thresholdOverride) {
      Message msg = me.mQueue.next(); // might block
      if (msg == null) {
         // 取出的消息为空时，代表 MessageQueue 已经退出，返回 false，终止循环
         return false;
      }
      ...
      // 执行消息
      msg.target.dispatchMessage(msg);
      ...
   }
   ```
3. `MessageQueue.next()` 方法中主要处理了以下核心逻辑：
   1. 如果是同步屏障消息，则优先取出异步消息进行处理；
   2. 如果到了同步消息的执行时间，则取出同步消息进行处理；
   3. 如果当前没有消息要处理，则处理空闲消息。
   ```java
   for (;;) {
      ...
      nativePollOnce(ptr, nextPollTimeoutMillis);

      synchronized (this) {
          final long now = SystemClock.uptimeMillis();
          Message prevMsg = null;
          Message msg = mMessages;
          if (msg != null && msg.target == null) {
              // 如果是同步屏障消息，优先取出异步消息进行处理
              do {
                  prevMsg = msg;
                  msg = msg.next;
              } while (msg != null && !msg.isAsynchronous());
          }
          if (msg != null) {
              if (now < msg.when) {
                  // 还没到消息的执行时间，计算等待时间，继续循环
                  nextPollTimeoutMillis = (int) Math.min(msg.when - now, Integer.MAX_VALUE);
              } else {
                  // 可以执行消息，从链表中取出
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
              nextPollTimeoutMillis = -1;
          }
          ...
          // 如果当前没有消息要处理，则计算是否有空闲消息要处理
          if (pendingIdleHandlerCount < 0 && (mMessages == null || now < mMessages.when)) {
              pendingIdleHandlerCount = mIdleHandlers.size();
          }
          ...
      }
      ...
      // 调用 queueIdle 方法处理空闲消息
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
   }
   ```
4. `handler.dispatchMessage` 方法中处理消息的逻辑如下：
   1. 如果消息有 callback，则执行 `msg.callback`；
   2. 如果当前 Handler 传递了 callback 参数，则执行 `mCallback.handleMessage(msg)`；
   3. 最后执行 Handler 重写的 `handleMessage(msg)` 方法。
   ```java
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

## IdleHandler

IdleHandler 用来处理 MessageQueue 中的空闲消息，空闲消息是指当 MessageQueue 中没有消息需要处理时，会调用 IdleHandler 的 `queueIdle` 方法，处理这些低优先级的消息。此部分逻辑在 `MessageQueue.next()` 方法中。

## 同步屏障和异步消息

可以通过 `MessageQueue.postSyncBarrier()` 方法，增加同步屏障消息，同步屏障消息的 msg.target 为 null。

异步消息和同步消息没有区别，只有在增加了同步屏障消息后才出现差异。异步消息在初始化 Handler 时使用 mAsynchronous 参数来标识，如果为 true，则表示异步消息，否则为同步消息。此外，也可以通过 `Message.setAsynchronous` 方法来单独设置消息为异步消息。

当 `MessageQueue.next()` 方法检测到同步屏障消息时，回去获取所有的异步消息进行处理。也就是说，同步屏障消息为 Handler 提供了一种优先处理异步消息的机制。但是同步屏障不会自己移除，需要通过 `MessageQueue.removeSyncBarrier` 手动移除，否则会造成同步消息无法处理。

## Handler 与内存泄漏

当 Handler 是匿名内部类或非静态内部类时，会持有外部类的引用，如果 Activity 切换后，Handler 没有处理完消息，就会导致内存泄漏。同时主线程 Looper 生命周期和进程生命周期一致。

内存泄漏的引用链为：Looper -> MessageQueue -> Message -> Handler -> Activity。

解决方法如下：

1. Activity 销毁时，调用 `handler.removeCallbacksAndMessages` 清空 MessageQueue 中的消息和 callback；
2. 使用静态内部类和弱引用来解决内存泄漏问题。
