# Java基础复习

## 一、Java语言的运行机制

### 1. JVM

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\JVM.jpg)

+ java虚拟机(JVM)
  + 不同操作系统上的JVM不相同
+ java垃圾自动回收机制
  + 垃圾回收机制在Java程序运行过程中自动进行，程序员无法精确控制和干预
  + 但java仍然会出现内存泄漏和内存溢出的问题

### 2. JVM、JRE、JDK

**JDK**：Java Development Kit java开发工具包

+ JDK是提供给Java开发人员使用的，其中包含了java的开发工具，也包括了JRE

**JRE**：Java Runtime Environment Java运行环境

+ JRE包括了Java虚拟机(Java Virtual Machine)和java程序所需要的核心类库等

**三者的关系**

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\JDKJREJVM.jpg)

### 3. Java的特点 --- 面向对象

#### 3.1 面向过程和面向对象

面向过程强调的是功能行为，以函数为最小单位，考虑怎么做

面向对象，将功能封装进对象，强调具备了功能的对象，以类/对象为最小单位，考虑谁来做

**面向对象的思想**

+ **类(Class)和对象(Object)**是面向对象的核心概念

  + 类是对一类事物的描述，是抽象的，概念上的定义

  + 对象是实际存在的该类事物的每个个体，因而也称为实例

+ 在Java中**万事万物皆对象**

  + 在Java语言的范畴中，都将功能、结构等封装到类中，通过类的实例化，来调用具体的功能结构
  + 涉及到Java语言与前端HTML、后端的数据库交互时，前后端的结构在Java层面交互时，都体现为类、对象

#### 3.2 面向对象的三大特征

##### 3.2.1 封装

程序设计的要求：**高内聚，低耦合**

+ 高内聚：类的内部数据操作细节自己完成，不允许外部干涉

+ 低耦合：仅对外暴露少量的方法用于使用

封装性的**设计思想**：隐藏对象内部的复杂性，只对外公开简单的接口，便于外界调用，从而提高系统的可扩展性，可维护性。通俗的说，**把该隐藏的隐藏起来，该暴露的暴露出来，这就是封装性的设计思想**

封装性的**体现**：

+ (1). 将类的属性私有化，同时，提供公共的方法来获取(getXXX)和设置(setXXX)属性的值

+ (2). 不对外暴露的私有方法

**JavaBean**：是一种Java语言写成的可重用组件

JavaBean的特点：

+ 类是公共的
+ 有一个无参的公共的构造器
+ 有属性，且有对应的get，set方法

用户可以使用JavaBean将功能、处理、值、数据库访问和其他任何可以用Java代码创建的对象进行打包，并且其他的开发者可以通过内部的jsp页面、Servlet、其他JavaBean、applet程序或者应用来使用这些对象，用户可以认为JavaBean提供了一种随时随地的复制粘贴功能，而不用关心任何改变



类的构造器

+ 作用：

  (1). 创建对象    (2). 初始化对象结构  

##### 3.2.2 继承

继承性的好处：

+ 减少了代码的冗余，提高了代码的复用性
+ 便于功能的扩展
+ 为之后多态性的使用，提供了前提

继承性的体现：一旦子类继承父类以后，子类中就获取了父类中声明的所有属性和方法

+ **注意：父类中声明为 private的属性和方法，子类继承父类之后，仍然认为获取了父类中私有的结构，只是因为封装性的影响，使得子类不能直接调用父类的结构而已**

Java中继承性的规定：

+ Java中只支持**单继承和多层继承**
  + 一个类可以被多个子类继承
  + 一个子类只能有一个父类
+ 子父类是相对的概念，子类继承父类之后，就获取了直接父类以及所有间接父类中所有的属性和方法

方法的重写：

+ 子类重写的方法必须和父类被重写的方法具有相同的方法名称和参数列表
+ 子类重写的方法的返回值类型不能大于(返回类型是父类返回类型或其子类)父类被重写的方法的返回值类型
+ 子类重写的方法使用的访问权限不能小于父类被重写的方法的访问权限
+ 子类不能重写父类中声明为private权限的方法

super关键字

+ super可以用来调用父类的属性、方法、构造器，调用时可以省略 super.
+ 当子类和父类中有同名的属性，方法时，要调用父类的属性就需要显示的写出"super."
+ 在子类的构造器中显示的使用"super(形参列表)"的方式，调用父类中声明的指定的构造器
+ "super(形参列表)"调用父类构造器必须写在子类构造器的首行
+ 在类的构造器中针对于"this(形参列表)"调用本类中的其他构造器和"super(形参列表)"调用父类的构造器只能二选一

**子类对象的实例化过程**：

+ 从结果上来看，创建子类对象之后，在堆空间中，就会加载所有父类中声明的属性

+ 从过程上看，当通过子类的构造器创建子类对象时，一定会直接或间接调用其父类的构造器，进而调用父类的父类的构造器，直到调用了java.lang.Object的空参构造器为止

  ![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\super.jpg)

+ 虽然在创建子类的对象时，调用了父类的构造器，但自始至终就创建过一个对象，就是子类的对象

##### 3.3.3多态

对象的多态性：**父类的引用指向子类的对象**

多态的使用：

+ 当调用子父类同名同参数的方法时，实际上执行的是子类重写父类的方法
+ 有了对象的多态性以后，在编译期，只能调用父类中声明的方法，但在运行的时候，实际执行的是子类重写父类的方法
+ 总结：编译看左，运行看右

多态性的使用前提：

+ 类的继承关系
+ 方法的重写

多态的应用：模板方法设计模式(TemplateMethod)

+ 抽象类体现的就是一种模板模式的设计，抽象类作为多个子类的通用模板，子类在抽象类的基础上进行扩展、改造，但子类总体上会保留抽象类的行为方式
+ 解决的问题：
  + 当功能内部一部分实现是确定的，一部分实现是不确定的，这时可以把不确定的部分暴露除去，让子类去实现
  + 在软件开发中实现一个算法时，整体步骤很固定、通用，这些步骤已经在父类中写好了，但是某些部分易变，易变的部分可以抽象出来，供不同子类实现

#### 3.3 对象的内存解析

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\MemoryStruct.jpg)

+ **堆：存放对象实例**，几乎所有的对象实例都在这里分配内存，java虚拟机中的规范描述是：所有的对象实例以及数组都要在堆上分配
+ **栈：**指虚拟机栈，虚拟机栈**用于存储局部变量**，局部变量表存放了编译期可知长度的各种基本数据类型(boolean,byte,char,short,int,float,long,double)、对象引用；方法执行完，自动释放内存
+ **方法区：**用于存储已被虚拟机加载的**类信息，常量，静态变量，即时编译器编译后的代码**等数据

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\object.jpg)

*注意：对象中的变量都是局部变量*

#### 3.4 变量类型

**实例变量：**每个对象都独立的拥有一套类中的非静态属性，当修改其中一个对象中的非静态属性时，不会导致其他对象中同样的属性值的修改

**静态变量：**多个对象共享同一个静态变量，当通过某一个对象修改静态变量时，会导致其他对象调用此静态变量时，是修改过的

**static修饰属性：**

+ 静态变量随着类的加载而加载，可以通过"类.静态变量"的方式进行调用
+ 静态变量的加载要早于对象的创建
+ 由于类只会加载一次，则静态变量在内存中也只会存在一份，保存在方法区的静态域中

静态变量的内存解析：

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\staticMemory.jpg)

**static修饰方法：静态方法**

+ 随着类的加载而加载，可以通过"类.方法"的方式进行调用
+ 静态方法中只能调用静态属性和方法，非静态方法即可以调用非静态方法和属性也能调用静态的方法和属性
+ 在静态方法中不能使用this，super关键字

**static修饰符的使用：**

+ 开发中，如何确定一个属性是否要声明为static

  属性是可以被多个对象所共享的，不会随着对象的不同而不同

+ 开发中，如何确定一个方法是否要声明为static

  操作静态属性的方法，通常设置为静态方法

代码块：

+ 用来初始化类、对象，只能使用static修饰或者没有修饰符

+ 静态代码块

  + 内部可以有输出语句

  + 随着类的加载而**执行**，只会执行一次
  + 初始化类的信息

+ 非静态代码块

  + 内部可以有输出语句
  + 随着对象的创建而执行，**每创建一个对象，就执行一次非静态代码块**
  + 可以在创建对象时，对对象的属性进行初始化

#### 3.5 一些重要的关键字

##### 3.5.1 abstract

abstract可以用来修饰**类、方法**

**抽象类：**

+ 不能够实例化
+ 抽象类中一定有构造器，便于子类实例化时调用
+ 抽象类中可以没有抽象方法

**抽象方法：**

+ 抽象方法只有方法声明，没有方法体
+ 包含抽象方法的类，一定是一个抽象类
+ 非抽象的子类必须要重写所有父类中的抽象方法

##### 3.5.2 interface

接口就是规范，定义的是一组规则；继承是一个"是不是"的关系，而接口是"能不能"的关系

+ 接口中不能定义构造器，接口不能实例化
+ 实现某个接口的类，必须实现接口中所有的方法
+ 接口与接口之间可以继承，并且可以多继承
+ JDK中：除了定义全局常量和抽象方法之外，还可以静态方法，默认方法

接口的应用：代理模式和工厂模式

