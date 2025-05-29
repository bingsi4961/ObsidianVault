---
title: Java SE17 技術手冊 CH11 ~ CH??
tags: [入庫]

---

# Java SE17 技術手冊 CH11 ~ CH??
###### tags: `書籍筆記`

---

## 第11章 執行緒與並行API

### 11.1.2 Thread 與 Runnable《p.11-4》

```java=
java.lang.Thread 一般類別
java.lang.Runnable 介面
java.lang.Thread.sleep(100)	靜態方法

public interface java.lang.Runnable {
    public abstract void run();
}

public class java.lang.Thread implements java.lang.Runnable {
    
    // target 是 java.lang.Thread 建構時，所傳入的類別實例
    private Runnable target;	
	
    public Thread(java.lang.Runnable target) {
        this(null, target, "Thread-" + nextThreadNum(), 0);
    }
	
    public Thread(ThreadGroup group, Runnable target, String name, long stackSize) {
        this(group, target, name, stackSize, null, true);
    }
	
    public static native void sleep(long millis) throws InterruptedException;
	
    public synchronized void start() { 
        // doSome()
        // start0();
    }
	
    private native void start0();    // 似乎是從這裡呼叫 run()
	 
    @Override
    public void run() {
        // target 是 java.lang.Thread 建構時，所傳入的類別實例
        if (target != null) {
            target.run();
        }
    }

    public static final int MIN_PRIORITY = 1;
    public static final int NORM_PRIORITY = 5;
    public static final int MAX_PRIORITY = 10;

    public final void setPriority(int newPriority) {
        // doSome();
    }    
}
```

### 11.1.3 執行緒生命週期《p.11-6》

```java=
public class java.lang.Thread implements java.lang.Runnable {

    public static native Thread currentThread();
	
    private volatile boolean interrupted;	//中斷標誌
	
    // new java.lang.Thread().interrupt() 設定中斷標誌為True
    // java.lang.Thread.currentThread().interrupted() 回傳中斷標誌結果(true/false)，並清除中斷標誌 (設定為false)
    // new java.lang.Thread().isInterrupted() 回傳中斷標誌結果(true/false)，但不清除中斷標誌
	
    public void interrupt() {
        interrupted = true;
        // inform VM of interrupt
        interrupt0();
    }
	
    public static boolean interrupted() {
        Thread t = currentThread();    // 故此靜態方法，僅能在子線程內使用
        boolean interrupted = t.interrupted;

        if (interrupted) {
            t.interrupted = false;	// 清除中斷標誌
            clearInterruptEvent();
        }
        return interrupted;
    }
	
    // 不清除中斷標誌，只回傳中斷標誌結果
    public boolean isInterrupted() {
        return interrupted;
    }
}
```

* new java.lang.Thread().interrupt() 用於中斷執行緒。如果任何執行緒處於休眠或等待狀態(sleep()、wait()、join())，那麼使用 interrupt()，可以通過丟擲 java.lang.InterruptedException 來中斷執行緒執行。
* 如果執行緒未處於休眠或等待狀態，則呼叫 interrupt() 將執行正常行為，並且不會中斷執行緒，但會將中斷標誌設定為true。

