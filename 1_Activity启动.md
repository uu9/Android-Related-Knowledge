```
"main@19885" prio=5 tid=0x2 nid=NA runnable
  java.lang.Thread.State: RUNNABLE
	  at com.example.testperformanceappnewversion.MainActivity.<init>(MainActivity.java:10)
	  at java.lang.Class.newInstance(Class.java:-1)
	  at android.app.AppComponentFactory.instantiateActivity(AppComponentFactory.java:95)
	  at androidx.core.app.CoreComponentFactory.instantiateActivity(CoreComponentFactory.java:45)
	  at android.app.Instrumentation.newActivity(Instrumentation.java:1273)
	  at android.app.ActivityThread.performLaunchActivity(ActivityThread.java:3660)
	  at android.app.ActivityThread.handleLaunchActivity(ActivityThread.java:3937)
	  at android.app.servertransaction.LaunchActivityItem.execute(LaunchActivityItem.java:103)
	  at android.app.servertransaction.TransactionExecutor.executeCallbacks(TransactionExecutor.java:135)
	  at android.app.servertransaction.TransactionExecutor.execute(TransactionExecutor.java:95)
	  at android.app.ActivityThread$H.handleMessage(ActivityThread.java:2288)
	  at android.os.Handler.dispatchMessage(Handler.java:106)
	  at android.os.Looper.loopOnce(Looper.java:210)
	  at android.os.Looper.loop(Looper.java:299)
	  at android.app.ActivityThread.main(ActivityThread.java:8293)
	  at java.lang.reflect.Method.invoke(Method.java:-1)
	  at com.android.internal.os.RuntimeInit$MethodAndArgsCaller.run(RuntimeInit.java:556)
	  at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1045)
```
栈信息，注意需要反着看，中间没啥用的逻辑直接删了。

```
@UnsupportedAppUsage
    public static void main(String[] argv) {
        ZygoteServer zygoteServer = null;

        // Zygote是所有Android进程的模板，如果在其中创建线程就会导致所有的Android子进程不干净，
        // 这里设置创建子线程就会出错以此保证进程是干净的
        ZygoteHooks.startZygoteNoThreadCreation();

        Runnable caller;
        // 初始化Zygote（同时会判断是否为主Zygote，主不会加载类和执行命令）
        // 如果是在子进程，运行caller
    }
```

Activity Thread
``` 
public static void main(String[] args) {

        Looper.prepareMainLooper();

        ActivityThread thread = new ActivityThread();
	// 会有个seq号
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```

直接看消息机制loopOnce
```
private static boolean loopOnce(final Looper me,
            final long ident, final int thresholdOverride) {
        Message msg = me.mQueue.next(); // might block
        if (msg == null) {
            // No message indicates that the message queue is quitting.
            return false;
        }

        // Make sure the observer won't change while processing a transaction.
        final Observer observer = sObserver;

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
        Object token = null;
        if (observer != null) {
            token = observer.messageDispatchStarting();
        }
        long origWorkSource = ThreadLocalWorkSource.setUid(msg.workSourceUid);
        try {
            msg.target.dispatchMessage(msg);
            if (observer != null) {
                observer.messageDispatched(token, msg);
            }
            dispatchEnd = needEndTime ? SystemClock.uptimeMillis() : 0;
        } catch (Exception exception) {
            if (observer != null) {
                observer.dispatchingThrewException(token, msg, exception);
            }
            throw exception;
        } finally {
            ThreadLocalWorkSource.restore(origWorkSource);
            if (traceTag != 0) {
                Trace.traceEnd(traceTag);
            }
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

        return true;
    }
```
