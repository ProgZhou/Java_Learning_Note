# Java八股

## 一、JavaSE --- 基础部分

### 1. 面向对象

什么是面向对象？

对比面向过程，是两种不同的处理问题的角度

面向过程注重事情的每一个步骤及顺序，面向对象更注重事情有哪些参与者(对象)、及各自需要做什么

比如洗衣机洗衣服

面向过程会将任务拆解成一系列的步骤(函数)：

(1). 打开洗衣机 ---- (2). 放衣服 ---- (3). 放洗衣粉 ---- (4). 清洗 ---- (5). 烘干

面向对象会拆解出人和洗衣机两个对象：

人：打开洗衣机，放衣服，放洗衣粉

洗衣机：洗衣服，烘干

从示例来看，面向过程比较直接高效，而面向对象更易于复用、扩展和维护

面向对象的三大特性：

**封装**：封装的意义，在于明确标识出允许外部使用的所有成员函数和数据项

内部细节对外部调用透明，外部调用无需修改或者关心内部实现

1、JavaBean的属性私有，提供get，set对外访问，因为属性的赋值或者获取逻辑只能由JavaBean本身决定，不能由外部直接修改

2、orm框架，操作数据库，我们不需要关心链接是如何建立的，sql是如何执行的，只需要引入mybatis，调用方法即可

**继承**：继承基类的方法，并做出自己的改变和扩展

子类共性的方法或者属性直接使用父类的，而不需要自己再定义，只需扩展自己个性化的

**多态**：基于对象所属的类不同，外部对同一个方法的调用，实际执行的逻辑不同

多态的三个条件：继承，方法重写，父类引用指向子类对象

但父类无法调用子类特有的功能

### 2. JRE、JDK、JVM

JDK：Java Development Kit  Java开发工具

JRE：Java Runtime Environment   Java运行时环境

JVM：Java Virtual Machine Java虚拟机

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\JDK.jpg)

#### 2.1 字节码是什么？采用字节码的好处是什么

在Java中，JVM可以理解的代码就叫做字节码(即扩展名为.class的文件)，**它不面向任何特定的处理器，只面向虚拟机**，Java语言通过字节码的方式，在一定程度上解决了传统解释型语言执行效率低的问题，同时又保留了解释性语言可移植的特点，因此java程序无须重新编译便可在多种不同操作系统的计算机上运行

**java语言的编译运行的过程：**

.java文件源码  -----  javac编译  -----  .class字节码文件  -----  解释器 / JIT（just-in-time complication）编译器  -----  机器可理解的代码 -----  OS运行

格外注意的是 `.class->机器码` 这一步。在这一步 JVM 类加载器首先加载字节码文件，然后通过解释器逐行解释执行，这种方式的执行速度会相对比较慢。而且，有些方法和代码块是经常需要被调用的(也就是所谓的热点代码)，所以后面引进了 JIT（just-in-time compilation） 编译器，而 JIT 属于运行时编译。当 JIT 编译器完成第一次编译后，其会将字节码对应的机器码保存下来，下次可以直接使用。而我们知道，机器码的运行效率肯定是高于 Java 解释器的。这也解释了我们为什么经常会说 **Java 是编译与解释共存的语言** 。

**编译型语言和解释型语言：**

- **编译型** ：编译型语言会通过编译器将源代码一次性翻译成可被该平台执行的机器码。一般情况下，编译语言的执行速度比较快，开发效率比较低。常见的编译性语言有 C、C++、Go、Rust 等等。
- **解释型** ：解释型语言会通过解释器一句一句的将代码解释（interpret）为机器代码后再执行。解释型语言开发效率比较快，执行速度比较慢。常见的解释性语言有 Python、JavaScript、PHP 等等。

`Java为什么要采用编译和解释并存的特点：*解释器与编译器两者各有优势：当程序需要*迅速启动和执行*的时候，解释器可以首先发挥作用，省去编译的时间，立即执行。在程序运行后，随着时间的推移，编译器逐渐发挥作用，把越来越多的代码编译成本地代码之后，可以获取*更高的执行效率*。当程序运行环境中*内存资源限制较大*（如部分嵌入式系统中），可以使用*解释器执行节约内存*，反之可以使用*编译执行来提升效率*。此外，如果编译后出现“罕见陷阱”，可以通过逆优化退回到解释执行。*`



### 3. == 与 equals的区别

== 比较的是栈中的值，对于基本数据类型来说是变量的值，对于引用类型是堆中内存对象的地址

+ 可以使用在基本数据类型变量和引用数据类型变量中
+ 如果比较的是基本数据类型的变量：比较两个变量保存的数据是否相等(类型不一定相同)
+ 如果比较的是引用数据类型的变量：比较两个对象的地址值是否相同，即两个引用是否指向同一个对象实体

equals 在Object类中默认也是采用==比较，通常在其他类中会重写，具体要看是否重写了equals方法

```java
//Object类中的equals
public boolean equals(Object obj) {
        return (this == obj);
    }
//String中的equals
public boolean equals(Object anObject){
    if(this == anObject){
        return true;
    }
    if(anObject instanceof String){
        String anotherString = (String)anObject;
        int n = value.length;
        if(n == anotherString.value.length){
            //比较两个字符串中的内容是否相同
            char v1[] = value;  //1.11中是byte[]
            char v2[] = anotherString.value;
            int i = 0;
            while(n-- != 0){
                if(v1[i] != v2[i])
                    return false;
                i++;
            }
            return true;
        }
    }
    return false;
}
```

String类中被重写的equals()方法其实是比较两个字符串的内容

### 4. final

关键字，表示最终的

+ 修饰类：表示类不可以被继承；比如String类，System类，StringBuffer类
+ 修饰方法：表示方法不可被子类覆盖(子类不能重写)，但可以重载
+ 修饰变量：表示变量一旦被赋值就不可以更改它的值

(1). 修饰成员变量

+ 如果final修饰的是静态变量，只能在静态初始化块中指定初始化值或者声明该类变量时指定初始值
+ 如果final修饰的是成员变量，可以在非静态初始化块、声明该变量进行显示初始化(final int a = 0;)或者构造器中执行初始化赋值

(2). 修饰局部变量

系统不会为局部变量进行初始化，局部变量必须由程序员显示初始化，因此使用final修饰局部变量时，即可以在定义时指定默认值(后面的代码不能对变量再赋值)，也可以不指定默认值，而在后面的代码中对final变量赋初值(仅一次)

```java
public class FinalVal{
	final static int a = 0;  //修饰静态变量声明时赋值，或者在静态代码块中赋值
/*	static{
		a = 0;
	}*/
	final int b = 0;  //修饰普通成员变量，在声明时赋值，或者在普通代码块中赋值，或者在构造器中赋值
	/*{
		b = 0;
	}*/
    //使用final修饰形参时，表明此形参是一个常量；当调用此方法时，给常量形参符一个值，一旦赋值之后，就只能在方法内使用赋值以后的值，但不能进行重新赋值
    public void show(final int num){
        System.out.println(num);
        //num = 20; 非法，不能修改
    }
    public static void main(String[] args){
        final int localA;  //局部变量只声明，不初始化，不会报错，与final无关
        localA = 0; //在使用之前一定要赋值
     	show(10);  //在传递时赋值
    }
}
```

(3). 修饰基本类型数据和引用类型数据

+ 如果是基本数据类型的变量，则其数值一旦在初始化之后便不能更改
+ 如果是引用类型的变量，则在对其初始化之后便不能再让其指向另一个对象(不能直接使用 = )，**但引用的值是可变的**

```java
public class FinalReferenceTest{
	public static void main(String[] args){
		final int[] iArr = {1,2,3,4};
		iArr[2] = 0; //合法
		iArr = null //不合法，iArr不能重新赋值
		
		final Person p = new Person(25);
		p.setAge(24);  //合法
		p = null; //非法
	}
}
```

static final 用来修饰属性：全局常量

**为什么局部内部类和匿名内部类只能访问局部final变量？**

编译之后会生成两个class文件，Test.class   Test1.class



首先需要知道的一点是，内部类和外部类是同一个级别的，内部类不会因为定义在方法中就会随着方法的执行完毕被销毁



### 5. String、StringBuffer、StringBuilder的区别和使用场景

String内部存储数据的char[]数组(1.11为byte[]数组)是final修饰的，不可变，每次操作都会生成一个新的String对象

+ 保存字符串的数组被 `final` 修饰且为私有的，并且`String` 类没有提供/暴露修改这个字符串的方法。
+ `String` 类被 `final` 修饰导致其不能被继承，进而避免了子类破坏 `String` 不可变。

StringBuffer和StringBuilder都是在原对象上操作

+ `StringBuilder` 与 `StringBuffer` 都继承自 `AbstractStringBuilder` 类，在 `AbstractStringBuilder` 中也是使用字符数组保存字符串，不过没有使用 `final` 和 `private` 关键字修饰，最关键的是这个 `AbstractStringBuilder` 类还提供了很多修改字符串的方法比如 `append` 方法。

StringBuffer是线程安全的

StringBuilder线程不安全的

StringBuffer方法都是synchronized修饰的

性能： StringBuilder > StringBuffer > String



场景：经常需要改变字符串内容时，使用后面两个

优先使用StringBuilder，多项成使用共享变量时使用StringBuffer

### 6. 重载和重写的区别

**重载**，发生在同一个类中，方法名必须要相同，参数类型不同，个数不同，顺序不同，方法返回值和访问修饰符可以不同，发生在编译时(**重载与方法返回值和修饰符无关**)

**重写**，发生在父子类中，方法名，参数列表必须相同，返回值范围小于等于父类，抛出的异常范围小于等于父类，访问修饰符范围大于等于父类，如果父类方法访问修饰符为private，则子类就不能重写该方法



### 7. List和Set的区别

List：有序，按对象进入的顺序保存对象，可重复，允许多个null元素对象，可以使用iterator取出所有元素，再逐一遍历，还可以使用get(int index)获取指定下标的元素

Set：无序，不可重复，最多允许有一个null元素对象，取元素时**只能使用Iterator接口取出所有元素**，再逐一遍历各个元素 

Set底层使用HashMap进行实现



### 8. 接口和抽象类的区别

+ 抽象类可以存在普通成员方法，而接口中只能存在public abstract方法
+ 抽象类中的成员变量可以是各种类型的，而接口中的成员变量只能是public static final类型的
+ 抽象类只能单继承，接口可以多继承

接口的设计目的，是对类的行为进行约束

抽象类的设计目的是代码的复用

抽象类是模板，接口是规范



### 9. hashCode与equals

equals方法：equals它的作用是判断两个对象是否相等，**如果对象重写了equals()方法，通常是比较两个对象的内容是否相等；如果没有重写，比较两个对象的地址是否相同，等价于“==”**。同样的，equals()定义在JDK的Object.java中，这就意味着Java中的任何类都包含有equals()函数。

hashCode方法：作用是获取哈希码；实际上是返回一个int整数，定义在Object类中，作用是确定该对象在哈希表中的索引位置(调用的是底层c++的代码)；散列表中存储的是键值对，特点是，能根据键快速的检索出对应的值

**为什么要有hashCode？**

以HashSet检测重复为例子：对象加入HashSet时，HashSet会先计算对象的hashCode值来判断对象加入的位置，看该位置是否有值，如果没有HashSet会假设对象没有重复出现，但是如果发现有值(哈希冲突)，这时会调用equals方法来检查两个对象是否真的相同；这样就大大降低调用equals方法的次数，提高了执行速度

hashCode的性质：

+ 如果两个对象相等，则hashCode一定也是相同的

+ 两个对象相等，对两个对象分别调用equals方法都返回true

+ 两个对象有相同的hashCode值，它们不一定相等

+ 因此，equals方法被重写过，hashCode方法也必须被重写

  + **假设有一个Person类，只重写了Person的equals()。但是，很奇怪的发现：HashSet中仍然有重复元素：p1 和 p2。为什么会出现这种情况呢？**

    **这是因为虽然p1 和 p2的内容相等，但是它们的hashCode()不等；所以，HashSet在添加p1和p2的时候，认为它们不相等。**

