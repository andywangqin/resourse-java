### 1、简介
---
wait(),notify()和notifyAll()都是java.lang.Object的方法：

wait(): Causes the current thread to wait until another thread invokes the notify() method or the notifyAll() method for this object.

notify(): Wakes up a single thread that is waiting on this object's monitor.

notifyAll(): Wakes up all threads that are waiting on this object's monitor.

这三个方法，都是Java语言提供的实现线程间阻塞(Blocking)和控制进程内调度(inter-process communication)的底层机制。在解释如何使用前，先说明一下两点：

1. 正如Java内任何对象都能成为锁(Lock)一样，任何对象也都能成为条件队列(Condition queue)。而这个对象里的wait(), notify()和notifyAll()则是这个条件队列的固有(intrinsic)的方法。

2. 一个对象的固有锁和它的固有条件队列是相关的，为了调用对象X内条件队列的方法，你必须获得对象X的锁。这是因为等待状态条件的机制和保证状态连续性的机制是紧密的结合在一起的。

(An object's intrinsic lock and its intrinsic condition queue are related: in order to call any of the condition queue methods on object X, you must hold the lock on X. This is because the mechanism for waiting for state-based conditions is necessarily tightly bound to the mechanism fo preserving state consistency)

根据上述两点，在调用wait(), notify()或notifyAll()的时候，必须先获得锁，且状态变量须由该锁保护，而固有锁对象与固有条件队列对象又是同一个对象。也就是说，要在某个对象上执行wait，notify，先必须锁定该对象，而对应的状态变量也是由该对象锁保护的。

### 2、为什么在执行wait, notify时，必须获得该对象的锁？
---







参考：
[]()
