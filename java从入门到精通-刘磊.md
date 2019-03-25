# java从入门到精通

> 学习过程参考了程序员面试指南和极客时间的java核心36讲

***

## 第一节 线程

​     1.1 程序 进程 线程

​	理解线程首先要区分好程序，进程和线程的区别

​		程序：其实是一个静态的概念的概念，是应用执行的蓝本，存放在外存上，没有运行的软件叫程序(就比如我们			把QQ下载到了D盘但是并没有运行它的状态)

​		进程：用来描述程序的动态执行过程(我们运行了QQ)

​		线程：是进程中相对独立的一个程序段的执行单元（我们一边和a聊天，一边和b聊天）

​		一个进程中可以有多个线程

​	1.2 创建线程

​		1.2.1 Thread类和Runnable接口

​		创建线程有两种方法:继承Thread类或者事项Runnable接口

​    	   重写run方法，run方法就类似于主线程的main方法，run方法运行结束，其对应的线程也会结束

```java
public class xiancheng1 extends Thread{
	public static void main(String [] args){
	System.out.println("来自于主线程的输出");
	xiancheng1 xc=new xiancheng1();
	xc.start();
}
public void run(){
	System.out.println("来自于子线程的输出");
}
}
```
~~~java
public class RunTest {
public static void main(String [] args){
	//创建一个实现了Runable接口的对象
	MyThread thread=new MyThread();
	//以实现了runable接口的对象作为参数实例化Thread对象
	Thread t=new Thread(thread);
	t.start();
}
}
class MyThread implements Runnable
{
	public void run() {
		System.out.println("thread body");
	}
	
}

~~~

1.2.2 Callable和Future接口

继承Thread类或者实现Runnable接口可以完成线程的创建，在线程start后就会执行run方法中的内容。如果我们希望run方法有个返回值，我们并不能得到这个方法的返回值，并且如果我们希望查看run方法是不是运行完成或者终止run方法的进行都是不能做到了，为了解决这个问题java5以后新增了callable和future接口，callable接口中提供了一个call方法，这个call方法类似于前面提到的run方法，但是它可以有返回值，这个返回值可以被Future接口实现类获取，call方法可以声明和抛出异常。

下面提供了一段使用这两个接口的实例：

~~~java
public class CallableTest2 {
	public static void main(String [] args){
		//声明对象
		MyCallable m1=new MyCallable(1000);
		MyCallable m2=new MyCallable(2000);
		//将cllable对象封装到FutureTask对象
		FutureTask<String> ft1=new FutureTask<String>(m1);
		FutureTask<String> ft2=new FutureTask<String>(m2);
		//创建线程池
		ExecutorService executor=Executors.newFixedThreadPool(2);
		executor.execute(ft1);
		executor.execute(ft2);
		while(true){
			try{
				if(ft1.isDone() && ft2.isDone()){
					System.out.println("done");
					executor.shutdown();
					return;
				}
				// 任务1没有完成，会等待，直到任务完成 
				if(!ft1.isDone()){
					System.out.println("future1 output"+ft1.get());
				}
				System.out.println("wait future2");
				//这里应该就是规定查询的时间
				String s=ft2.get(200L,TimeUnit.MILLISECONDS);
				if(s!=null){
					System.out.println("future2 output"+s);
				}
			}catch(Exception e){
				
			}
			}	
	}
}
class MyCallable implements Callable<String>{
	private long waitTime;
	public MyCallable(int time){
		this.waitTime=time;
	}
	@Override
	public String call() throws Exception {
		Thread.sleep(waitTime);
		return Thread.currentThread().getName();
	}
	
}

~~~

Future接口中的方法

1.boolean cancel（）：取消任务执行，其中参数指定是否立即中断任务执行或者等待任务结束

2.boolean isCancelled（）：确认任务是否已经取消，任务正常完成前将其取消则返回true

3.boolean isDone（）：确认任务是否已经完成，任务正常终止、异常或取消都将返回true

4.V get ：该方法等待任务执行结束然后获得V类型的结果

5.V get(long timeout,TimeUnit unit):参数timeout指定超时时间，unit指定时间的单位

其实Future接口的工作主要有三个：1.得到返回值  2.判断线程是不是结束  3.终止线程

1.3 线程的生命周期

线程的生命周期：创建状态new 可运行状态 runnable 阻塞状态blocked 等待状态Time  计时状态Time Waiting 终止状态

