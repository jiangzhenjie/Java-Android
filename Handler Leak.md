# 非Static Handler 可能引起Activity内存泄露#
---------------
### Warning ###

	This Handler class should be static or leaks might occur   

	In Android, Handler classes should be static or leaks might occur, 
	Messages   enqueued on the application thread's MessageQueue also 
	retain their target Handler. If the Handler is an inner class, its 
	outer class will be retained as well. To avoid leaking the outer class,
	declare the Handler as a static nested class with a WeakReference to 
	its outer class.

### 为什么Handler不声明为static可能引起内存泄露？ ###
1. 当Android应用启动的时候，UI Thread会创建一个Looper对象，Looper对象负责管理Message Queue，Looper对象的生命周期是整个应用程序。
2. Handler和Looper有关联，Handler可以向Message Queue中发送Message对象或者Runnable对象，也负责处理从消息队列中取出的Message对象。
3. 非静态内部类会引用外部类对象。（静态内部类不会引用外部类对象) 
4. 假如存在这样的代码：     
  
    	handler.sendMessageDelayed(msg, 1000);
		finish();   

     当Activity finish后，一般情况该Activity对象会在某个时刻被GC回收。但因为延时消息仍然存在UI Thread的Message Queue中,而该消息引用着Handler对象，Handler对象又引用着Activity，所以，Activity无法被GC回收，从而造成内存泄露。

### 如何修改代码？ ###
原始代码：

	private Handler handler = new Handler(){
    	@Override
		public void handleMessage(Message msg) {
    		switch (msg.what) {
    			case WHAT_LOGIN_REQUEST:
    				if (msg.obj != null) {
    					UserBean userBean = (UserBean) msg.obj;
    					loginSuccess(userBean);
    				} 
    				break;
    			default:
    				break;
    		}
    	}
    };

改进代码：

	private static class LoginHandler extends Handler {
		private WeakReference<LoginActivity> mOuter = null;

		public LoginHandler(LoginActivity ac) {
			mOuter = new WeakReference<LoginActivity>(ac);
		}

		@Override
		public void handleMessage(Message msg) {
			LoginActivity ac = mOuter.get();
			if (ac == null)
				return;
			switch (msg.what) {
			case WHAT_LOGIN_REQUEST:
				if (msg.obj != null) {
					UserBean userBean = (UserBean) msg.obj;
					ac.loginSuccess(userBean);
				}
				break;
			default:
				break;
			}
		}
	}

将Handler声明为static，写一个静态内部类，传递当前Activity生成WeakReference。可以根据WeakReference Activity判断当前Activity的状态，当WeakReference Activity为空时，即Activity可能被回收了，则可以不在处理消息。

### 延伸 ###
当写内部类时，需要考虑是否能够控制该内部类的生命周期。不能时，最好声明为static。

### 参考 ###
1. [内部Handler类引起内存泄露](http://blog.chengyunfeng.com/?p=468#ixzz3JEk0CVqJ)
2. [Android Handler leak 分析及解决办法](http://my.oschina.net/dragonboyorg/blog/160986)