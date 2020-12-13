## Java反射

#### 1、Class类

在面向对象的世界里，万事万物皆对象。在java语言中 **类** 是对象，类是 **java.lang.Class** 类的实例对象。

表示一个类的实例（类的实例化）：

```java
//Person类
public class Person{
    
}
```

一般情况下通过 `Person person = new Person()` 来表示一个类的实例。这里的 `person` 是 **Person** 类的实例对象，同时 **Person** 类也是一个实例对象，它是 **Class** 类的实例对象。

> 任何一个类都是 **Class** 类的实例对象。

**Class** 的实例对象有三种表达形式：

```java
//1、Class c = 类名.class
Class c1 = Person.class;    //任何一个类都有一个隐含的静态成员变量

//2、Class c = 对象名.getClass();
Person person = new Person();
Class c2 = person.getClass();    //通过类的对象的getClass()方法创建

//3、Class c = Class.forName("类的完整路径");
Class c3 = null;
try {
	c3 = Class.forName("com.entity.Person");
} catch (ClassNotFoundException e) {
	e.printStackTrace();
}
```

通过这三种方法创建的类对象 c1、c2、c3都是相等的， **一个类只可能是Class类的一个实例对象** 。

在万事万物皆对象的世界中，类也是对象，类是Class类的实例对象，这个对象称之为该类的 **类类型** 。例如：c1、c2、c3都是Person类的类类型。

#### 2、Java动态加载类

**Class.forName(“类的全称”)：** 不仅表示了类的类类型，还代表了动态加载类。编译时加载类是 **静态加载类** ，运行时加载类是 **动态加载类** 。

new 创建对象是静态加载类，在编译时就需要加载所有可能使用到的类。通过动态加载类可以实现使用哪个类就加载哪个类。

```java
public static void main(String[] args){
    try{
        //动态加载类，在运行时加载
        Class c = Class.forName(args[0]);
        //通过类型装换创建对象
        c.newInstance();//创建对象时，不知道加载类的类型时无法强转为想要的类型，可以通过java多态的思想，用一类类的接口作为参数类型接收。
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

#### 3、Java获取方法信息

```java
public static void main(String[] args){
    Class c1 = int.class;		//int的类类型
    Class c2 = String.class;	//String类的类类型	String类字节码
    Class c3 = double.class;	//double数据类型的字节码表示方式
    Class c4 = Double.class;
    Class c5 = void.class;		//void关键字的类类型
    System.out.println(c1.getName());	//int
    System.out.println(c2.getName());	//java.lang.String
    System.out.println(c2.getSimpleName());	//String	不带包名的类的名称
}
```

```java
public class ClassUtil {
	/**
	 * 打印类的信息包括类的成员函数，成员变量
	 * @param obj 该对象所属类的信息
	 */
	public static void printClassMessage(Object obj) {
		//获取类的信息，首先要获取类的类类型
		Class<? extends Object> c = obj.getClass();	//传递的是哪个子类的对象c就是该子类的类类型
		//获取类的名称
		System.out.println("类的名称是：" + c.getName());
		/*
		 * Method类，方法对象
		 * 一个成员方法就是一个Method对象
		 * getMethods()	方法获取的是所有的public的方法，包括父类继承而来的
		 * getDeclaredMethods()	获取的是所有该类自己声明的方法，不问访问权限，不包括父类继承的方法
		 */
		Method[] ms = c.getMethods();	//c.getDeclaredMethods();
		for(int i = 0; i < ms.length; i++) {
			//得到方法的返回值类型的类类型
			Class returnType = ms[i].getReturnType();
			//得到返回值的名字
			System.out.print(returnType.getName() + " ");
			//得到方法的名称
			System.out.print(ms[i].getName() + "(");
			//获取参数类型 --> 得到的是参数列表的类型的类类型
			Class[] paramType = ms[i].getParameterTypes();
			for (Class class1 : paramType) {
				System.out.print(class1.getName() + ",");
			}
			System.out.println(")");
		}
	}
}
```