+ hashCode()的默认行为是对堆上的对象产生独特值，如果没有重写hashCode，则该class的两个对象无论如何都不会相等

### 10. ArrayList和LinkedList区别

ArrayList：基于动态数组，连续内存存储，适合下标访问(随机访问)；

扩容机制：因为数组长度固定，超出长度时，需要新建数组，然后将老数组的数据拷贝到新数组，如果不是尾插，还会涉及到元素的移动

```java
transient Object[] elementData; //ArrayList中默认存储数据的数组
private static final Object[] DEFAULTCAPACITY_EMPTY_ELEMENTDATA = {};
public ArrayList() {
    //在使用ArrayList list = new ArrayList();底层Object[] elementData初始化为{}
	//第一次调用add()方法时，底层会创建长度为10的数组，并将数据添加至elementData中
    this.elementData = DEFAULTCAPACITY_EMPTY_ELEMENTDATA;
}
//add()方法
private void add(E e, Object[] elementData, int s) {
    //如果是第一次添加数据
    if (s == elementData.length)
        elementData = grow();
    elementData[s] = e;
    size = s + 1;
}
//grow()方法，size为列表的长度
private Object[] grow() {
    return grow(size + 1);
}
private Object[] grow(int minCapacity) {
    return elementData = Arrays.copyOf(elementData,
                                           newCapacity(minCapacity));
}
//扩容
private int newCapacity(int minCapacity) {
        // overflow-conscious code
    int oldCapacity = elementData.length;
    //默认扩容为原容量的1.5倍
    int newCapacity = oldCapacity + (oldCapacity >> 1);
    //如果扩容之后容量仍然不够
        if (newCapacity - minCapacity <= 0) {
            //这个判断表示此次扩容是因为第一次调用add()方法引发的
            if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA)
                //这里的DEFAULT_CAPACITY是10，会创建一个长度为10(如果待扩容的容量小于10)的数组
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

LinkedListed：基于链表，可以存储在分散的内存中，适合做数据插入以及删除操作，不适合查询，需要逐一遍历

遍历LinkedList建议使用Iterator不能使用for循环，因为每次for循环体内通过get(i)取得某一元素时都需要对list重新遍历，性能消耗极大

```java
//LinkedList内部结构
transient int size = 0;
//Pointer to first node.
transient Node<E> first;
//Pointer to last node.
transient Node<E> last;
```

### 11. HashMap和HashTable的区别

区别：

+ HashMap方法没有synchronized修饰，线程不安全；HashTable线程安全
+ HashMap允许key和value为null，而HashTable不允许

底层实现：数组 + 链表实现

jdk1.8开始链表高度到8，数组长度超过64，链表转换为红黑树，元素以内部类Node结点存在

+ 计算key的hash值，二次hash然后对数组长度取模，对应数组下标
+ 如果没有产生hash冲突，则直接创建Node存入数组
+ 如果产生hash冲突，先进行equals比较，相同则取代该元素，不同，则判断链表高度，链表高度达到8，并且数组长度到64则转变为红黑树，长度(红黑树的结点数)低于6，则将红黑树转换为链表
+ key为null，存在下标为0的位置

数组扩容：当所需要的容量 >= 数组容量(默认为16) * 加载因子(默认为0.75)时进行扩容，默认容为原来的两倍

**HashMap原理**

+ new HashMap(): 底层没有创建一个长度为16的数组

+ jdk8.0底层的数组是Node[]数组，而非Entry[]

+ 首次调用put方法时，底层创建长度为16的数组

+ 加载因子：默认0.75，作用：提前扩容

  临界值：容量 \* 加载因子，当容量超过临界值时，进行扩容

+ jdk8.0当中，Map的底层结构是数组 + 链表 + 红黑树，当数组的某一个索引位置上的元素以链表形式存在的数据个数 > 8且当前数组的长度 > 64时，此时此索引位置上的所有数据改为使用红黑树存储

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
            else if (p instanceof TreeNode)   //当前链表是否已经转换为红黑树
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



### 12. ConcurrentHashMap原理



### 13. Java泛型，什么是类型擦除

**Java 泛型（generics）** 是 JDK 5 中引入的一个新特性, 泛型提供了编译时类型安全检测机制，该机制允许程序员在编译时检测到非法的类型。泛型的本质是参数化类型，也就是说所操作的数据类型被指定为一个参数。

泛型就是允许在定义类，接口时通过一个标识符标识类中某个属性的类型或者是某个方法的返回值及参数类型；这个类型参数在使用时确定

Java的泛型是伪泛型，这是因为Java在运行期间，所有的泛型信息都会被擦掉，这也就是通常所说的类型擦除

```java
List<Integer> list = new ArrayList<>();

list.add(12);
//这里直接添加会报错
list.add("a");
Class<? extends List> clazz = list.getClass();
Method add = clazz.getDeclaredMethod("add", Object.class);
//但是通过反射添加是可以的
//这就说明在运行期间所有的泛型信息都会被擦掉
add.invoke(list, "kl");
System.out.println(list);
```

#### 常用的通配符有哪些？

**常用的通配符为： T，E，K，V，？**

- ？ 表示不确定的 Java 类型
- T (type) 表示具体的一个 Java 类型
- K V (key value) 分别代表 Java 键值中的 Key Value
- E (element) 代表 Element

#### 你的项目中哪里用到了泛型？

- 可用于定义通用返回结果 `CommonResult<T>` 通过参数 `T` 可根据具体的返回类型动态指定结果的数据类型
- 定义 `Excel` 处理类 `ExcelUtil<T>` 用于动态指定 `Excel` 导出的数据类型
- 用于构建集合工具类。参考 `Collections` 中的 `sort`, `binarySearch` 方法
- ......

### 14. Exception 和 Error 有什么区别？

在 Java 中，所有的异常都有一个共同的祖先 `java.lang` 包中的 `Throwable` 类。`Throwable` 类有两个重要的子类:

- **`Exception`** :程序本身可以处理的异常，可以通过 `catch` 来进行捕获。`Exception` 又可以分为 Checked Exception (受检查异常，必须处理) 和 Unchecked Exception (不受检查异常，可以不处理)。
- **`Error`** ：`Error` 属于程序无法处理的错误 ，我们没办法通过 `catch` 来进行捕获 。例如Java 虚拟机运行错误（`Virtual MachineError`）、虚拟机内存不够错误(`OutOfMemoryError`)、类定义错误（`NoClassDefFoundError`）等 。这些异常发生时，Java 虚拟机（JVM）一般会选择线程终止。

错误处理的继承体系：

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\types-of-exceptions-in-java.png)

#### Checked Exception 和 Unchecked Exception 有什么区别？

**Checked Exception** 即受检查异常，Java 代码在编译过程中，如果受检查异常没有被 `catch`/`throw` 处理的话，就没办法通过编译 。

比如下面这段 IO 操作的代码：

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\checked-exception.png)

除了`RuntimeException`及其子类以外，其他的`Exception`类及其子类都属于受检查异常 。常见的受检查异常有： IO 相关的异常、`ClassNotFoundException` 、`SQLException`...。

**Unchecked Exception** 即 **不受检查异常** ，Java 代码在编译过程中 ，我们即使不处理不受检查异常也可以正常通过编译。

`RuntimeException` 及其子类都统称为非受检查异常，例如：`NullPointerException`、`NumberFormatException`（字符串转换为数字）、`ArrayIndexOutOfBoundsException`（数组越界）、`ClassCastException`（类型转换错误）、`ArithmeticException`（算术错误）等。

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\unchecked-exception.png)

### 15. 包装类型的常量池技术了解么？

Java 基本类型的包装类的大部分都实现了常量池技术。

`Byte`,`Short`,`Integer`,`Long` 这 4 种包装类默认创建了数值 **[-128，127]** 的相应类型的缓存数据，`Character` 创建了数值在 **[0,127]** 范围的缓存数据，`Boolean` 直接返回 `True` or `False`。

**Integer 缓存源码：**

```java
public static Integer valueOf(int i) {
    if (i >= IntegerCache.low && i <= IntegerCache.high)
        return IntegerCache.cache[i + (-IntegerCache.low)];
    return new Integer(i);
}
private static class IntegerCache {
    static final int low = -128;
    static final int high;
    static {
        // high value may be configured by property
        int h = 127;
    }
}
```

**`Character` 缓存源码:**

```java
public static Character valueOf(char c) {
    if (c <= 127) { // must cache
      return CharacterCache.cache[(int)c];
    }
    return new Character(c);
}

private static class CharacterCache {
    private CharacterCache(){}
    static final Character cache[] = new Character[127 + 1];
    static {
        for (int i = 0; i < cache.length; i++)
            cache[i] = new Character((char)i);
    }

}
```

**`Boolean` 缓存源码：**

```java
public static Boolean valueOf(boolean b) {
    return (b ? TRUE : FALSE);
}
```

如果超出对应范围仍然会去创建新的对象，缓存的范围区间的大小只是在性能和资源之间的权衡。

两种浮点数类型的包装类 `Float`,`Double` 并没有实现常量池技术。

```java
Integer i1 = 33;
Integer i2 = 33;
System.out.println(i1 == i2);// 输出 true

Float i11 = 333f;
Float i22 = 333f;
System.out.println(i11 == i22);// 输出 false

Double i3 = 1.2;
Double i4 = 1.2;
System.out.println(i3 == i4);// 输出 false
```

下面我们来看一下问题。下面的代码的输出结果是 `true` 还是 `false` 呢？

```java
Integer i1 = 40;
Integer i2 = new Integer(40);
System.out.println(i1==i2);   //返回false
```

`Integer i1=40` 这一行代码会发生装箱，也就是说这行代码等价于 `Integer i1=Integer.valueOf(40)` 。因此，`i1` 直接使用的是常量池中的对象。而`Integer i2 = new Integer(40)` 会直接创建新的对象。

### 16. 什么是序列化?什么是反序列化?

如果我们需要持久化 Java 对象比如将 Java 对象保存在文件中，或者在网络传输 Java 对象，这些场景都需要用到序列化。

简单来说：

- **序列化**： 将数据结构或对象转换成二进制字节流的过程
- **反序列化**：将在序列化过程中所生成的二进制字节流转换成数据结构或者对象的过程

对于 Java 这种面向对象编程语言来说，我们序列化的都是对象（Object）也就是实例化后的类(Class)，但是在 C++这种半面向对象的语言中，struct(结构体)定义的是数据结构类型，而 class 对应的是对象类型。

维基百科是如是介绍序列化的：

> **序列化**（serialization）在计算机科学的数据处理中，是指将数据结构或对象状态转换成可取用格式（例如存成文件，存于缓冲，或经由网络中发送），以留待后续在相同或另一台计算机环境中，能恢复原先状态的过程。依照序列化格式重新获取字节的结果时，可以利用它来产生与原始对象相同语义的副本。对于许多对象，像是使用大量引用的复杂对象，这种序列化重建的过程并不容易。面向对象中的对象序列化，并不概括之前原始对象所关系的函数。这种过程也称为对象编组（marshalling）。从一系列字节提取数据结构的反向操作，是反序列化（也称为解编组、deserialization、unmarshalling）。

综上：**序列化的主要目的是通过网络传输对象或者说是将对象存储到文件系统、数据库、内存中。**



### 17. 为什么在Java中只有值传递机制

值传递：方法接收的是实参值的拷贝，会创建副本

引用传递：方法接收的直接是实参所引用的对象在堆中的地址，不会创建副本，对形参的修改将影响实参的值

- 如果参数是基本类型的话，很简单，传递的就是基本类型的字面量值的拷贝，会创建副本。
- 如果参数是引用类型，传递的就是实参所引用的对象在堆中地址值的拷贝，同样也会创建副本。

示例：

```java
public class Person {
    private String name;
   // 省略构造函数、Getter&Setter方法
}

