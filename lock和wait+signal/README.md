# lock和wait+signal

在编程中，我们经常使用多线程来提升性能，所以这就涉及到**互斥和同步**的问题了。而在编程中，我们一般都是通过如下方式来完成多线程的互斥和同步:

* lock | unlock 
* signal + wait(timeout) 
* join
* sleep

## C语言

在Linux C编程中，我们通常使用**pthread**类库来完成跨平台的多线程控制，如下是几个常用的API：

* pthread_mutex_lock()：占有互斥锁（阻塞操作）
* pthread_mutex_unlock(): 释放互斥锁
* pthread_cond_signal(): 唤醒第一个调用pthread_cond_wait()而进入睡眠的线程
* pthread_cond_wait(): 等待条件变量的特殊条件发生
* pthread_cond_timedwait():等待条件变量的特殊条件发生或者timeout
* pthread_join()：阻塞当前的线程，直到另外一个线程运行结束
* sleep() ： 休眠固定时间，**不过这个API是Linux原生提供，不能跨平台。**

注意：**pthread类库是glibc(绝大多数Linux平台标准C库。)的一部分。这些功能都是通过中断号进入内核来完成的，而非仅仅做了Linux兼容API。**具体可见 glibc-2.23\sysdeps\nacl\nacl-interface-list.h ，声明wait(timeout)功能中断号文件。

## Java

在Java中，多线程的控制已经是一个统一标准了，一般都是通过Java原生API或者JUC来实现并发控制。这里来说说原生的API：

* synchronized : 实现了lock 和 unlock的功能
* Object#wait : 等待信号发生，并且可以实现超时等待
* Object#notify ： 通知信号发生
* Object#notifyAll ：通知信号发生
* Thread#join :阻塞当前的线程，直到另外一个线程运行结束
* Thread#sleep ： 休眠固定时间

通过这些API，我们基本上能实现Java的多线程并发控制，当然了可以使用最新的JUC并发库的API。**而这些API底层也是可以通过pthread这个C库来实现。**

## 示例-Timer

Java中Timer的实现原理就是通过 wait 和 notify 以及 synchronized  来实现的：

	Timer timer = new Timer();
	timer.schedule(new TimerTask() {
		@Override
		public void run() {

		}
	}, 1000);

其实，Timer持有TimerImpl，其实Impl就是一个Thread实现，它一直阻塞等待task的到来，见run代码：

	public void run() {
		while (true) {
			TimerTask task;
			synchronized (this) {
				// need to check cancelled inside the synchronized block
				if (cancelled) {
					return;
				}
				//等待任务
				if (tasks.isEmpty()) {
					if (finished) {
						return;
					}
					// no tasks scheduled -- sleep until any task appear
					try {
						//等待任务加入后，会通过notify来通知它可以运行
						this.wait();
					} catch (InterruptedException ignored) {
					}
					continue;
				}
				....
				if (timeToSleep > 0) {
					// sleep!
					try {
						//延迟执行代码
						this.wait(timeToSleep);
					} catch (InterruptedException ignored) {
					}
					continue;
				}

				// no sleep is necessary before launching the task
				...
			}

			...
			try {
				//具体执行任务
				task.run();
				taskCompletedNormally = true;
			} finally {
				...
			}
		}
	}

可以看到TimerImpl的run代码会通过wait来阻塞等待任务加入queue，然后通过notify告知它可以运行task。timer#schedule最后会调用TimerImpl#insertTask，具体代码如下：

	private void insertTask(TimerTask newTask) {
		// callers are synchronized
		tasks.insert(newTask);
		this.notify();
	}

**所以任务加入队列后，通过notify来告知阻塞在等待任务的线程（TimerImpl#run）。**这样子就实现了Timer的功能了，并且通过wait(timeout)实现了delay的功能。

## 总结

多线程的控制，其实大多都是依赖：

* **lock**
* **signal + wait**

这两种类型的API来完成并发控制。

**而在此基础上，我们可以实现各种各样的多线程并发控制，比如说：MQ，CountDownLatch等。**