+ **代理模式**：

  代理模式是Java开发中使用较多的一种设计模式，代理模式就是为其它对象提供一种代理以控制对这个对象的访问

  ```java
  public class NetWorkTest{
  	public static void main(String[] args){
          Server server = new Server();
          ProxyServer proxyServer = new ProxyServer(server);
          proxyServer.browse();
      }
  }
  
  interface NetWork{
  	public void browse();
  }
  
  //被代理类
  class Server implements NetWork{
  	@Override
  	public void browse(){
  		System.out.println("真实服务器访问网络");
  	}
  }
  //代理类
  class ProxyServer implements NetWork{
  
  	private NetWork netWork;
  	
  	public ProxyServer(NetWork netWork){
  		this.NetWork = netWork;
  	}
  	
  	public void check(){
  		System.out.println("连网之前的检查");
  	}
  	
  	@Override
  	public void browse(){
  		check();
  		netWork.browse();
  	}
  }
  ```

  代理模式的应用场景：

  + 安全代理：屏幕对真实角色的直接访问
  + 远程代理：通过代理类处理远程方法调用
  + 延迟加载：先加载轻量级的代理对象，真正需要再加载真实对象

+ **工厂模式**：

  实现了创建者与调用者的分离，即将创建对象的具体过程屏蔽隔离起来，达到提高灵活性的目的

  工厂模式的分类：

  + 简单工厂
  + 工厂方法
  + 抽象工厂

  ```java
  interface Car{
  	public void run();	
  }
  class Audi implements Car{
  	@Override
  	public void run(){
  		System.out.println("奥迪");
  	}
  }
  class BYD implements Car{
  	@Override
  	public void run(){
  		System.out.println("比亚迪");
  	}
  }
  class CarFactory{
      public static Car getCar(String type){
          if("奥迪".equals(type)){
              return new Audi();
          } else if("比亚迪".equals(type)){
              return new BYD();
          }
      }
  }
  ```


#### 3.6 内部类

Java中，允许一个类的定义位于另一个类的内部，前者称为内部类，后者称为外部类

内部类的分类：

+ 成员内部类：

  一方面作为外部类的成员

  + 调用外部类的结构
  + 可以被static修饰
  + 可以被四种不同的权限修饰(作为外部类只能被public和默认修饰符修饰)

  另一方面，作为一个类

  + 类内可以定义属性、方法、构造器等
  + 可以被final修饰，表示不能被继承
  + 可以被abstract修饰，表示不能被实例化

+ 局部内部类：方法内，代码块内，构造器内

```java
public class InnerClassTest{
    public static void main(String[] args){
        //创建内部类的实例(静态的成员内部类)：
        Person.Dog dog = new Person.Dog();
        
        //创建Bird实例(非静态的成员内部类)
        Person p = new Person();
        //变量.方法
        Person.Bird bird = p.new Bird();
    }
}

class Person{
    String name;
    int age;
    public void show(){
    }
    
	//静态成员内部类
	static class Dog{
	}
	//非静态成员内部类
	class Bird{
        //成员内部类中能够定义属性、方法、构造器
        String name;
        public void sing(){
            //可以调外部类的结构
            //Person.this.show();
            show();
        }
        public Bird(){}
	}
	
	{	//代码块内部的局部内部类
		class BB{
		}
	}	
	public void method(){
		//方法内的局部内部类
		class AA{
		}
	}
}
```



### 4. Java方法

#### 4.1 方法的重载

判断方法是否是重载方法：**两相同一不同**

两相同：在同一个类中，方法名相同

一不同：参数列表不同，或者是参数数目不同，或者是参数类型不同

**注意：判断是否为重载方法与方法的返回类型，方法体都没有关系**

#### 4.2 方法形参的值传递机制

变量的赋值：

+ 如果变量是基本数据类型，此时赋值的是变量所保存的数据值
+ 如果变量是引用数据类型(除去基本数据类型的其他类型)，此时赋值的是变量所保存的数据的地址

方法的值传递：

+ 如果参数是基本数据类型，此时实参赋值给形参的是实参所保存的数据值

+ 如果参数是引用数据类型，此时实参赋值给形参的是所保存的地址值

  ![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\dataTrans.jpg)

## 二、Java的数据类型

### 1. 数组

#### 1.1 一维数组

JVM中的内存结构

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\JVMMemeory.jpg)

一维数组在JVM中的创建，回收：

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\TrashReget.jpg)

在堆中，如果没有指针指向某块区域时，就会在不确定的时间将那块区域进行回收

#### 1.2 二维数组

二维数组初始化在JVM中的过程

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\ArrayTwo.jpg)

#### 1.3 对象数组

对象数组内存解析：

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\objectArray.jpg)

### 2. 包装类

针对八种基本数据类型定义相应的引用类型 ---- 包装类

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\Wrapper.jpg)

基本类型，包装类与String类间的转换

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\Wrapper_Base.jpg)

#### 2.1 自动拆箱和装箱

JDK5.0新特性：自动装箱和自动拆箱

**自动装箱**：基本数据类型 ----> 包装类

```java
int number = 1;
Integer in = number;    //可以直接将基本数据类型的对象赋值给包装类
```

**自动拆箱**：包装类 ----> 基本数据类型

```java
Boolean bool = true;
boolean b = bool;   //可以直接将一个包装类的数据赋值给基本数据类型
```

#### 2.2 关于包装类的细节

```java
public void test(){
	Object o1 = true ? new Integer(1) : new Double(2.0);
	System.out.println(o1);
}
//结果输出为1.0
```

在编译的时候要求等号右边的数据类型统一，所以Integer会被提升为Double，输出为1.0

```java
public void test(){
	Integer i = new Integer(1);
	Integer j = new Integer(1);
	System.out.println(i == j);   //false
	
	Integer m = 1; 
	Integer n = 1;
	System.out.println(m == n); //true
	
	Integer x = 128;
	Integer y = 128;
	System.out.println(x == y);  //false
}
```

在Integer中的源码中有：在内部声明了一个缓存，缓存中定义了一个Integer的数组，保存了[-128, 127]之间的整数，如果自动装箱的方式给Integer赋值的范围在-128 --- 127内时，可以直接使用数组中的元素，就不需要再new一个Integer对象了；当超过128时，就需要重新new一个对象

```java
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static final Integer[] cache;
    static Integer[] archivedCache;

    static {
        // high value may be configured by property
        int h = 127;
        String integerCacheHighPropValue =
            VM.getSavedProperty("java.lang.Integer.IntegerCache.high");
        if (integerCacheHighPropValue != null) {
            try {
                int i = parseInt(integerCacheHighPropValue);
                i = Math.max(i, 127);
                // Maximum array size is Integer.MAX_VALUE
                h = Math.min(i, Integer.MAX_VALUE - (-low) -1);
            } catch( NumberFormatException nfe) {
                // If the property cannot be parsed into an int, ignore it.
            }
        }
        high = h;

        // Load IntegerCache.archivedCache from archive, if possible
        VM.initializeFromArchive(IntegerCache.class);
        int size = (high - low) + 1;

        // Use the archived cache if it exists and is large enough
        if (archivedCache == null || size > archivedCache.length) {
            Integer[] c = new Integer[size];
            int j = low;
            for(int k = 0; k < c.length; k++)
                c[k] = new Integer(j++);
            archivedCache = c;
        }
        cache = archivedCache;
        // range [-128, 127] must be interned (JLS7 5.1.7)
        assert IntegerCache.high >= 127;
    }

    private IntegerCache() {}
}
```

### 3. 字符串

#### 3.1 String的特性

String类是一个final类，不可被继承

字符串是常量，用双引号表示，创建之后不能修改

String对象的字符内容是存储在一个char型的字符数组中

```java
private final char value[];
```

String类实现了Serialiazble接口，支持序列化

String类也实现了Comparable接口，可以比较大小

#### 3.2 String的内存解析

通过字面量的方式给一个字符串赋值，此时的字符串值声明在字符串常量池中，字符串常量池中是不会存储相同内容的字符串

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\String.jpg)

**字符串不可变的体现：**

+ 当字符串重新赋值时，需要重写指定内存区域赋值，不能使用原有的value进行赋值
+ 当对现有的字符串进行连接操作时，也需要指定内存区域赋值，不能使用原有的value值
+ 当调用String的replace方法修改字符或字符串时，也需要指定内存区域赋值

```java
String s1 = "abc";
String s2 = new String("abc");
```

两种方式创建字符串的区别

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\newString.jpg)

具体实例：

```java
class Person{
	String name;
	int age;
	Person(String name, int age){
		this.name = name;
		this.age = age;
	}
	Person(){}
}
@Test
public void test(){
    Person p1 = new Person("Tom", 20);
    Person p2 = new Person("Tom", 20);
    System.out.println(p1.name == p2.name);  //输出true
}
```

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\Person.jpg)

问题：

```java
public class StringTest{
	String str = new String("good");
	char[] ch = {'t', 'e', 's', 't'};
	
	public void change(String str, char[] ch){
		str = "test ok";
		ch[0] = 'b';
	}
	public static void main(String[] args){
		StringTest ex = new StringTest();
        ex.change(ex.str, ex.ch);
        System.out.println(ex.str);  //"good" String的不可变性
        System.out.println(ex.ch);   //"best"
	}
}
```



#### 3.3 字符串的拼接

+ 常量与常量的拼接结果在常量池中，且常量池中不会存在相同内容的常量

+ 只要有一个是变量，结果就在堆中