public static void main(String[] args) {
    Person xiaoZhang = new Person("小张");
    Person xiaoLi = new Person("小李");
    swap(xiaoZhang, xiaoLi);
    System.out.println("xiaoZhang:" + xiaoZhang.getName());
    System.out.println("xiaoLi:" + xiaoLi.getName());
}

public static void swap(Person person1, Person person2) {
    Person temp = person1;
    person1 = person2;
    person2 = temp;
    System.out.println("person1:" + person1.getName());
    System.out.println("person2:" + person2.getName());
}
```

输出：

```
person1:小李
person2:小张
xiaoZhang:小张
xiaoLi:小李
```

可以发现，即使是给方法传入了引用类型的值，其值依然没有改变，其实在内存中的结构是这样的

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\java-value-passing-03.png)

形参仅仅是拷贝两个实参的地址，交换也只是形参所指的地址交换，而并不是实参交换



## 二、JavaEE部分 --- 框架

### 1. Spring是什么

Spring是开源的J2EE框架。它是一个**容器框架**，用来装JavaBean(Java对象)，中间层框架可以起一个连接作用，比如说把Structs和Hibernate粘合在一起运用，可以让我们的企业开发更快、更简洁

Spring是一个轻量级的**控制反转(IOC)**和**面向切面(AOP)**的容器框架

+ 从大小与开销两方面而言Spring都是轻量级的
+ 通过控制反转的技术达到**松耦合**的目的
+ 提供了面向切面编程的丰富支持，允许**通过分离应用的业务逻辑与系统服务进行内聚性的开发**
+ 包含管理应用对象的配置和生命周期，这个意义上是一个容器
+ 将简单的组件配置、组合成复杂的应用，这个意义上是一个框架

### 2. 对AOP的理解

系统是由许多不同的组件所组成的，每一个组件各负责一块特定功能，当然会存在很多组件是跟业务无关的，例如日志、事务管理和安全这样的核心服务组件，这些核心服务组件经常需要融入到具体的业务逻辑中，如果我们为每一个具体业务逻辑操作都添加这样的代码，很明显代码冗余太多，**因此我们需要将这些公共的代码逻辑抽象出来变成一个切面，然后注入到目标对象(具体业务)中去**

AOP正是基于这样的一个思路实现的，**通过动态代理的方式，将需要注入切面的对象进行代理**，在进行调用的时候，将公共的逻辑直接添加进去，而不需要修改原有的业务逻辑代码，只需要在原来的业务逻辑基础之上做一些增强功能即可

**AOP**：将程序中的交叉业务逻辑(比如安全、日志、事物等)，封装成一个切面，然后注入到目标对象(具体业务逻辑)中去，**AOP可以对某个对象或某些对象的功能进行增强，比如对象中的方法进行增强**，可以在执行某个方法之前额外的做一些事情，在某个方法执行之后额外的做一些事情

通俗描述：**在不修改源代码的情况下，在主干功能里面添加新功能**

底层原理：动态代理



### 3. 对IOC的理解

**IOC容器**：实际上就是个map(key, value)，里面存的就是各种对象(在xml里面配置的bean结点，@Repository，@Service，@Controller，@Component)，在项目启动的时候会读取配置文件里面的bean结点，**根据全限定类名使用反射创建对象放到map中**；或者扫描有注解的类，也是通过反射创建对象并且放到map中

这个时候map中就有各种各样的对象了，接下来在代码里需要用到里面的对象时，再通过DI注入(@Autowired, @Resource等注解，xml里bean结点内ref属性，在项目启动时根据id注入)

**控制反转**：没有引入IOC容器之前，对象A依赖于对象B，自己必须主动去创建对象B，无论创建还是使用对象B，控制权都在自己手上

```java
class A{
	B b = new B();
    public void show(){
        System.out.println(b);
    }
    ...
}
```

引入IOC容器之后，对象A与对象B之间失去了直接联系，当对象A运行到需要对象B的时候，IOC容器会主动创建一个对象B注入到对象A需要的地方

```java
class A{
	@Autowired
	B b;
}
```

通过这两种方法的对比，不难看出：**对象A获得依赖对象B的过程，由主动行为变成了被动行为**，控制权颠倒过来了，这就是控制反转这个名称的由来

全部对象的控制权上缴给第三方IOC容器，所以，IOC容器成了整个系统的关键核心，它起到了一种类似粘合剂的作用，把系统中所有对象粘合在一起发挥作用

**依赖注入**：容器能够知道哪个组件运行的时候，需要另一个组件（比如Controller组件需要Service层的组件），容器通过反射的方式，将容器中准备好的Service对象注入到Controller组件中

底层原理：xml解析，工厂模式，反射

+ 谁控制谁：在之前的编程过程中，都是需要什么对象自己去创建什么对象，由程序员自己来控制对象，而有了IOC容器之后，就会变成由IOC容器来控制对象
+ 控制什么：bean创建和管理的权力，控制bean的生命周期
+ 什么是反转：在没有IOC容器之前我们都是在对象中主动去创建依赖的对象，这是正转的，而有了IOC容器之后，把这个权利交给了 Spring 容器，而不是自己去控制，就是反转。由之前的自己主动创建对象，变成现在被动接收别人给我们的对象的过程，这就是反转。
+ 哪些方面被反转：依赖的对象

简单来说，控制反转就是bean的创建和管理交给第三方，在spring中就是IOC容器，容器负责创建bean，管理bean的生命周期

> IOC是控制反转，其中控制指的是资源的获取方式，主要分为两种：
>
> + 主动式：要什么资源都自己创建
>
>   ```java
>   public class BookServlet {
>       BookService bs = new BookService();
>       
>       AirPlane ap = new AirPlane();
>       ...
>       public void method1(){
>           bs.m();
>           ...
>       }
>   }
>   ```
>
> + 被动式：资源的获取不是我们自己创建，而是交给一个容器来创建和设置
>
>   ```java
>   public class BookServlet{
>       BookService bs;
>         
>       public void test1() {
>           bs.check();
>           ...
>       }
>   }
>   ```
>
>   所谓容器就是管理所有的组件，假设BookServlei受容器管理，BookService也受容器管理，容器可以自动探查出哪些组件需要用到另一些组件，容器可以帮我们创建BookService对象，并把BookService对象赋值过去
>
>   而容器的作用就是主动的new资源变为被动的接收资源

### 4. BeanFactory和ApplicationContext

ApplicationContext是BeanFactory的子接口

ApplicationContext提供了完整的功能

+ 继承了MessageSource，因此支持国际化
+ 统一的资源文件访问方式
+ 提供在监听器注册bean的事件
+ 同时加载多个配置文件
+ 载入多个(由继承关系)上下文，使得每一个上下文都专注于一个特定的层次，比如应用的web层
+ 主要实现类是ClassPathXmlApplicationContext和FileSystemXmlApplicationContext，前者默认从类路径加载配置文件，后者默认从文件系统中装载配置文件

两者的区别：

+ **BeanFactory采用延迟加载的方式来注入bean**，即只有在使用到某个bean时，才对bean进行加载实例化，这样，就不能发现一些存在的spring的配置问题；如果bean的某一个属性没有注入，BeanFactory加载之后，直至第一次调用getBean()方法才会抛出异常
+ **ApplicationContext在容器启动时，一次性创建了所有bean**，这样，在容器启动时，就可以发现spring中存在的配置错误，这样有利于检查所依赖属性是否注入；ApplicationContext启动后预载入所有的单实例bean，通过预载入单实例bean，确保当你需要的时候，无需等待
+ 相对于基本的BeanFactory，ApplicationContext唯一不足就是占用内存空间，当应用程序bean较多时，程序启动较慢
+ BeanFactory通常以编程的方式被创建，ApplicationContext还能以声明的方式创建，如使用ContextLoader

#### 4.1 BeanFactory和FactoryBean的区别

+ BeanFactory是一个工厂类，**它负责生产和管理bean的一个工厂**，**所有的Bean都是由BeanFactory(也就是IOC容器)来进行管理的**，BeanFactory只是个接口，并不是IOC容器的具体实现，但是Spring容器给出了很多种实现
+ FactoryBean是一个bean接口，在某些情况下，实例化Bean过程比较复杂，如果按照传统的方式，则需要在<bean>中提供大量的配置信息。配置方式的灵活性是受限的，这时采用编码的方式可能会得到一个简单的方案。Spring为此提供了一个org.springframework.bean.factory.FactoryBean的工厂类接口，用户可以通过实现该接口定制实例化Bean的逻辑。



### 5. Spring Bean的生命周期

Spring中bean生命周期的普通理解：

1、通过构造器创建bean实例(无参构造器)，使用反射的方式生成bean对象

2、为bean的属性设置值和对其他bean的引用(调用set方法)

*把bean实例传递给bean后置处理器的方法*

3、调用bean的初始化的方法(需要进行配置，bean标签的init-method属性，规定方法不能带任何参数)

*把bean实例传递给bean后置处理器的方法*

4、获取bean对象并使用

5、当容器关闭时，调用bean的销毁方法(需要进行配置，bean标签的destroy-method属性，规定方法不能带任何参数)

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\Bean.jpg)

补充：后置处理器的使用

BeanPostProcessor[interface]：bean后置处理器

```java
//编写一个类实现BeanPostProcessor接口，实现里面的两个方法
public class PostProcessor implements BeanPostProcessor {
    //在初始化方法执行之前执行
    @Override
    public Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("post processor before init...");
        return bean;
    }
	//在初始化方法执行完成之后执行
    @Override
    public Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
        System.out.println("post processor after init...");
        return bean;
    }
}
```

```xml
<!--在xml文件中进行配置
配置之后，默认对所有的bean都生效
-->
<bean id="orders" class="com.SpringXML.bean.Orders" init-method="init" destroy-method="destroy">
        <property name="name" value="Phone"/>
    </bean>
    <bean id="postProcessor" class="com.SpringXML.bean.PostProcessor"/>
