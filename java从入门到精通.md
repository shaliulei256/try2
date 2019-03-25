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

​		创建线程有两种方法:继承Thread类或者事项Runnable接口

​    	   重写run方法，run方法就类似于主线程的main方法，run方法运行结束，其对应的线程也会结束

​	      ~~~ java

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

​	 ~~~