+ 如果拼接的结果调用intern()方法，返回值就在常量池中

  ![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\concate.jpg)

#### 3.4 String与byte[]的转换

```java
@Test
public void test(){
	String str1 = "abc123";
    //使用默认的字符集进行转换
	byte[] bytes = str1.getBytes();
	System.out.println(Arrays.toString(bytes));
}
//输出每个字符对应的ASCII码构成的字节数组
```

编码：String ---- byte[]

解码：byte[] ---- String

#### 3.5 StringBuffer和StringBuilder

**StringBuffer: **

继承于AbstractStringBuilder

```java
//1.8版本
StringBuffer sb1 = new StringBuffer();//new char[16]相当于底层创建了一个长度为16的数组
//底层构造器
public StringBuffer(){
	super(16);
}
public StringBuffer(String str) {
        super(str.length() + 16);
        append(str);
    }
//1.11版本，AbstractStringBuilder就使用byte[]来存储数据
public StringBuffer() {
        super(16);
    }
AbstractStringBuilder() {
        value = EMPTYVALUE;
    }
//private static final byte[] EMPTYVALUE = new byte[0];
```

扩容：如果要添加的数据底层数组装不下了，那就需要扩容底层数组

```java
public synchronized StringBuffer append(String str) {
        toStringCache = null;
        super.append(str); //调用父类的append方法
        return this;
    }
public AbstractStringBuilder append(String str) {
        if (str == null) {
            return appendNull();
        }
        int len = str.length();
    	//判断添加之后长度是否超过数组的容量
        ensureCapacityInternal(count + len);
        putStringAt(count, str);
        count += len;
        return this;
    }
//扩容规则：
private int newCapacity(int minCapacity) {
        // 扩容为原来容量的两倍 + 2
        int oldCapacity = value.length >> coder;
        int newCapacity = (oldCapacity << 1) + 2;
    //其他一些特殊情况
        if (newCapacity - minCapacity < 0) {
            newCapacity = minCapacity;
        }
        int SAFE_BOUND = MAX_ARRAY_SIZE >> coder;
        return (newCapacity <= 0 || SAFE_BOUND - newCapacity < 0)
            ? hugeCapacity(minCapacity)
            : newCapacity;
    }
```

**StringBuilder: **

继承于AbstractStringBuilder

底层源码与StringBuffer相似，只是方法中没有synchronize

常用方法总结：

增：append()方法，可以使用方法链的调用方式

删：delete(int start, int end)方法

改：setCharAt(int n ,char ch) / replace(int start, int end, String str)

查：charAt(int index)

长度：length()

遍历：for() + charAt()



String、StringBuffer、StringBuilder的异同：

String：不可变的字符序列；

StringBuffer：可变的字符序列；线程安全的，效率偏低

StringBuilder：可变的字符序列；jdk5.0新增，线程不安全的，效率高

三者底层都是使用char型数组存储(之后版本使用byte[]数组)

### 4. 集合

#### 4.1 集合框架

Collection接口：单列集合，用来存储一个个对象

+ List接口：存储有序的、可重复的数据 --- "动态"数组

  ArrayList、LinkedList、Vector

+ Set接口：存储无序的、不可重复的数据

  HashSet、LinkedHashSet、TreeSet

+ Map接口：双列集合，用来存储一对(key - value)

  HashMap、LinkedHashMap、TreeMap、Properties

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\Collection.jpg)

#### 4.2 Collection的常用方法

**contains(Object obj)**方法：判断当前集合中是否含有obj对象

+ 向Collection接口的实现类的对象中添加数据obj，要求obj所在的类重写equals()方法

**remove(Object obj)**方法：从当前集合中移除obj元素，删除成功返回true，否则返回false

+ 仍然是调用对象的equals方法判断对象是否在集合中

**retainAll(Collection coll)**方法：交集，获取当前结合和coll1集合的交集，并返回当前集合

#### 4.3 Collection子接口 --- List

List集合类中元素有序，且可重复，集合中的每个元素都有其对应的顺序索引

List容器中的元素都对应一个整型的序号记载其在容器中的为止，可以根据序号存取容器中的元素

主要实现类:

1. ArrayList: List接口主要实现类，线程不安全的，效率较高，底层使用Object[]实现

2. LinkedList: 当需要频繁地进行数据的插入，删除操作时，使用效率比ArrayList高，底层使用双向链表实现

3. Vector: List接口古老的实现类，在List接口出现之前就已经存在，线程安全，效率较低，底层使用Object[]实现

ArrayList源码：

```java
transient Object[] elementData; //ArrayList中默认存储数据的数组
//在使用ArrayList list = new ArrayList();底层Object[] elementData初始化为{}
//第一次调用add()方法时，底层会创建长度为10的数组，并将数据添加至elementData中
public ArrayList() {
        this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
    }
//private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};

//ArrayList的扩容机制：
private int newCapacity(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
    //一般情况下扩容为原来的1.5倍
        int newCapacity = oldCapacity + (oldCapacity >> 1);
    //如果扩容完之后仍然小于需求的容量
        if (newCapacity - minCapacity <= 0) {
            //如果是第一次添加，创建默认容量10或者需求的容量
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                return Math.max(DEFAULT_CAPACITY, minCapacity);
            if (minCapacity < 0) // overflow
                throw new OutOfMemoryError();
            return minCapacity;
        }
        return (newCapacity - MAX_ARRAY_SIZE <= 0)
            ? newCapacity
            : hugeCapacity(minCapacity);
    }
```

LinkedList源码：

```java
//头结点
transient Node<E> first;
//尾结点
transient Node<E> last;

LinkedList list = new LinkedList(); //内部声明Node类型的first和list属性，默认值为null
list.add(123); //调用add方法创建Node对象
```



#### 4.4 Collection子接口 --- Set

Set接口的实现类：

+ HashSet：作为Set接口的主要实现类，线程不安全，可以存储null值 ，底层数组 + 链表
+ LinkedHashSet：HashSet的子类
+ TreeSet：可以按照添加元素的指定属性进行排序
  + 向TreeSet中添加的数据，要求是相同类的对象
  + 两种排序方式：自然排序(实现Comparable接口)和定制排序(实现Comparator接口)
  + 自然排序：比较两个对象是否相同的标准为：compareTo()返回0，不再是equals()方法
  + 定制排序：比较；两个对象是否相同的标准为compare()返回0，也不是equals()方法

Set的存储**无序性和不可重复性**：

+ **无序性**：不等于随机性，存储的数据在底层数组中按照散列的方式进行添加，通过hashCode确定元素在数组中的位置
+ **不可重复性**：保证添加的元素按照equals()方法判断时，不能返回true，即：相同的元素只能添加一个

Set中添加元素的过程(HashSet)：

+ 首先调用元素所在类的hashCode()方法，计算元素的哈希值
+ 接着通过某种算法(散列函数)计算出元素在HashSet底层数组的存放位置
+ 判断数组此位置上没有其他元素，如果位置上没有其他元素，则元素直接添加成功
+ 如果此位置上有其他元素(以链表形式存储的多个元素)，则比较待添加元素和链表上元素的hash值
  + 如果hash值都不相同，则元素添加成功
  + 如果hash值相同，则需要调用待添加元素的equals()方法，如果返回true，添加失败；否则继续比较，直到与链表上的元素均不相同则添加成功

**Set示例：**

```java
HashSet set = new HashSet();
Person p1 = new Person(1001, "AA");
Person p2 = new Person(1002, "BB");

set.add(p1);
set.add(p2);
System.out.println(set);
//输出[Person{id=1001, name="AA"}, Person{id=1002, name="BB"}]

p1.name = "CC";
//执行remove方法时，使用hashCode()方法进行计算，算出待删除对象的位置，由于p1对象中的属性值发生变化，所以计算出的hashCode与之前的hashCode不一样，有很大概率不会定位到原来的位置上，所以不会删除原来位置上的p1对象
set.remove(p1);
System.out.println(set);
//输出[Person{id=1001, name="BB"}, Person{id=1002, name="CC"}]
set.add(new Person(1001, "CC"));
//在添加的时候，与上面的计算相同，原来位置上的p1，虽然是Person{1001, "CC"}但hashCode计算是由Person{1001, "AA"}计算出来的，所以一开始的hashCode就不相同，添加成功
System.out.println(set);
set.add(new Person(1001, "AA"));
//在添加时，计算出来的hashCode与没修改的p1相同，但调用equals()方法时，返回false，添加成功
System.out.println(set);
```



#### 4.5 Collection子接口 --- Map

Map的接口继承关系：

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\Map.jpg)

Map：双列数据，存储key - value对的数据

+ HashMap：作为Map的主要实现类，线程不安全，效率高；HashMap可以存储null的key和value，HashMap底层使用数组 + 链表 + 红黑树(jdk8.0之后)实现
  + LinkedHashMap：HashMap的子类，线程安全，效率低
+ TreeMap：保证按照添加的key - value对进行排序(**按照key排序**)，实现排序遍历，底层使用红黑树

Map的结构：

+ Map中的key：无序的，不可重复的，使用Set存储所有的key
+ Map中的value：无序的，可重复的，使用Collection存储所有的value
+ 一个键值对：key - value构成一个Entry对象
+ Map中的entry：无序的，不可重复的，使用Set存储所有的entry

HashMap底层实现原理：

