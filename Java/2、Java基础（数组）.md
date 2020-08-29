## Java基础（数组）

#### 1、数组的定义：

```java
数据类型[] 数组名 = new 数据类型[元素个数或数组长度];

int[] x = new int[100];
```

上述语句表示在内存中定义了100个int类型的变量，其初始值都是0。此时内存中的存储地址是连续的。

```java
public static void main(String[] args) {
    int[] arr; // 声明变量
    arr = new int[3]; // 创建数组对象
    System.out.println("arr[0]=" + arr[0]); //访问数组中的第一个元素
    System.out.println("arr[1]=" + arr[1]); //访问数组中的第二个元素
    System.out.println("arr[2]=" + arr[2]); //访问数组中的第三个元素
    System.out.println("数组的长度是： " + arr.length); //打印数组长度
}
```

```java
//输出结果
arr[0]=0
arr[1]=0
arr[2]=0
数组的长度是：3
```

元素默认值：

数据类型 | 默认初始化值
:---:|:---:
byte、short、int、long | 0
float、double | 0.0
char | 一个空字符（空格），即 '\u0000'
boolean | false
引用数据类型 | null，表示变量不引用任何对象

#### 2、数组初始化：

在定义数组时只指定长度，由系统自动为元素赋值的方式称作动态初始化。在定义数组时就为数组的每个元素进行赋值称为静态初始化。

```java
/**
 * 类型[] 数组名 = new 类型[]{元素1,元素2,...}
 * 类型[] 数组名 = {元素1,元素2,...}
 */
public static void main(String[] args) {
    int[] arr = { 1, 2, 3, 4 }; // 静态初始化
    // 下面的代码是依次访问数组中的元素
    System.out.println("arr[0] = " + arr[0]);
    System.out.println("arr[1] = " + arr[1]);
    System.out.println("arr[2] = " + arr[2]);
    System.out.println("arr[3] = " + arr[3]);
}
```
#### 3、数组的遍历：

依次访问数组中的每个元素称为数组的遍历。一般情况下会通过`for`循环进行数组的遍历。

```java
public static void main(String[] args){
    int[] arr = {1,2,3,4,5};//静态初始化
    //使用for循环依次输出数组中的元素
    for (int i = 0; i < arr.length; i++){
        System.out.println(arr[i]);//通过下标访问数组中的元素
    }
}
```

这里需要注意的是for循环中，循环条件的设置，数组的是下标是从0开始一直到==数组长度-1==。如果for循环中写成了 `i <= arr.length` 程序会抛出 `ArrayIndexOutOfBoundsException` 数组下标越界异常。

#### 4、二维数组的定义：

二维数组可以理解为在一维数组中在嵌套一个一维数组。

```java
//定义一个3*4的二维数组，二维数组的长度为3，每个元素是一个长度为4的数组
int[][] arr1 = new int[3][4];

//定义一个长度为3的数组，每个元素的长度不确定
int[][] arr2 = new int[3][];

//定义一个二维数组，长度为3，每个数组元素的长度不一致
int[][] arr3 = {{1,2}, {3,4,5,6}, {7,8,9}}
```

**注意：** 当声明一个二维数组时可以不指定数组的长度，但如果想对该数组进行赋值时需注意：
```java
public static void main(String[] args) {
	String[][] strings = new String[1][];
	
	//strings[0][0] = "123";//错误
	//String a = "123";
	//strings[0][0] = a;//错误
	String[] a = {"123"};
	strings[0] = a;
	
	System.out.println(strings[0][0]);
}
```
二维数组可以理解为一个一维数组中存储的每一个元素都是一个一维数组，即这个数组中每个元素存储的都是一个对象，不能直接进行赋值。

#### 5、二维数组的遍历：

二维数组可以通过两个for循环嵌套进行数组的遍历。

```java
public static void main(String[] args){
    int[][] arr = {{1,2}, {3,4,5,6}, {7,8,9}};
    
    for(int i = 0; i < arr.length; i++){
        for(int j = 0; j < arr[i].length; j++){
            System.out.print(arr[i][j] + "  ");
        }
        System.out.println();
    }
}
```

#### 6、练习：

要统计一个公司三个销售小组中每个小组的总销售额。以及整个公司的销售额。第一小组销售额为{11,12}万元第二小组销售额为{21, 22, 23}万元第三小组销售额为{31, 32, 33, 34}万元。

```java
public static void main(String[] args) {

	int[][] arr = { { 11, 12 }, { 21, 22, 23 }, { 31, 32, 33, 34 } };
	// 统计每个小组的营业额，以及整个公司的营业额
	int zu = 0;
	int all = 0;
	for (int i = 0; i < arr.length; i++) {
		for (int j = 0; j < arr[i].length; j++) {
			zu += arr[i][j];
		}
		all += zu;
		System.out.println("第" + (i + 1) + "小组的营业额为:" + zu + "万元");
		zu = 0;
	}
	System.out.println("整个公司的营业额为:" + all + "万元");
}
```