```



### 6. Spring中bean的作用域

1、**singleton：单例**，默认的bean作用域，**每个容器中只有一个bean的实例**，单例的模式由BeanFactory自身来维护，该对象的生命周期是与Spring IOC容器一致的（但在第一次被注入时才会创建）；**设置为单例时，Spring文件加载时就会创建单实例对象**

2、**prototype：多例，为每个bean请求提供一个实例，在每次注入时都会创建一个新的对象**；设置为多例时，不是在Spring文件加载时创建对象，而是在每次调用getBean方法时创建对象

3、request：bean被定义为在每个HTTP请求中创建一个单例对象，也就是说在单个请求中都会复用这一个单例对象

4、session：与request范围类似，确保每个session中有一个bean的实例，在session过期后，bean会随之失效

*5、application：bean被定义为在ServletContext的生命周期中复用一个单实例对象*

*6、websocket：bean被定义为在websocket的生命周期中复用一个单例对象*



### 7. Spring中的设计模式

1、简单工厂：由一个工厂类根据传入的参数，动态地决定应该创建哪一个产品类

`Spring中的BeanFactory就是简单工厂模式的体现，根据传入一个唯一的标识来获得bean对象，但是否是在传入参数后创建还是传入参数前创建这个要根据具体情况来定`

2、工厂方法

`实现FactoryBean接口的bean是一类叫做factory的bean，其特点是，spring会在使用getBean()调用获得该bean时，会自动调用该bean的getObject()方法，所以返回的不是factory这个bean，而是这个bean.getObject()的返回值`

3、单例模式：保证一个类仅有一个实例，并提供一个访问它的全局访问点

`spring对单实例的实现：spring中的单例模式完成了后半句话，即提供了全局的访问点BeanFactory，但没有从构造器级别取控制单例，这是因为spring管理的是任意的java对象`

`Spring中bean默认都是单例的`

4、适配器模式：

`spring定义了一个适配器接口，使得每一种Controller有一种对应的适配器实现类，让适配器代替Controller执行相应的方法，这样在扩展Controller时，只需要增加一个适配器类就完成了SpringMVC的扩展了`

5、装饰器模式：动态地给一个对象添加一些额外的职责，就增加功能来说，Decorator模式相比生成子类更为灵活

`Spring中用到的装饰器模式在类名上有两种表现：一种是类名中含有Wrapper的，另一种是类名中含有Decorator的`

6、动态代理模式

`切面在应用运行的时刻被注入，一般情况下，在注入切面时，AOP容器会为目标对象动态地创建一个代理对象`

7、模板方法：

`spring中postProcessorBeanFactory，onRefresh，initPropertyValue方法`



### 8. Bean的自动装配

开启自动装配，只需要在xml配置文件\<bean>中定义"autowire"属性

```xml
<bean id="" class="com.xxx.xxx.xxx" autowire=""/>
```

autowire属性有五种装配方式：

+ 默认情况，自动配置是通过"ref"属性手动设置的

  `手动装配：在bean标签之间使用<property>标签设置value或者ref属性`

+ byName -- 根据bean的属性名称进行自动装配

  ```xml
  <!--示例：假设Customer类中有一个Person类的属性，属性名为person
  class Customer{
  	Person person;
  }
  -->
  <bean id="customer" class="com.xxx.Coustomer" autowire="byName"/>
  <bean id="person" class="com.xxx.Person"/>
  ```

+ byType -- 根据bean的类型进行自动装配

  ```xml
  <!--示例：假设Customer类中有一个Person类的属性，属性名为person
  class Customer{
  	Person person;
  }
  -->
  <bean id="customer" class="com.xxx.Coustomer" autowire="byType"/>
  <bean id="person" class="com.xxx.Person"/>
  ```

+ constructor -- 类似byType，不过是应用于构造器的参数，如果一个bean与构造器参数的类型形同，则进行自动装配，否则导致异常

  ```xml
  <!--示例：假设Customer类中有一个Person类的属性，属性名为person，并且有一个有参构造器
  class Customer{
  	Person person;
  	Customer(Person person){
  		this.person = person;
  	}
  }
  -->
  <bean id="customer" class="com.xxx.Coustomer" autowire="constructor"/>
  <bean id="person" class="com.xxx.Person"/>
  ```

+ autodetect -- 如果有默认的构造器，则通过constructor方式进行自动装配，否则使用byType方式进行自动装配

+ @Autowired自动装配bean



### 9. SpringMVC的工作流程

1、用户发送请求至前端控制器DispatcherServlet

2、DispatcherServlet收到请求调用HandlerMapping处理器映射器

3、处理器映射器找到具体的处理器，生成处理器以及处理器拦截器，一并返回给DispatcherServlet

4、DispatcherServlet调用HandlerAdapter处理器适配器

5、HandlerAdapter经过适配器调用具体的处理器(Controller)

6、Controller执行完成后返回一个ModelAndView对象

7、HandlerAdatper将Controller执行结果ModelAndView返回给DispatcherServlet

8、DispatherServlet将返回的ModelAndView传给ViewResolver视图解析器

9、ViewResolver解析后返回具体的View对象

10、DispatcherServlet根据View进行渲染视图

11、最后DispatcherServlet将结果响应给用户



### 10. SpringMVC的主要组件

Handler：也就是处理器（可以理解为Controller），它直接对应MVC中的Controller层，它的具体表现形式有很多，可以是类，也可以是方法，在Controller层中@RequestMapping标注的所有方法都可以看成是一个Handler

**1、HandlerMapping**

处理器映射器，根据用户请求的资源URI来查找Handler，在SpringMVC中会有很多请求，每个请求都需要一个Handler处理，具体接收到一个请求之后使用哪个Handler进行，这就是HandlerMapping需要做的事情

**2、HandlerAdapter**

处理器适配器，因为SpringMVC中的Handler可以是任意形式，只要能处理请求就ok，但是Servlet需要的处理方法的结构是固定的，都是以request和response为参数的方法，Handler Adapter的作用就是让固定的Servlet处理方调用灵活的Handler

3、HandlerExceptionResolver

处理器异常解析器，当其他组件在干活的过程中出现异常，需要有一个专门的角色对异常进行处理，在SpringMVC中就是HandlerExceptionResolver，具体来说，此组件的作用就是根据异常设置ModelAndView，之后再交给render方法进行渲染

4、ViewResolver

视图解析器，用来将Sring类型的视图名称和Locale解析为View类型的视图，View是用来渲染页面的，也就是将程序返回的参数填入参数模板里，生成html文件，需要处理两个关键问题：使用哪个模板？用什么技术填入参数？这其实是ViewResolver主要要做的工作，ViewResolver需要找到渲染所用的模板和所用的技术进行渲染，具体的渲染过程则交由不同的视图自己完成



### 11. Mybatis的优缺点

优点：

1、基于SQL语句变成，相当灵活，不会对应用程序或者数据库的现有设计造成任何影响，SQL写在XML里，解除了SQL与程序代码的耦合，便于统一管理；提供XML标签，支持编写动态SQL语句，并可重用

2、与JDBC相比，减少了50%以上的代码量，消除了JDBC大量代码冗余，不需要手动开关连接

3、很好的与各种数据库兼容（只要JDBC支持的数据库Mybatis都支持）

4、能够与Spring很好的集成

5、提供映射标签，支持对象与数据库的ORM字段关系映射；提供对象关系映射标签，支持对象关系组件维护

缺点：

1、SQL语句编写工作量较大，尤其当字段多，关联表多时，对开发人员编写SQL语句的功底有一定要求

2、SQL语句依赖于数据库，导致数据库移植性差，不能随意更换数据库

### 12. \#{}与${}的区别

\#{}是预编译处理，占位符，${}是字符串替换，是拼接符

Mybatis在处理#{}时，会将sql中的#{}替换为?，调用PreparedStatement来赋值

Mybatis在处理${}时，就是把${}替换为变量值，调用Statement来赋值

#{}的变量替换是在DBMS中，变量替换后，#{}对应的变量自动加上单引号

${}的变量替换是在DBMS外，变量替换后，${}对应的变量不会加上单引号

使用#{}可以有效的放置SQL注入，提高系统安全性

### 13. Spring事务

在使用Spring框架时，可以有两种使用事务的方式，一种是编程式，一种是声明式的（@Transactional注解），一般Spring的事务添加在JavaEE三层结构的Service层上（业务逻辑层）

首先，事务这个概念是数据库层面的，Spring只是基于数据库中的事务进行了扩展，以及提供了一些能让程序员更加方便操作事务的方式

在一个方法上加上@Transactional注解后，Spring会基于这个类生成一个代理对象，会将这个代理对象作为bean，当在使用这个代理对象的方法时，如果这个方法上存在@Transactional注解，那么代理逻辑会先把事务的自动提交设置为false，然后再去执行原本的业务逻辑方法，如果执行业务逻辑方法没有出现异常，那么代理逻辑中就会将事务进行提交，如果执行业务逻辑方法出现了异常，那么则会将事务进行回滚

Spring事务隔离级别就是数据库的隔离级别，外加一个默认级别

```
假设数据库配置的隔离级别是Read Commited，而spring配置的隔离级别是Repeatable Read，请问这时隔离级别是以哪一个为准？
以spring中的事务为准，如果数据库不支持spring中设置的事务隔离级别，那么效果就取决于具体的数据库


```







### 14. Spring事务传播机制

多个事务方法相互调用时，事务如何在这些方法中传播

`方法A是一个事务的方法，方法A执行过程中调用了方法B，那么方法B有无事务以及方法B对事务的要求不同都会对方法A的事务具体执行过程造成影响，同时方法A的事务对方法B的事务执行也有影响，这种影响具体是什么就由两个方法所定义的事务传播类型决定`

REQUIRED（Spring默认的事务传播类型）：如果当前没有事务，则自己新建一个事务，如果当前存在事务，则加入这个事务

SUPPORTS：当前存在事务，则加入当前事务，如果当前没有事务，就以非事务方法执行

MANDATORY：当前存在事务，则加入当前事务，如果当前事务不存在，则抛出异常

REQUIRES_NEW：创建一个新事务，如果存在当前事务，则挂起该事务

NOT_SUPPORTED：以非事务方式执行，如果当前存在事务，则挂起当前事务

NEVER：不使用事务，如果当前事务存在，则抛出异常

NESTED：如果当前事务存在，则嵌套在事务中执行，否则REQUIRED的操作一样（开启一个事务）

https://blog.csdn.net/weixin_39625809/article/details/80707695



### 15. Spring循环依赖

什么是循环依赖：一个或多个对象实例之间存在直接或间接的依赖关系，这种依赖关系构成了一个环形调用

循环依赖的三种情况：

+ 自己依赖自己的直接依赖

  ```java
  @Component
  public class A{
  	@Autowired
  	private A a;
  }
  ```

  ![](C:\Users\86198\Desktop\JavaEE\Java八股\image\自依赖.png)

+ 两个对象直接的直接依赖

  ```java
  @Component
  public class A{
  	@Autowired
  	private B b;
  }
  
  @Component
  public class B{
      @Autowired
      private A a;
  }
  ```

  ![](C:\Users\86198\Desktop\JavaEE\Java八股\image\两个对象直接依赖.png)

+ 多个对象之间的间接依赖

  ```java
  @Component
  public class A{
  	@Autowired
  	private B b;
  }
  
  @Component
  public class B{
  	@Autowired
  	private C c;
  }
  
  @Component
  public class C{
  	@Autowired
  	private A a;
  }
  ```

  ![](C:\Users\86198\Desktop\JavaEE\Java八股\image\多个对象直接依赖.jpg)

**Spring内部如何解决循环依赖？**

Spring解决循环依赖的前置条件：

1. 出现循环依赖的Bean必须要是**单例**

2. 依赖注入的方式不能全是构造器注入的方式

   全是构造器注入指的是：

   ```java
   @Component
   public class A {
   // @Autowired
    private B b;
    public A(B b) {
   
    }
   }
   @Component
   public class B {
   
   // @Autowired
    private A a;
   
    public B(A a){
   
    }
   }
   ```

   A中注入B的方式是通过构造器，B中注入A的方式也是通过构造器，这个时候循环依赖是无法被解决

Spring的三级缓存 --- Spring内部维护了三个Map，也就是通常说的三级缓存

+ *singletonObjects* 它是我们最熟悉的朋友，俗称“单例池”“容器”，缓存创建完成单例Bean的地方。
+ *singletonFactories* 映射创建Bean的原始工厂
+ *earlySingletonObjects* 映射Bean的早期引用，也就是说在这个Map里的Bean不是完整的，甚至还不能称之为“Bean”，只是一个Instance（没有被注入属性的对象）.

```java
public class DefaultSingletonBeanRegistry extends SimpleAliasRegistry implements SingletonBeanRegistry {
    
    ...
        
	private final Map<String, Object> singletonObjects = new ConcurrentHashMap<>(256);

	/** Cache of singleton factories: bean name to ObjectFactory. */
	private final Map<String, ObjectFactory<?>> singletonFactories = new HashMap<>(16);

	/** Cache of early singleton objects: bean name to bean instance. */
	private final Map<String, Object> earlySingletonObjects = new HashMap<>(16);
    
    ...
}
```

Spring解决循环依赖主要是靠singletonObjects这一级缓存，简单来理解的话，解决循环依赖就是在singletonObjects这个map中查找是否有已经需要注入的bean，如果有，那么就将map中的值注入，如果没有就创建

https://mp.weixin.qq.com/s/5mwkgJB7GyLdKDgzijyvXw



### 16. Springboot自动配置原理

@Import + @Configuration + Spring API

自动配置类由各个starter提供，使用@Configuration + @Bean定义配置类，放到META-INF/spring.factories下

使用Spring API扫描META-INF/spring.factories下的配置类

使用@Import导入自动配置类

在Springboot的主启动类上有一个@SpringBootApplication注解

```java
@SpringBootConfiguration
@EnableAutoConfiguration
@ComponentScan(excludeFilters = { @Filter(type = FilterType.CUSTOM, classes = TypeExcludeFilter.class),
		@Filter(type = FilterType.CUSTOM, classes = AutoConfigurationExcludeFilter.class) })
