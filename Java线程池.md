# Java线程池 #
----------------------------------------
## 为什么要使用线程池？ ##
1. 创建线程和销毁线程开销很大，耗费JVM大量工作；
2. 线程太多管理困难。

## 如何实现线程池？ ##

实现线程池至少需要四部分：   
1. 线程管理器（ThreadPool）：用于创建和管理线程池，一般包含创建线程池，销毁线程池，添加新任务，等待所有线程结束等这些方法。   
2. 工作线程：线程池中的线程。一般是一个循环执行任务的线程，在没有任务时等待。   
3. 任务接口：供工作线程调度的任务接口。   
4. 任务队列：用于存放任务。   

    /**
     * 线程池管理器
     * @author JiangZhenJie
     * @date 2014-10-21
     */
    public class ThreadPool extends ThreadGroup {
    
    	private static int threadID = 1234;
    	private boolean isClosed = false;
    	private LinkedList<ThreadTask> taskQueue;  //任务队列
    
    	/**
    	 * 创建线程池
    	 * @param numThread
    	 */
    	public ThreadPool(int numThread) {
    		super("ThreadPool-" + threadID);
    		super.setDaemon(true); //设置为true，当线程池中所有线程被撤销时，线程池自动撤销
    		taskQueue = new LinkedList<ThreadTask>();
    		for (int i = 0; i < numThread; i++) {
    			new WorkThread().start();
    		}
    	}
    	
    	/**
    	 * 关闭线程池
    	 */
    	public synchronized void close(){
    		if(!isClosed){
    			isClosed = true;
    			taskQueue.clear();
    			interrupt(); //中断所有工作线程，继承自ThreadGroup
    		}
    	}
    	
    	/**
    	 * 执行线程
    	 * @param task
    	 */
    	public synchronized void execute(ThreadTask task) {
    		if(isClosed){
    			return ;
    		}
    		if (task != null) {
    			taskQueue.add(task);
    			notify();
    		}
    	}
    
    	/**
    	 * 等待工作线程将所有任务执行完成
    	 */
    	public void join(){
    		synchronized (this) {
    			isClosed = true;
    			notifyAll();
    		}
    		Thread[] threads = new Thread[this.activeCount()];
    		int count = enumerate(threads); //将所有活动线程复制到新创建的线程组
    		for (int i = 0; i < count; i++) {
    			Thread thread = threads[i];
    			try {
    				thread.join();
    			} catch (InterruptedException e) {
    				e.printStackTrace();
    			}
    		}
    	}
    	
    	/**
    	 * 获取线程任务
    	 * @return
    	 */
    	protected synchronized ThreadTask getTask() {
    		if (taskQueue.size() != 0) {
    			ThreadTask task= taskQueue.removeFirst();
    			return task;
    		}
    		return null;
    	}
    
    	/**
    	 * 工作线程
    	 * @author JiangZhenJie
    	 * @date 2014-10-21
    	 */
    	private class WorkThread extends Thread {
    		@Override
    		public void run() {
    			while (!isInterrupted()) {
    				ThreadTask task = null;
    				task = getTask();
    				if (task != null) {
    					task.perform();
    				}
    			}
    		}
    	}
    }
    
    /**
     * 任务接口
     * @author JiangZhenJie
     * @date 2014-10-21
     */
    public interface ThreadTask {
    	public void perform()；
    }

## 如何优化？ ##

1. 动态增加工作线程。
2. 优化工作线程数目。
3. 提供多个线程池。


参考：[线程池的介绍及简单实现](http://http://www.ibm.com/developerworks/cn/java/l-threadPool/)