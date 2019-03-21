#### ① Synchronized 深度解析

 Synchronized 的两种用法

1. 对象锁 (包括方法锁，默认锁对象为this当前实例对象)和同步代码块锁(自己指定锁对象)
2. 类锁(指synchronizd修饰静态的方法或指定锁为Class对象)

第一种用法： 对象锁

代码块形式：手动指定锁对象

方法锁形式： synchronized 修饰普通方法，锁对象默认为this.

```
public class SynchronizedObjectCodeBlock implements Runnable {

    static SynchronizedObjectCodeBlock instance = new SynchronizedObjectCodeBlock();
    Object lock = new Object(); //手动指定对象
    Object lock2 = new Object();
    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        while (t1.isAlive() || t2.isAlive()) {

        }
        System.out.println("finished");
    }

    @Override
    public void run() {
        synchronized (lock) {
            System.out.println("我是对象锁的代码块形式，我叫" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "运行结束");
        }
        synchronized (lock2) {
            System.out.println("我是对象锁的代码块形式，我叫" + Thread.currentThread().getName());
            try {
                Thread.sleep(3000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            System.out.println(Thread.currentThread().getName() + "运行结束");
        }
    }
}
```

普通方法锁

```
public class SynchronizedObjectMethod implements Runnable {

    static SynchronizedObjectMethod instance = new SynchronizedObjectMethod();

    public static void main(String[] args) {
        Thread t1 = new Thread(instance);
        Thread t2 = new Thread(instance);
        t1.start();
        t2.start();
        while (t1.isAlive() || t2.isAlive()) {

        }
        System.out.println("finished");
    }

    @Override
    public void run() {
        method();
    }

    public synchronized void method() {
        System.out.println("我的对象锁的方法修饰符形式，我叫" + Thread.currentThread().getName());
        try {
            Thread.sleep(3000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
        System.out.println(Thread.currentThread().getName() + "finish");

    }
}

```

第二种用法： 类锁

概念(重要)： Java类可能有很多个对象，但只有一个Class对象

形式2： synchronized (*.class) 代码块

本质： 所以所谓的类锁，不过是Class对象的锁而已。

用法和效果： 类锁只能在 同一时刻被一个对象拥有。

形式1： synchronized 加在 static方法上