public @interface SpringBootApplication{}
```

其中，@EnableAutoConfiguration注解

```java
@AutoConfigurationPackage
@Import(AutoConfigurationImportSelector.class)
public @interface EnableAutoConfiguration {}
```

这个@AutoConfigurationPackage实质上是一个@Import注解

```java
@Import(AutoConfigurationPackages.Registrar.class)  //给容器中导入一个组件
public @interface AutoConfigurationPackage {}

//利用Registrar给容器中导入一系列组件
//将指定的一个包下的所有组件导入进来？MainApplication 所在包下。
```

另一个@Import(AutoConfigurationImportSelector.class)注解

```
1、利用getAutoConfigurationEntry(annotationMetadata);给容器中批量导入一些组件
2、调用List<String> configurations = getCandidateConfigurations(annotationMetadata, attributes)获取到所有需要导入到容器中的配置类
3、利用工厂加载 Map<String, List<String>> loadSpringFactories(@Nullable ClassLoader classLoader)；得到所有的组件
4、从META-INF/spring.factories位置来加载一个文件。
	默认扫描我们当前系统里面所有META-INF/spring.factories位置的文件
    spring-boot-autoconfigure-2.3.4.RELEASE.jar包里面也有META-INF/spring.factories
5、虽然我们127个场景的所有自动配置启动的时候默认全部加载。xxxxAutoConfiguration
按照条件装配规则（@Conditional），最终会按需配置。
```

### 17. springboot的starter

starter就是定义一个starter的jar包，写一个@Configuration配置类，将这些bean定义在里面，然后再starter包的META-INF/spring.factories中写入该配置类，springboot会按照约定来加载该配置类

开发人员只需要将相应的starter包依赖进应用，进行相应的属性配置，就可以直接进行代码开发，使用对应的功能了，比如mybatis-spring-boot-starter

## 三、数据库部分 --- SQL / NoSQL

### ==SQL部分==

### 1. 索引的基本原理

索引用来快速地寻找那些具有特定值的记录，如果没有索引，一般来说执行查询时遍历整张表

索引的原理：就是把无序的数据变成有序的查询

1、把创建了索引的列的内容进行排序

2、对排序结果生成倒排表

3、在倒排表内容上拼上数据地址链

4、在查询的时候，先拿到倒表内容，再取出数据地址链，从而拿到具体数据

**mysql的索引原理**

索引本质上是数据结构 ---- B+树

具体的存储引擎使用不同的索引

InnoDB，MyISAM：B+树

Memory：Hash

### 2. 什么是数据库, 数据库管理系统, 数据库系统, 数据库管理员?

* **数据库** : 数据库(DataBase 简称 DB)就是信息的集合或者说数据库是由数据库管理系统管理的数据的集合。
* **数据库管理系统** : 数据库管理系统(Database Management System 简称 DBMS)是一种操纵和管理数据库的大型软件，通常用于建立、使用和维护数据库。
* **数据库系统** : 数据库系统(Data Base System，简称 DBS)通常由软件、数据库和数据管理员(DBA)组成。
* **数据库管理员** : 数据库管理员(Database Administrator, 简称 DBA)负责全面管理和控制数据库系统。

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\数据库基础知识.jpg)

### 3. 什么是元组, 码, 候选码, 主码, 外码, 主属性, 非主属性？

* **元组** ： 元组（tuple）是关系数据库中的基本概念，关系是一张表，表中的每行（即数据库中的每条记录）就是一个元组，每列就是一个属性。 在二维表里，元组也称为行。
* **码** ：码就是能唯一标识实体的属性，对应表中的列。
* **候选码** ： 若关系中的某一属性或属性组的值能唯一的标识一个元组，而其任何、子集都不能再标识，则称该属性组为候选码。例如：在学生实体中，“学号”是能唯一的区分学生实体的，同时又假设“姓名”、“班级”的属性组合足以区分学生实体，那么{学号}和{姓名，班级}都是候选码。
* **主码** : 主码也叫主键。主码是从候选码中选出来的。 一个实体集中只能有一个主码，但可以有多个候选码。
* **外码** : 外码也叫外键。如果一个关系中的一个属性是另外一个关系中的主码则这个属性为外码。
* **主属性** ： 候选码中出现过的属性称为主属性。比如关系 工人（工号，身份证号，姓名，性别，部门）. 显然工号和身份证号都能够唯一标示这个关系，所以都是候选码。工号、身份证号这两个属性就是主属性。如果主码是一个属性组，那么属性组中的属性都是主属性。
* **非主属性：** 不包含在任何一个候选码中的属性称为非主属性。比如在关系——学生（学号，姓名，年龄，性别，班级）中，主码是“学号”，那么其他的“姓名”、“年龄”、“性别”、“班级”就都可以称为非主属性。

### 4. 主键和外键的区别

* **主键(主码)** ：主键用于唯一标识一个元组，不能有重复，不允许为空。一个表只能有一个主键。
* **外键(外码)** ：外键用来和其他表建立联系用，外键是另一表的主键，外键是可以有重复的，可以是空值。一个表可以有多个外键。

### 5. 数据库范式

**1NF(第一范式)**

属性（对应于表中的字段）不能再被分割，也就是这个字段只能是一个值，不能再分为多个其他的字段了。**1NF 是所有关系型数据库的最基本要求** ，也就是说关系型数据库中创建的表一定满足第一范式。

**2NF(第二范式)**

2NF 在 1NF 的基础之上，消除了非主属性对于码的部分函数依赖。如下图所示，展示了第一范式到第二范式的过渡。第二范式在第一范式的基础上增加了一个列，这个列称为主键，非主属性都依赖于主键。

![第二范式](C:\Users\86198\Desktop\JavaEE\Java八股\image\数据库范式)

一些重要的概念：

* **函数依赖（functional dependency）** ：若在一张表中，在属性（或属性组）X 的值确定的情况下，必定能确定属性 Y 的值，那么就可以说 Y 函数依赖于 X，写作 X → Y。
* **部分函数依赖（partial functional dependency）** ：如果 X→Y，并且存在 X 的一个真子集 X0，使得 X0→Y，则称 Y 对 X 部分函数依赖。比如学生基本信息表 R 中（学号，身份证号，姓名）当然学号属性取值是唯一的，在 R 关系中，（学号，身份证号）->（姓名），（学号）->（姓名），（身份证号）->（姓名）；所以姓名部分函数依赖与（学号，身份证号）；
* **完全函数依赖(Full functional dependency)** ：在一个关系中，若某个非主属性数据项依赖于全部关键字称之为完全函数依赖。比如学生基本信息表 R（学号，班级，姓名）假设不同的班级学号有相同的，班级内学号不能相同，在 R 关系中，（学号，班级）->（姓名），但是（学号）->(姓名)不成立，（班级）->(姓名)不成立，所以姓名完全函数依赖与（学号，班级）；
* **传递函数依赖** ： 在关系模式 R(U)中，设 X，Y，Z 是 U 的不同的属性子集，如果 X 确定 Y、Y 确定 Z，且有 X 不包含 Y，Y 不确定 X，（X∪Y）∩Z=空集合，则称 Z 传递函数依赖(transitive functional dependency) 于 X。传递函数依赖会导致数据冗余和异常。传递函数依赖的 Y 和 Z 子集往往同属于某一个事物，因此可将其合并放到一个表中。比如在关系 R(学号 , 姓名, 系名，系主任)中，学号 → 系名，系名 → 系主任，所以存在非主属性系主任对于学号的传递函数依赖。。

**3NF(第三范式)**

3NF 在 2NF 的基础之上，消除了非主属性对于码的传递函数依赖 。符合 3NF 要求的数据库设计，**基本**上解决了数据冗余过大，插入异常，修改异常，删除异常的问题。比如在关系 R(学号 , 姓名, 系名，系主任)中，学号 → 系名，系名 → 系主任，所以存在非主属性系主任对于学号的传递函数依赖，所以该表的设计，不符合 3NF 的要求。

**总结**

* 1NF：属性不可再分。
* 2NF：1NF 的基础之上，消除了非主属性对于码的部分函数依赖。
* 3NF：3NF 在 2NF 的基础之上，消除了非主属性对于码的传递函数依赖 。

### 6. 什么是存储过程

我们可以把存储过程看成是一些 SQL 语句的集合，中间加了点逻辑控制语句。存储过程在业务比较复杂的时候是非常实用的，比如很多时候我们完成一个操作可能需要写一大串 SQL 语句，这时候我们就可以写有一个存储过程，这样也方便了我们下一次的调用。存储过程一旦调试完成通过后就能稳定运行，另外，使用存储过程比单纯 SQL 语句执行要快，因为存储过程是预编译过的。

存储过程在互联网公司应用不多，因为存储过程难以调试和扩展，而且没有移植性，还会消耗数据库资源。

### 7. drop、delete和truncate的区别

* drop(丢弃数据): `drop table 表名` ，**直接将表都删除掉**，在删除表的时候使用。
* truncate (清空数据) : `truncate table 表名` ，**只删除表中的数据，再插入数据的时候自增长 id 又从 1 开始**，在清空表中数据的时候使用。
* delete（删除数据） : `delete from 表名 where 列名=值`，删除某一列的数据，如果不加 where 子句和`truncate table 表名`作用类似。

truncate 和不带 where 子句的 delete、以及 drop 都会删除表内的数据，但是 **truncate 和 delete 只删除数据不删除表的结构(定义)，执行 drop 语句，此表的结构也会删除，也就是执行 drop 之后对应的表不复存在。**

truncate 和 drop 属于 DDL(数据定义语言)语句，操作立即生效，原数据不放到 rollback segment 中，不能回滚，操作不触发 trigger。而 delete 语句是 DML (数据库操作语言)语句，这个操作会放到 rollback segement 中，事务提交之后才生效。

* DML 是数据库操作语言（Data Manipulation Language）的缩写，是指对数据库中表记录的操作，主要包括表记录的插入（insert）、更新（update）、删除（delete）和查询（select），是开发人员日常使用最频繁的操作。
* DDL （Data Definition Language）是数据定义语言的缩写，简单来说，就是对数据库内部的对象进行创建、删除、修改的操作语言。它和 DML 语言的最大区别是 DML 只是对表内部数据的操作，而不涉及到表的定义、结构的修改，更不会涉及到其他对象。DDL 语句更多的被数据库管理员（DBA）所使用，一般的开发人员很少使用。



### 8. 查询优化

#### 8.1 关联查询优化





### ==NoSQL部分==

### 1. Redis事务

Redis事务是一个单独的隔离操作：事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断

Redis 可以通过 **`MULTI`，`EXEC`，`DISCARD` 和 `WATCH`** 等命令来实现事务(transaction)功能。

+ Multi命令：输入Multi命令之后，输入的命令都会依次进入命令队列中，但不会执行
+ exec命令：会将命令队列中的命令按顺序执行
+ discard命令：命令组队的过程中，可以通过discard命令来放弃组队，它会清空命令事务中保存的所有命令

```bash
> multi    // 开启事务
OK
> set k1 v1
QUEUED
> get k1
QUEUED
> exec
1) OK
2) "v1"
```

当multi组队过程中某个命令报错（指的是输入的命令不符合语法规范，编译报错），这将导致队列中所有的命令都会被取消

```bash
> multi
OK
> set k1 v1
QUEUED
> get k1
QUEUED
> set k2
(error) ERR wrong number of arguments for 'set' command
> get k2
QUEUED
> exec
(error) EXECABORT Transaction discarded because of previous errors
```

如果执行阶段某个命令出错，则只有报错的命令不会被执行，而其他命令都会执行，事务不会回滚

```bash
> multi    // 开启事务
OK
> set k1 v1
QUEUED
> incr k1
QUEUED
> get k1
QUEUED
> exec
1) OK
2)(error) ERR value is not an integer or out of range
3)"v1"
```

+ watch命令，在multi命令之前可以使用watch命令监视一个或者多个key，如果在事务执行之前这个(或这些) key 被其他命令所改动，那么事务将被打断。

Redis事务的特性：

**单独的隔离操作** 

+ 事务中的所有命令都会序列化、按顺序地执行。事务在执行的过程中，不会被其他客户端发送来的命令请求所打断。 

**没有隔离级别的概念** 

+ 队列中的命令没有提交之前都不会实际被执行，因为事务提交前任何指令都不会被实际执行

**不保证原子性** 

+ 事务中如果有一条命令执行失败，其后的命令仍然会被执行，没有回滚 

### 2. Redis与Memcache的区别

**共同点：**

+ 都是基于内存的数据库，一般都用来当作缓存
+ 都有过期策略
+ 两者的性能都非常高

**区别：**

+ **Redis 支持更丰富的数据类型（支持更复杂的应用场景）**。Redis 不仅仅支持简单的 k/v 类型的数据，同时还提供 list，set，zset，hash 等数据结构的存储。Memcached 只支持最简单的 k/v 数据类型。
+ **Redis 支持数据的持久化，可以将内存中的数据保持在磁盘中，重启的时候可以再次加载进行使用,而 Memcached 把数据全部存在内存之中。**
+ **Memcached 是多线程，非阻塞 IO 复用的网络模型；Redis 使用单线程的多路 IO 复用模型。** （Redis 6.0 引入了多线程 IO ）
+ **Memcached 没有原生的集群模式，需要依靠客户端来实现往集群中分片写入数据；但是 Redis 目前是原生支持 cluster 模式的。**

### 3. Redis的适用场景

1、用作缓存，减轻MySQL的查询压力，提升系统的性能

2、适合制作排行榜，可以利用redis的SortedSet集合实现

3、计数器，利用Redis中原子性的自增操作，可以统计用户的点赞数，用户访问量等；如果使用MySQL，频繁地读写会带来相当大的压力

4、Session 共享：Session 是保存在服务器的文件中，如果是集群服务，同一个用户过来可能落在不同机器上，这就会导致用户频繁登陆；采用 Redis 保存 Session 后，无论用户落在那台机器上都能够获取到对应的 Session 信息。



**Redis不适用的场景**

+ 数据量太大、数据访问频率非常低的业务都不适合使用 Redis，数据太大会增加成本，访问频率太低，保存在内存中纯属浪费资源。



### 4. Redis持久化

持久化是在指定的时间间隔内将内存中的数据集快照写入磁盘， 它恢复时是将快照文件直接读到内存里

Redis支持两种不同方式的持久化操作：

+ RDB
+ AOF

**RDB：**Redis 可以通过创建快照来获得存储在内存里面的数据在某个时间点上的副本。Redis 创建快照之后，可以对快照进行备份，可以将快照复制到其他服务器从而创建具有相同数据的服务器副本（Redis 主从结构，主要用来提高 Redis 性能），还可以将快照留在原地以便重启服务器的时候使用。

快照持久化是 Redis 默认采用的持久化方式，在 Redis.conf 配置文件中默认有此下配置：

```
save 900 1           #在900秒(15分钟)之后，如果至少有1个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 300 10          #在300秒(5分钟)之后，如果至少有10个key发生变化，Redis就会自动触发BGSAVE命令创建快照。