+ new HashMap(): 底层没有创建一个长度为16的数组
+ jdk8.0底层的数组是Node[]数组，而非Entry[]
+ 首次调用put方法时，底层创建长度为16的数组
+ jdk8.0当中，Map的底层结构是数组 + 链表 + 红黑树，当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8且当前数组的长度 > 64时，此时此索引位置上的所有数据改为使用红黑树存储

HashMap底层源码：

加载因子：默认0.75，作用：提前扩容

临界值：容量 \* 加载因子，当容量超过临界值时，进行扩容

```java
//HashMap的底层数组
transient Node<K,V>[] table;

public HashMap() {
        this.loadFactor = DEFAULT_LOAD_FACTOR; //将加载因子赋值为0.75
    }
//put方法,hash(key)计算key的hash值
public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }

final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        if ((tab = table) == null || (n = tab.length) == 0)		 //如果是第一次调用，创建一个Node数组
            n = (tab = resize()).length;
    	//判断当前位置上是否有值，如果没有，直接添加
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        else {
            //如果当前位置上不为空
            Node<K,V> e; K k;
            //判断当前位置上的key与要添加的key的hash值比较
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k)))) //并且比较key是否相等
                e = p;  
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            else {
                //循环遍历当前位置上的所有key
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        //如果当前位置上只有一个元素，直接将添加的key链接到当前位置上key的后面
                        p.next = newNode(hash, key, value, null);
                //当当前位置上的节点数大于8(默认值)时，就将当前的链表转换为红黑树        
                        if (binCount >= TREEIFY_THRESHOLD - 1) 
                            treeifyBin(tab, hash);
                        break;
                    }
                    //判断当前位置上所有key与要添加的key是否相同(Hash和equals)
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            if (e != null) {
                //如果key相同，替换value的值
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }

//部分resize()代码
final Node<K,V>[] resize() {
        Node<K,V>[] oldTab = table;
        int oldCap = (oldTab == null) ? 0 : oldTab.length;
        int oldThr = threshold;
        int newCap, newThr = 0;
        if (oldCap > 0) {
            if (oldCap >= MAXIMUM_CAPACITY) {
                threshold = Integer.MAX_VALUE;
                return oldTab;
            }
            else if ((newCap = oldCap << 1) < MAXIMUM_CAPACITY &&
                     oldCap >= DEFAULT_INITIAL_CAPACITY)
                newThr = oldThr << 1; // double threshold
        }
        else if (oldThr > 0) // initial capacity was placed in threshold
            newCap = oldThr;
        else { //当第一次调用普通方法时              
            newCap = DEFAULT_INITIAL_CAPACITY;//初始的数组长度16
            newThr = (int)(DEFAULT_LOAD_FACTOR * DEFAULT_INITIAL_CAPACITY);  //临界值0.75 * 16 = 12
        }
        if (newThr == 0) {
            float ft = (float)newCap * loadFactor;
            newThr = (newCap < MAXIMUM_CAPACITY && ft < (float)MAXIMUM_CAPACITY ?
                      (int)ft : Integer.MAX_VALUE);
        }
        threshold = newThr;
   //创建数组，长度为16 	
        Node<K,V>[] newTab = (Node<K,V>[])new Node[newCap];
    table = newTab;
   ......
       return newTab;
}


```



### 5. 泛型

#### 5.1 泛型的基本概念

泛型就是允许在定义类，接口时通过一个标识标识类中某个属性的类型或者是某个方法的返回值及参数类型；这个类型参数在使用时确定

**泛型的优势：**

+ 解决元素存储的安全性问题

+ 解决获取数据元素时，需要类型强制转换的问题

  ![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\Generic.jpg)

#### 5.2 泛型的使用

泛型是jdk1.5之后的新特性

+ 集合接口或集合类在jdk5.0时都修改为带泛型的结构
+ 在实例化集合类时，可以指明具体的泛型类型
+ 指明完以后，在集合类或接口中凡是定义类或接口时，内部结构(方法，构造器，属性等)使用泛型进行定义
+ 泛型的类型必须是类，不能是基本数据类型，需要用到基本数据类型的位置，拿包装类替换
+ 如果实例化时，没有指明泛型的类型，默认类型为java.lang.Object类型

**自定义泛型：**

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\SelfGeneric.jpg)

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\SelfGeneric2.jpg)

泛型方法：在方法中出现了泛型的结构，**泛型参数与类的泛型参数没有任何关系**，泛型方法可以声明为static，泛型参数是在调用方法时确定的，并非是在实例化时确定

**泛型的继承方式：**

+ 假设A类是B类的父类，但是泛型类型List\<A>与List\<B>不具备子父类关系
+ 通配符( ? )的使用，List\<A>与List\<B>没有关系，它们的公共父类是List\<?>
+ 对于使用通配符的集合比如List\<?>，不能向其内部添加数据，但可以添加null值；允许读取数据，读取的数据类型为Object
+ 有限制条件的通配符的使用：List\<? extends A>；List\<? super A>其中A代表可以是任何类

### 6. IO流

#### 6.1 File类

java.io.File类：文件和文件目录路径的抽象表示形式，与平台无关

File能新建、删除、重命名文件和目录，但**File不能访问文件内容本身**，如果需要访问文件内容本身，需要使用输入输出流

想要在Java程序中表示一个真实存在的文件或目录，那么必须有一个File对象，但是Java程序中的一个File对象，可能没有一个真实存在的文件或目录

File对象可以作为参数传递给流的构造器

File类型的常用方法：

```java
public String getAbsolutePath();  //获取绝对路径
public String getPath();  //获取路径
public String getName();  //获取名称
public String getParent(); //获取上层文件目录路径，若无，返回null
public long length();  //获取文件长度，即字节数
public long lastModified(); //获取最后一次的修改时间，毫秒值
public String[] list(); //获取指定目录下的所有文件或者文件目录的名称数组
public File[] listFiles(); //获取指定目录下的所有文件或者文件目录的File数组

public boolean renameTo(File dest); //把文件重命名为指定的文件路径
```

#### 6.2 IO流

IO技术常用于处理设备之间的数据传输，如读写文件，网络通讯等

Java程序中，对于数据的输入输出操作以"流(Stream)"的方式进行

java.io包下提供了各种"流"类和接口，用以获取不同种类的数据，并通过标准的方法输入或输出数据

Java IO原理：

+ 输入：读取外部数据到内存中
+ 输出：将程序数据输出到磁盘、光盘等存储设备中

流的分类：

+ 按照操作数据单位的不同分为：字节流(8bit)，字符流(16bit)

+ 按照数据流的流向不同分为输入流，输出流

  | 抽象基类   | 字节流       | 字符流 |
  | ---------- | ------------ | ------ |
  | **输入流** | InputStream  | Reader |
  | **输出流** | OutputStream | Writer |

IO流体系：

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\IO.jpg)

流读入操作的具体过程：

```java
@Test
public void test(){
	//1. 实例化File对象，指明要操作的文件
	File file = new File("hello.txt"); //当前Module下的文件
	//2. 提供具体的流
	FileReader fr = new FileReader(file);
	//3. 数据的读入
	//read()方法：返回读入的一个字符，如过达到文件末尾，返回-1
    int data = fr.read();
    while(data != -1){
        System.out.print((char) data);
        data = fr.read();
    }
    
    //4. 关闭流
    fr.close();
}
```

对读入操作的扩展：

```java
File file = new File("hello.txt");
FileReader fr = new FileReader(file);
char[] cBuffer = new char[5];
int l;
//int read(char[] c)：返回每次读入到数组中的字符个数，如果达到文件末尾，返回-1
while((l = fr.read(char)) != -1){
	for(int i = 0; i < l; i++){
        System.out.print(cBuffer[i]);
    }
}
fr.close();
```

流写操作具体流程：

```java
/*
输出操作，对应的File可以不存在，如果不存在目标文件，输出过程中会自动创建此文件；如果目标文件存在，可以指定是对原有文件覆盖还是追加
FileWriter fw = new FileWriter(file, true);
在原有文件内容之后追加
FileWriter fw = new FileWriter(file, false);
或者
FileWriter fw = new FileWriter(file);
对原有文件的覆盖
*/
@Test
public void test(){
	//1. 提供File类的对象，指明写出到的文件
	File file = new File("hello.txt");
	//2. 提供FileWriter的对象，用于数据的写出
	FileWriter fw = new FileWriter(file);
	//3. 写出的操作
	fw.write("Hello world");
	//4. 流的关闭
	fw.close();
}
```

图片复制操作：

```java
@Test
public void test(){
	File srcFile = new File("picture1.jpg");
	File destFile = new File("picture2.jpg");
	//字符流不能用来处理图片，同样的字节流不能用来处理文本
	FileInputStream fis = new FileInputStream(srcFile);
	FileOutputStream fos = new FileOutputStream(destFile);
	
	byte[] buffer = new byte[5];
	int len;
	while((len = fis.read(buffer)) != -1){
		fos.write(buffer, 0, len);
	}
	fos.close();
	fis.close();
}
```

缓冲流的使用：

