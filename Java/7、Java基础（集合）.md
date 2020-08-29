## Java基础（集合）

#### 1、集合和数组：

- **数组：**

  数组中只能存储一类相同类型的数据，在数组声明时该数据类型就已经确定（如：int、String...）。数组的长度在声明数组时就需要确定且不能变动。数组只能通过下标进行数组元素的访问和操作。

- **集合：**

  集合是Java提供的一种数据操作的API，集合的长度是一种动态长度，可以随意扩容。集合中存储的数据类型默认是**Object**类型，因此一个集合中可以存储各种对象，只能是一个对象。

#### 2、List集合：

List集合继承了**Collection**接口，是有序、可重复的集合。

- **ArrayList集合：**

  ArrayList作为List的典型实现，完全实现了List的全部接口功能，它是基于数组实现的List类，它封装了一个Object[]类型的数组，长度可以动态的增长。如果在创建ArrayList时没有指定**Object[]**数组的长度，它默认创建一个长度为10的数组，当新添加的元素已经没有位置存放的时候，ArrayList就会自动进行扩容，扩容的长度为原来长度的1.5倍。它的线程是不安全的。 

  - **常用方法：**

    |             方法              |                             作用                             |
    | :---------------------------: | :----------------------------------------------------------: |
    |         **add(E e)**          |                将指定的元素添加到此列表的尾部                |
    |      **get(int index)**       |                 返回此列表中指定位置上的元素                 |
    |     **indexOf(Object o)**     | 返回此列表中首次出现的指定元素的索引，或没有该元素元素，则返回 -1 |
    |     **remove(int index)**     |                  移除该列表中指定位置的元素                  |
    | **set(int index, E element)** |           使用指定的element元素替换指定位置的元素            |
    |          **size()**           |                     返回集合中元素的个数                     |

  - **ArrayList的底层实现：**

    ```java
    //构造方法
    ArrayList(int initialCapacity);//创建一个指定的大小的Object[]数组作为集合
    
    ArrayList(); //创建的是一个空的数组
    
    ArrayList(Collection<? extends E> c);//将集合转化为ArrayList,在底层实现中，先调用集合的toArray()方法，并赋值给elementData ， 然后进行类型的判断，是如果类型不是Object[]类型，那么将使用反射生成一个Object[]的数组，并重新赋值给elementData
    ```

    **扩容检测：**

    ```java
    public boolean add(E e) {
    	// 关键 -> 添加之前，校验容量
    	ensureCapacityInternal(size + 1); 
    	
    	// 修改 size，并在数组末尾添加指定元素
    	elementData[size++] = e;
    	return true;
    }
    ```

    数组的容量固定，在向集合中添加元素前首先要考虑其容量是否饱和。若接下来的操作会使元素超过数组的长度时，必须对其进行扩容操作。扩容的本质是创建一个容量更大的新数组，将旧数组中的元素复制到新的数组中。

    **数组遍历：**

    ```java
    List<String> list = new ArrayList<>() ;
    
    //第一种 for循环
    for (int i = 0; i < list.size(); i++) {
        list.get(i);
    }
    //第二种 迭代器
    for (Iterator iter = list.iterator(); iter.hasNext(); ) {
        iter.next();
    }
    //第三种 foreach
    for (Object obj : list){
        
    } 
    
    //第四种 ， 只支持JDK1.8+ lambda表达式
    list.forEach(
                    e->{
                        ;
                    }
            );
    ```

  - **总结：**

    1、ArrayList是基于数组实现的List类。会自动的进行扩容，采用Arrays.copyOf()实现。

    2、如果在创建ArrayList时，可以知道ArrayList元素的数量最好指定初始容量，这样可以避免ArrayList的自动多次扩容问题。

    3、与 LinkedList 相比，ArrayList 适用于频繁查询的场景，而不适用于频繁修改的场景； 与 Vector 相比，ArrayList 的所有操作都是非线程安全的。

    4、线程不安全。

- **LinkedList集合：**

  **LinkedList** 是线程不安全的，允许元素为null的双向链表。其底层数据结构是链表。相比**ArrayList**它的存储空间不连续，不需要批量扩容，也不需要预留空间，空间利用率比**ArrayList**高。当需要随机访问元素时，时间效率低。

  - **常用方法：**

    |             方法              |               作用               |
    | :---------------------------: | :------------------------------: |
    |         **add(E e)**          |   将指定元素添加到此列表的尾部   |
    | **add(int index, E element)** |  在列表的指定位置插入指定的元素  |
    |       **addFirst(E e)**       |    将指定元素插入到列表的开头    |
    |       **addLast(E e)**        |    将指定元素插入到列表的结尾    |
    |      **get(int index)**       |     返回列表中指定位置的元素     |
    |     **indexOf(Object o)**     | 返回列表中首次出现指定元素的索引 |
    |     **remove(int index)**     |        移除指定位置的元素        |
    |          **size()**           |       返回列表中的元素个数       |
    | **set(int index, E element)** | 将指定位置的元素替换成指定的元素 |

  - **LinkedList底层实现：**

    ```java
    private static class Node<E> {
    	E item;//元素值
    	Node<E> next;//后置节点
        Node<E> prev;//前置节点
    
        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    } 
    ```

    LinkedList的底层是一个双向链表。

    **add()插入单个节点：**

    ```java
    //在尾部插入一个节点： add 
    public boolean add(E e) {
        linkLast(e);
        return true; 
    }  
    
    //生成新节点 并插入到 链表尾部， 更新 last/first 节点。 
    void linkLast(E e) {
        final Node<E> l = last; //记录原尾部节点
        final Node<E> newNode = new Node<>(l, e, null);//以原尾部节点为新节点的前置节点
        last = newNode;//更新尾部节点
        if (l == null)//若原链表为空链表，需要额外更新头结点
            first = newNode;
        else//否则更新原尾节点的后置节点为现在的尾节点（新节点）
            l.next = newNode;
        size++;//修改size
        modCount++;//修改modCount 
    }
    ```

  - **总结：**

    1、通过下标获取某个node 的时候，（add select），会根据index处于前半段还是后半段 进行一个折半，以提升查询效率。

    2、删除一定会修改modCount。按下标删，也是先根据index找到Node，然后去链表上unlink掉这个Node。 按元素删，会先去遍历链表寻找是否有该Node，如果有，去链表上unlink掉这个Node。

