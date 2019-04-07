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

> 公平性是针对获取锁而言的，如果一个锁是公平的，那么锁的获取顺序就应该符合请求的绝对时间顺序，也就是FIFO
>
> 当构造函数中的参数为`true`时，即为公平锁

~~~java
new ReentrantLock(true)
~~~

### ReentrantLlock与Synchronized区别

> 1. 尝试非阻塞地获取锁：当前线程尝试获取锁，如果这一时刻锁没有被其他线程获取到，则成功获取并尺有锁
> 2. 能被中断地获取锁：与synchronized不同，获取到锁的线程能够相应中断，当获取到锁的线程被中断时，中断异常将会被抛出，同时锁会被释放
> 3. 超时获取锁，在指定的截止时间之前获取锁，如果截止时间到了仍无法获取锁，则返回