1.4 线程调度

 线程睡眠：需要让当前正在执行的线程暂停一段时间，则通过Thread类的静态方法sleep使其进入等待状态
         	（注意：sleep是Thread的方法，而notify是Object的方法）
 线程让步：调用yield方法可以实现进程让步，它与sleep类似，也会暂停正在执行的线程，让当前线程交出CPU权限，单yoeld方法只能让拥有相同优先级或优先级更高的线程有获取CPU执行的机会。如果可运行线程队列中的线程的优先级都没有当前线程的优先级高，则当前线程会继续执行
yield是Thread类的方法
线程协作：
若需要一个线程运行到某一个点时，等待另一个线程运行结束后才能继续进行，这种情况可以通过join方法来实现。
在A线程中，需要等到B线程运行结束后再继续A线程，写法是在A线程中写上B.join（）

~~~java
public class joinTest {
	public static void main(String [] args){
		subThread st=new subThread();
		st.start();
		for(int i=0;i<10;i++){
			System.out.println("我是主线程");
			if(i==5){
				try {
					st.join();
				} catch (InterruptedException e) {
					// TODO Auto-generated catch block
					e.printStackTrace();
				}	
			}
			try {
				Thread.sleep(1000);
			} catch (InterruptedException e) {
				// TODO Auto-generated catch block
				e.printStackTrace();
			}
		}
	}

}
class subThread extends Thread{
	public void run(){
		for(int i=1;i<=10;i++){
			System.out.println("我是子线程");
		}
		try {
			sleep(1000);
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}

~~~



1.5 守护线程

线程可以分为两类一类是守护线程，一类是用户线程。

守护线程主要负责一些后台的维护，例如jvm的垃圾回收线程

守护线程依赖于创建他的线程而用户线程不依赖，就是说守护线程会随着创建他的线程的结束而结束而用户线程一直会运行到自己结束

可以通过setDaemon来设置一个线程是用户线程还是守护线程，但是这个修改必须在线程start以前修改。

1.6 线程同步

这个是线程问题的一个难点，在进行多线程程序设计时，有时需要多个线程共享一段代码，从而实现共享同一个私有成员或类的静态成员的目的，如果不对线程进行控制，就很容易造成线程无序地访问这些共享资源，而导致无法得到正确的结果，这些问题通常成为线程安全问题。

下面的例子就是一个线程安全的问题：

~~~java
public class ThreadUnsafe {
	public static void main(String [] args){
		ShareData sd=new ShareData();
		for(int i=0;i<10;i++){
			new Thread(sd).start();
		}	
	}
}
class ShareData implements Runnable{
	public int num=0;
	private void add(){
		int temp;
		for(int i=0;i<1000;i++){
			temp=num;
			temp++;
			num=temp;
		}
		System.out.println(Thread.currentThread().getName()+"-"+num);
		
	}
	public void run(){
		add();
	}
	
}

~~~

我们本来的期望是，用10个线程每个线程将num的值增加1000,10个线程就加到了10000.但是运行结果并不是我们想要的结果，问题就处在各个线程对num的调用的无序性，为了解决这个问题java为我们提供了锁机制，使用synchronized修饰代码块或者方法，被修饰的代码块一旦被一个线程获取。另外一个线程就不能使用，我们可以将上文中的add方法加锁

~~~java
private synchronized void add(){
		int temp;
		for(int i=0;i<1000;i++){
			temp=num;
			temp++;
			num=temp;
		}
		System.out.println(Thread.currentThread().getName()+"-"+num);
		
	}
~~~



但是synchronized不是完美的，试想一下如果有两个对象A和B，如果不恰当的使用sychronized关键字管理线程对特定对象的访问，被允许执行的线程拥有对变量或对象的排他性访问权，线程1先访问A后访问B，线程2先访问B后访问A，1得不到B就不放A，2得不到A就不放B这就僵持住了

线程间的通信

java 控制线程间通信的方法有三个：wait（） notify（）notifyAll（） 他们都是Object类的final类

wait（）：让出资源，线程从可运行状态进入到等待状态，这里要注意和sleep的区别，sleep不会让出资源

notify():可以唤醒等待队列中的第一个等待同一个共享资源的线程

notifyAll：唤醒所有正在等待同一个资源的线程



​		

