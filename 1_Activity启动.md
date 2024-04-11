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

``` Activity Thread
public static void main(String[] args) {
        Trace.traceBegin(Trace.TRACE_TAG_ACTIVITY_MANAGER, "ActivityThreadMain");

        // Install selective syscall interception
        AndroidOs.install();

        // CloseGuard defaults to true and can be quite spammy.  We
        // disable it here, but selectively enable it later (via
        // StrictMode) on debug builds, but using DropBox, not logs.
        CloseGuard.setEnabled(false);

        Environment.initForCurrentUser();

        // Make sure TrustedCertificateStore looks in the right place for CA certificates
        final File configDir = Environment.getUserConfigDirectory(UserHandle.myUserId());
        TrustedCertificateStore.setDefaultUserDirectory(configDir);

        // Call per-process mainline module initialization.
        initializeMainlineModules();

        Process.setArgV0("<pre-initialized>");

        Looper.prepareMainLooper();

        // Find the value for {@link #PROC_START_SEQ_IDENT} if provided on the command line.
        // It will be in the format "seq=114"
        long startSeq = 0;
        if (args != null) {
            for (int i = args.length - 1; i >= 0; --i) {
                if (args[i] != null && args[i].startsWith(PROC_START_SEQ_IDENT)) {
                    startSeq = Long.parseLong(
                            args[i].substring(PROC_START_SEQ_IDENT.length()));
                }
            }
        }
        ActivityThread thread = new ActivityThread();
        thread.attach(false, startSeq);

        if (sMainThreadHandler == null) {
            sMainThreadHandler = thread.getHandler();
        }

        if (false) {
            Looper.myLooper().setMessageLogging(new
                    LogPrinter(Log.DEBUG, "ActivityThread"));
        }

        // End of event ActivityThreadMain.
        Trace.traceEnd(Trace.TRACE_TAG_ACTIVITY_MANAGER);
        Looper.loop();

        throw new RuntimeException("Main thread loop unexpectedly exited");
    }
```