```java
//实现非文本文件的复制
@Test
public void test(){
	File srcFile = new File("p1.jpg");
	File destFile = new File("p2.jpg");
	
	//2. 造流，先造结点流，再造缓冲流
	FileInputStream fis = new FileInputStream(file);
	FileOutputStream fos = new FileOutputStream(file);
	
	BufferedInputStream bis = new BufferedInputStream(fis);
	BufferedOutputStream bos = new BufferedOutputstream(fos);
	
	//3. 复制操作
	byte[] buffer = new byte[10];
	int len;
	while((len = bis.read(buffer)) != -1){
		bos.write(buffer, 0, len);		
	}
    
    //4. 资源的关闭，先关闭外层流（缓冲流），再关闭内层流（结点流）
    bos.close();
    bis.close();
    //在关闭外层流的同时，内层流会自动进行关闭
    // fis.close();
    // fos.close();
}

//实现文本文件的复制
@Test
public void test2(){
    BufferedReader br = new BufferedReader(new FileReader("hello.txt"));
    BufferedWriter bw = new BufferedWriter(new FileWriter("hello1.txt"));
    
    String data;
    while((data = br.readLine()) != null){
        bw.write(data);
        bw.newLine();  //换行操作
    }
    
    br.close();
    bw.close();
}
```

#### 6.3 转换流

转换流提供了字符流和字节流之间的转换，转换流属于字符流

Java API提供了两个转换流：

+ InputStreamReader: 将InputStream转换为Reader
+ OutputStreamWriter: 将Writer转换为OutputStream

转换流的基本操作：

```java
@Test
public void test(){
	FileInputStream fis = new FileInputStream("test.txt");
	InputStreamReader isr = new InputStreamReader(fis, "UTF-8"); //指明字符集，具体使用哪个字符集，取决于文件保存时使用的字符集
    char[] buffer = new char[1024];
    int len;
    while((len = isr.read(buffer)) != -1){
        String str = new String(buffer, 0, len);
        System.out.println(str);
    }
    isr.close();
}
```

#### 6.4 其他流

**标准输入输出流**：

+ System.in: 标准输入流，默认从键盘输入

+ System.out: 标准输入流，默认从控制台输出

  ```java
  //System类的setIn(InputStream is) / setOut(PrintStream ps)方式重新指定输入输出流
  //从键盘输入，从控制台输出
  @Test
  public void test(){
      InputStreamReader isr = new InputStreamReader(System.in);
      BufferedReader gr = new BufferedReader(isr);
      while(true){
          String data = br.readLine();
          if("exit".equals(data)){
              break;
          }
          System.out.println(data.toUpperCase());
      }
      br.close();
  }
  ```

  

**打印流：**

+ PrintStream和PrintWriter
+ 提供了一系列重载的println方法用于多种数据的输出

**对象流：**

+ 用于存储和读取基本数据类型数据或对象的处理流，它的强大之处就是可以把java中的对象写入到数据源中，也能把对象从数据源中还原回来
+ 序列化：用ObjectOutputStream类保存基本类型数据或对象的机制
+ 反序列化：用ObjectInputStream类读取基本类型数据或对象的机制
+ ObjectOutputStream和ObjectInputStream不能序列化static和transient修饰的成员变量

对象的序列化机制：

+ 允许把内存中的Java对象转换成平台无关的二进制流，从而允许把这种二进制流持久地保存在磁盘上，或通过网络将这种二进制流传输到另一个网络节点，当其他程序获取了这种二进制流，就可以恢复成原来的Java对象
+ 序列化的好处在于可将任何实现了Serializable接口的对象转换为字节数据，使其在保存和传输时可被还原
+ 序列化是RMI(远程方法调用)过程的参数和返回值都必须实现的机制，而RMI是JavaEE的基础，因此序列化机制是JavaEE平台的基础

```java
//序列化过程
@Test
    public void testObjectOutputStream(){
        ObjectOutputStream oos = null;
        try {
            oos = new ObjectOutputStream(new FileOutputStream("test.dat"));
            oos.writeObject(new String("I am programmer"));
            oos.flush();
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if(oos != null){

                try {
                    oos.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
//反序列化过程：将磁盘文件中的对象还原为内存中的一个java对象
@Test
    public void testInputStream(){
        ObjectInputStream ois = null;
        try {
            ois = new ObjectInputStream(new FileInputStream("test.dat"));
            Object obj = ois.readObject();
            System.out.println(obj);
        } catch (IOException e) {
            e.printStackTrace();
        } catch (ClassNotFoundException e) {
            e.printStackTrace();
        } finally {
            if(ois != null){

                try {
                    ois.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
    }
```

自定义序列化对象：

类需要满足：

+ 需要实现接口：Serializable
+ 需要当前类提供一个全局常量 serialVersionUID

```java
public class Person implements Serializable{
    public static final long serialVersionUID = 654646846846L;
}
```



### 7. 反射

反射被视为动态语言的关键，反射机制允许程序在**执行期间**借助于反射API取得任何类的内部信息，**并能直接操作任意对象的内部属性及方法(包括私有的)**

Java反射机制的应用：

+ 在运行时判断任意一个对象所属的类
+ 在运行时构造任意一个类的对象
+ 在运行时判断任意一个类所具有的成员变量和方法
+ 在运行时获取泛型信息
+ 在运行时调用任意一个对象的成员变量和方法
+ 在运行时处理注解
+ 生成动态代理

Java反射常用的API：

+ java.lang.Class
+ java.lang.reflect.Method：代表类的方法
+ java.lang.reflect.Field：代表类的成员变量
+ java.lang.reflect.Constructor：代表类的构造器

#### 7.1 反射的实例操作

使用反射创建一个类的对象：

```java
@Test
    public void test1(){
        Class<Person> pClass = Person.class;
        try {
            //获取类的构造器
            Constructor<Person> cons = pClass.getConstructor(String.class, int.class);
            //创建对象
            Person person = cons.newInstance("Tom", 12);
            System.out.println(person);
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```

通过反射调用类的属性和方法：

```java
            Field age = pClass.getDeclaredField("age");
            age.set(person, 20);
            Method show = pClass.getDeclaredMethod("show");
            show.invoke(person);
```

通过反射调用类的私有构造器、属性和方法：

```java
 Field name = pClass.getDeclaredField("name");
//设置属性可访问
            name.setAccessible(true);
            name.set(person, "Jerry");
            System.out.println(person);
            Method showNation = pClass.getDeclaredMethod("showNation", String.class);
            showNation.setAccessible(true);
```

#### 7.2 Class类

类的加载过程：

+ 程序经过javac.exe命令之后，会生成一个或多个字节码文件(.class结尾)
+ 使用java.exe命令，对某个java字节码文件进行解释运行，相当于将某个字节码文件加载到内存中，此过程就称为类的加载过程
+ 加载到内存中的类，就称为运行时类，此运行时类就作为Class的一个实例

获取Class实例的方式：

```java
//方式一：调用运行时类的属性.class
Class pClass1 = Person.class;

//方式二：通过运行时类的对象，调用getClass()方法
Person p1 = new Person();
Class pClass2 = p1.getClass();

//方式三：调用Class的静态方法：forName(String classPath)
Class pClass3 = class.forName("com.JavaReflect.Person"); //全类名

System.out.println(pClass1 == pClass2); //返回true

//方式四：使用类的加载器：ClassLoader
ClassLoader loader = ReflectTest.class.getClassLoader();
            Class<?> loadClass = loader.loadClass("com.JavaReflect.Person");

```

*注意：加载到内存中的运行时类，会缓存一段时间，在此时间之内，可以通过不同的方式来获取此运行时类，不同的方式获取到的运行时类是相同的*

**哪些类型可以有Class对象**：

+ class，外部类，内部类，局部内部类，匿名内部类
+ interface，接口
+ []：数组，只要数组的元素类型与维度一样就是同一个Class
+ enum：枚举
+ annotation：注解
+ void

**类的加载过程：**

+ 将类的class文件读入内存，并为之创建一个java.lang.Class对象，此过程由类加载器完成
+ 将类的二进制数据合并到JRE中
+ JVM负责对类进行初始化

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\classLoad.jpg)

#### 7.3 类的加载器

类加载的作用：将class文件字节码内容加载到内存中，并将这些静态数据**转换成方法区的运行时数据结构**，然后在堆中生成一个代表这个类的java.lang.Class对象，作为方法区中类数据的访问入口

类缓存：标准的JavaSE类加载器可以按要求查找类，但一旦某个类被加载到类加载器中，它将维持加载一段时间，不过JVM垃圾回收机制可以回收这些Class对象

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\ClassLoader.jpg)

**使用ClassLoader加载配置文件**

```java
@Test
public void test1(){
    Properties properties = new Properties();
    //使用ClassLoader加载配置文件
    ClassLoader classLoader = ClassLoaderTest.class.getClassLoader();
    //文件在模块的src文件夹下
    InputStream stream = classLoader.getResourceAsStream("jdbc.properties");
    try {
        properties.load(stream);
        String username = properties.getProperty("username");
        String password = properties.getProperty("password");
        System.out.println("username = " + username);
        System.out.println("password = " + password);
    } catch (IOException e) {
        e.printStackTrace();
    }
}
```

#### 7.4 通过反射创建运行时类对象

实例：

```java
@Test
public void test(){
    Class<Person> pClass = Person.class;
    try {
        Person person = pClass.getDeclaredConstructor().newInstance();
        System.out.println(person);
    } catch (Exception e) {
        e.printStackTrace();
    }
}
```

**newInstance()方法：**调用此方法，创建对应运行时类的对象，实际上还是调用类的空参构造器来创建对象，所以要求类中必须有空参构造器

