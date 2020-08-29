## Java基础（类、对象）

#### 1、类、对象：

类描述的是一类事物的共同的特征和行为。每一个事物都可以看作是一个对象，类是这一类事物的抽象描述。比如：可以把每个人看作是不同的对象，每个人都有姓名、身高、体重；同时每个人都能够吃饭、喝水。如果把这些用Java语言表述该怎么做呢？把所有人的共同特征提取出来作为类的属性，把人们的行为作为类的方法。就可以表示为：

```java
public class Person{
    String name;//姓名属性
    int age;//年龄属性
    float height;//身高属性
    
    public void eat(){
        //描述吃东西的行为
    }
}
```

现在已经用Java语言对人共同的特征和属性进行了表述，也就是定义了一个 `class` 其中包含了属性（人共同的特征）和方法（人共同的行为）。这样的一个class可以看做是一个模板，并不是真正的“人”。如果想完整地表述一个人的特征和行为就需要进行“实例化”，将抽象的表述具体化。

```java
public static void main(String[] args){
    //Person xiaoming 表示的是声明一个变量用来保存人的实例，这个变量的数据类型是自己定义的类
    Person xiaoming = new Person();//通过new关键字实例化对象
    xiaoming.name = "小明";//设置具体的名称
    xiaoming.age = 20;//设置具体的年龄
    xiaoming.height = 180;//设置具体的身高
   
    xiaoming.eat(); //通过 对象名.方法名() 的方式进行方法的调用
}
```
#### 2、方法调用中的参数传递：

```java
public static void main(String[] args){
    show("小明");
    int sum = add(1,2);
}

public static void show(String name){
    System.out.println("name = " + name);
}

public static int add(int a, int b){
    return a + b;
}
```

在上面的代码中有两个方法调用。 `show("小明")` 是一个“带参无返回值”的方法调用。其中 `show("小明")` 中的“小明”是实际参数，具体方法实现 `public static void show(String name)` 中的name是一个形式参数。实参是调用者传递给调用方法的实际的参数，形参是用来接收调用这传递的内容，其作用范围是调用的方法，生命周期是随方法的调用开始到方法调用结束销毁。另一个方法是有返回值的方法调用，在调用方法执行完后会返回一个数据给调用者。

#### 3、变量的作用范围：

根据变量的作用范围可以将变量分为局部变量和全局变量。局部变量的作用范围是在变量定义的大括号之中，在方法外面无法进行访问。全局变量的作用范围是整个类，所有的方法都可以访问到这个变量。

**注意：** 在同一个方法中不允许有同名的局部变量，在不同的方法中可以有同名的局部变量。在同一个类中成员变量（全局变量）和局部变量同名时，局部变量拥有更高的优先级。局部变量在定义后只有赋值才能使用。局部变量是在栈中分配内存，全局变量是在堆中分配内存。

#### 4、构造方法：

1、方法名与类名相同。

2、无返回值类型。

3、主要用来对类的成员变量进行初始化。

4、当创建对象时（执行new关键字）被调用。

5、当未定义构造方法时，系统会默认提供无参的构造方法，如果定义了构造方法，系统不再提供无参构造。

#### 5、抽象类和抽象方法：

如果一个类中没有包含足够的信息来描绘一个具体的对象，这样的类就是抽象类。抽象类除了不能实例化以外类的其他功能依然存在，成员变量、成员方法、构造方法的访问和普通类一样。抽象类不能被实例化所以抽象类必须被继承才能被使用。

在抽象类中可以声明抽象方法，抽象方法没有方法体，子类继承抽象类后必须实现抽象类中的抽象方法。

#### 6、内部类：

成员内部类，可以使用成员修饰符 public static ...；内部类也是一个类也可以继承、实现接口

调用规则：内部类可以使用外部类成员，包括私有的成员变量；外部类要使用内部类成员就必须建立内部类对象

```java
public class Outer {
    private int a = 1;
    //内部类
    class Inner{
        public void inner(){
            System.out.println("内部类方法inner" + a);
        }
    }
}

public class Test{
    //内部类依靠外部类对象进行访问
    //格式：外部类名.内部类名 变量 = new 外部类对象().new 内部类对象()
    Outer.Inner in = new Outer().new Inner();
    in.inner();
}
```

```java
//同名变量的调用
public class Outer{
    int i = 1;
    class Inner{
        int i = 2;
        public void inner(){
            int i = 3;
            System.out.println(i);				// i = 3
            System.out.println(this.i);			// i = 2
            System.out.println(Outer.this.i);	// i = 1
        }
    }
}
```

```java
//局部内部类，将一个类定义在方法中
public class Outer{
    
    public void out(){
        //局部内部类
        class Inner{
            public void inner(){
                
            }
        }
        Inner in = new Inner();
        in.inner();
    }
}
/**
 * 局部内部类的调用：
 * 在方法中创建局部内部类的对象，通过该对象调用方法
 */
```

```java
//匿名内部类，简化问题：定义实现类，重写方法，建立实现类的对象
/**
 * 定义实现类，重写方法，创建实现类对象
 * 格式：new 接口或父类(){
 *     重写抽象方法;
 * };
 */
```

