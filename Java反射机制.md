#Java反射学习笔记#
**声明：该笔记是观看慕课网 [反射——Java高级开发必须懂的](http://www.imooc.com/learn/199 ) 时所作。该视频讲师非常专业，课程易懂，可以移步观看。**
#一、 Class类的使用 #
1. 类是对象吗？在Java里面，万事万物皆对象。类是java.lang.Class的实例对象。任何类都是Class的实例对象。
2. 如何创建Class类的实例对象？
	1. 第一种方法：类名.class  任何一个类都有一个隐含的静态变量class
	2. 第二种方法：对象.getClass()
类也是一个对象，是Class类的实例对象，这个对象称为该类的类类型。
	3. 第三种方法：Class.forName("packet_name")   

			public class Demo{
				Foo foo1 = new Foo();
				//第一种方法
				Class c1 = Foo.class;
				//第二种方法
				Class c2 = foo1.getClass();
				//第三种方法
				Class c3 = Class.forName("com.jzj.demo.Foo");
				//每个类的都只有一个类类型，所以c1 = c2 = c3
				//可以使用类类型创建类的对象实例
				Foo foo2 = (Foo)c1.newInstance();
			}
			class Foo{}
3. 基本数据类型或者对象类型都有类类型。    
	
#二、 Class类的动态加载 #

1. 静态加载：程序在编译时刻加载。使用new创建对象属于静态加载。
2. 动态加载：程序在运行时刻加载。使用Class.forName("packet_name")属于动态加载。

## 静态加载 ##

	public class Office 
	{
		public static void main(String[] args) 
		{
			if("Word".equals(args[0]))
			{
				//new 静态加载类，在编译时如果找不到该类会报错
				Word word = new Word();
				word.start();
			}
			if("Excel".equals(args[0]))
			{
				Excel excel = new Excel();
				excel.start();
			}
			// 如果需要更多的功能，在需要更多的if...
		}
	}

## 动态加载 ##

	public class OfficeBetter 
	{
		public static void main(String[] args) 
		{
			try{
				//动态加载类
				Class c = Class.forName(args[0]);
				OfficeAble oa = (OfficeAble)c.newInstance();
				oa.start();
				//加入新功能时，无需修改此类
			}catch(Exception e){
				e.printStackTrace();
			}
		}
	}

	class Word implements OfficeAble
	{
		public void start()
		{
			System.out.println("word...start...");
		}
	}

#三、 获取类的信息 #

1. 获取类的成员变量
    
        /**
    	 * 获取类的成员变量
    	 * @param obj
    	 */
    	public static void printFieldMessage(Object obj) {
    		Class c = obj.getClass();
    		System.out.println("该类的成员变量：");
    		/*
    		 * 成员变量也是对象，一切皆对象
    		 * getFields()得到该类的public成员变量
    		 * getDeclaredFields() 获得该类自己声明的全部成员变量
    		 */
    		Field[] fields = c.getDeclaredFields();
    		for (Field field : fields) {
    			//获取成员变量的类型的类类型
    			Class type =field.getType();
    			System.out.println(type.getName()+" " +field.getName());
    		}
    	}
2. 获取类的构造函数

	   	/**
    	 * 获取类的构造函数
    	 * @param obj
    	 */
    	public static void printConstructorMessage(Object obj){
    		Class c = obj.getClass();
    		System.out.println("该类的构造函数：");
    		/*
    		 * 构造函数也对象
    		 * getConstructors 获取该类的public构造函数
    		 * getDeclaredConstructors 获取该类全部的构造函数
    		 */
    		Constructor[] cons = c.getDeclaredConstructors();
    		for (Constructor constructor : cons) {
    			System.out.print(constructor.getName()+"(");
    			/*
    			 * 获取构造函数参数的类类型
    			 */
    			Class[] parameterTypes = constructor.getParameterTypes();
    			for (Class class1 : parameterTypes) {
    				System.out.print(class1.getName() + ",");
    			}
    			System.out.println(")");
    		}
    	}
3. 获取类的成员函数

	    /**
    	 * 打印类的全部信息，包括类的成员变量，成员函数。
    	 * 
    	 * @param obj
		 *              该类的信息
    	 */
    	public static void printMethodClassMessage(Object obj) {
    		Class c = obj.getClass();
    		System.out.println("该类的成员函数：");
    		/*
    		 * 成员函数也是对象
    		 * getMethods 获取类的public成员函数，包括从父类继承的
    		 * getDeclaredMethods 获取该类自己声明的全部函数
    		 */
    		Method[] methods = c.getMethods();
    		for (Method method : methods) {
    			//获取成员函数的返回值类型的类类型
    			Class returnType = method.getReturnType();
    			System.out.print(returnType.getName()+" " + method.getName()+"(");
    			//获取成员函数参数类型的类类型
    			Class[] parameterTypes = method.getParameterTypes();
    			for (Class class1 : parameterTypes) {
    				System.out.print(class1.getName()+",");
    			}
    			System.out.println(")");
    		}
    	}
    
**总结：要想获取一个类的信息，只需要获得这个类的类类型。通过类的类类型可以获取类的全部信息。**

# 四、方法反射的基本操作 #

1. 如何获取某个方法？   
   方法的名称和方法的参数列表唯一决定某个方法。
2. 方法反射的操作？
   method.invoke(对象，参数列表)
    
        public class ClassDemo2 {
    	public static void main(String[] args) {
    		A a1 = new A();
    		Class c = a1.getClass();
    		// 获取方法---->方法名称和参数列表唯一确定方法
    		try {
    			Method method1 = c.getMethod("print", int.class, int.class);
    			// 正常情况下通过对象调用方法
    			// a1.print(10, 20);
    			// 反射----->反过来，用方法调用对象
    			method1.invoke(a1, 10, 20);
    
    			Method method2 = c.getDeclaredMethod("print");
    			method2.invoke(a1);
    
    		} catch (Exception e) {
    			e.printStackTrace();
    		}
    	}
    	}
    
    	class A {
    	public void print() {
    		System.out.println("hello world !");
    	}
    
    	public void print(int a, int b) {
    		System.out.println(a + b);
    	}
		}

# 五、通过反射了解集合泛型的本质 #

Java的泛型只是在编译时防止输入错误。编译完成后，泛型将会去掉。

	 public class ClassDemo3 {
    	public static void main(String[] args) {
    		//不使用泛型，list1可以存任何类型
    		ArrayList list1 = new ArrayList(); 
    		ArrayList<String> list2  = new ArrayList<String>();
    		Class c1 = list1.getClass();
    		Class c2 = list2.getClass();
    		System.out.println(c1 == c2);
    		//--->输出true，说明泛型只在编译时起作用，反射是在运行时才加载的。说明Java编译后集合的泛型会去泛型化
    		
    		list2.add("hello");
    		// list2.add(20); //报错
    		
    		//通过反射添加元素
    		try {
    			Method method = c2.getMethod("add", Object.class);
    			method.invoke(list2, 20);  //通过反射，能将整型添加到list2中
    			System.out.println(list2.size());  //输出2
    			//结论：通过反射绕过了编译，Java的集合泛型只在编译时起作用
    		} catch (Exception e) {
    			e.printStackTrace();
    		} 
    	}
    }