save 60 10000        #在60秒(1分钟)之后，如果至少有10000个key发生变化，Redis就会自动触发BGSAVE命令创建快照。
```



RDB的优点：

+ 适合大规模的数据恢复
+ 对数据完整性和一致性要求不高更适合使用
+ 节省磁盘空间
+ 恢复速度快

RDB的不足：

+ Fork的时候，内存中的数据被克隆了一份，大致2倍的膨胀性需要考虑
+ 虽然Redis在fork时使用了**写时拷贝技术**,但是如果数据庞大时还是比较消耗性能。
+ 在备份周期在一定间隔时间做一次备份，所以如果Redis意外down掉的话，就会丢失最后一次快照后的所有修改。

**AOF：**



### 5. Redis缓存穿透

**缓存穿透**指的是客户端想要访问的某个数据在数据库中并不存在，在Redis中也不存在，所以每次针对这个key的访问都不会经过缓存，直接访问数据库，如果在短时间内这种请求过多，导致数据库访问量加大，会使得数据库服务器崩溃，这就是缓存穿透

举个例子：某个黑客故意制造我们缓存中不存在的 key 发起大量请求，导致大量请求落到数据库。

![缓存穿透情况](C:\Users\86198\Desktop\JavaEE\Java八股\image\Redis缓存穿透.jpg)

**缓存穿透的解决方法**

1、缓存无效key，如果一个查询返回的数据为空（不管是数据是否不存在），我们仍然把这个空结果（null）进行缓存，将这个无效key的过期时间设置的短一些比如1min

2、布隆过滤器，把所有可能存在的请求的值都存放在布隆过滤器中，当用户请求过来，先判断用户发来的请求的值是否存在于布隆过滤器中。不存在的话，直接返回请求参数错误信息给客户端，存在的话才会走下面的流程。

布隆过滤器实际上是一个bit 向量或者说 bit 数组

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\布隆过滤器.jpg)



布隆过滤器添加数据的过程：如果我们要映射一个值到布隆过滤器中，我们需要使用**多个不同的哈希函数**生成**多个哈希值，**并对每个生成的哈希值指向的 bit 位置 1，例如针对值 “baidu” 和三个不同的哈希函数分别生成了哈希值 1、4、7，则上图转变为：

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\布隆过滤器添加数据1.jpg)

判断一个key是否在布隆过滤器中：

(1)、按照之前的hash函数计算key的三个哈希值（实际中不一定是三个）

(2)、得到值之后判断位数组中的每个元素是否都为 1，如果值都为 1，那么说明这个值在布隆过滤器中，如果存在一个值不为 1，说明该元素不在布隆过滤器中。

比如现在要判断tencent是否在有效key中，假设tecent通过hash计算出来的值为2，4，6，比较时发现2位置上的值为0，说明tencent不在布隆过滤器中

但布隆过滤器会出现误判的情况（key不存在的情况不会误判），因为随着数据的增加，bit数组的值不断会被填满，假设现在查询key='taobao'，计算出来的hash值恰好是1，4，7，去比较的时候发现都是1，布隆过滤器就会认为这个值是存在的，但其实这是baidu的hash值，taobao并不存在

带上布隆过滤器之后的服务处理过程：

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\加入布隆过滤器后的缓存处理流程.jpg)

### 6. Redis缓存雪崩

缓存雪崩是指：缓存中的**大量key值在同一时间过期**，导致后面的请求全部落到数据库上，造成数据库短时间内承受了大量请求，可能导致数据库服务器崩溃

**缓存雪崩的解决办法**

1、采用Redis集群，避免单个Redis宕机后整个服务无法使用的情况

2、构建多级缓存，nginx缓存 + redis缓存 + 其他缓存（ehcache）

3、使用锁或者队列的方式来限制访问流量

4、将缓存失效的时间分散开来，设置key过期时间时，使用随机数，避免在同一时间大量key的同时失效

## 四、分布式与微服务

### 1. CAP理论

**CAP** 也就是 **Consistency（一致性）**、**Availability（可用性）**、**Partition Tolerance（分区容错性）** 这三个单词首字母组合。

- **一致性（Consistency）** : 所有节点访问同一份最新的数据副本
- **可用性（Availability）**: 非故障的节点在合理的时间内返回合理的响应（不是错误或者超时的响应）。
- **分区容错性（Partition tolerance）** : 分布式系统出现网络分区的时候，仍然能够对外提供服务。

> 网络分区：
>
> 分布式系统中，多个节点之前的网络本来是连通的，但是因为某些故障（比如部分节点网络出了问题）某些节点之间不连通了，整个网络就分成了几块区域，这就叫网络分区。

CAP理论的核心是：一个分布式系统不可能同时很好的满足一致性，可用性和分区容错性这三个需求，**最多只能同时较好的满足两个**

![](C:\Users\86198\Desktop\JavaEE\SpringCloud\CAP理论.jpeg)

CA：单点集群，满足一致性，可用性，通常在可扩展性上不太强大

CP：满足一致性，分区容忍性的系统，通常性能不是特别高

AP：满足可用性，分区容错性的系统，通常可能对一致性的要求低一些

CAP理论就是说在分布式存储系统中，最多只能实现上面的两点，而由于当前的网络硬件肯定会出现延迟丢包等问题，所以**分区容错性必须要实现**

CAP理论的应用实例：

1. **ZooKeeper 保证的是 CP。** 任何时刻对 ZooKeeper 的读请求都能得到一致性的结果，但是， ZooKeeper 不保证每次请求的可用性比如在 Leader 选举过程中或者半数以上的机器不可用的时候服务就是不可用的。
2. **Eureka 保证的则是 AP。** Eureka 在设计的时候就是优先保证 A （可用性）。在 Eureka 中不存在什么 Leader 节点，每个节点都是一样的、平等的。因此 Eureka 不会像 ZooKeeper 那样出现选举过程中或者半数以上的机器不可用的时候服务就是不可用的情况。 Eureka 保证即使大部分节点挂掉也不会影响正常提供服务，只要有一个节点是可用的就行了。只不过这个节点上的数据可能并不是最新的。
3. **Nacos 不仅支持 CP 也支持 AP。**

### 2. RPC

**RPC（Remote Procedure Call）** 即远程过程调用，通过名字我们就能看出 RPC 关注的是远程调用而非本地调用。

**为什么要 RPC  ？** 因为，两个不同的服务器上的服务提供的方法不在一个内存空间，所以，需要通过网络编程才能传递方法调用所需要的参数。并且，方法调用的结果也需要通过网络编程来接收。但是，如果我们自己手动网络编程来实现这个调用过程的话工作量是非常大的，因为，我们需要考虑底层传输方式（TCP还是UDP）、序列化方式等等方面。

**RPC基本原理**

1. **客户端（服务消费端）** ：调用远程方法的一端。
1. **客户端 Stub（桩）** ： 这其实就是一代理类。代理类主要做的事情很简单，就是把你调用方法、类、方法参数等信息传递到服务端。
1. **网络传输** ： 网络传输就是你要把你调用的方法的信息比如说参数啊这些东西传输到服务端，然后服务端执行完之后再把返回结果通过网络传输给你传输回来。网络传输的实现方式有很多种比如最近基本的 Socket或者性能以及封装更加优秀的 Netty（推荐）。
1. **服务端 Stub（桩）** ：这个桩就不是代理类了。我觉得理解为桩实际不太好，大家注意一下就好。这里的服务端 Stub 实际指的就是接收到客户端执行方法的请求后，去指定对应的方法然后返回结果给客户端的类。
1. **服务端（服务提供端）** ：提供远程方法的一端。

![](C:\Users\86198\Desktop\JavaEE\SpringCloud\RPC基本原理.jpg)

RPC过程：

1. 服务消费端（client）以本地调用的方式调用远程服务；
1. 客户端 Stub（client stub） 接收到调用后负责将方法、参数等组装成能够进行网络传输的消息体（序列化）：`RpcRequest`；
1. 客户端 Stub（client stub） 找到远程服务的地址，并将消息发送到服务提供端；
1. 服务端 Stub（桩）收到消息将消息反序列化为Java对象: `RpcRequest`；
1. 服务端 Stub（桩）根据`RpcRequest`中的类、方法、方法参数等信息调用本地的方法；
1. 服务端 Stub（桩）得到方法执行结果并将组装成能够进行网络传输的消息体：`RpcResponse`（序列化）发送至消费方；
1. 客户端 Stub（client stub）接收到消息并将消息反序列化为Java对象:`RpcResponse` ，这样也就得到了最终结果。over!

### 3. 分布式ID

**什么是分布式ID**

以MySQL为例，在一个业务数据量不大的情况下，单库单表完全可以支撑现有的业务，当数据量增大时，也可以使用MySQL主从复制来承载这些数据

但当数据再次扩展，数据量已经大到主从复制也容不下的情况，这时，就需要对数据库进行分库分表，将不同的数据库部署在不同的服务器中；但分库分表后，需要有一个唯一ID来标识一条数据，每张表中的自增的主键ID显然不能满足要求，这时，需要生成一个`全局唯一的ID`来标识这些数据，这就是分布式ID

**分布式ID的要求**

- **全局唯一** ：ID 的全局唯一性肯定是首先要满足的！
- **高性能** ： 分布式 ID 的生成速度要快，对本地资源消耗要小。
- **高可用** ：生成分布式 ID 的服务要保证可用性无限接近于 100%。
- **方便易用** ：拿来即用，使用方便，快速接入！



**雪花算法**

Snowflake 是 Twitter 开源的分布式 ID 生成算法。Snowflake 由 64 bit 的二进制数字组成，这 64bit 的二进制被分成了几部分，每一部分存储的数据都有特定的含义

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\雪花算法.jpg)

Snowflake ID组成结构：正数位（占1比特）+ 时间戳（占41比特）+ 机器ID（占5比特）+ 数据中心（占5比特）+ 自增值（占12比特），总共64比特组成的一个Long类型。

- 第一个bit位（1bit）：Java中long的最高位是符号位代表正负，正数是0，负数是1，一般生成ID都为正数，所以默认为0。
- 时间戳部分（41bit）：毫秒级的时间，不建议存当前时间戳，而是用（当前时间戳 - 固定开始时间戳）的差值，可以使产生的ID从更小的值开始；41位的时间戳可以使用69年，(1L << 41) / (1000L * 60 * 60 * 24 * 365) = 69年
- 工作机器id（10bit）：也被叫做`workId`，这个可以灵活配置，机房或者机器号组合都可以。
- 序列号部分（12bit），自增值支持同一毫秒内同一个节点可以生成4096个ID

```java
/**
 * Twitter的SnowFlake算法,使用SnowFlake算法生成一个整数，然后转化为62进制变成一个短地址URL
 *
 * https://github.com/beyondfengyu/SnowFlake
 */