```java
//使用反射创建运行时类的不确定性
@Test
public void test1() throws Exception {
    String classPath = "";
    for (int i = 0; i < 10; i++) {
        int num = new Random().nextInt(3);
        switch (num){
            case 0:
                classPath = "java.util.Date";
                break;
            case 1:
                classPath = "java.lang.Object";
                break;
            case 2:
                classPath = "com.JavaReflect.Person";
        }
        Object instance = getInstance(classPath);
        System.out.println(instance);
    }

}

public Object getInstance(String classPath) throws Exception {
    Class<?> object = Class.forName(classPath);
    return object.getDeclaredConstructor().newInstance();
}
```

#### 7.5 获取运行时类的完整结构

获取运行时类的属性：

```java
@Test
public void test(){
    Class<Person> personClass = Person.class;
    //getFields()方法：获取当前运行时类及其父类中声明为public访问权限的属性
    Field[] fields = personClass.getFields();
    for (Field field: fields) {
        System.out.println(field);
    }
    //getDeclaredFields()方法：获取当前运行时类中声明的所有属性，不包含父类中声明的属性
    Field[] declaredFields = personClass.getDeclaredFields();
        for (Field field: declaredFields) {
            System.out.println(field);
        }
}
```

获取属性的结构：权限修饰符，数据类型，变量名：

```java
@Test
public void test1(){
    Class<Person> personClass = Person.class;
    Field[] fields = personClass.getDeclaredFields();
    for (Field field: fields) {
        //1. 获取权限修饰符，默认使用int值表示
        int modifiers = field.getModifiers();        System.out.print(Modifier.toString(modifiers) + "\t");
            //2. 数据类型
            Class<?> type = field.getType();
            System.out.print(type.getName() + "\t");
            //3. 变量名
            String name = field.getName();
            System.out.println(name);
    }
}
```

#### 7.6 动态代理

代理模式的原理：使用一个代理将对象包装起来，然后用该代理对象取代原始对象，**任何对原始对象的调用都要通过代理**，代理对象决定是否以及何时将方法调用转到原始对象上

动态代理：动态代理指客户**通过代理类来调用其他对象的方法**，并且是在程序运行时根据需要动态创建目标类的代理对象

动态代理的示例：

```java
public interface UserDAO {
    public void login(String username, String password);
}
public class UserDaoImpl implements UserDAO{
    @Override
    public void login(String username, String password) {
        if("admin".equals(username) && "123456".equals(password)){
            System.out.println("login successful");
        } else{
            System.out.println("login failed");
        }
    }
}
class ProxyFactory{
    ////调用此方法，返回一个代理类的对象
    //obj: 被代理对象
    public static Object getProxyInstance(Object obj){
        //newProxyInstance的三个参数：
        //ClassLoader：类加载器
        //Class<?>[] interfaces：被代理对象所在类实现的接口
        //Interface InvocationHandler：执行被代理对象中被增强的方法
        return Proxy.newProxyInstance(obj.getClass().getClassLoader(), obj.getClass().getInterfaces(),
                new InvocationHandler() {
                    //当通过代理类对象，调用方法a时，就会自动地调用invoke方法
            //将被代理类要执行的方法a的功能声明在invoke方法中
                    @Override
                    public Object invoke(Object proxy, Method method, Object[] args) throws Throwable {
                        //proxy：代理类对象
                //method: 代理类调用的方法
                        System.out.println("before logging...check authority");
                        Object returnObj = method.invoke(obj, args);
                        System.out.println("after logging...get into main page");
                        return returnObj;
                    }
                });
    }
}
public class ProxyTest {
    @Test
    public void testProxy(){
        UserDAO userDAO = new UserDaoImpl();
        UserDAO proxyInstance = (UserDAO) ProxyFactory.getProxyInstance(userDAO);
        proxyInstance.login("admin", "123456");
    }
}
输出结果：
before logging...check authority
login successful
after logging...get into main page
```

### 8. 多线程

#### 8.1 线程的生命周期

新建：当一个Thread类或其子类的对象被声明并创建时，新生的线程对象处于新建状态

就绪：处于新建状态的线程被start()后，将进入线程队列等待CPU时间片

运行：当就绪的线程被调用并获得CPU资源时，便进入运行状态，run()方法定义了线程的操作和功能

阻塞：在某种特殊情况下，被人为挂起或执行输入输出操作时，让出CPU并临时终止自己的执行，进入阻塞状态

死亡：线程完成了它的全部工作或线程被提前强制性中止或出现异常导致结束

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\线程生命周期.jpg)

#### 8.2 线程安全问题

多个线程执行的不确定性引起执行结果的不稳定

多个线程对账本的共享，会造成操作的不完整性，会破外数据

典型案例：取钱

假设银行卡中有3000元，现在同时有两个请求，都需要取2000元，两个线程同时进入判断 if(2000 < 3000)，最后取出之后会发现卡中只剩下-1000元

卖票问题：

```java
public class Window implements Runnable{
	private int tickets = 100;
	
	@Override
	public void run(){
		while(true){
			if(tickets > 0){
				System.out.println(Thread.currentThread().getName() + " : ticket" + tickets--);
			}
		}
	}
	
}
```

出现问题：错票，重票，当某个线程操作的过程中，还未操作完毕，其他的线程就参与进来，也操作车票

解决方法：当一个线程在操作tickets时，其他线程不能参与进来，直到当前线程操作完毕之后，其他线程才可以开始操作tickets

**synchronized**

+ 同步代码块

  ```java
  synchronized(同步监视器){
  	//操作
  }
  ```

  共享数据：多个线程需要共同操作的变量

  同步监视器：锁，**任何一个类的对象都可以充当锁，多个线程必须要共用同一把锁**

+ 同步方法

  ```java
  public synchronized void function(){
  	//操作共享数据的代码
  }
  ```

  如果操作共享数据的代码刚好被定义在同一个方法中，那么就可以将这个方法声明为synchronized

#### 8.3 Lock

java.util.concurrent.locks.Lock接口是控制多个线程对共享资源进行访问的工具

ReentrantLock类实现了Lock，它拥有与synchronized相同的并发性和内存语义，在实现线程安全的控制中，比较常用的是ReentrantLock，可以显示加锁、释放锁

```java
//Lock锁的使用
public class Window implements Runnable{
    private int tickets = 100;
    //创建Lock的实例对象
    private Lock lock = new ReentrantLock();
    
    @Override
    public void run(){
        while(true){
            try{
                lock.lock();
                
                if(tickets > 0){
                    try{
                        Thread.sleep(100);
                    } catch(Exceprion e){
                        e.printstack();
                    }
                    
                    System.out.println(Thread.currentThread().getName() + ": ticket " + tickets--);
                }
            } finally{
                //释放锁
                lock.unlock();
            }
            
        }
    }
}
```



**Lock与synchronized的区别**

+ synchronized机制在执行完相应的代码逻辑之后，自动的释放同步监视器；Lock的方式需要手动地启动同步和释放同步
+ Lock只有代码块锁，synchronized有代码块锁和方法锁
+ 使用Lock锁，JVM将花费较少的来调度线程，性能更好

#### 8.4线程通信

示例：两个线程交替打印1-100

```java
public class Number implements Runnable{
	int num = 1;
	
	@Override
	public void run(){
		while(true){
            synchronized(this){
                //唤醒阻塞的线程
                notify();
               if(num <= 100){
				System.out.println(Thread.currentThread().getName() + ":" + num);
				num++;
                try{
                    //调用wait方法线程进入阻塞状态，并且会释放锁
                	wait();    
                } catch(Exception e){
                    e.printStackTrace();
                }   
                    
			} else{
				break;
			} 
          }
			
		}
	}
}

class Test{
    @Test
    public void test(){
        Number number = new Number();
        
        Thread t1 = new Thread(number);
        Thread t2 = new Thread(number);
    }
}
```

+ wait()方法：一旦执行此方法，当前线程就进入阻塞状态，并释放同步锁
+ notify()：一旦执行此方法，就会唤醒被线程被wait的一个线程，如果有多个线程，就唤醒优先级高的那个
+ notifyAll()：一旦执行此方法，就会唤醒所有被wait的线程
+ 这个三个方法是线程通信的方法，必须使用在同步代码块或同步方法中，调用者都是同步代码块的同步监视器
+ 这三个方法都声明在Object类中

*sleep和wait的异同*：

> 相同点：一旦执行方法都可以让当前线程进入阻塞状态
>
> 不同点：
>
> ​	1). 两个方法声明的位置不同，sleep方法在Thread类中；wait方法声明在Object类中
>
> ​	2). 调用的范围不同：sleep方法的调用没有限制；wait方法必须由同步监视器调用，所以必须在同步代码块和同步方法中调用
>
> ​	3). sleep方法不会释放同步监视器，而wait方法会释放同步监视器 

#### 8.5 创建线程的方式

**1、继承Thread类，重写run方法**



**2、实现Runnable接口，实现run方法**



**3、实现Callable接口，实现call()方法**

+ 创建一个实现Callable的实现类
+ 实现call方法，将此线程需要执行的操作声明在call方法中
+ 创建Callable接口实现类的对象
+ 将此Callable的实现类对象作为参数传递到FutureTask构造器中，去创建FutureTask的对象
+ 将FutureTask的对象作为参数传递到Thread类的构造器中，并调用start()方法启动线程

