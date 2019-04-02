---
title: ReentrantLock
date: 2019-04-02 16:18:12
tags: 多线程
---

### 基础应用

~~~java
private static int count = 0;

    public static void main(String[] args) throws InterruptedException {
        Lock lock = new ReentrantLock();
        Thread t1 = new Thread(new CountUtil(lock));
        Thread t2 = new Thread(new CountUtil(lock));
        t1.start();
        t2.start();
        t1.join();
        t2.join();
        System.out.println(count);
    }

    static class CountUtil implements Runnable{

        private final Lock lock;

        public CountUtil(Lock lock) {
            this.lock = lock;
        }

        @Override
        public void run() {
            lock.lock();
            for (int i = 0; i < 100; i++) {
                count++;
            }
            lock.unlock();
        }
    }
~~~

> 在使用ReentrantLock时可以调用`lock`方法和`unlock`方法进行加锁和释放锁操作

### 重入性

> ReentrantLock是一个可重入的锁。即如下代码不会造成死锁

~~~java
public void test() {
    lock.lock();
    test();
    lock.unlock();
}
~~~

> 1. 线程再次**获取**锁的时候，需要判断获取锁的线程是否是当前线程，如果是当前线程则将当前状态位+1
> 2. 在锁**释放**时将状态位-1，当计数为0时表示锁成功释放

### 公平性

