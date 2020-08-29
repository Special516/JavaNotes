## Java基础（方法）

在Java中方法就是用来完成解决某件事情或实现某个功能的办法。可以在程序中引用方法名称和所需的参数来实现方法的调用。

#### 1、方法的语法格式：

```java
/**
 * 修饰符 返回值类型 方法名(参数类型 参数名1, 参数类型 参数名2){
 *      执行语句;
 *      ......
 *      return 返回值;
 * }
 */
```

**修饰符：** 访问修饰符的作用是对方法的访问权限进行限定，public表示公共的方法，在任何地方都可以进行访问；protected表示受保护的，可以在同类、同包、子类(包括不同包子类)中访问；default(默认访问修饰符)，可以在同类、同包中访问；private(私有的)只能在同类中进行访问。

**返回值类型：** 用于限定方法返回值的类型。

**参数名：** 一个变量，用于接收调用方法时传入的参数。

**return关键字：** 用于结束方法并返回方法指定的值。

**返回值：** 被return语句返回的值，该值会返回给调用者。

```java
public static void main(String[] args){
    int area = getArea(3,5);
    System.out.println("This area is :" + area);
}

public static int getArea(int x, int y){
    int temp = x * y;
    return temp;
}
```
#### 2、方法的重载：

方法的重载就是方法名相同，方法的参数类型和参数个数不同。

```java
public static int add(int x, int y);

public static int add(int x, int y, int z);

public static int add(double x, double y);
```

虽然上面的三个方法名称相同，但因为它们的参数类型和个数不同，因此形成了方法重载。在调用 `add` 方法时，只要传入的参数不同得到的结果就不同。

**注意：** 

1、重载方法参数列表必须不同。

2、重载只与方法名和参数类型相关，与返回值无关。

3、重载与具体的变量名无关。

#### 3、方法的重写： 

方法重写的前提是类之间存在继承关系，当父类方法无法满足子类需求的时候就需要进行方法重写。方法重写需要注意：

1、方法名与形参列表完全一致。

2、子类的权限修饰符必须大于或者等于父类的权限修饰符。

3、子类的返回值类型必须小于或者等于父类的返回值类型。

#### 4、形参和实参：

在调用方法时可能需要进行参数的传递，在方法中的参数被称作 `形参` 调用方法处传递过去的参数被称作是 `实参` 。

```java
public static void main(String[] args){
      int num = 10;
      change(num);//此处参数传递为实参
      System.out.println("num = " + num);//此处输出 num = 10
      
      School school = new School();
      school.name = "小学";
      change(school);
      System.out.println("schoolName = " school.name);//此处输出 schoolName = 大学
  }
  
  public static void change(int a){//此处的参数为形参
      a = 20;
  }
  
  public static void change(School school){
      school.name = "大学";
  }
```

  **注意：** 方法调用的参数传递过程中，如果传递的是基本数据类型（八大基本数据类型）方法中对数据进行的操作不会影响调用处的数据值。上面的例子中在 `main` 方法中定义了一个 `num` 变量并赋值为 `10` 随后调用 `change(num)` 方法，在方法中对 `num` 重新赋值。此时再主方法中打印出的 `num` 值依然为 `10` 这就说明方法调用时传递的参数为基本数据类型时，实参的值不会被改变，方法调用传递的是值。

在 `main` 方法中通过 `new` 关键字创建一个对象，然后调用 `change(school)` 方法对对象中的属性进行重新赋值操作。此时主方法中打印的 `schoolName` 为调用方法中修改的值。说明方法调用传递参数为 `引用数据类型` 时，方法中对形参的修改会影响实际参数的值，方法调用传递的是地址。

>`String` 是一种引用数据类型，但当方法调用传递的参数为String时，方法内部的改动不会影响调用处的实际参数值。