```java
//说明：Callable接口支持泛型Callable<T>
class NumberThread implements Callable{
	
    @Override
    public Object call throws Exception{
        int sum = 0;
        for(int i = 0; i <= 100; i += 2){
            System.out.println(i);
            sum += i;
        }
        return sum;
    }
}

class TestThread{
    @Test
    public void test(){
        NumberThread num = new NumberThread();
        //需要使用Future接口的实现类来创建线程
        FutureTask f = new FutureTask(num);
        Thread t = new Thread(f);
        t.start();
       	try{
            //get()方法的返回值即为Callable实现类重写的call方法的返回值
            Object sum = f.get();
            System.out.println(sum);
        } catch(Exception e){
            e.printStackTrace();
        }
 		       
    }
}
```

相比于Runnable接口创建线程，实现Callable创建线程的优势：

+ call方法可以有返回值
+ call方法可以抛出异常，被外面的操作捕获，获取异常的信息
+ Callable接口支持泛型

**4、使用线程池创建线程**

为什么使用线程池：经常创建和销毁、使用量特别大的资源，比如并发情况下的线程对性能影响很大

方法：提前创建好多个线程，放入线程池中，使用时直接获取，使用完放回池中，可以避免频繁地创建销毁、实现重复利用，类似生活中的公共交通工具

**线程池的好处：**

+ 提高相应速度，减少了创建线程的时间

+ 降低资源消耗，重复利用线程池中线程，不需要每次都要创建

+ 便于线程的管理

  corePoolSize: 核心池的大小

  maximumPoolSize: 最大线程数

  keepAliveTime: 线程没有任务时最多保持多长时间后会终止

  ...

```java
public class ThreadPoolTest{
	@Test
    public void test(){
        //Executor:创建线程池的工具类
        //ExecutorService: 接口
        ExecutorService service = Executors.newFixedThreadPool(10);
        //可以设置线程池的属性，需要使用ExecutorService的实现类设置
        ThreadPoolExecutor pool = (ThreadPoolExecutor) service;
        pool.setCorePoolSize(15);
        //pool.setxxx();...
        
        //传入实现Runnable接口的线程
        service.execute(new Runnable(){
           @Override
            public void run(){
                //线程要做的事情
                ...
            }
        });
        //传入实现Callable接口的线程
        service.submit(new Callable(){
            @Override
            public Object call(){
                //线程要做的事情
                ...
            }
        });
        //关闭线程池
        service.shutdown();
    }
}
```



## 三、设计模式

### 1. 单例模式

基本介绍：

单例模式就是采取一定的方法保证在整个的软件系统中，对某个类只能存在一个对象实例，并且该类只提供一个取得其对象实例的方法

单例模式的实现：

饿汉式：在系统加载的时候就创建对象

```java
//饿汉式单例
public class Bank {
    //内部创建类的对象，要求也必须是静态的
    private static Bank bank = new Bank();
    //私有化类构造器
    private Bank(){
    }
    //提供公共的静态方法，返回类对象
    public static Bank getInstance(){
        return bank;
    }
}
```

懒汉式：在需要对象的时候创建对象

```java
//懒汉式单例
public class Order {
    //私有化类构造器
    private Order(){}
    //声明当前类的实例变量
    private static Order order = null;
    //声明一个public静态的返回当前类对象的方法
    public static Order getInstance(){
        return order == null ? new Order() : order;
    }
}
```

单例模式的优点：减少了系统性能开销，当对象的产生需要比较多的资源时，如读取配置、产生其他依赖对象时，则可以通过在应用启动时直接产生一个单例对象，然后永久驻留内存的方式来解决

### 2. 工厂模式

#### 2.1 简单工厂



#### 2.2 工厂方法



### 3. 原型模式

**基本介绍：**

1、原型模式(prototype)是指：用原型实例指定创建对象的种类，并且通过拷贝这些原型创建新的对象

2、原型模式是一种创建型设计模式，允许一个对象再创建另外一个可定制的对象，无需直到如何创建的细节

3、工作原理：通过将一个原型对象传给那个要创建的对象，待创建的对象通过请求原型对象的拷贝创建对象，即对象.clone()

```java
//clone()方法的使用，需要所在的类实现Cloneable接口
public class Sheep implements Cloneable{
    private String name;
    private Integer age;
    private String color;

    public Sheep() {
    }

    public Sheep(String name, Integer age, String color) {
        this.name = name;
        this.age = age;
        this.color = color;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public Integer getAge() {
        return age;
    }

    public void setAge(Integer age) {
        this.age = age;
    }

    public String getColor() {
        return color;
    }

    public void setColor(String color) {
        this.color = color;
    }

//    @Override
//    public String toString() {
//        return "Sheep{" +
//                "name='" + name + '\'' +
//                ", age=" + age +
//                ", color='" + color + '\'' +
//                '}';
//    }

    //克隆该实例，使用默认的clone方法来完成
    @Override
    public Object clone() throws CloneNotSupportedException {
        Sheep sheep = null;
        sheep = (Sheep) super.clone();
        return sheep;
    }
}
//测试
public class PrototypeTest {
    @Test
    public void testPrototype() throws CloneNotSupportedException {
        Sheep sheep = new Sheep("Tom", 1, "white");
        Sheep sheep1 = (Sheep) sheep.clone();
        System.out.println(sheep);
        System.out.println(sheep1);
    }
}
输出：
com.DesignPattern.Prototype.Sheep@c81cdd1
com.DesignPattern.Prototype.Sheep@1fc2b765
```

从上面的例子中可以看出，使用clone方法创建的对象的地址是不同的，即clone方法创建了一个新的对象，而不是把元对象的地址赋给了一个新的引用变量

区分两个概念：深拷贝  --- 浅拷贝

```java
/*
在Sheep类对象的克隆过程中，name属性是String类型的，是一个引用类型
对它的拷贝有两种方式： 
	1、直接将源对象中的name的引用值拷贝给新对象的name字段   浅拷贝
	2、或者是根据原Sheep对象中的name指向的字符串对象创建一个新的相同的字符串对象，将这个新字符串对象的引用赋给新拷贝的Sheep对象的name字段。    深拷贝
这两种拷贝方式分别叫做浅拷贝和深拷贝。
*/
public class PrototypeTest {
    @Test
    public void testPrototype() throws CloneNotSupportedException {
        Sheep sheep = new Sheep("Tom", 1, "white");
        Sheep sheep1 = (Sheep) sheep.clone();
        System.out.println(sheep.getName() == sheep1.getName());
    }
}
输出：true
```

**所以，clone方法执行的是浅拷贝**

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\clone.png)

如果要实现深拷贝，那么需要除了调用源对象类中的clone方法得到新的对象， 还要将该类中的引用变量也clone出来

```java
//使用clone方法实现深拷贝
class Body implements Cloneable{
		public Head head;
		public Body() {}
		public Body(Head head) {this.head = head;}
 
		@Override
		protected Object clone() throws CloneNotSupportedException {
			Body newBody =  (Body) super.clone();
			newBody.head = (Head) head.clone();
			return newBody;
		}
		
	}
class Head implements Cloneable{
		public  Face face;
		
		public Head() {}
		public Head(Face face){this.face = face;}
    //属性的引用类型也需要实现Cloneable接口并重写clone方法
		@Override
		protected Object clone() throws CloneNotSupportedException {
			return super.clone();
		}
	} 

```

**原型模式在Spring中的使用：**

Spring中原型bean的拆功能键，就是原型模式的应用

```java
/*
配置文件: beans.xml
<bean id="id01" class="com.xxx.Monster" scope="prototype"/>
*/
public class Test{
	@Test
	public void test(){
		ApplicationContext applicationContext = new ClassPathXmlApplicationContext("beans.xml");
		Object bean1 = applicationContext.getBean("id01");
		System.out.println(bean1);
		Object bean2 = applicationContext.getBean("id01");
		System.out.println(bean2);
		System.out.println(bean1 == bean2); //返回false，使用原型模式进行克隆
	}
}
```

### 4. 适配器模式

基本介绍：

1、适配器模式(Adaptor Pattern)将一个类的接口转换成客户希望的另外一个接口，使得原本由于接口不兼容而不能一起工作的那些类能一起工作，其别名为包装器(Wrapper)

在软件设计领域就是：需要开发的具有某种业务功能的组件在现有的组件库中已经存在，但它们与当前系统的接口规范不兼容，如果重新开发这些组件成本又很高，这时用适配器模式能很好地解决这些问题。

2、适配器模式属于结构性模式

3、主要分为三类：类适配器模式，对象适配器模式、接口适配器模式

工作原理：

1、适配器模式：将一个类的接口转换成另一种接口，让原本接口不兼容的类可以兼容

2、从用户的角度看不到被适配者

3、用户调用适配器转换出来的目标接口方法，适配器再调用被适配者的相关接口方法

4、用户收到反馈结果，感觉只是和目标接口交互

5、适配器的关键代码：**适配器继承或依赖已有的对象，实现想要的目标接口。**

适配器模式（Adapter）包含以下主要角色。

1. 目标（Target）接口：当前系统业务所期待的接口，它可以是抽象类或接口。
2. 适配者（Adaptee）类：它是被访问和适配的现存组件库中的组件接口。
3. 适配器（Adapter）类：它是一个转换器，通过继承或引用适配者的对象，把适配者接口转换成目标接口，让客户按目标接口的格式访问适配者。