#### 3、Set集合：

Set集合继承自Collection接口。Set集合中不允许有重复的元素存在。Set集合中可以存在null元素，且只能有一个。Set集合中存储的元素是无序的。

- **HashSet集合:**

  实现了Set接口，有哈希表支持，其底层是通过HashMap实现。不保证迭代顺序，不保证顺序恒久不变。

  - **常用方法：**

    |         方法         |                  作用                   |
    | :------------------: | :-------------------------------------: |
    |     **add(E e)**     | 如果set集合中为包含此元素则添加指定元素 |
    |    **iterator()**    |   返回此Set集合中元素进行迭代的迭代器   |
    | **remove(Object o)** |    如果指定元素存在集合中则进行移除     |
    |      **size()**      |           返回Set中元素的数量           |

  - **add(E e):**

    Set集合中不允许有重复的元素，但当向HashSet集合中添加对象元素时，会出现添加相同数据的情况。

    ```java
    public static void main(Stirng[] args){
        Set<Student> set = new HashSet<>();//定义一个HashSet集合
        //向集合中添加元素
        set.add(new Student("张三",20));
        set.add(new Student("李四",21));
        set.add(new Student("张三",20));
    }
    ```

    > 在上面的代码中向集合中插入了两条相同的“张三”数据。按照一般的思维，这两条相同的数据是不能全部添加进Set集合中的，但在这里却可以。其实，在进行HashSet插入操作时底层会生成对象对应的哈希值，如果计算的哈希值相等，则会调用equals方法进行对象判断，如果对象判断相等则会覆盖原来的信息，如果不相等则进行数据插入。由于上面的代码中没有对Student类重写 `hashCode()` 和 `equals()` 方法，当计算哈希值时出现哈希冲突，就去调用 `equals()` 进行判断。在没有重写之前会调用父类Object中的equals方法。Object中判断的是地址，所以会将相同内容的对象认定为不同。如果要不重复数据则需要重写 `hashCode()` 和 `equals()` 方法。

- **TreeSet集合：**

  使用元素的自然顺序对元素进行排序。TreeSet实例使用 `compareTo` 方法对所有元素进行比较。可以自定义规则实现自定义排序。

  让类实现 `Comparable` 接口，并实现 `compareTo` 方法，在方法中书写自定义的排序规则，可以在TreeSet集合中插入元素时对该类的对象进行自动排序。

#### 4、Map集合：

Map是一个键值对的集合，将建映射到值，键和值是一一对应的关系，其中Map中的**Key（键）**是不允许重复的，但是**Value（值）**可以重复。

- **HashMap集合：**

  基于哈希表的Map接口实现，允许null值和null键。

  - **常用方法：**

    |          方法           |               作用               |
    | :---------------------: | :------------------------------: |
    | **put(K key, V value)** | 在此映射中关联指定的值和指定的键 |
    |   **get(Object key)**   |       返回指定键所映射的值       |
    | **remove(Object key)**  |       移除指定键的映射关系       |
    |     **entrySet()**      |    返回包含映射关系的Set集合     |
    |      **keySet()**       |       返回包含键的Set集合        |
    |       **size()**        |        返回键-值映射数量         |

#### 5、集合的遍历：

- **for循环：**

  对于 **Collection** 集合可以使用for循环进行遍历：

  ```java
  for(int i = 0; i < list.size(); i++){
      System.out.println(list.get(i));
  }
  
  for(String str : set){
      System.out.println(str);
  }
  ```

- **Iterator迭代器：**

  集合还可以使用iterator迭代器进行遍历。

  ```java
  //遍历List集合
  for(Iterator iterator = list.iterator();iterator.hasNext();){                    
      int i = (Integer) iterator.next();                   
      System.out.println(i);               
  }
  
  //遍历Set集合
  Iterator<String> it = set.iterator();  
  while (it.hasNext()) {  
  	String str = it.next();  
  	System.out.println(str);  
  }
  
  //遍历Map集合
  //1、通过map.keySet遍历key和value
  Set<Student> set = map.keySet();
  for (Student stu : set) {
  	System.out.println(stu.getName() + "\t" + stu.getAge() + "\t" + map.get(stu));
  }
  
  //2、通过Map.entrySet使用iterator遍历key和value
  Set<Map.Entry<Student, String>> entries = map.entrySet();
  Iterator<Entry<Student, String>> it = entries.iterator();
  while (it.hasNext()) {
  	Entry<Student, String> entry = it.next();
  	System.out.println(entry.getKey().getName() + "\t" + entry.getKey().getAge() + "\t" + entry.getValue());
  }
  
  for(Entry<Student, String> entry : map.entrySet()) {
  	System.out.println(entry.getKey().getName() + "\t" + entry.getKey().getAge() + "\t" + entry.getValue());
  }
  ```

  