- [Java里一个线程调用了Thread.interrupt()到底意味着什么？](https://www.zhihu.com/question/41048032)
- [停止多執行緒](http://n.sfs.tw/content/index/16083)
- [Java中interrupt的使用](https://www.cnblogs.com/jenkov/p/juc_interrupt.html)

```java=
/**
 * 主線程改變了子線程的中斷標誌為 true，
 * 由於子線內部並沒有進行檢測和處理中斷標誌，故子線程並不會終止，會無限列印下去
 */
java.lang.Thread thrd1 = new java.lang.Thread(() -> {
    int follow = 0;
    while (true) {
        out.printf("follow is %d %n", ++follow);
    }
});

thrd1.start();

try {
    java.lang.Thread.sleep(1000);
    thrd1.interrupt();
} catch (java.lang.InterruptedException e) {
    e.printStackTrace();
}

/*
    follow is 244302 
    follow is 244303 
    follow is 244304 
    follow is 244305 
    follow is 244306 
    follow is 244307
    無限列印下去 ...
*/

```

```java=
/**
 * 主線程改變了子線程的中斷標誌為 true，
 * 由於子線程偵測到中斷標誌為 true，故跳出 while 迴圈
 * 清理資源後，結束子線程
 */
java.lang.Thread thrd2 = new java.lang.Thread(() -> {
    int follow = 0;
    while (!java.lang.Thread.currentThread().isInterrupted()) {
        out.printf("follow is %d %n", ++follow);
        out.printf("isInterrupted() is %b %n", Thread.currentThread().isInterrupted());
    }
    out.println("清理資源");
});

thrd2.start();

try {
    java.lang.Thread.sleep(10);
    thrd2.interrupt();
} catch (java.lang.InterruptedException e) {
    e.printStackTrace();
}

/*
    follow is 1 
    isInterrupted() is false 
    follow is 2 
    isInterrupted() is false 
    follow is 3 
    isInterrupted() is false 
    follow is 4 
    isInterrupted() is false 
    follow is 5 
    isInterrupted() is false 
    follow is 6 
    isInterrupted() is false 
    清理資源
*/
```

```java=

/**
 * 主線程改變了子線程的中斷標誌為 true，Thread.sleep(3000) "立即"丟出 InterruptedException
 * 中斷停止工作(sleep、wait、join)的執行緒
 */
java.lang.Thread thrd3 = new java.lang.Thread(() -> {
    while (!Thread.currentThread().isInterrupted()) {
        try {
            java.lang.Thread.sleep(3000);
        } catch (java.lang.InterruptedException ex) {
            // 異常被捕獲後，阻塞被打破、★★ 中斷標誌被設為false
            out.printf("subNode1：%b %n", Thread.currentThread().isInterrupted());
            err.printf("subNode1：回收資源 %n");
            ex.printStackTrace();
            // throw new java.lang.RuntimeException("Thread interrupted..." + e); // 丟出錯誤給主線程，並結束子線程？？
        }
        err.printf("subNode2：while() %n");
    }
});

thrd3.start();

try {

    java.lang.Thread.sleep(1000);
    thrd3.interrupt();
    out.printf("node1：%b %n", thrd3.isInterrupted());

    java.lang.Thread.sleep(1000);
    thrd3.interrupt();
    out.printf("node2：%b %n", thrd3.isInterrupted());

} catch (InterruptedException e) {
    e.printStackTrace();
}

/*
    node1：false    // 可能是 Thread.sleep() 已發生 InterruptedException，中斷標誌被設為false
    subNode1：false
    subNode1：回收資源 
    java.lang.InterruptedException: sleep interrupted
        at java.base/java.lang.Thread.sleep(Native Method)
        at prjMultiThread1/tw.idv.cko2x.InterruptDemo.lambda$0(InterruptDemo.java:57)
        at java.base/java.lang.Thread.run(Thread.java:833)
    subNode2：while() 
    node2：true    // 可能是 Thread.sleep() 還未發生 InterruptedException，中斷標誌仍然是 True
    subNode1：false 
    subNode1：回收資源 
    java.lang.InterruptedException: sleep interrupted
        at java.base/java.lang.Thread.sleep(Native Method)
        at prjMultiThread1/tw.idv.cko2x.InterruptDemo.lambda$0(InterruptDemo.java:57)
        at java.base/java.lang.Thread.run(Thread.java:833)
    subNode2：while() 
    subNode2：while() 
    subNode2：while() 
    subNode2：while() 
    無限列印下去 ...
*/
```

```java=
/*
 * interrupted() 回傳中斷標誌結果(true/false)，並清除中斷標誌 (設定為false)
 */
java.lang.Thread thrd4 = new java.lang.Thread(() -> {
    while (true) {
        if (Thread.interrupted()) {
            out.println("清理資源");
            break;	// 跳出while迴圈，然後結束子線程
        }
        err.printf("subNode1：while() %n");
    }
});

thrd4.start();

try {
    java.lang.Thread.sleep(5);
    thrd4.interrupt();
} catch (InterruptedException e) {
    e.printStackTrace();
}

/*
    subNode1：while() 
    subNode1：while() 
    subNode1：while() 
    subNode1：while() 
    subNode1：while() 
    subNode1：while() 
    subNode1：while() 
    subNode1：while() 
    subNode1：while() 
    subNode1：while() 
    清理資源
*/

```

### *- 安插執行緒《 p.11-11》*

```java=
out.println("main thread start ...");

Thread thrdA = new Thread(() -> {
    out.println("Thread A start ...");
    for (int i = 0; i < 5; i++) {
        try {
            Thread.sleep(1000);
            out.printf("i：%d for Thread A %n", i);
        } catch (InterruptedException e) {
            // TODO 自動產生的 catch 區塊
            e.printStackTrace();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
    out.println("Thread A end ...");
});


Thread thrdB = new Thread(() -> {
    out.println("Thread B start ...");
    for (int i = 0; i < 5; i++) {
        try {
            Thread.sleep(1000);
            out.printf("i：%d for Thread B %n", i);
        } catch (InterruptedException e) {
            // TODO 自動產生的 catch 區塊
            e.printStackTrace();
        } catch (Exception ex) {
            ex.printStackTrace();
        }
    }
    out.println("Thread B end ...");
});

thrdA.start();
thrdB.start();

// thrdA、thrdB 同時執行，兩者都執行完後，再回到主線程執行
// thrdA.join();
// thrdB.join();

thrdA.join(2000);    // 2秒後，主線程就不管 thrdA，繼續主線程原本的流程
thrdB.join();        // 雖然主線程不管 thrdA了，但仍然要等 thrdB 完成後，再繼續主線程原本的流程

out.println("main thread end ...");

/*
    main thread start ...
    Thread A start ...
    Thread B start ...
    i：0 for Thread B 
    i：0 for Thread A 
    i：1 for Thread B 
    i：1 for Thread A 
    i：2 for Thread B 
    i：2 for Thread A 
    i：3 for Thread B 
    i：3 for Thread A 
    i：4 for Thread B 
    Thread B end ...
    main thread end ...
    i：4 for Thread A 
    Thread A end ...
*/
```

### *- 停止執行緒《 p.11-13》*

```java=
 var s = new Runnable() {
    private boolean isContinue = true;
    private int i = 0;

    public void Stop() {
        isContinue = false;
    }

    @Override
    public void run() {
        while (isContinue) {
            System.out.printf("%d--%n", ++i);
        }
    }
};

Thread thrd = new Thread(s);
thrd.start();

try {
    java.lang.Thread.sleep(500);
    s.Stop();
} catch (InterruptedException e) {
    e.printStackTrace();
} catch (Exception e) {
    e.printStackTrace();
}
```

### 11.1.4 關於ThreadGroup《p.11-14》

```java=
public class java.lang.Thread implements java.lang.Runnable {
	
    private volatile String name;
    private static int threadInitNumber;

    public static native Thread currentThread();

    private static synchronized int nextThreadNum() {
        return threadInitNumber++;
    }

    public Thread() {
        // to A
        this(null, null, "Thread-" + nextThreadNum(), 0);
    }

    public Thread(Runnable target) {
        // to A
        this(null, target, "Thread-" + nextThreadNum(), 0);
    }

    Thread(Runnable target, @SuppressWarnings("removal") AccessControlContext acc) {
        // to C
        this(null, target, "Thread-" + nextThreadNum(), 0, acc, false);
    }

    public Thread(ThreadGroup group, Runnable target) {
        // to A
        this(group, target, "Thread-" + nextThreadNum(), 0);
    }

    public Thread(String name) {
        // to A
        this(null, null, name, 0);
    }

    public Thread(ThreadGroup group, String name) {
        // to A
        this(group, null, name, 0);
    }

    public Thread(Runnable target, String name) {
        // to A
        this(null, target, name, 0);
    }

    public Thread(ThreadGroup group, Runnable target, String name) {
        // to A
        this(group, target, name, 0);
    }

    // A
    public Thread(ThreadGroup group, Runnable target, String name, long stackSize) {
        // to C
        this(group, target, name, stackSize, null, true);
    }

    // B
    public Thread(ThreadGroup group, Runnable target, String name, long stackSize, boolean inheritThreadLocals) {
        // to C
        this(group, target, name, stackSize, null, inheritThreadLocals);
    }

    // C
    private Thread(ThreadGroup g, Runnable target, String name,
                   long stackSize, AccessControlContext acc,
                   boolean inheritThreadLocals) {

        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;
        Thread parent = currentThread();
        // 創建 Thread 實例，在執行 start()前，都還是主線程在跑 ??

        this.target = target;
    }

    public final String getName() {
        return name;
    }

    public final synchronized void setName(String name) {        
        if (name == null) {
            throw new NullPointerException("name cannot be null");
        }

        this.name = name;        
    }
}

public class myThread extends Thread {
    public myThread() {		
        super();
        // 因為繼承 Thread，所以系統會自動執行父類別(Thread)建構子
        // 建構子會指定線程名稱 ("Thread-" + nextThreadNum()) 給變數 name
        // Thread.getName() 回傳變數 name
        // Thread.setName(xxx) 設定變數 name

        System.out.println();
    }
}
```

- [多线程中Thread.currentThread().getName()和this.getName()的区别](https://blog.csdn.net/lzm18064126848/article/details/54023275)
- [Thread.currentThread().getName() 和 this.getName()区别详解](https://www.cnblogs.com/jichi/p/15916171.html)

:::info
selfDef extends Thread => selfDef 為一個線程類別 ( 有自已的 線程名稱變數、start()、run() 等… )
var Thrd1 = new Thread(selfDef) => Thrd1 為另一個線程類別
Thrd1.start() => 以 Thrd1 線程來跑 selfDef.run()
:::

:::success
selfDef implements Runnable => selfDef 為一個自訂類別
var Thrd1 = new Thread(selfDef) => Thrd1 為一個線程類別
Thrd1.start() => 以 Thrd1 線程來跑 selfDef.run()
:::

* currentThread.getName() 代表當前執行線程的名稱，實際上是 thead.getName()。
* this.getName() 代表線程類別的名稱，實際上是 target.getName()。

1. 創建 Thread 實例時，在執行 start()前，都還是主線程在跑 ??
1. 自訂類別繼承Thread類別，就算是一個線程類別 (有線程名稱變數、start()、run() 等…)，但還沒啟動

### *- java.lang.ThreadGroup.activeCount()《 p.11-15》*

```java=
public class activeCountDemo {
    public static void main(String[] args) throws InterruptedException {

        ThreadGroup tg1 = new java.lang.ThreadGroup("group1");
        Thread thrd1 = new java.lang.Thread(tg1, new some(), "group1's - thrd1");
        Thread thrd2 = new java.lang.Thread(tg1, new some(), "group1's - thrd2");
        thrd1.start();
        thrd2.start();		
        out.println(tg1.activeCount());    // 取得線程群組中，有在活動的線程數量

        // group1's - thrd1
        // 1
        // group1's - thrd2
    }
}

class some implements java.lang.Runnable {
    @Override
    public void run() {		
        try {
            //Thread.sleep(500);
            out.println(Thread.currentThread().getName());		
        }catch(Exception e) {
            e.printStackTrace();
        }		
    }
}

```

### *- java.lang.ThreadGroup.uncaughtException(Thread t, Throwable e)《 p.11-15》*

1. 在 Runnable.run() 中，無法丟出受檢例外 (checked Exception)
2. 在 Runnable.run() 中，可丟出非受檢例外 (unChecked Exception)，但在主線程補捉不到(無法處理)，只會在主控台顯示Exception
3. 子線程發生Exception時，無法丟回主線程處理，因主線程已經跑後面的程序了。
4. 可利用 ThreadGroup.uncaughtException() 來處理


```java=
Thread thrd1 = new Thread(() -> {
    try {
        Thread.sleep(1010);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    throw new RuntimeException("在子線程發生非受檢例外!!");
}, "Thread is thrd1");

try {
    thrd1.start();
    for (int i = 0; i < 5; i++) {
        Thread.sleep(500);
        out.println(i);
    }
} catch (java.lang.Exception ex) {
    out.printf("主線程處理 [%s]", ex.getMessage());
    // 子線程丟出非受檢例外，但主線程無法補捉及處理
}

/*
    0
    1
    Exception in thread "Thread is thrd1" java.lang.RuntimeException: 在子線程發生非受檢例外!!
        at prjThreadGroup/tw.idv.cko2x.ThreadException.lambda$0(ThreadException.java:15)
        at java.base/java.lang.Thread.run(Thread.java:833)
    2
    3
    4
*/

```

```java=
// java.lang.ThreadGroup implements java.lang.Thread.UncaughtExceptionHandler
// 所以 ThreadGroup 本身就有實作 uncaughtException(Thread t, Throwable e)
ThreadGroup tg1 = new ThreadGroup("group1") {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        out.printf("透過 ThreadGroup.uncaughtException 處理 [%s-%s] %n", t.getName(), e.getMessage());
    }
};

Thread thrd1 = new Thread(tg1, () -> {
    try {
        Thread.sleep(1010);
    } catch (InterruptedException e) {
        e.printStackTrace();
    }
    throw new RuntimeException("在子線程發生非受檢例外!!");
}, "Thread is thrd1");

try {
    thrd1.start();
    for (int i = 0; i < 5; i++) {
        Thread.sleep(500);
        out.println(i);
    }
} catch (java.lang.Exception ex) {
    out.printf("主線程處理 [%s]", ex.getMessage());
}

/*
    0
    1
    透過 ThreadGroup.uncaughtException 處理 [Thread is thrd1-在子線程發生非受檢例外!!] 
    2
    3
    4
*/
```

### *- java.lang.Thread.UncaughtExceptionHandler 介面《 p.11-16》*

- [UncaughtExceptionHandler的作用以及簡單用法](https://blog.csdn.net/qq_42799615/article/details/111658473)

```java=
public class main_getName {
    public static void main(String[] args) {

        ThreadGroup tg1 = new ThreadGroup("group1");

        // Thread thrd1 = new Thread(tg1,new SubThrdIns(),"group1's thread -01");
        Thread thrd1 = new Thread(tg1, () -> {
            throw new java.lang.RuntimeException("子線程發生非受檢例外!!");
        }, "group1's thread -01");

        // thrd1.setUncaughtExceptionHandler(new ThrdExceptHandler());
        thrd1.setUncaughtExceptionHandler((t, e) -> System.out.printf("Thread is %s-%s", t.getName(), e.getMessage()));

        thrd1.start();
    }
}

class SubThrdIns implements java.lang.Runnable {
    @Override
    public void run() {
        throw new java.lang.RuntimeException("子線程發生非受檢例外!!");
    }
}

class ThrdExceptHandler implements java.lang.Thread.UncaughtExceptionHandler {
    @Override
    public void uncaughtException(java.lang.Thread t, java.lang.Throwable e) {
        System.out.printf("Thread is %s-%s", t.getName(), e.getMessage());
    }
}
```

```java=
public class java.lang.Thred implements Runnable {	
    
    public void run() {
        if (target != null) {
            target.run();
        }
    }

    // 此介面寫在類別裡
    public interface UncaughtExceptionHandler {
        void uncaughtException(Thread t, Throwable e);
    }
}

public class java.lang.ThreadGroup implements 
                java.lang.Thread.UncaughtExceptionHandler {

    private final ThreadGroup parent;
    String name;
    int maxPriority;
    ThreadGroup groups[];

    private ThreadGroup() {     // called from C code
        this.name = "system";
        this.maxPriority = Thread.MAX_PRIORITY;
        this.parent = null;
    }

    public ThreadGroup(String name) {
        this(Thread.currentThread().getThreadGroup(), name);
    }

    public final String getName() {
        return name;
    }

    public final int getMaxPriority() {
        return maxPriority;
    }
    
    // 實作 java.lang.Thread.UncaughtExceptionHandler.uncaughtException()
    public void uncaughtException(Thread t, Throwable e) {
        if (parent != null) {
            // ① 父類ThreadGroup.uncaughtException(t,e)
            parent.uncaughtException(t, e);
        } else {
            Thread.UncaughtExceptionHandler ueh =
                Thread.getDefaultUncaughtExceptionHandler();
            if (ueh != null) {
                // ② 預設UncaughtExceptionHandler.uncaughtException(t, e)
                ueh.uncaughtException(t, e);
            } else if (!(e instanceof ThreadDeath)) {
                System.err.print("Exception in thread \""
                                 + t.getName() + "\" ");
                e.printStackTrace(System.err);
                // ④ e.printTrace()
            }

            // ③ java.lang.ThreadDeath 系統嚴重錯誤，不處理
        }
    }
}
```

```java=
// 对所有线程对象设置异常处理器
Thread.setDefaultUncaughtExceptionHandler(Thread.UncaughtExceptionHandler)
Thread.UncaughtExceptionHandler ueh = Thread.getDefaultUncaughtExceptionHandler()

// 对指定线程对象设置异常处理器
Thrd1.setUncaughtExceptionHandler(Thread.UncaughtExceptionHandler);
Thread.UncaughtExceptionHandler ueh = Thrd1.getUncaughtExceptionHandler()

// 如果 setUncaughtExceptionHandler、setDefaultUncaughtExceptionHandler 同時包含
// 那麽會被 setUncaughtExceptionHandler 處理，setDefaultUncaughtExceptionHandler 則忽略。

```

```java=
// 例外處理順序第2
// ThreadGroup 本身就有實作 Thread.UncaughtExceptionHandler 介面，可參考上方程式碼
// 此為覆寫 ThreadGroup.uncaughtException()
ThreadGroup tg1 = new ThreadGroup("group1") {
    @Override
    public void uncaughtException(Thread t, Throwable e) {
        out.printf("ThreadGroup.uncaughtException() handle Exception：Thread is %s-%s %n", t.getName(),
                e.getMessage());
    }
};

// 例外處理順序第3
// 对所有线程对象设置异常处理器
java.lang.Thread.setDefaultUncaughtExceptionHandler((t, e) -> {
    out.printf("Thread.setDefaultUncaughtExceptionHandler() handle Exception：Thread is %s-%s %n", t.getName(),
            e.getMessage());
});

Thread thrd1 = new Thread(tg1, () -> {
    throw new java.lang.RuntimeException("子線程thrd1發生非受檢例外!!");
}, "No.1 for group1's thread");

Thread thrd2 = new Thread(tg1, () -> {
    throw new java.lang.RuntimeException("子線程thrd2發生非受檢例外!!");
}, "No.2 for group's thread");

// 例外處理順序第1
// 各線程要各自設置異常處理器
thrd1.setUncaughtExceptionHandler(
        (t, e) -> out.printf("thrd1.setUncaughtExceptionHandler() handle Exception：Thread is %s-%s %n",
                t.getName(), e.getMessage()));

thrd1.start();
thrd2.start();

// 若上述「三種例外處理 UncaughtExceptionHandler 介面都沒有實作」
// 則系統會跑 new ThreadGroup().uncaughtException(Thread t Throwable e) 的原生程式碼，可參考上方程式碼。
```

### 11.1.5 synchronized 與 volatile《p.11-17》

``` mermaid
stateDiagram-V2

[*] --> start()
start() --> Runnable
Runnable --> Running : Scheduler


Running --> Wait_Blocked : wait()
Wait_Blocked --> Runnable : notify()、notifyAll()

Running --> IO_Blocked : 等待輸入輸出
IO_Blocked --> Runnable : 輸入輸出完成

Running --> Lock_Blocked : 嘗試執行 synchronized
Lock_Blocked --> Runnable : 取得鎖定

Running --> Sleep_Blocked : 睡覺中
Sleep_Blocked --> Runnable : 睡醒了

Running --> Dead : run()執行完畢
Dead --> [*]

```

* 執行緒要執行 synchronized 區塊前，要先取得物件的內部鎖
* synchronized 區塊，同時只允許一個 Thread 執行
* synchronized 區塊執行過程中，允許切換執行緒，但內部鎖未釋放
---
* 呼叫物件的 wait()方法，線程會釋放內部鎖，並進入物件的「等待集(Wait Set)」而處於阻斷狀態。
* 呼叫物件的 notify()方法，會從該物件的「等待集(Wait Set)」中隨機通知一個線程進入 Runnable；但不會釋放目前線程的內部鎖。
* 呼叫物件的 notifyAll()方法，會從該物件的「等待集(Wait Set)」中通知所有線程進入 Runnable；但不會釋放目前線程的內部鎖。
* 物件「等待集(Wait Set)」中的線程，似乎不同於其他類型的阻斷線程(I/O、Sleep、Locker)。
---
* 線程在 sleep()、I/O、synchronized {} 進入Blocked 狀態時，不會釋放內部鎖。
* 比起執行緒輪流切換，wait() 是明確的將「線程進入Wait Blocked」及「釋放內部鎖」，並切換其他執行緒，立即性會比較高。
---
* 在 Running 嚐試要執行 synchronized 區塊的線程，因未取得內部鎖而進入 Locker Blocked 阻斷狀態；待取得內部鎖後，線程會先回到 Runnable 狀態，等待CPU排班器排入 Running。(不是先排入Running，再嚐試取得內部鎖，這樣可能會比較浪費效能) ??
* 存在「等待集(Wait Set)」的線程，收到 notify() 通知後，會與其他線程競爭內部鎖取得後，線程會先回到 Runnable 狀態，等待CPU排班器排入 Running。??

<br>

```java=
public class ArrayList {

    Object[] elems;
    int next;
    int times;

    public ArrayList() {
        this(16);
    }

    public ArrayList(int capacity) {
        elems = new Object[capacity];
    }

    // 鎖定整個方法
    public synchronized void add(Object o) {
        addFuncSrc(o);
    }

    // 鎖定區塊
    public void add2(Object o) {
        synchronized (this) {
            addFuncSrc(o);
        }
    }

    // 由調用端來鎖定
    public void add3(Object o) {
        addFuncSrc(o);
    }

    private void addFuncSrc(Object o) {
        times++;
        out.printf("第%d次 - %s - add() - begin %n", times, Thread.currentThread().getName());

        if (next == elems.length) {
            elems = java.util.Arrays.copyOf(elems, elems.length * 2);
            out.printf("第%d次 - %s - add() - copyOf %n", times, Thread.currentThread().getName());
        }

        elems[next++] = o;
        out.printf("第%d次 - %s - add() - end %n", times, Thread.currentThread().getName());
        out.println("--------------------------------------");
    }

    public Object get(int index) {
        return elems[index];
    }

    public int size() {
        return next;
    }
}

public class syncDemo {
    public static void main(String[] args) {

        ArrayList alist = new ArrayList(5);

        Thread t1 = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(50);
                    // alist.add("t1's element");
                    // alist.add2("t1's element");
                    synchronized (alist) {
                        alist.add3("t1's element");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "thrd1");

        Thread t2 = new Thread(() -> {
            while (true) {
                try {
                    Thread.sleep(50);
                    // alist.add("t2's element");
                    // alist.add2("t2's element");
                    synchronized (alist) {
                        alist.add3("t2's element");
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            }
        }, "thrd2");

        t1.start();
        t2.start();
    }
}
```

```java=
public static void main(String[] args) {
    
    // 匿名類別
    var Material = new Object() {
        private int data1 = 0;
        private int data2 = 0;

        // 因為是不同的物件內部鎖，故兩個 Thread 互不等待
        private Object lock1 = new Object();
        private Object lock2 = new Object();

        // CPU Process 輪流執行各個 Thread (有可能 synchronized 區塊 執行到一半時，CPU 切到另一個 Thread)
        // synchronized 區塊，同時只能有一個 Thread 執行			
        public void doSome() {	
            synchronized (lock1) {
                out.printf("data1:%d %n", data1++);
            }
        }

        public void doOther() {
            synchronized (lock2) {
                out.printf("data2:%d %n", data2--);
            }
        }
    };

    Thread t1 = new Thread(() -> {
        while (true) {
            Material.doSome();
        }
    });

    Thread t2 = new Thread(() -> {
        while (true) {
            Material.doOther();
        }
    });

    t1.start();
    t2.start();
}
```

### 11.2.1 Lock、ReadWriteLock 與 Condition《p.11-36》

```java=
(itf) java.util.concurrent.locks.Lock.lock();
(itf) java.util.concurrent.locks.Lock.unlock();
(cls) java.util.concurrent.locks.ReentrantLock.lock();
(cls) java.util.concurrent.locks.ReentrantLock.unlock();
※ ReentrantLock implements Lock
```

### *- 使用 java.util.concurrent.locks.Lock《 p.11-36》*

```java=
package java.util.concurrent.locks;

public interface Lock {
    void lock();
    void unlock();
    boolean tryLock();
}
```

### *- 使用 java.util.concurrent.locks.ReadWriteLock《 p.11-40》*

```java=
public class ReentrantLock implements Lock {
	
    public ReentrantLock() {
        sync = new NonfairSync();
    }

    public ReentrantLock(boolean fair) {
        sync = fair ? new FairSync() : new NonfairSync();
    }

    public void lock() {
        sync.lock();
    }

    public void unlock() {
        sync.release(1);
    }	

    // 查詢當前執行緒是否保持此鎖
    public boolean isHeldByCurrentThread() {
        return sync.isHeldExclusively();
    }

    // 如果鎖是自由的並且被當前執行緒獲取
    // 或者當前執行緒已經保持該鎖，則返回 true；否則返回 false
    public boolean tryLock() {
        return sync.tryLock();
    }
}

public interface ReadWriteLock {
    Lock readLock();
    Lock writeLock();
}

public class ReentrantReadWriteLock implements ReadWriteLock {
	
    public static class ReadLock implements Lock { ... }
    public static class WriteLock implements Lock { ... }

    private final ReentrantReadWriteLock.ReadLock readerLock;
    private final ReentrantReadWriteLock.WriteLock writerLock;

    public ReentrantReadWriteLock.ReadLock  readLock()  { return readerLock; }
    public ReentrantReadWriteLock.WriteLock writeLock() { return writerLock; }
	
}
```

```java=
ReentrantLock 實作 Lock
// RLock implements Lock
---------------------------
ReentrantReadWriteLock 實作 ReadWriteLock
// RRWLock implements RWLock
-----------------------------------------
ReentrantReadWriteLock.ReadLock 實作 Lock
ReentrantReadWriteLock.WriteLock 實作 Lock
// RRWLock.ReadLock implements Lock
// RRWLock.WriteLock implements Lock
```

```java=
ReentrantReadWriteLock.ReadLock(內部類別) rlc = new ReentrantReadWriteLock().readLock(); --> 回傳類別 (私有變數 readerLock)
ReentrantReadWriteLock.WriteLock(內部類別) wlc = new ReentrantReadWriteLock().writeLock(); --> 回傳類別 (私有變數 writerLock)

var rlc = new ReentrantReadWriteLock().readLock();
var wlc = new ReentrantReadWriteLock().writeLock();
rlc.lock(); / rlc.unlock(); / rlc.tryLock(); (從Lock繼承下來)
wlc.lock(); / wlc.unlock(); / wlc.tryLock(); (從Lock繼承下來)
```

```java=
--- ★★★ ------
要執行      new ReentrantReadWriteLock().readLock().lock()
必需未執行  new ReentrantReadWriteLock().writeLock().lock() 

要執行     new ReentrantReadWriteLock().writeLock().lock()
必需未執行 new ReentrantReadWriteLock().writeLock().lock()
必需未執行 new ReentrantReadWriteLock().readLock().lock()
```

### *- 使用 java.util.concurrent.locks.StampedLock《 p.11-42》*

```java=
package java.util.concurrent.locks;
public class StampedLock {

    // 取得「寫鎖」成功，回傳大於 0 的 Long整數 
    // 取得「寫鎖」失敗，代表有其他的「讀鎖、寫鎖」正在進行，線程進入 wait_blocked
    public long writeLock() {	}

    // 取得「悲觀讀鎖」成功，回傳大於 0 的 Long整數 
    // 取得「悲觀讀鎖」失敗，代表有其他的「寫鎖」正在進行，線程進入 wait_blocked	
    public long readLock()  {	}

    // 取得「樂觀讀鎖」成功，回傳大於 0 的 Long整數 
    // 取得「樂觀讀鎖」失敗，回傳 0，代表有其他的「寫鎖」正在進行，線程不會進入 wait_blocked
    // 不會真正執行上鎖，其他的「寫鎖」可以同時進行
    public long tryOptimisticRead() {	}

    // 釋放鎖需要一個戳記(Stamp)，這個 Stamp 必須是和「成功取鎖時所得到的 Stamp」相同
    public void unlockWrite(long stamp) {	}
    public void unlockRead(long stamp)  {	}

    // true: 其他線程沒有發生「寫鎖」，所取得的資料正確，未被修改
    // false: 其他線程發生「寫鎖」，資料可能被修改；本線程需要 lock.readLock()，等待「寫鎖」釋放，重新取得正確資料 
    public boolean validate(long stamp) {	}

    // 讀鎖和寫鎖相互轉換的功能
    public long tryConvertToOptimisticRead(long stamp) {   }
    public long tryConvertToReadLock(long stamp) {   }
    public long tryConvertToWriteLock(long stamp) {   }

    // 若使用 Thread.interrupt()，會造成 CPU飆升
    public long readLockInterruptibly() throws InterruptedException { }	
    public long writeLockInterruptibly() throws InterruptedException { }

}
```
- [StampedLock：比讀寫鎖更快的鎖](https://www.cnblogs.com/myworld7/p/12332911.html)

:::success
- 允許多個線程同時取得「讀鎖」(得到的Stamp不同)
- 同時只允許一個線程持有「寫鎖」
- 不允許兩個線程同時取得「寫鎖」及「悲觀讀鎖」
- 允許同時取得「(一個) 寫鎖」及「(多個) 樂觀讀鎖」
- tryOptimisticRead 取到 stamp>0 的樂觀讀鎖後，writeLock 仍然可以取到 stamp>0 的寫鎖
- stamp 需要宣告在各自的讀寫函式裡
:::


:::info
- 樂觀讀鎖的意思，就是指樂觀的估計讀鎖期間，大概不會有寫鎖發生。
- 悲觀讀鎖 則是讀鎖期間，拒絕寫鎖的發生，也就是寫鎖必須等待讀鎖完成。
::: 

:::success
- 無論寫鎖還是讀鎖，都不支援Conditon等待。
- StampedLock是不可重入的，即使當前線程持有了鎖，再次獲取鎖還是會返回一個新的數據戳，所以要注意鎖的釋放參數，使用不小心可能會導致死鎖。
:::

```java=
public static void main(String[] args) {		
    var aryList = new ArrayList4<String>();

    new Thread(() -> {
        while (true) {
            aryList.add("thrd1-obj");
        }
    }, "thrd1").start();

    new Thread(() -> {
        while (true) {
            aryList.add("thrd2-obj");
        }
    }, "thrd2").start();

    new Thread(() -> {
        while (true) {
            aryList.get(0);
        }
    }, "thrd3").start();

    new Thread(() -> {
        while (true) {
            aryList.size();
        }
    }, "thrd4").start();

}
```

```java=
public class ArrayList4<E> {

    private Object[] elems;
    private int next;
    StampedLock slc = new StampedLock();

    public ArrayList4() {
        this(5);
    }

    public ArrayList4(int capacity) {
        elems = new Object[capacity];
    }

    public void add(E o) {
        // 取得寫鎖後，其他線程不可取得「讀鎖 (slc.readLock)」並進入 wait_blocked
        // 取得寫鎖後，其他線程 tryOptimisticRead() 回傳 0，程式繼續執行
        long stamp = slc.writeLock();
        try {
            if (next == elems.length) {
                elems = java.util.Arrays.copyOf(elems, elems.length * 2);
            }
            elems[next++] = o;
        } finally {
            slc.unlockWrite(stamp);
        }
    }

    public E get(int idx) {

        // 樂觀讀鎖不會真正上鎖，其他線程的「寫鎖」可以同時進行
        // 所以 tryOptimisticRead() 不需要解除鎖定
        // 若 stamp=0，代表「寫鎖 (writeLock)」 正在進行，本線程不會阻塞，程式繼續往下跑
        long stamp = slc.tryOptimisticRead();

        E txtElem = (E) elems[idx];

        // slc.validate(stamp) =>
        // true: 其他線程沒有發生「寫鎖」，所取得的資料正確，未被修改
        // false: 其他線程發生「寫鎖」，資料可能被修改		
        // 所以等待「寫鎖」釋放後，目前線程執行「讀鎖 readLock」，重新取得正確資料
        // validate(stamp=0) => false		
        if (!slc.validate(stamp)) {
            stamp = slc.readLock();
            try {
                txtElem = (E) elems[idx];
            } finally {
                slc.unlockRead(stamp);
            }
        }

        return txtElem;
    }

    public int size() {
        long stamp = slc.tryOptimisticRead();
        int rtnNext = next;

        if (!slc.validate(stamp)) {
            stamp = slc.readLock();
            try {
                rtnNext = next;
            } finally {
                slc.unlockRead(stamp);
            }
        }

        return rtnNext;
    }
}
```

【測試程式】

:::spoiler
```java=
public static void main(String[] args) {	
 
    var aryList = new ArrayList4<String>();

    new Thread(() -> {
        while (true) {
            aryList.add("thrd1");
        }
    }, "thrd1").start();

    new Thread(() -> {
        while (true) {
            aryList.add("thrd2");
        }
    }, "thrd2").start();

    new Thread(() -> {
        while (true) {
            aryList.get(0);
        }
    }, "thrd3").start();

    new Thread(() -> {
        while (true) {
            aryList.size();
        }
    }, "thrd4").start();
}
```

```java=
public class ArrayList4<E> {

    private Object[] elems;
    private int next;

    StampedLock slc = new StampedLock();

    public ArrayList4() {
        this(5);
    }

    public ArrayList4(int capacity) {
        elems = new Object[capacity];
    }

    public void add(E o) {

        String thrdName = Thread.currentThread().getName();

        out.printf("%s/add()-> line1 未執行 writeLock %n", thrdName);
        long stamp = slc.writeLock();
        out.printf("%s/add()-> line2 已執行 writeLock，stamp 為 %d %n", thrdName, stamp);

        try {

            if (next == elems.length) {
                elems = java.util.Arrays.copyOf(elems, elems.length * 2);
                out.printf("%s/add()-> Arrays.copyOf (%s)，stamp 為 %d %n", thrdName, elems.length, stamp);
            }

            String aryTxt = String.format("%s-idx%d-len%d", o, next, elems.length);
            elems[next++] = aryTxt;
            out.printf("%s/add()-> line3 in writeLock，elems[next++]=%s，stamp 為%d %n", thrdName, aryTxt, stamp);

        } finally {
            out.printf("%s/add()-> line4 未執行 unlockWrite，stamp 為%d %n", thrdName, stamp);
            slc.unlockWrite(stamp);
            out.printf("%s/add()-> line5 已執行 unlockWrite，stamp 為%d %n", thrdName, stamp);
        }

    }

    public E get(int idx) {

        long stamp;
        E txtElem;

        out.println("get()-> line1 未執行 tryOptimisticRead");
        stamp = slc.tryOptimisticRead();
        out.printf("get()-> line2 已執行 tryOptimisticRead，stamp 為 %d %n", stamp);

        txtElem = (E) elems[idx];
        out.printf("get()-> line3 取得 txtElem on tryOptimisticRead，stamp 為 %d %n", stamp);

        if (!slc.validate(stamp)) {
            out.printf("get()-> line4 slc.validate(stamp) is false，stamp 為 %d %n", stamp);

            out.println("get()-> line5 未執行 readLock");
            stamp = slc.readLock();
            out.printf("get()-> line6 已執行 readLock，stamp 為 %d %n", stamp);

            try {
                txtElem = (E) elems[idx];
                out.printf("get()-> line7 取得 txtElem on readLock，stamp 為 %d %n", stamp);
            } finally {
                out.printf("get()-> line8 未執行 unlockRead，stamp 為 %d %n", stamp);
                slc.unlockRead(stamp);
                out.printf("get()-> line9 已執行 unlockRead，stamp 為 %d %n", stamp);
            }
        }

        out.printf("get()-> line10 elems[%d] = %s，stamp 為 %d %n", idx, txtElem, stamp);
        return txtElem;
    }

    public int size() {

        long stamp;
        int rtnNext;

        out.println("size()-> line1 未執行 tryOptimisticRead");
        stamp = slc.tryOptimisticRead();
        out.printf("size()-> line2 已執行 tryOptimisticRead，stamp 為 %d %n", stamp);

        rtnNext = next;
        out.printf("size()-> line3 取得 next on tryOptimisticRead，stamp 為 %d %n", stamp);

        if (!slc.validate(stamp)) {
            out.printf("size()-> line4 slc.validate(stamp) is false，stamp 為 %d %n", stamp);

            out.println("size()-> line5 未執行readLock");
            stamp = slc.readLock();
            out.printf("size()-> line6 已執行 readLock，stamp 為 %d %n", stamp);

            try {
                rtnNext = next;
                out.printf("size()-> line7 取得 next on readLock，stamp 為 %d %n", stamp);
            } finally {
                out.printf("size()-> line8 未執行 unlockRead，stamp 為 %d %n", stamp);
                slc.unlockRead(stamp);
                out.printf("size()-> line9 已執行 unlockRead，stamp 為 %d %n", stamp);
            }
        }

        out.printf("size()-> line10 size()=%d，stamp 為 %d %n", rtnNext, stamp);
        return rtnNext;
    }
}
```
:::

### *- 使用 java.util.concurrent.locks.Condition《 p.11-44》*

```java=
package java.util.concurrent.locks;
public interface Lock {
    void lock();
    void unlock();
    boolean tryLock();
    Condition newCondition();
}

public interface Condition {
    void await() throws InterruptedException;	// 會釋放鎖，並進行 wait_blocked
    void signal();
    void signalAll();
}

package java.lang;
public class Object {
    public final void wait() throws InterruptedException { wait(0L); }
    public final native void wait(long timeoutMills) throws InterruptedException;
    public final native void notify()
    public final native void notifyAll();
}
```

:::success
- ReentrantLock (實作Lock) 搭配 Condition (await、signal、signalAll)，用來改善 Synchronized 搭配 wait()、notify()、notifyAll()
- ReentrantLock (實作Lock) 基本上效果跟 Synchronized 一樣，再提供更多彈性的用法
- Synchronized 利用 java.lang.Object (類別) 鎖定，並使用 Object.wait()、Object.notify()、Object.notifyAll() 來操作
- 利用 ReentrantLock 鎖定，並使用 ReentrantLock.Condition.await()、ReentrantLock.Condition.signal()、ReentrantLock.Condition.signalAll() 來操作
:::


### 11.2.2 使用 Executor《p.11-47》

```java=
public static void main(String[] args) {

    String[] urls = { "http://urlA", "http://urlB", "http://urlC" };
    String[] files = { "Encoding.html", "Encoding2.html", "Encoding3.html" };

    for (int i = 0; i < 3; i++) {
        var url = urls[i];
        var file = files[i];

        // ======= 語法1 ========
        var myRunb = new myRunnable(url, file); // 執行計劃
        new myExecutor().execute(myRunb); // 開始執行

        // ======= 語法2 ========
        var myRunb2 = new Runnable() {
            public void run() {
                out.printf("要從網址 %s 下載檔案 %s %n", url, file);
            }
        };
        new myExecutor().execute(myRunb2);

        // ======= 語法3 ========
        Runnable myRunb3 = () -> out.printf("要從網址 %s 下載檔案 %s %n", url, file);
        new myExecutor().execute(myRunb3);

        // ======= 語法4 ========
        Runnable myRunb4 = () -> out.printf("要從網址 %s 下載檔案 %s %n", url, file);
        var myExer = new Executor() {
            public void execute(Runnable r) {
                r.run();
            }
        };
        myExer.execute(myRunb4);

        // ======= 語法5 ========
        Runnable myRunb5 = () -> out.printf("要從網址 %s 下載檔案 %s %n", url, file);
        // ((Executor) (r -> r.run())).execute(myRunb5);
        Executor myExer2 = r -> r.run();
        // Executor myExer2 = r -> new Thread(r).start();
        myExer2.execute(myRunb5);

        // ======= 語法6 ========
        Runnable myRunb6 = () -> out.printf("要從網址 %s 下載檔案 %s %n", url, file);
        Actuator myActuator = r -> r.run();
        myActuator.go(myRunb6);

        // ======= 語法7(多執行緒的跑法) ========
        Runnable myRunb7 = () -> out.printf("要從網址 %s 下載檔案 %s %n", url, file);
        // Executors.newCachedThreadPool() 產生 ThreadPoolExecutor 實例，轉型並回傳 ExecutorService
        // ExecutorService 繼承父介面 Executor，故也有 execute(Runnable r) 方法簽章
        ExecutorService myES = Executors.newCachedThreadPool();
        // ★ myES 不用自已實作 execute(), 因為 ThreadPoolExecutor(ExecutorService) 已有實作
        myES.execute(myRunb7);
        myES.shutdown();
        // List<Runnable> runbs = myES.shutdownNow();
    }
}

interface Actuator {
    void go(Runnable r);
}

class myExecutor implements Executor {
    public void execute(Runnable r) {
        r.run();
    }
}

class myRunnable implements Runnable {
    String url;
    String file;

    myRunnable(String url, String file) {
        this.url = url;
        this.file = file;
    }

    public void run() {
        out.printf("要從網址 %s 下載檔案 %s %n", url, file);
    }
}

```

### *- java.util.concurrent.FutureTask《 p.11-52》*

```java=

public interface Callable<V> {
    V call() throws Exception;
}

public interface Runnable {
    public abstract void run();	
}

public interface Future<V> {
    boolean cancel(boolean mayInterruptIfRunning);
    boolean isCancelled();
    boolean isDone();
    V get() throws InterruptedException, ExecutionException;
    V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException;
}

public interface RunnableFuture<V> extends Runnable, Future<V> {
    void run();	// 跟 Runnable.run() 一樣
}

// 等同實作 Runnable、Future<V> 
public class FutureTask<V> implements RunnableFuture<V> {
	
    private volatile int state;
    private static final int NEW          = 0;
    private static final int COMPLETING   = 1;
    private static final int NORMAL       = 2;
    private static final int EXCEPTIONAL  = 3;
    private static final int CANCELLED    = 4;
    private static final int INTERRUPTING = 5;
    private static final int INTERRUPTED  = 6;	
	
    public FutureTask(Runnable runnable, V result) {
        this.callable = Executors.callable(runnable, result);
        this.state = NEW;
    }
	
    public FutureTask(Callable<V> callable) {
        if(callable == null) throws new NullPointerException();
        this.callable = callable;
        this.state = NEW;
    }
	
    public void run() {
        .....
        Callable<V> c = callable;
        V result = c.call();
        set(result);
        .....
    }
	
    protected void set(V v) {
        .....
        outcome = v;        
        .....
    }
	
    public boolean cancel(boolean mayInterruptIfRunning) {
        .....
        t.interrupt();		
        .....
    }
		
    public boolean isCancelled() {
        return state >= CANCELLED;
    }

    public boolean isDone() {
        return state != NEW;
    }
	
    // ★ 由 run() 執行，再由 get() 取結果
    public V get() throws InterruptedException, ExecutionException {
        int s = state;
        if (s <= COMPLETING) s = awaitDone(false, 0L);
        return report(s);
    }
	
    // ★ 由 run() 執行，再由 get() 取結果
    public V get(long timeout, TimeUnit unit) throws InterruptedException, ExecutionException, TimeoutException {
        if (unit == null) throw new NullPointerException();
        int s = state;
        if (s <= COMPLETING && (s = awaitDone(true, unit.toNanos(timeout))) <= COMPLETING) throw new TimeoutException();
        return report(s);
    }
	
    private V report(int s) throws ExecutionException {
        Object x = outcome;
        if (s == NORMAL) return (V)x;
        if (s >= CANCELLED) throw new CancellationException();
        throw new ExecutionException((Throwable)x);
    }
}

```

```java=
static long Fibonacci(long n) {
    if (n <= 1) return n;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}

public static void main(String[] args) throws Exception {

    // 作法一
    // FutureTask implements RunnableFuture extends Runnable, Future<V>
    // 執行新線程
    FutureTask<Long> The30FibFuture = new FutureTask<>(() -> Fibonacci(30));
    new Thread(The30FibFuture).start();

    // 作法二
    // newCachedThreadPool() 從線程池取得某條線程執行
    ExecutorService service = Executors.newCachedThreadPool();
    Future<Long> The30FibFuture = service.submit(() -> Fibonacci(45));
    // submit() =>
    // (1) ftask = new FutureTask<T>(Callable);
    // (2) service.execute(ftask) -> ftask.run() -> Callable.call()
    // (3) return ftask;
    service.shutdown(); // 在指定執行的 Runnable 完成後，關閉 ExecutorService

    while (!The30FibFuture.isDone()) {
        out.println("請做別的事~");
    }
    out.printf("第30個費式數:%d %n", The30FibFuture.get());
}

```

### *- Executor、FutureTask 的用法整理*

```java=
/*
 * 記憶：Executor.execute(Runnable)
 */

Runnable myRunb = () -> out.printf("要從網址 %s 下載檔案 %s %n", url, file);
Executor myExecutor = r -> r.run();
// Executor myExecutor = r -> new Thread(r).start();
myExecutor.execute(myRunb);
// ((Executor) (r -> r.run())).execute(myRunb);

// -> new Executor().execute(new Runnable())
// --> Runnable.run()
// --> new Thread(Runnable).start()

---------------------------------------------------------------------------------------------
/*
 * Executors.newCachedThreadPool() 產生 ExecutorService/ThreadPoolExecutor 型態的實例
 * ExecutorService.execute(Runnable)
 * ExecutorService.execute() 已內建實作，不需要自已實作
 * 記憶：ThreadPoolExecutor.execute(Runnable):void
 */

Runnable myRunb = () -> out.printf("要從網址 %s 下載檔案 %s %n", url, file);
// Executors.newCachedThreadPool() 產生 ThreadPoolExecutor 實例，轉型並回傳 ExecutorService
ExecutorService myES = Executors.newCachedThreadPool();
// ExecutorService 繼承父介面 Executor，故也有 execute(Runnable r) 方法簽章
// ★ myES 不用自已實作 execute(), 因為 ThreadPoolExecutor(ExecutorService) 已有實作
myES.execute(myRunb);
myES.shutdown();
// List<Runnable> runbs = myES.shutdownNow();

---------------------------------------------------------------------------------------------
/* 
 * Executors.newCachedThreadPool() 產生 ExecutorService(ThreadPoolExecutor) 型態的實例
 * ExecutorService.submit(Callable) 產生 Future/FutureTask 的實例
 * 透過 Future.get() 取得 Callable() 的結果
 * 記憶：ThreadPoolExecutor.submit(Callable<V>):FutureTask<V>
 */

// Thread Pool 發現有執行緒執行完成後，會將資源回收重新的再使用。如果沒有回收的資源，那就會再去建立執行緒
ExecutorService service = Executors.newCachedThreadPool();
// () -> Fibonacci(45) 為 Callable<Long> 的匿名類別
Future<Long> The30FibFuture = service.submit(() -> Fibonacci(45));
// submit() =>
// (1) ftask = new FutureTask<T>(Callable);
// (2) service.execute(ftask) -> [new] Thread(ftask).run() 
//     -> ftask.run() -> ftask.result = Callable.call() / ftask.get() return ftask.result
// (3) return ftask;
service.shutdown(); // 在指定執行的 Runnable 完成後，關閉 ExecutorService
if(The30FibFuture.isDone()) out.printf("第30個費式數:%d %n", The30FibFuture.get());

---------------------------------------------------------------------------------------------
/* 
 * Thread.start() >> Thread.run() >> FutureTask.run() >> Callable<V>.call():V
 * FutureTask.get() 取得 call()的回傳值
 * 記憶：Thread(FutureTask<V>(Callable<V>)).start()
 */

// () -> Fibonacci(30) 為 Callable<Long> 的匿名類別
FutureTask<Long> The30FibFuture = new FutureTask<>(() -> Fibonacci(30));
new Thread(The30FibFuture).start();
if(The30FibFuture.isDone()) out.printf("第30個費式數:%d %n", The30FibFuture.get());
// -> new Thread(new FutureTask(new Callable())).start()
// --> Thread.run()
// ---> FutureTask.run()
// ----> Callable.call()

---------------------------------------------------------------------------------------------
ThreadPoolExecutor.execute(Runnable) => new Runnable(){}.run() 沒有回傳值
ThreadPoolExecutor.execute(FutureTask) => new FutureTask().run() >> new Callable(){}.call() 有回傳值，並存在 FutureTask 實例中，透過 FutureTask.get() 取得
```

### *- ExecutorService.invokeAll()、ExecutorService.invokeAny()《 p.11-55》*

```java=
static long Fibonacci(long n) {
    if (n <= 1)
        return n;
    return Fibonacci(n - 1) + Fibonacci(n - 2);
}

static long Multiply2(long m) {
    return m * 2;
}

public static void main(String[] args) throws Exception {
    
    ArrayList<Callable<Long>> aList = new ArrayList<>();
    aList.add(() -> Fibonacci(30));
    aList.add(() -> Multiply2(5));

    // ===================================================
    ExecutorService service = Executors.newCachedThreadPool();
    // invokeAll():等所有 Callable 都執行完成後，才會往下執行
    List<Future<Long>> futures = service.invokeAll(aList);
    service.shutdown();

    for (Future<Long> future : futures) {
        out.println(future.get());
    }

    // ===================================================
    ExecutorService service2 = Executors.newCachedThreadPool();
    Long futrueValue = service2.invokeAny(aList);
    service2.shutdown();
    // 因為 Multiply2(5) 比較早完成，所以結果為 10
    out.println(futrueValue);
}
```

### *- 使用 ScheduledThreadPoolExecutor《 p.11-55》*

:::info
* 快取執行緒池 newCachedThreadPool(): ExecutorService (ThreadPoolExecutor) / 每個執行緒預設可idle的時間為60秒
* 固定執行緒池 newFixedThreadPool(): ExecutorService (ThreadPoolExecutor)
* 排程執行緒池 newScheduledThreadPool(): ScheduledExecutorService (ScheduledThreadPoolExecutor)
* 單一執行緒循序進行任務 newSingleThreadExecutor(): ExecutorService (ThreadPoolExecutor)
* 單一執行緒排程進行任務 newSingleThreadScheduledExecutor(): ScheduledExecutorService (ScheduledThreadPoolExecutor)
:::

```java=
execute():void
submit(Callable<V> c):Future<V>
submit(Runnable r):Future<?>
schedule(Callable<V> c):ScheduledFuture<V>
schedule(Runnable r):ScheduledFuture<?>
scheduleWithFixedDalay():ScheduledFuture<?>
scheduleAtFixedRate():ScheduledFuture<?>
```

```java=
① (itf)ScheduledFuture<V> extends (itf)Delayed, (itf)Future<V>
② (itf)RunnableFuture<V> extends (itf)Runnable, (itf)Future<V>
③ (itf)RunnableScheduledFuture<V> extends RunnableFuture<V>, ScheduledFuture<V>
※ RunnableScheduledFuture<V> extends Dalayed,Runnable,Future<V>

(cls)FutureTask<V> implements (itf)RunnableFuture<V> --> ②
(cls)ScheduledFutureTask<V> implements (itf)RunnableScheduledFuture<V> --> ③
(cls)ScheduledFutureTask<V> extends (cls)FutureTask<V>
(itf)RunnableScheduledFuture<V> extends (itf)RunnableFuture<V>,  ScheduledFuture<V>

(cls)ThreadPoolExecutor is-a (itf)ExecutorService
(cls)ScheduledThreadPoolExecutor is-a (itf)ScheduledExecutorService
(cls)ScheduledThreadPoolExecutor extends (cls)ThreadPoolExecutor
(itf)ScheduledExecutorService extends (itf)ExecutorService
```
- [電腦的核心 (Core) 和執行緒 (Thread)](https://jenifers001d.github.io/2020/08/04/%E9%9B%BB%E8%85%A6%E7%9F%A5%E8%AD%98/what-is-core-and-thread/)

:::info
1. 雖然三個 Thread 像是同時在作業，但實際上是一個 Core 非常快速的切換不同 Runnable 在執行，讓表面看起來，像是三個 Thread 同時作業
1. Runnable 執行時，若發生需要等待的情況，則 Core 會先執行別項 Runnable；表面看起來像一個 Thread 在等待，另外兩個 Thread 在執行
1. 所有 Runnable 啟動後，所有 Runnable 立即開始首次延遲數秒；後續再以 Thread 的數量，來執行對應數量的 Runnable
1. Thread 要等目前 Runnable 執行完成後，才會開始執行另一個 Runnable
1. 執行 Runnable，從開始至結束，都是由同一個 Thread 來完成
1. 以排程的方式來重覆執行 Runnable，第1次是以 Thread-A來執行，第2次就有可能以 Thread-B 來執行，看當下哪一個 Thread 閒置。
:::

```java=
static SimpleDateFormat sdf = new SimpleDateFormat("mm:ss");
```

#### (1) newFixedThreadPool_submit_Loop(int thrdCnt)
:::spoiler

```java=
// ==========================================
// 【固定數量執行緒池】
// 建立執行緒池，當中包括5個可重複使用的執行緒
// 如果所需的執行緒數量超過5個，那多出的任務就會被放入 Queue 等待被執行

// newFixedThreadPool() 建立 ThreadPoolExecutor 實例，並轉型為 ExecutorService
ExecutorService service = Executors.newFixedThreadPool(5);
for (int i = 0; i < 10; i++) {
    final int count = i;

    Runnable runb = new Runnable() {
        public void run() {
            out.printf("%d-%s %n", count, new java.util.Date());
            try {
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }
    };

    // submit(Runnable) 與 submit(Callable) 的實作相同，只是差異有沒有回傳值
    // RunnableFuture<Void> ftask = newTaskFor(task, null);
    // execute(ftask);
    // return ftask;
    service.submit(runb);
    //Future<?> future = service.submit(runb);
    //service.execute(runb);
}
service.shutdown(); // 關閉 Thread Pool

// 1-Mon Jul 17 17:27:43 CST 2023
// 4-Mon Jul 17 17:27:43 CST 2023
// 3-Mon Jul 17 17:27:43 CST 2023
// 2-Mon Jul 17 17:27:43 CST 2023
// 0-Mon Jul 17 17:27:43 CST 2023
// ---- 因 Thread Pool 宣告5個固定執行緒，故0~4先執行
// ---- Thread.sleep(2000)，0~4完成後，再執行 5~9
// 5-Mon Jul 17 17:27:45 CST 2023
// 6-Mon Jul 17 17:27:45 CST 2023
// 7-Mon Jul 17 17:27:45 CST 2023
// 8-Mon Jul 17 17:27:45 CST 2023
// 9-Mon Jul 17 17:27:45 CST 2023
```
:::

#### (2) newScheduledThreadPool_scheduleWithFixedDelay_Loop(int thrdCnt)
:::spoiler
```java=
static void newScheduledThreadPool_scheduleWithFixedDelay_Loop(int thrdCnt) {

    ScheduledExecutorService service = Executors.newScheduledThreadPool(thrdCnt);
	
    for (int i = 1; i <= 5; i++) {
        final int j = i;

        out.printf("Runb(%d)-ThrdNm(%s)-Time(%s)-Runnable(%d) 啟動 %n", j, Thread.currentThread().getName(),
                sdf.format(new java.util.Date()), j);

        service.scheduleWithFixedDelay(() -> {
            try {
                out.printf("Runb(%d)-ThrdNm(%s)-Time(%s) %n", j, Thread.currentThread().getName(),
                        sdf.format(new java.util.Date()));
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, 1, 2, TimeUnit.SECONDS);

        // 1 => 延遲1秒後，執行首次 Runnable
        // 2 => 經過首次 Runnable 後，之後每次先延遲2秒，再執行 Runnable
    }

    // 因 Runnable 會持續執行，不能關閉 ScheduledExecutorService (ScheduledThreadPoolExecutor)
    // service.shutdown();

    /*
        Runb(1)-ThrdNm(main)-Time(37:46)-Runnable(1) 啟動 
        Runb(2)-ThrdNm(main)-Time(37:46)-Runnable(2) 啟動 
        Runb(3)-ThrdNm(main)-Time(37:46)-Runnable(3) 啟動 
        Runb(4)-ThrdNm(main)-Time(37:46)-Runnable(4) 啟動 
        Runb(5)-ThrdNm(main)-Time(37:46)-Runnable(5) 啟動

        ------------------------------------------------------------------------------ 
        Runb(1)-ThrdNm(pool-1-thread-1)-Time(37:47) >> 完成時間 37:50 >> 下次執行時間 37:52
        Runb(2)-ThrdNm(pool-1-thread-2)-Time(37:47) >> 完成時間 37:50 >> 下次執行時間 37:52
        Runb(3)-ThrdNm(pool-1-thread-3)-Time(37:47) >> 完成時間 37:50 >> 下次執行時間 37:52

        ------------------------------------------------------------------------------ 
        Runb(4)-ThrdNm(pool-1-thread-1)-Time(37:50) >> 完成時間 37:53 >> 下次執行時間 37:55
        Runb(5)-ThrdNm(pool-1-thread-3)-Time(37:50) >> 完成時間 37:53 >> 下次執行時間 37:55
        ※ Runb(4、5) 預計 37:47 執行(首次延遲1秒)，因 thread(1~3) 正在跑 Runb(1~3)，故 37:50 才執行
        ※ pool-1-thread-2 37:50 開始閒置

        ------------------------------------------------------------------------------ 
        Runb(1)-ThrdNm(pool-1-thread-2)-Time(37:52) >> 完成時間 37:55 >> 下次執行時間 37:57
        Runb(3)-ThrdNm(pool-1-thread-3)-Time(37:53) >> 完成時間 37:56 >> 下次執行時間 37:58
        Runb(2)-ThrdNm(pool-1-thread-1)-Time(37:53) >> 完成時間 37:56 >> 下次執行時間 37:58
        ※ Runb(1) 預計 37:52 執行，因 thread-2 正在閒置，故執行之
        ※ Runb(2、3) 預計 37:52 執行，因 thread(1、3) 正在跑 Runb(4、5)，故 37:53 才執行

        ------------------------------------------------------------------------------ 
        Runb(4)-ThrdNm(pool-1-thread-2)-Time(37:55) >> 完成時間 37:58 >> 下次執行時間 38:00
        Runb(5)-ThrdNm(pool-1-thread-3)-Time(37:56) >> 完成時間 37:59 >> 下次執行時間 38:01
        ※ Runb(4) 預計 37:55 執行，因 thread-2 剛好執行完 Runb(1)，故執行之
        ※ Runb(5) 預計 37:55 執行，因 thread-3 正在跑 Runb(3)，故 37:56 才執行
        ※ pool-1-thread-1 37:56 開始閒置

        ------------------------------------------------------------------------------  
        Runb(1)-ThrdNm(pool-1-thread-1)-Time(37:57) >> 完成時間 38:00 >> 下次執行時間 38:02
        Runb(3)-ThrdNm(pool-1-thread-2)-Time(37:58) >> 完成時間 38:01 >> 下次執行時間 38:03
        Runb(2)-ThrdNm(pool-1-thread-3)-Time(37:59) >> 完成時間 38:02 >> 下次執行時間 38:04
        ※ Runb(1) 預計 37:57 執行，因 thread-1 正在閒置，故執行之
        ※ Runb(3) 預計 37:58 執行，因 thread-2 剛好執行完 Runb(4)，故執行之
        ※ Runb(2) 預計 37:58 執行，因 thread-3 正在跑 Runb(5)，故 37:59 才執行

        ------------------------------------------------------------------------------ 
        Runb(4)-ThrdNm(pool-1-thread-1)-Time(38:00) >> 完成時間 38:03 >> 下次執行時間 38:05
        Runb(5)-ThrdNm(pool-1-thread-2)-Time(38:01) >> 完成時間 38:04 >> 下次執行時間 38:06
        ※ Runb(4) 預計 38:00 執行，因 thread-1 剛好執行完 Runb(1)，故執行之
        ※ Runb(5) 預計 38:01 執行，因 thread-2 剛好執行完 Runb(3)，故執行之
        ※ pool-1-thread-3 38:02 開始閒置
    */
}
```
:::

#### (3) newScheduledThreadPool_scheduleAtFixedRate(int thrdCnt)
:::spoiler

```java=
static void newScheduledThreadPool_scheduleAtFixedRate(int thrdCnt) {
	
    ScheduledExecutorService service = Executors.newScheduledThreadPool(thrdCnt);

    out.printf("Runb(%d)-ThrdNm(%s)-Time(%s)-Runnable 啟動 %n", 0, Thread.currentThread().getName(),
            sdf.format(new java.util.Date()));

    service.scheduleAtFixedRate(() -> {
        try {
            out.printf("Runb(%d)-ThrdNm(%s)-Time(%s) %n", 0, Thread.currentThread().getName(),
                    sdf.format(new java.util.Date()));
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }, 1, 2, TimeUnit.SECONDS);

    // 1 => 延遲1秒後，執行首次 Runnable
    // 2 => 預設 Runnable 會執行2秒，因睡3秒超過2秒，故馬上執行下次 Runnable
    // 4 => 預設 Runnable 會執行4秒，因睡3秒未超過4秒，故需等滿4秒後，才執行下次 Runnable

    // 因 Runnable 會持續執行，不能關閉 ScheduledExecutorService (ScheduledThreadPoolExecutor)
    // service.shutdown();

    /* 執行結果
    Runb(0)-ThrdNm(main)-Time(17:06)-Runnable 啟動 
    Runb(0)-ThrdNm(pool-1-thread-1)-Time(17:07) 
    Runb(0)-ThrdNm(pool-1-thread-1)-Time(17:10) 
    Runb(0)-ThrdNm(pool-1-thread-2)-Time(17:13) 
    Runb(0)-ThrdNm(pool-1-thread-2)-Time(17:16) 
    Runb(0)-ThrdNm(pool-1-thread-2)-Time(17:19)	
    每次執行 Runnable時，系統會從 ThreadPool 取得任一個 Thread 來執行
    從結果看到有使用到 thread-1、thread-2
    */	
}
```
:::

#### (4) newScheduledThreadPool_scheduleAtFixedRate_Loop(int thrdCnt)
:::spoiler

```java=
static void newScheduledThreadPool_scheduleAtFixedRate_Loop(int thrdCnt) {

    ScheduledExecutorService service = Executors.newScheduledThreadPool(thrdCnt);

    for (int i = 1; i <= 5; i++) {
        final int j = i;

        out.printf("Runb(%d)-ThrdNm(%s)-Time(%s)-Runnable(%d) 啟動 %n", j, Thread.currentThread().getName(),
                sdf.format(new java.util.Date()), j);

        service.scheduleAtFixedRate(() -> {
            try {
                out.printf("Runb(%d)-ThrdNm(%s)-Time(%s) %n", j, Thread.currentThread().getName(),
                        sdf.format(new java.util.Date()));
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, 1, 2, TimeUnit.SECONDS);

        // 1 => 延遲1秒後，執行首次 Runnable
        // 2 => 預設 Runnable 會執行2秒，因睡3秒超過2秒，故馬上執行下次 Runnable
        // 4 => 預設 Runnable 會執行4秒，因睡3秒未超過4秒，故需等滿4秒後，才執行下次 Runnable
    }

    // 因 Runnable 會持續執行，不能關閉 ScheduledExecutorService (ScheduledThreadPoolExecutor)
    // service.shutdown();

    /*
    Runb(1)-ThrdNm(main)-Time(26:41)-Runnable(1) 啟動 
    Runb(2)-ThrdNm(main)-Time(26:41)-Runnable(2) 啟動 
    Runb(3)-ThrdNm(main)-Time(26:41)-Runnable(3) 啟動 
    Runb(4)-ThrdNm(main)-Time(26:41)-Runnable(4) 啟動 
    Runb(5)-ThrdNm(main)-Time(26:41)-Runnable(5) 啟動 

    =================================================================================== 
    Runb(2)-ThrdNm(pool-1-thread-2)-Time(26:42) >> 完成時間 26:45 >> 下次執行時間 26:45
    Runb(3)-ThrdNm(pool-1-thread-3)-Time(26:42) >> 完成時間 26:45 >> 下次執行時間 26:45
    Runb(1)-ThrdNm(pool-1-thread-1)-Time(26:42) >> 完成時間 26:45 >> 下次執行時間 26:45

    =================================================================================== 
    -- Runb(4~5) 應該在 26:42 執行，但因執行緒正在跑 Runb(1~3)，故等到 26:45 執行
    -- Runb(1) 上次執行完成後，馬上就執行下次 Runb(1)
    Runb(4)-ThrdNm(pool-1-thread-2)-Time(26:45) >> 完成時間 26:48 >> 下次執行時間 26:48
    Runb(5)-ThrdNm(pool-1-thread-3)-Time(26:45) >> 完成時間 26:48 >> 下次執行時間 26:48
    Runb(1)-ThrdNm(pool-1-thread-1)-Time(26:45) >> 完成時間 26:48 >> 下次執行時間 26:48

    =================================================================================== 
    -- Runb(2、3) 應該在 26:45 執行，但因執行緒正在跑 Runb(1、4、5)，故等到 26:48 執行
    -- Runb(4) 上次執行完成後，馬上就執行下次 Runb(4)
    Runb(2)-ThrdNm(pool-1-thread-2)-Time(26:48) >> 完成時間 26:51 >> 下次執行時間 26:51
    Runb(4)-ThrdNm(pool-1-thread-1)-Time(26:48) >> 完成時間 26:51 >> 下次執行時間 26:51
    Runb(3)-ThrdNm(pool-1-thread-3)-Time(26:48) >> 完成時間 26:51 >> 下次執行時間 26:51

    =================================================================================== 
    -- Runb(5、1) 應該在 26:48 執行，但因執行緒正在跑 Runb(2、3、4)，故等到 26:51 執行
    -- Runb(2) 上次執行完成後，馬上就執行下次 Runb(2)
    Runb(5)-ThrdNm(pool-1-thread-2)-Time(26:51) >> 完成時間 26:54 >> 下次執行時間 26:54
    Runb(1)-ThrdNm(pool-1-thread-1)-Time(26:51) >> 完成時間 26:54 >> 下次執行時間 26:54
    Runb(2)-ThrdNm(pool-1-thread-3)-Time(26:51) >> 完成時間 26:54 >> 下次執行時間 26:54
    */
}

```
:::

#### (5) newSingleThreadScheduledExecutor_schedule_Loop()
:::spoiler

```java=
static void newSingleThreadScheduledExecutor_schedule_Loop() {

    ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

    for (int i = 1; i <= 5; i++) {
        final int j = i;

        out.printf("Runb(%d)-ThrdNm(%s)-Time(%s)-Runnable(%d) 啟動 %n", j, Thread.currentThread().getName(),
                sdf.format(new java.util.Date()), j);

        scheduler.schedule(() -> {
            try {
                out.printf("Runb(%d)-ThrdNm(%s)-Time(%s) %n", j, Thread.currentThread().getName(),
                        sdf.format(new java.util.Date()));
                Thread.sleep(2000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, 3, TimeUnit.SECONDS);

    }		

    scheduler.shutdown();	// 等 Runnable 執行完成，才會關閉 scheduler
    //scheduler.shutdownNow();	// 立即關閉  scheduler

    /*
    // scheduler.shutdown();	// 等 Runnable 執行完成，才會關閉 scheduler
    Runb(1)-ThrdNm(main)-Time(49:10)-Runnable(1) 啟動 
    Runb(2)-ThrdNm(main)-Time(49:10)-Runnable(2) 啟動 
    Runb(3)-ThrdNm(main)-Time(49:10)-Runnable(3) 啟動 
    Runb(4)-ThrdNm(main)-Time(49:10)-Runnable(4) 啟動 
    Runb(5)-ThrdNm(main)-Time(49:10)-Runnable(5) 啟動 
    Runb(1)-ThrdNm(pool-1-thread-1)-Time(49:13) 
    Runb(2)-ThrdNm(pool-1-thread-1)-Time(49:15) 
    Runb(3)-ThrdNm(pool-1-thread-1)-Time(49:17) 
    Runb(4)-ThrdNm(pool-1-thread-1)-Time(49:19) 
    Runb(5)-ThrdNm(pool-1-thread-1)-Time(49:21) 
    */

    /*
    //scheduler.shutdownNow();	// 立即關閉  scheduler
    Runb(1)-ThrdNm(main)-Time(52:25)-Runnable(1) 啟動 
    Runb(2)-ThrdNm(main)-Time(52:25)-Runnable(2) 啟動 
    Runb(3)-ThrdNm(main)-Time(52:25)-Runnable(3) 啟動 
    Runb(4)-ThrdNm(main)-Time(52:25)-Runnable(4) 啟動 
    Runb(5)-ThrdNm(main)-Time(52:25)-Runnable(5) 啟動 
    */
}
```
:::

#### (6) newSingleThreadScheduledExecutor_scheduleAtFixedRate_Loop()
:::spoiler

```java=
static void newSingleThreadScheduledExecutor_scheduleAtFixedRate_Loop() {

    ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();

    for (int i = 1; i <= 5; i++) {
        final int j = i;

        out.printf("Runb(%d)-ThrdNm(%s)-Time(%s)-Runnable(%d) 啟動 %n", j, Thread.currentThread().getName(),
                sdf.format(new java.util.Date()), j);

        scheduler.scheduleAtFixedRate(() -> {
            try {
                out.printf("Runb(%d)-ThrdNm(%s)-Time(%s) %n", j, Thread.currentThread().getName(),
                        sdf.format(new java.util.Date()));
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, 1, 2, TimeUnit.SECONDS);
    }

    // scheduler.shutdown(); 會立即關閉 scheduler ?? 沒等 Runnable 執行完成
    // scheduler.shutdownNow();

    /*
        Runb(1)-ThrdNm(main)-Time(12:32)-Runnable(1) 啟動 
        Runb(2)-ThrdNm(main)-Time(12:32)-Runnable(2) 啟動 
        Runb(3)-ThrdNm(main)-Time(12:32)-Runnable(3) 啟動 
        Runb(4)-ThrdNm(main)-Time(12:32)-Runnable(4) 啟動 
        Runb(5)-ThrdNm(main)-Time(12:32)-Runnable(5) 啟動 
        Runb(1)-ThrdNm(pool-1-thread-1)-Time(12:33) 
        Runb(2)-ThrdNm(pool-1-thread-1)-Time(12:36) 
        Runb(3)-ThrdNm(pool-1-thread-1)-Time(12:39) 
        Runb(4)-ThrdNm(pool-1-thread-1)-Time(12:42) 
        Runb(5)-ThrdNm(pool-1-thread-1)-Time(12:45) 
        Runb(1)-ThrdNm(pool-1-thread-1)-Time(12:48) 
        Runb(2)-ThrdNm(pool-1-thread-1)-Time(12:51) 
        Runb(3)-ThrdNm(pool-1-thread-1)-Time(12:54) 
        Runb(4)-ThrdNm(pool-1-thread-1)-Time(12:57) 
        Runb(5)-ThrdNm(pool-1-thread-1)-Time(13:00) 
        Runb(1)-ThrdNm(pool-1-thread-1)-Time(13:03) 
        Runb(2)-ThrdNm(pool-1-thread-1)-Time(13:06) 
    */

    /*
        // scheduler.shutdown(); 會立即關閉  scheduler ?? 沒等 Runnable 執行完成
        Runb(1)-ThrdNm(main)-Time(00:50)-Runnable(1) 啟動 
        Runb(2)-ThrdNm(main)-Time(00:50)-Runnable(2) 啟動 
        Runb(3)-ThrdNm(main)-Time(00:50)-Runnable(3) 啟動 
        Runb(4)-ThrdNm(main)-Time(00:50)-Runnable(4) 啟動 
        Runb(5)-ThrdNm(main)-Time(00:50)-Runnable(5) 啟動
    */
}
```
:::

#### (7) newSingleThreadScheduledExecutor_scheduleAtFixedRate_schedule_cancel()
:::spoiler

```java=
static void newSingleThreadScheduledExecutor_scheduleAtFixedRate_schedule_cancel() {
	
    ScheduledExecutorService scheduler = Executors.newSingleThreadScheduledExecutor();
    int j = 0;

    out.printf("Runb(%d)-ThrdNm(%s)-Time(%s)-Runnable(%d) 啟動 %n", j, Thread.currentThread().getName(),
            sdf.format(new java.util.Date()), j);

    ScheduledFuture<?> future = scheduler.scheduleAtFixedRate(() -> {
        try {
            out.printf("Runb(%d)-ThrdNm(%s)-Time(%s) %n", j, Thread.currentThread().getName(),
                    sdf.format(new java.util.Date()));
            Thread.sleep(2000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }, 2, 5, TimeUnit.SECONDS);

    out.printf("Runb(%d)-ThrdNm(%s)-Time(%s)-future.cancel() 開始計時 %n", j, Thread.currentThread().getName(),
            sdf.format(new java.util.Date()));

    scheduler.schedule(() -> {
        future.cancel(false);
        out.printf("Runb(%d)-ThrdNm(%s)-Time(%s)-Runnable(%d) 結束 %n", j, Thread.currentThread().getName(),
                sdf.format(new java.util.Date()), j);
    }, 13, TimeUnit.SECONDS);

    /*
        Runb(0)-ThrdNm(main)-Time(10:26)-Runnable(0) 啟動 
        Runb(0)-ThrdNm(main)-Time(10:26)-future.cancel() 開始計時 
        Runb(0)-ThrdNm(pool-1-thread-1)-Time(10:28) 
        Runb(0)-ThrdNm(pool-1-thread-1)-Time(10:33) 
        Runb(0)-ThrdNm(pool-1-thread-1)-Time(10:38)
        Runb(0)-ThrdNm(pool-1-thread-1)-Time(10:40)-Runnable(0) 結束 
        ※ future.cancel() 預計在 10:39 開始執行
        ※ 因 Runnable sleep(2000)，故 Runb(0) 在 10:40 執行結束，才開始執行 future.cancel()
    */
}
```
:::

---

## 第12章 Lambda

### 12.2.1 使用 Optional 取代 null《p.12-18》

```java=
import java.util.Optional
Optional.of(value) // 建立 value (non-null) 的 Optional 實例 / 若 value 為 null，則會丟出 java.lang.NullPointerException
Optional.empty()   // 建立 null 的 Optional 實例
Optional.get()     // 取得 value / 如果 value 為 null，則丟出 java.util.NoSuchElementException
Optional.isPresent()            // 若 value 為 non-null，則 true / 反之則為 false
Optional.orElse(defaulValue)    // 若 value 為 null，回傳 defaulValue / 反之則回傳 value 
Optional.orElseThrow            // 若 value 為 non-null，回傳 value / 反之則丟出 java.util.NoSuchElementException
Optional.ofNullable(value)      // 若 value 為 null，則調用 Optional.empty() / 反之則調用 Optional.of(value)
```

```java=
package java.util;

// final，類該不能被繼承
public final class Optional<T> {

    // final，變數不可被修改值，其值在初始化后不能被修改
    private static final Optional<?> EMPTY = new Optional<>(null);
    private final T value;

    private Optional(T value) {
        this.value = value;
    }
	
    public static<T> Optional<T> empty() {
        @SuppressWarnings("unchecked")
        Optional<T> t = (Optional<T>) EMPTY;
        return t;
    }
	
    // 如果 value 為 null，則丟出 java.lang.NullPointerException
    public static <T> Optional<T> of(T value) {
        return new Optional<>(Objects.requireNonNull(value));
    }
	
    @SuppressWarnings("unchecked")
    public static <T> Optional<T> ofNullable(T value) {
        return value == null ? (Optional<T>) EMPTY
                             : new Optional<>(value);
    }
	
    // 如果 value 為 null，則丟出 java.util.NoSuchElementException
    public T get() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
	
    public boolean isPresent() {
        return value != null;
    }
	
    public boolean isEmpty() {
        return value == null;
    }
	
    public T orElse(T other) {
        return value != null ? value : other;
    }

    public T orElseThrow() {
        if (value == null) {
            throw new NoSuchElementException("No value present");
        }
        return value;
    }
}
```