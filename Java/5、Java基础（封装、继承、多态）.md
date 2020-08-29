## Java基础（封装、继承、多态）

封装、继承、多态是java面向对象编程的三大特性。

#### 1、封装：

- **什么是封装：**

  在面向对象的程序设计中，封装是将某些信息隐藏在类的内部，不允许外部程序直接访问；

  通过该类提供的公有的方法对隐藏信息进行访问和操作。

- **封装的特点：**

  封装可以对成员变量进行更精确的控制，能够隐藏类的实例细节，方便进行修改；

  封装只能通过特定的方法进行访问。

- **封装的实现：**

  属性私有化：修改属性的可见性来限制属性的访问，一般会在属性前添加 `private` 关键字使属性私有化；

  提供对外访问的方法：对每个属性都提供对外的访问方法，即创建一对赋值取值的方法，一般都是 `getter / setter` 方法。

  对属性的合法值进行判断：可以在 `getter/setter` 方法中对输入进行合法值的判断，保证数据的合法性。

- **实例：**

  ```java
  public class Person{
      //1、将属性私有化，外界无法访问
      private String name;
      private int age;
      
      //2、提供getter和setter对外访问的方法
      public void setName(String name){
          this.name = name;
      }
      public String getName(){
          return this.name;
      }
      
      public void setAge(int age){
          //可以在赋值的方法中对传入的值进行判断，如果不符合我们的预期可以不进行设置值或给一个默认值
          if(age < 0){
              System.out.println("年龄没有负数");
              this.name = 20;
          } else {
               this.age = age;
          }       
      }
      public int getAge(){
          return this.age;
      }
  }
  ```

  通过上面的代码即可完成对 `Person` 的封装。现在类中的 `name` 和 `age` 属性不能再外部直接进行访问，只能通过 `getter/setter` 方法进行操作。

#### 2、继承：

- **什么是继承：**

  继承是一种类与类之间的关系，继承是将具有共同特征或行为的类进行抽取，将这些共同的特征或行为组成一个类作为父类，子类继承父类，子类拥有父类的属性和方法。通常来说子类和父类是 `is - a` 的关系。一个类只能继承一个父类，在Java中所有类的父类都是 `Object` 。

- **为什么要使用继承：**

  假如现在需要对宠物狗和宠物猫进行封装以供后面的业务使用，在完成这两个类的封装后发现宠物狗和宠物猫都有名称、年龄、健康值等属性，在这两个类中这些代码是重复的。如果就这两个类也许不会感觉到什么，那如果现在有100只不同的宠物呢，它们都有这些属性，再写100遍？如果应需求变动需要加入昵称这个属性呢，再改100个类？

  现在讲将这些共有的属性和方法进行抽取，构成一个父类宠物类，在这个父类中就封装宠物的名称、年龄、健康值。然后让宠物狗和宠物猫继承这个宠物类，在这两个子类中不需要再声明这些公有的属性。子类继承父类后可以直接使用父类的属性和方法。现在再操作上面需求就会简单很多了。

- **继承的特点：**

  子类可以拥有父类非私有的属性和方法；

  子类可以有自己的属性和方法；

  子类可以根据自己的特性对父类的方法进行重写；

  一个类只能有一个父类，不能实现多继承，但可以实现多重继承，即一个类继承一个父类，这个父类又继承另一个父类，这样最后的那个类就拥有上面连个类的非私有属性和方法；

  父类的私有属性和方法子类不能继承；

  子类和父类不在同一个包中，父类使用默认访问权限的属性或方法不能被继承；

  父类的构造方法子类不能继承。

- **继承实例：**

  ```java
  //父类
  public class Pet{
      private String name;
      private int age;
      private int health;
      
      public void setName(String name){
          this.name = name;
      }
      public String getName(){
          return this.name;
      }
      public void setAge(int age){
          this.age = age;
      }
      public int getAge(){
          return this.age;
      }
      public void setHealth(int health){
          this.health = health;
      }
      public int getHealth(){
          return this.health;
      }
  }
  
  //宠物猫子类 通过extends关键字实现继承
  public class Dog extends Pet{
      
  }
  
  //宠物狗子类
  public class Cat extends Pet{
      
  }
  ```

  上面代码中虽然没有给子类中设置任何属性，但通过对象对 `name、age、health` 这些属性进行操作。继承后类的加载顺序为：父类属性 ---> 父类的构造方法 --->子类属性 ---> 子类的构造方法

#### 3、多态：

- **什么是多态：**

  多态是同一个行为具有多种不同表现形式的能力。同一个操作使用不同的实例能够得到不同的效果。比如打印操作，黑白打印机和彩色打印机打印出来的效果是不同的。这里的打印可以看做一个方法，黑白打印机和彩色打印机可以看做是两个对象，不同的对象执行相同的操作，得到的结果是不一样的，这就是多态的体现。

- **多态的特点：**

  消除类型之间的耦合关系、可替换性、可扩充性、接口性、灵活性、简化性。多态存在的不要条件：继承、重写、父类引用指向子类实例。使用多态调用方法时，首先检查父类中是否有该方法如果没有则编译错误，如果有再去调用子类的同名方法。

  多态可以使类具有良好的扩展性并可以对所有类的对象进行通用处理。

- **多态案例：**

  ```java
  public abstract class Animal {  
      abstract void eat();  
  }  
    
  public class Cat extends Animal {  
      public void eat() {  
          System.out.println("吃鱼");  
      }  
      public void work() {  
          System.out.println("抓老鼠");  
      }  
  }  
    
  public class Dog extends Animal {  
      public void eat() {  
          System.out.println("吃骨头");  
      }  
      public void work() {  
          System.out.println("看家");  
      }  
  }
  
  public class Test {
      public static void main(String[] args) {
        show(new Cat());  // 以 Cat 对象调用 show 方法
        show(new Dog());  // 以 Dog 对象调用 show 方法
                  
        Animal a = new Cat();  // 向上转型  
        a.eat();               // 调用的是 Cat 的 eat
        Cat c = (Cat)a;        // 向下转型  
        c.work();        // 调用的是 Cat 的 work
    }  
              
      public static void show(Animal a)  {
      	a.eat();  
          // 类型判断
          if (a instanceof Cat)  {  // 猫做的事情 
              Cat c = (Cat)a;  
              c.work();  
          } else if (a instanceof Dog) { // 狗做的事情 
              Dog c = (Dog)a;  
              c.work();  
          }  
      }  
  }
  ```

  `Animal` 是一个抽象父类，`Cat` 和 `Dog` 是继承自 `Animal` 的两个子类，在子类中重写了父类的的抽象方法 `eat` 方法，做出了具体的实现方式。另外这两个子类中还拥有两个独立的 `work()` 方法。在测试类中调用 `show()` 方法，方法中的形参数据类型为 `Animal` 可以接收其子类类型。但不同子类的 `work()` 方法表现形式不同，且父类中并没有 `work()` 方法的声明，直接调用会出错。这里使用 `instanceof` 做类型判断后将 `Animal` 对象向下转型为相应的子类对象，然后再调用对应的方法。在主方法中 `Animal a = new Cat();` 是一个自动转型的操作。抽象类不能被实例化但抽象类可以作为数据类型接收其子类的实例。后面的 `a.eat();` 调用的就是 `Cat` 类中的 `eat()` 方法。

	​				