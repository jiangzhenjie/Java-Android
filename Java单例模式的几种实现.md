# Java 单例模式的几种实现

Tags : Java 设计模式

---

 1. 经典模式，线程不安全。
 
        public class Singleton1{
        	private static Singleton1 instance ;
        	private Singleton1(){
        	}
        	public static Singleton1 getInstance(){
        		if(instance == null){
        			instance = new Singleton1();
        		}
        		return instance;
        	}
    	}

 2. 方法级同步，线程安全，但性能非常低

     	public class Singleton2{
    		private static Singleton2 instance ;
    		private Singleton2(){
    		}
    		public synchronized static Singleton2 getInstance(){
    			if(instance == null){
    				instance = new Singleton2();
    			}
    			return instance;
    		}
    	}
 3. 虽然进行了同步，线程依然不安全，只要线程进入if(instance == null) 分支就会创建多个实例
 
	    public class Singleton3{
    		private static Singleton3 instance ;
    		private Singleton3(){
    		}
    		public static Singleton3 getInstance(){
    			if(instance == null){
    				synchronized(Singleton3.class){
    					instance = new Singleton3();
    				}
    			}
    			return instance;
    		}
    	}
 4. 双重校验锁，线程安全，推荐写法

	    public class Singleton4{
    		private static Singleton4 instance ;
    		private Singleton4(){
    		}
    		public static Singleton4 getInstance(){
    			if(instance == null){
    				synchronized(Singleton4.class){
    					if(instance == null){
    						instance = new Singleton4();
    					}
    				}
    			}
    			return instance;
    		}
    	}
 5. 线程安全，但没达到懒加载，只要类被装载，就会创建实例

		public class Singleton5{
			private static Singleton5 instance = new Singleton5();
			private Singleton5(){
			}
			public static Singleton5 getInstance(){
				return instance;
			}
		}

 6. 静态内部类，线程安全，推荐写法
    
    	public class Singleton6{
    		private static class SingletonHodler{
    			private static final Singleton6 INSTANCE = new Singleton6();
    		}
    		private Singleton6(){
    		}
    		public static final Singleton6 getInstance(){
    			return SingletonHodler.INSTANCE;
    		}
    	}