public class SnowFlakeShortUrl {

    /**
     * 起始的时间戳
     */
    private final static long START_TIMESTAMP = 1480166465631L;

    /**
     * 每一部分占用的位数
     */
    private final static long SEQUENCE_BIT = 12;   //序列号占用的位数
    private final static long MACHINE_BIT = 5;     //机器标识占用的位数
    private final static long DATA_CENTER_BIT = 5; //数据中心占用的位数

    /**
     * 每一部分的最大值
     */
    private final static long MAX_SEQUENCE = -1L ^ (-1L << SEQUENCE_BIT);
    private final static long MAX_MACHINE_NUM = -1L ^ (-1L << MACHINE_BIT);
    private final static long MAX_DATA_CENTER_NUM = -1L ^ (-1L << DATA_CENTER_BIT);

    /**
     * 每一部分向左的位移
     */
    private final static long MACHINE_LEFT = SEQUENCE_BIT;
    private final static long DATA_CENTER_LEFT = SEQUENCE_BIT + MACHINE_BIT;
    private final static long TIMESTAMP_LEFT = DATA_CENTER_LEFT + DATA_CENTER_BIT;

    private long dataCenterId;  //数据中心
    private long machineId;     //机器标识
    private long sequence = 0L; //序列号
    private long lastTimeStamp = -1L;  //上一次时间戳

    private long getNextMill() {
        long mill = getNewTimeStamp();
        while (mill <= lastTimeStamp) {
            mill = getNewTimeStamp();
        }
        return mill;
    }

    private long getNewTimeStamp() {
        return System.currentTimeMillis();
    }

    /**
     * 根据指定的数据中心ID和机器标志ID生成指定的序列号
     *
     * @param dataCenterId 数据中心ID
     * @param machineId    机器标志ID
     */
    public SnowFlakeShortUrl(long dataCenterId, long machineId) {
        if (dataCenterId > MAX_DATA_CENTER_NUM || dataCenterId < 0) {
            throw new IllegalArgumentException("DtaCenterId can't be greater than MAX_DATA_CENTER_NUM or less than 0！");
        }
        if (machineId > MAX_MACHINE_NUM || machineId < 0) {
            throw new IllegalArgumentException("MachineId can't be greater than MAX_MACHINE_NUM or less than 0！");
        }
        this.dataCenterId = dataCenterId;
        this.machineId = machineId;
    }

    /**
     * 产生下一个ID
     *
     * @return
     */
    public synchronized long nextId() {
        long currTimeStamp = getNewTimeStamp();
        if (currTimeStamp < lastTimeStamp) {
            throw new RuntimeException("Clock moved backwards.  Refusing to generate id");
        }

        if (currTimeStamp == lastTimeStamp) {
            //相同毫秒内，序列号自增
            sequence = (sequence + 1) & MAX_SEQUENCE;
            //同一毫秒的序列数已经达到最大
            if (sequence == 0L) {
                currTimeStamp = getNextMill();
            }
        } else {
            //不同毫秒内，序列号置为0
            sequence = 0L;
        }

        lastTimeStamp = currTimeStamp;

        return (currTimeStamp - START_TIMESTAMP) << TIMESTAMP_LEFT //时间戳部分
                | dataCenterId << DATA_CENTER_LEFT       //数据中心部分
                | machineId << MACHINE_LEFT             //机器标识部分
                | sequence;                             //序列号部分
    }
    
