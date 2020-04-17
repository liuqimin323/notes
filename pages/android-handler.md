# Android Handler

A Handler allows you to send and process Message and Runnable objects associated with a thread's MessageQueue. As threads are not allowed accessing UI thread. When a thread needs update UI, it can send a message to UI thread Message Queue through Handler object.

![image](http://qny.ivanfan.site/handler_Looper_MessageQueue.jpg)

## Usages:

### Method 1: post(Runnable)
1. Create a thread, implement run() to process time consuming task.
2. Outside the thread, create a handler, override runnable to process UI related task.

```java
new Thread(new Runnable() {
    @Override
    public void run() {
        // do something
        handler.post(new Runnable() {
            @Override
            public void run() {
                // update UI
            }
        });
    }
});
```

### Method 2: sendMessage(Message)
1. Create a thread, override `run()` to process time consuming tasts.
2. Create a Message through `Message.obtain()`, set up `what` label and data
3. Create a Handler, override `handleMessage()` to handle received message
4. send message in the thread.

```java
// create handler
private handler handler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
        super.handlerMessage(msg)
        switch(msg.what) { 
            case 1:
            //...
            break;
        }
    }
}

new Thread(new Runnable() {
    @Override
    public void run() {
        // do something
        // create message
        Message msg = Message.obtain();
        msg.obj = data;
        msg.what = 1;
        handler.sendMessage(msg);
    }
}).start();

```

## Potential issues

### Message leak
If a handler is inside an Activity, when the Activity is finished, handler may not finished yet, then Activity has memory leak. So whenever creates and uses a handler, it is better be static inner class.

### Null Pointer
If the activity is finished, and released resources in `onDestroy()`, and at the meantime, the handler is processing `handlerMessage()` but related resources is released, then it makes them Null Pointer.

### Improvement
1. make Handler static. If it holds an activity, make the activity WeakReference.

```java
    private static class MyHandler extends handler {
        private final WeakReference<Activity> mActivityRef;
        // ......
    }
```

2. Instead of try catch, remove the message queue in `onDestroy()`:
```java
@Override
protected void onDestroy() {
    super.onDestroy();
    handler.removeCallbacks(postRunnable); // if handler has callback
    handler.removeMessage(what); // if handler handled message
}
```

## Handler Work Flow

![image](http://qny.ivanfan.site/handler.png)

1. When a handler seend message or post runnable, it will call `enqueueMessage` and insert the message into the MessageQueue
2. When Looper repeatly get messages from MessageQueue. Looper will dispatch message, and call `handler.handleMessage()`

### Handler send message

1. Send Message
```java
    Message msg = Message.obtain();
    msg.obj = data;
    msg.what = 1;
    handler.sendMessage(msg);
```

Both of `handler.sendMessage` and `handler.post(Runnable)` will call `handler.sendMessageDelayed` -> `handler.sendMessageAtTime` -> `MessageQueue.enqueueMessage`.

2. Enqueue Message
In `handler.enqueueMessage`, it wil set the handler to be the `message.target`. In `MessageQueue.enqueueMessage`, it will add the message to queue.

3. Set and get Looper through ThreadLocal
When creating a Looper, it will create a message queue. Looper can only be created by `Looper.prepare`, so one Thread only allows one Looper.

```java
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
```

A Thread Local is an instance holding Looper, so the handler can get the Looper through ThreadLocal.

ThreadLocal.get():
```java
    /**
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
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue();
    }

```
4. Looper dispatch message.
in `Looper.loop()`, it gets next message thorugh `MessageQueue.next()`. If the message is null, ends cycle, else Looper will dispatch message by `msg.target.dispatchMessage` to call back to handler.

```java
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

        // Allow overriding a threshold with a system prop. e.g.
        // adb shell 'setprop log.looper.1000.main.slow 1 && stop && start'
        final int thresholdOverride =
                SystemProperties.getInt("log.looper."
                        + Process.myUid() + "."
                        + Thread.currentThread().getName()
                        + ".slow", 0);

        boolean slowDeliveryDetected = false;

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
            long slowDispatchThresholdMs = me.mSlowDispatchThresholdMs;
            long slowDeliveryThresholdMs = me.mSlowDeliveryThresholdMs;
            if (thresholdOverride > 0) {
                slowDispatchThresholdMs = thresholdOverride;
                slowDeliveryThresholdMs = thresholdOverride;
            }
            final boolean logSlowDelivery = (slowDeliveryThresholdMs > 0) && (msg.when > 0);
            final boolean logSlowDispatch = (slowDispatchThresholdMs > 0);

            final boolean needStartTime = logSlowDelivery || logSlowDispatch;
            final boolean needEndTime = logSlowDispatch;

            if (traceTag != 0 && Trace.isTagEnabled(traceTag)) {
                Trace.traceBegin(traceTag, msg.target.getTraceName(msg));
            }

            final long dispatchStart = needStartTime ? SystemClock.uptimeMillis() : 0;
            final long dispatchEnd;
            try {
                msg.target.dispatchMessage(msg);
                dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
            } finally {
                if (traceTag != 0) {
                    Trace.traceEnd(traceTag);
                }
            }
            if (logSlowDelivery) {
                if (slowDeliveryDetected) {
                    if ((dispatchStart - msg.when) <= 10) {
                        Slog.w(TAG, "Drained");
                        slowDeliveryDetected = false;
                    }
                } else {
                    if (showSlowLog(slowDeliveryThresholdMs, msg.when, dispatchStart, "delivery",
                            msg)) {
                        // Once we write a slow delivery log, suppress until the queue drains.
                        slowDeliveryDetected = true;
                    }
                }
            }
            if (logSlowDispatch) {
                showSlowLog(slowDispatchThresholdMs, dispatchStart, dispatchEnd, "dispatch", msg);
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

5. Handler handle message
In `handler.dispatchmessage()`, if message has callback, call message callback. Else if handler has callback, call handler `callback.handleMessage`. Else if `handler.handleMessage` if override, it will call `handleMessage`.

## Create custom handler is another Thread
1. In the thread, create a Looper through Looper.preopare()
2. Create Handler
3. run message though created Looper.