```java
/*类适配器案例：手机充电的电压问题
  一般国内的标准电压为220V
  手机的充电电压为5V左右，所以需要一个适配器将电压转换后再给手机充电
*/
//插孔输出220V的电压
public class Voltage_220 {
    public int output220(){
        System.out.println("Output 220V...");
        return 220;
    }
}
//充电器输出5V的电压，适配接口
public interface Charger {
    public int output5();
}
//适配器，可以理解为充电器中的降压器
public class VoltageAdaptor extends Voltage_220 implements Charger{
    @Override
    public int output5() {
        //首先获取到原始电压
        int srcVol = output220();
        //进行降压转换
        int dstVol = srcVol / 44;
        //最后输出
        return dstVol;
    }
}
```

对象适配器：基本思路与类的适配器模式相同，不是继承src类，而是持有src类的实例，以解决兼容性问题

```java
public class VoltageAdaptor implements Charger{
	//变成持有对象而不是继承类
	private Voltage_220 vol220;
	//通过构造器传入实例
	public VoltageAdaptor(Voltage_220 vol220){
		this.vol220 = vol220;
	}
    @Override
    public int output5() {
        int dstVol = 0;
        if(vol220 != null){
        	int srcVol = vol220.output220();
        	dstVol = srcVol / 44;
        }
        return dstVol;
    }
}
```

**适配器模式在SpringMvc中的应用：**Spring MVC中的适配器模式主要用于执行目标 Controller中的请求处理方法。当Spring容器启动后，会将所有定义好的适配器对象存放在一个List集合中，当一个请求来临时，DispatcherServlet 会通过 handler 的类型找到对应适配器，并将该适配器对象返回给用户，然后就可以统一通过适配器的 hanle() 方法来调用 Controller 中的用于处理请求的方法。通过适配器模式我们将所有的 `controller` 统一交给 `HandlerAdapter` 处理，免去了写大量的 `if-else` 语句对 `Controller` 进行判断，也更利于扩展新的 `Controller` 类型。

```java
//SpringMVC中提供的适配器接口
public interface HandlerAdapter {
	boolean supports(Object handler);
	
	@Nullable
	ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler) throws Exception;

	long getLastModified(HttpServletRequest request, Object handler);

}
//该接口有很多的实现类
//以HttpRequestHandlerAdapter为例
public class HttpRequestHandlerAdapter implements HandlerAdapter {
	//判断是适配器否是HttpRequestHandlerAdapter
	@Override
	public boolean supports(Object handler) {
		return (handler instanceof HttpRequestHandler);
	}

	@Override
	@Nullable
	public ModelAndView handle(HttpServletRequest request, HttpServletResponse response, Object handler)
			throws Exception {

		((HttpRequestHandler) handler).handleRequest(request, response);
		return null;
	}

	@Override
	public long getLastModified(HttpServletRequest request, Object handler) {
		if (handler instanceof LastModified) {
			return ((LastModified) handler).getLastModified(request);
		}
		return -1L;
	}

}
public class DispatcherServlet extends FrameworkServlet {
    private List<HandlerAdapter> handlerAdapters;
    
    //初始化handlerAdapters
    private void initHandlerAdapters(ApplicationContext context) {
        //..省略...
    }
    
    // 遍历所有的 HandlerAdapters，通过 supports 判断找到匹配的适配器
    protected HandlerAdapter getHandlerAdapter(Object handler) throws ServletException {
		for (HandlerAdapter ha : this.handlerAdapters) {
			if (logger.isTraceEnabled()) {
				logger.trace("Testing handler adapter [" + ha + "]");
			}
			if (ha.supports(handler)) {
				return ha;
			}
		}
	}
	
	// 分发请求，请求需要找到匹配的适配器来处理
	protected void doDispatch(HttpServletRequest request, HttpServletResponse response) throws Exception {
		HttpServletRequest processedRequest = request;
		HandlerExecutionChain mappedHandler = null;

		// Determine handler for the current request.
		mappedHandler = getHandler(processedRequest);
			
		// 确定当前请求的匹配的适配器.
		HandlerAdapter ha = getHandlerAdapter(mappedHandler.getHandler());

		ha.getLastModified(request, mappedHandler.getHandler());
					
		mv = ha.handle(processedRequest, response, mappedHandler.getHandler());
    }
	// ...省略...
}	

```

### 5. 装饰者模式

基本介绍：

1、装饰者模式是动态地将新功能附加到对象上，在对象功能扩展方面，它比继承更优弹性

2、装饰模式可以在不需要创造更多子类的情况下，将对象的功能加以扩展。

装饰者模式就像打包一个快递 ；比如我卖了一个陶瓷，一本书，其核心就是我们卖的东西。这些东西我们不可能直接去卖出去，因此我们会在外面包装好多保护东西，如泡沫塑料等等。

装饰者模式的引入：

```
咖啡馆订单项目：

1）、咖啡种类：Espresso、ShortBlack（浓缩）、LongBlack、Decaf （无糖）（这些就是相当咖啡的基础，单品咖啡，基础元素，所有的咖啡都是在此基础上混合起来了的）

2）、调料：Milk、Soy、Chocolate  （往基础品种中加入牛奶，巧克力等等，就组成了类似巧克力味咖啡，摩卡，卡夫奇诺等等。我们一般在咖啡厅一般都是直接点卡夫奇诺，其价格就是有单品咖啡和各种调料相加在一起的价格。所以下面的案例系统就是咖啡完整系统）

3）、咖啡馆订单项目设计原则（或者是需求）：扩展性好、改动方便、维护方便
```

装饰者模式的组成：

（1）Component：主体（装饰的主体是什么）他就像上面饮料例子的超类Drink类（抽象的超类）。

（2）ConcreteComponent：具体不同类型主题（比如具体要卖的陶瓷，衣服等等），就像上面的单品咖啡，用来被包装的物品。

（3）Decorator：装饰者（比如泡沫塑料，纸板等等），就像是各种调料。

简单地说：由上面的例子来说，这里的对象其实指的就是单品咖啡，附加的功能就是各种调料，是单品咖啡喝起来味道更好。也就是说装饰者模式可以动态的将调料添加到单品咖啡上混合出一个好的咖啡。而且是通过附加的方式添加上去，不是继承的方式添加上去的。

```java
public abstract class Drink {
    public String description = ""; //这个描述具体扩展出是什么单品或调料
    private float price = 0f;       //这里是具体的单品或调料的价格

    public void setDescription(String description)
    {
        this.description = description;
    }

    public String getDescription()
    {
        return description + "-" + this.getPrice();
    }
    public float getPrice()
    {
        return price;
    }
    public void setPrice(float price)
    {
        this.price = price;
    }
    public abstract float cost();  //这个是用抽象是因为，在单品种直接返回价格即可，
    //但在调料中，调料是不能单独存在的,是跟着实体，具体实体一起的，价格不仅是调料还有单品的价格，调料cost方法是是要递归去获取所有调料的价格
}
public class Coffee extends Drink{
    @Override
    public float cost() {
        return super.getPrice();
    }
}
//具体的一种咖啡单品
class Decaf extends Coffee{
    public Decaf() {
        super.setDescription("Decaf");
        super.setPrice(3.0f);
    }
}
//装饰者分支，中间层，就相当于那些调料的总和
public class Decorator extends Drink{
    //这个装饰者包装的是一个Coffee单品，但不知道是哪种具体的单品，所以需要一个基类
    private Drink drink;

    public Decorator() {
    }
    public Decorator(Drink drink) {
        this.drink = drink;
    }
    //这里的价格计算就有所跟单品不一样了。首先他要计算自己的价格，比如，巧克力，牛奶多少钱，还有要计算的就是前面
    @Override
    public float cost() {
        return super.getPrice() + drink.getPrice();
    }

    @Override
    public String getDescription() {
        return super.description + "-" + super.getPrice() + "&&" + drink.getDescription();
    }

}
//具体的一种调料
class Chocolate extends Decorator{
    public Chocolate(Drink drink) {
        super(drink);
        super.setDescription("chocolate");
        super.setPrice(2.0f);
    }
}
//测试方法：买一杯加巧克力的Decaf咖啡
public class DecoratorTest {
    @Test
    public void test(){
        Drink drink = new Decaf();
        drink = new Chocolate(drink);
        System.out.println("cost: " + drink.cost());
    }
}
```

装饰者模式的继承关系：

![](C:\Users\86198\Desktop\JavaEE\Java基础复习\image\Decorator.png)

总结来说，装饰者模式由四部分构成：

**Component（抽象构件）**：它是具体构件和抽象装饰类的共同父类，声明了在具体构件中实现的业务方法，它的引入可以使客户端以一致的方式处理未被装饰的对象以及装饰之后的对象，实现客户端的透明操作。

**ConcreteComponent（具体构件）**：它是抽象构件类的子类，用于定义具体的构件对象，实现了在抽象构件中声明的方法，装饰器可以给它增加额外的职责（方法）。

**Decorator（抽象装饰类）**：它也是抽象构件类的子类，用于给具体构件增加职责，但是具体职责在其子类中实现。它维护一个指向抽象构件对象的引用，通过该引用可以调用装饰之前构件对象的方法，并通过其子类扩展该方法，以达到装饰的目的。

**ConcreteDecorator（具体装饰类）**：它是抽象装饰类的子类，负责向构件添加新的职责。每一个具体装饰类都定义了一些新的行为，它可以调用在抽象装饰类中定义的方法，并可以增加新的方法用以扩充对象的行为。