    public static void main(String[] args) {
        SnowFlakeShortUrl snowFlake = new SnowFlakeShortUrl(2, 3);

        for (int i = 0; i < (1 << 4); i++) {
            //10进制
            System.out.println(snowFlake.nextId());
        }
    }
}
```



## 五、并发编程

### 1. 线程的生命周期，线程有哪些状态

1、 **初始(NEW)**：新创建了一个线程对象，但还没有调用start()方法。
**运行(RUNNABLE)**：Java线程中将就绪（ready）和运行中（running）两种状态笼统的称为“运行”。
线程对象创建后，其他线程(比如main线程）调用了该对象的start()方法。该状态的线程位于可运行线程池中，等待被线程调度选中，获取CPU的使用权，此时处于就绪状态（ready）。就绪状态的线程在获得CPU时间片后变为运行中状态（running）。
**阻塞(BLOCKED)**：表示线程阻塞于锁。
**等待(WAITING)**：进入该状态的线程需要等待其他线程做出一些特定动作（通知或中断）。

**超时等待(TIMED_WAITING)**：该状态不同于WAITING，它可以在指定的时间后自行返回。

**终止(TERMINATED)**：表示该线程已经执行完毕。

### 2. sleep()方法和wait()方法的区别

**共同点**：

+ `wait()、wait(long)、sleep(long)`的效果都是让当前线程暂时放弃CPU的使用权，进入阻塞状态
+ 两个方法都能够被打断唤醒(`interrupt`)

**不同点**

+ 方法归属不同
  + `sleep()`方法是Thread类中的静态方法
  + `wati()`方法是Object类中的方法，每个对象都有
+ 唤醒的时机不同
  + `sleep(long)`和`wait(long)`方法在线程等待指定时间之后就可以自动唤醒
  + `wait()`方法还能够被`notify() / notifyAll()`方法唤醒，无参的wait方法如果不被唤醒将一致等待下去
+ 锁特性不同
  + 如果想要调用wait方法，就必须先获取wait对象的锁，而sleep方法能够在任何地方使用
  + wait方法被调用之后，就会释放对象锁，允许其他线程获得锁
  + sleep方法如果在synchronized代码块中执行，并不会释放对象锁

### 3. 单例模式在多线程模式下的问题

经典的单例模式（懒汉式）

```java
public class Singleton{
    private static Singleton instance = null;
    private Singleton(){}
    public static Singleton getInstance(){
        if(instance == null){
            instance = new Singleton();
        }
        return instance;
    }
}
```

在多线程模式下由于多个线程都需要去做判断`if(instance == null)`，在判断的同时，已经进入if语句块的线程可能还没有执行完`new Singleton()`导致多个线程创建了多个实例，这就不符合单例模式的特征了

最简单的解决方法就是在方法上加上synchronized，但影响性能

可以使用双重校验锁

```java
public class Singleton{
    //注意记得加上volatile
    private static volatile Singleton instance = null;
    private Singleton(){}
    public static Singleton getInstance(){
        //创建完对象之后，这里instance == null始终为false，可以直接返回instance对象，无须再获取锁
        if(instance == null){
            //第一次进入if判断时，会对创建对象的代码进行保护
            synchronized(Singleton.class){
                if(instance == null){
                    instance = new Singleton();
                }
        	}
        }
        return instance;
    }
}
```



## 六、计算机网络

### 1. OSI七层体系&TCP-IP四层结构

OSI七层模型是国际标准化组织提出的一个网络分层模型

+ 应用层：各种应用程序协议，如HTTP、FTP等
+ 表示层：信息的语法语义以及它们的关联，如加密解密，转换翻译。压缩解压缩等
+ 会话层：不同机器上的用户之间建立及管理会话
+ 传输层：接收上一层的数据，在必要的时候把数据进行分割，并将这些数据交给网络层，且保证这些数据段有效到达对端
+ 网络层：控制子网的运行，如逻辑编址，分组传输、路由选择等
+ 数据链路层：物理寻址。同时将原始比特流转变为逻辑传输链路
+ 物理层：机械、电子、定时接口通信信道上的原始比特流传输

TCP-IP四层模型：

+ 应用层：**应用层位于传输层之上，主要提供两个终端设备上的应用程序之间信息交换的服务，它定义了信息交换的格式，消息会交给下一层传输层来传输。** 我们把应用层交互的数据单元称为报文。

  应用层协议定义了网络通信规则，对于不同的网络应用需要不同的应用层协议。在互联网中应用层协议很多，如支持 Web 应用的 HTTP 协议，支持电子邮件的 SMTP 协议等等。

+ 传输层：**传输层的主要任务就是负责向两台终端设备进程之间的通信提供通用的数据传输服务。** 应用进程利用该服务传送应用层报文。“通用的”是指并不针对某一个特定的网络应用，而是多种应用可以使用同一个运输层服务。

  **运输层主要使用以下两种协议：**

  1. **传输控制协议 TCP**（Transmisson Control Protocol）--提供**面向连接**的，**可靠的**数据传输服务。
  2. **用户数据协议 UDP**（User Datagram Protocol）--提供**无连接**的，尽最大努力的数据传输服务（**不保证数据传输的可靠性**）。

+ 网络层：**网络层负责为分组交换网上的不同主机提供通信服务。** 在发送数据时，网络层把运输层产生的报文段或用户数据报封装成分组和包进行传送。在 TCP/IP 体系结构中，由于网络层使用 IP 协议，因此分组也叫 IP 数据报，简称数据报。

  **网络层的还有一个任务就是选择合适的路由，使源主机运输层所传下来的分株，能通过网络层中的路由器找到目的主机。**

+ 网络接口层：我们可以把网络接口层看作是数据链路层和物理层的合体。

  1. 数据链路层(data link layer)通常简称为链路层（ 两台主机之间的数据传输，总是在一段一段的链路上传送的）。**数据链路层的作用是将网络层交下来的 IP 数据报组装成帧，在两个相邻节点间的链路上传送帧。每一帧包括数据和必要的控制信息（如同步信息，地址信息，差错控制等）。**
  2. **物理层的作用是实现相邻计算机节点之间比特流的透明传送，尽可能屏蔽掉具体传输介质和物理设备的差异**

网络体系结构的分层思想：

+ 每层在自己下层所提供服务的支持下，通过自身内部功能实现一种或几种特定的服务

### 2. http与https的区别

**http协议：**超文本传输协议，用来规范超文本的传输，超文本，也就是网络上的包括文本在内的各式各样的消息，具体来说，主要是来规范浏览器和服务器端的行为的。

http是一个**无状态**的协议，包含两个部分：请求和响应

HTTP 是应用层协议，它以 TCP（传输层）作为底层协议，默认端口为 80（HTTP协议本身是**无连接**的）. 通信过程主要如下：

1. 服务器在 80 端口等待客户的请求。
2. 浏览器发起到服务器的 TCP 连接（创建套接字 Socket）。
3. 服务器接收来自浏览器的 TCP 连接。
4. 浏览器（HTTP 客户端）与 Web 服务器（HTTP 服务器）交换 HTTP 消息。
5. 关闭 TCP 连接。

http协议的优点：扩展性强、速度快、跨平台支撑性好

> HTTP响应状态码：
>
> + 1xx表示通知信息，如请求收到了或正在处理
> + 2xx表示请求成功
> + 3xx表示重定向，如要完成请求还必须采取进一步行动
> + 4xx表示客户端差错，如请求中有错误的语法或不能完成
> + 5xx表示服务器的差错，如服务器失效无法完成请求

**https协议：**是 HTTP 的加强安全版本。HTTPS 是基于 HTTP 的，也是用 TCP 作为底层协议，并额外使用 SSL/TLS 协议用作加密和安全认证。默认端口号是 443.

http协议的优点：保密性好，信任度高

*SSL/TLS工作原理*

> 非对称加密：非对称加密采用两个密钥——一个公钥，一个私钥。在通信时，私钥仅由解密者保存，公钥由任何一个想与解密者通信的发送者（加密者）所知。

补充：ssh -- 安全网络传输协议，是目前较可靠，专为远程登录会话和其他网络服务提供安全性的协议。利用 SSH 协议可以有效防止远程管理过程中的信息泄露问题。SSH  建立在可靠的传输协议 TCP 之上。

**Telnet 和 SSH 之间的主要区别在于 SSH 协议会对传输的数据进行加密保证数据安全性。**

### 3. TCP三次握手四次挥手

#### 3.1 **TCP三次握手建立连接**

+ 第一次: `客户端 --- 服务器`。客户端向服务器提出连接建立请求，即发出同步请求报文。
+ 第二次: `服务器 --- 客户端`。服务器收到客户端的连接请求后，向客户端发出同意建立连接的同步确认报文。 
+ 第三次: `客户端 --- 服务器`。客户端在收到服务器的同步确认报文后，向服务器发出确认报文。
+ 当服务器收到来自客户端的确认报文后，连接即被建立。

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\TCP三次握手1.jpg)



![](C:\Users\86198\Desktop\JavaEE\Java八股\image\TCP三次握手2.jpg)

> 第三次客户端向服务器发送确认时是可以携带要传输的数据的，但前两次不行

**为什么需要三次握手，而不是两次？**

+ 三次握手的首要原因是防止旧的重复连接初始化造成混乱

  >假设有这样一种场景，客户端发送了第一个请求连接并且没有丢失，只是因为在网络结点中滞留的时间太长了，由于TCP的客户端迟迟没有收到确认报文，以为服务器没有收到，此时重新向服务器发送这条报文，此后客户端和服务器经过两次握手完成连接，传输数据，然后关闭连接。**此时此前滞留的那一次请求连接，网络通畅了到达了服务器**，这个报文本该是失效的，但是，**两次握手的机制将会让客户端和服务器再次建立连接**，这将导致不必要的错误和资源的浪费。（两次握手就是你发请求我确认，但我不管是新的请求还是旧的请求，只要这个请求有效）
  >
  >如果采用的是三次握手，就算是那一次失效的报文传送过来了，服务端接受到了那条失效报文并且回复了确认报文，但是客户端不会再次发出确认。由于服务器收不到确认，就知道客户端并没有请求连接。

+ 同步双方初始序列号

  > 序列号在 TCP 连接中占据着非常重要的作用，所以当客户端发送携带「初始序列号」的 `SYN` 报文的时候，需要服务端回一个 `ACK` 应答报文，表示客户端的 `SYN` 报文已被服务端成功接收，那当服务端发送「初始序列号」给客户端的时候，依然也要得到客户端的应答回应，**这样一来一回，才能确保双方的初始序列号能被可靠的同步。**

+ 避免资源浪费

  > 如果只有「两次握手」，当客户端的 `SYN` 请求连接在网络中阻塞，客户端没有接收到 `ACK` 报文，就会重新发送 `SYN` ，由于没有第三次握手，服务器不清楚客户端是否收到了自己发送的建立连接的 `ACK` 确认信号，所以每收到一个 `SYN` 就只能先主动建立一个连接
  >
  > 如果客户端的 `SYN` 阻塞了，重复发送多次 `SYN` 报文，那么服务器在收到请求后就会**建立多个冗余的无效链接，造成不必要的资源浪费。**

#### 3.2 **TCP四次挥手释放连接**

+ 第一次: 客户端 --- 服务器。客户端向服务器发出一个连接释放报文。
+ 第二次: 服务器 --- 客户端。服务器收到客户端的释放连接请求后，向客户端发出确认报文。
+ 第三次: 服务器 --- 客户端。服务器在发送完最后的数据后，向客户端发出连接释放确认报文。  
+ 第四次: 客户端 --- 服务器。客户端在收到服务器连接释放报文后，向服务器发出确认报文。

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\TCP四次挥手1.jpg)

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\TCP四次挥手2.jpg)

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\TCP四次挥手3.jpg)

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\TCP四次挥手4.jpg)

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\TCP四次挥手5.jpg)

### 4. TCP协议与UDP协议的区别

![](C:\Users\86198\Desktop\JavaEE\Java八股\image\tcp-vs-udp.jpg)

用户数据报协议（User Datagram Protocol，UDP）是OSI参考模型和TCP/IP模型中都有的一种面向无连接的传输层协议。

TCP适用于传输大文件，时延大

UDP适用于传输小文件，时延小

UDP协议的主要特点：

+ **UDP是无连接协议，在发送数据之前不需要建立连接。**
+ **UDP使用尽最大努力交付，不保证可靠交付，同时也不使用拥塞控制。**
+ **UDP是面向报文，没有拥塞控制，很适合多媒体通信的要求。**
+ **UDP支持一对一、一对多、多对一和多对多的交互通信。**
+ **UDP 的首部开销小，只有 8 个字节。**

UDP 在传送数据之前不需要先建立连接，远地主机在收到 UDP 报文后，不需要给出任何确认。虽然 UDP 不提供可靠交付，但在某些情况下 UDP 却是一种最有效的工作方式（一般用于即时通信），比如： QQ 语音、 QQ 视频 、直播等等

TCP 提供面向连接的服务。在传送数据之前必须先建立连接，数据传送结束后要释放连接。 TCP 不提供广播或多播服务。由于 TCP 要提供可靠的，面向连接的传输服务（TCP 的可靠体现在 TCP 在传递数据之前，会有三次握手来建立连接，而且在数据传递时，有确认、窗口、重传、拥塞控制机制，在数据传完后，还会断开连接用来节约系统资源），这难以避免增加了许多开销，如确认，流量控制，计时器以及连接管理等。这不仅使协议数据单元的首部增大很多，还要占用许多处理机资源。TCP 一般用于文件传输、发送和接收邮件、远程登录等场景。



### 5. TCP是如何实现可靠传输的

通过`校验和`、`序列号`、`确认应答`、`超时重传`、`连接管理`、`流量控制`、`拥塞控制`等机制来保证可靠性。

**（1）校验和**

在数据传输过程中，将发送的数据段都当做一个16位的整数，将这些整数加起来，并且前面的进位不能丢弃，补在最后，然后取反，得到校验和。

发送方：在发送数据之前计算校验和，并进行校验和的填充。接收方：收到数据后，对数据以同样的方式进行计算，求出校验和，与发送方进行比较。

**（2）序列号**

TCP 传输时将每个字节的数据都进行了编号，这就是序列号。序列号的作用不仅仅是应答作用，有了序列号能够将接收到的数据根据序列号进行排序，并且去掉重复的数据。

**（3）确认应答**

TCP 传输过程中，每次接收方接收到数据后，都会对传输方进行确认应答，也就是发送 ACK 报文，这个 ACK 报文中带有对应的确认序列号，告诉发送方，接收了哪些数据，下一次数据从哪里传。

**（4）超时重传**

在进行 TCP 传输时，由于存在确认应答与序列号机制，也就是说发送方发送一部分数据后，都会等待接收方发送的 ACK 报文，并解析 ACK 报文，判断数据是否传输成功。如果发送方发送完数据后，迟迟都没有接收到接收方传来的 ACK 报文，那么就对刚刚发送的数据进行重发。

TCP连接还有其他的计时器，比如坚持计时器用来防止通信双方陷入死锁的僵局；保活计时器，用来判断TCP连接是否正常

**（5）连接管理**

就是指三次握手、四次挥手的过程。

**（6）流量控制**

如果发送方的发送速度太快，会导致接收方的接收缓冲区填充满了，这时候继续传输数据，就会造成大量丢包，进而引起丢包重传等等一系列问题。TCP 支持根据接收端的处理能力来决定发送端的发送速度，这就是流量控制机制。

具体实现方式：接收端将自己的接收缓冲区大小放入 TCP 首部的『窗口大小』字段中，通过 ACK 通知发送端。

**（7）拥塞控制**

TCP 传输过程中一开始就发送大量数据，如果当时网络非常拥堵，可能会造成拥堵加剧。所以 TCP 引入了`慢启动机制`，在开始发送数据的时候，先发少量的数据探探路。

### 6. TCP如何提高传输效率

TCP 协议提高效率的方式有`滑动窗口`、`快重传`、`延迟应答`、`捎带应答`

**（1）滑动窗口**

如果每一个发送的数据段，都要收到 ACK 应答之后再发送下一个数据段，这样的话我们效率很低，大部分时间都用在了等待 ACK 应答上了。

为了提高效率我们可以一次发送多条数据，这样就能使等待时间大大减少，从而提高性能。窗口大小指的是无需等待确认应答而可以继续发送数据的最大值。

**（2）快重传**

`快重传`也叫`高速重发控制`。

那么如果出现了丢包，需要进行重传。一般分为两种情况：

情况一：数据包已经抵达，ACK被丢了。这种情况下，部分ACK丢了并不影响，因为可以通过后续的ACK进行确认；

情况二：数据包直接丢了。发送端会连续收到多个相同的 ACK 确认，发送端立即将对应丢失的数据重传。

**（3）延迟应答**

如果接收数据的主机立刻返回ACK应答，这时候返回的窗口大小可能比较小。

- 假设接收端缓冲区为1M，一次收到了512K的数据；如果立刻应答，返回的窗口就是512K；
- 但实际上可能处理端处理速度很快，10ms之内就把512K的数据从缓存区消费掉了；
- 在这种情况下，接收端处理还远没有达到自己的极限，即使窗口再放大一些，也能处理过来；
- 如果接收端稍微等一会在应答，比如等待200ms再应答，那么这个时候返回的窗口大小就是1M；

窗口越大，网络吞吐量就越大，传输效率就越高；我们的目标是在保证网络不拥塞的情况下尽量提高传输效率。

**（4）捎带应答**

在延迟应答的基础上，很多情况下，客户端服务器在应用层也是一发一收的。这时候常常采用捎带应答的方式来提高效率，而ACK响应常常伴随着数据报文共同传输。如：三次握手。

## 七、操作系统

### 1. 进程和线程的区别

进程是程序的一次执行，进程中的一个轻型运行实体，是系统进行资源分配和调度的独立单位，他的作用是使程序能够并发执行提高资源利用率和吞吐率。

进程是资源分配和调度的基本单位，因为进程的创建、销毁、切换产生大量的时间和空间的开销，进程的数量不能太多而线程是比进程更小的能独立运行的基本单位，他是进程的一个实体，可以减少程序并发执行时的时间和空间开销，使得操作系统具有更好的并发性。

线程基本不拥有系统资源，只有一些运行时必不可少的资源，比如程序计数器、寄存器和栈，进程则占有堆、栈。
