# 面向对象

具备某种功能的实体，称为对象。面向对象最小的程序单元是：类。面向对象思想是站在对象的角度思考问题，把多个功能合理放到不同对象里，强调的是具备某些功能的对象。

面向对象更加符合常规的思维方式，稳定性好，可重用性强，易于开发大型软件产品，有良好的可维护性。

在软件工程上，面向对象可以使工程更加模块化，实现更低的耦合和更高的内聚。

## 继承

继承是类与类的一种关系，是一种“is a”的关系。比如“狗”继承“动物”，这里动物类是狗类的父类或者基类，狗类是动物类的子类或者派生类。

java中的继承是单继承，即一个类只有一个父类。但是可以通过内部类继承其他类来实现多继承。

### 继承的好处

子类拥有父类的所有属性和方法（除了private修饰的属性不能拥有）从而实现了实现代码的复用；

### 抽象类

抽象类和抽象方法都使用 abstract 关键字进行声明。如果一个类中包含抽象方法，那么这个类必须声明为抽象类。

抽象类和普通类最大的区别是，抽象类不能被实例化，只能被继承。

### 接口

接口是抽象类的延伸，在 Java 8 之前，它可以看成是一个完全抽象的类，也就是说它不能有任何的方法实现。

从 Java 8 开始，接口也可以拥有默认的方法实现，这是因为不支持默认方法的接口的维护成本太高了。在 Java 8 之前，如果一个接口想要添加新的方法，那么要修改所有实现了该接口的类，让它们都实现新增的方法。

接口的成员（字段 + 方法）默认都是 public 的，并且不允许定义为 private 或者 protected。

接口的字段默认都是 static 和 final 的。

### 重写和重载

方法重载：在Java中，同一个类中的多个方法可以有相同的方法名称，但是有不同的参数列表，这就称为方法重载（method overloading）。

参数列表又叫参数签名，包括参数的类型、参数的个数、参数的顺序，只要有一个不同就叫做参数列表不同。

方法重写：相对继承而言，子类中对父类已经存在的方法进行区别化的修改。

#### 子类方法重写父类方法的原则 ####

遵循“两同两小一大”：
两同：方法名、形参
两小：子类的返回值类型和抛出异常应该比父类小或者相等
一大：子类的方法权限比父类的大或者相等。

### 继承的初始化顺序

1、初始化父类再初始化子类

2、先执行初始化对象中属性，再执行构造方法中的初始化。

java程序的执行顺序是：

　父类对象属性初始化---->父类对象构造方法---->子类对象属性初始化--->子类对象构造方法

### final关键字

1. final 修饰类，则该类不允许被继承。
2. final 修饰方法，则该方法不允许被重写。
3. final 修饰属性，则该类的该属性不会进行隐式的初始化，所以 该final 属性的初始化属性必须有值，或在**构造方法中赋值(但只能选其一，且必须选其一，因为没有默认值！)，**且初始化之后就不能改了，只能赋值一次。
4. final 修饰变量，则该变量的值只能赋一次值，在声明变量的时候才能赋值，即变为常量。

### super关键字

在对象的内部使用，可以代表父类对象。

1、访问父类的属性：

2、访问父类的方法：

## 封装

封装是将代码及其处理的数据绑定在一起的一种编程机制，该机制保证了程序和数据都不受外部干扰且不被误用。

### 封装的优点

1. 良好的封装能够减少耦合。
1. 类内部的结构可以自由修改。
1. 可以对成员变量进行更精确的控制。
1. 隐藏信息，实现细节。

### 封装实现

属性设置为private，添加set/get方法

### 访问修饰符

![](https://camo.githubusercontent.com/64cfa829e55ce7af78e136b035a6c3c1fd177558/68747470733a2f2f696d61676573323031352e636e626c6f67732e636f6d2f626c6f672f313138393331322f3230313730362f313138393331322d32303137303633303137343931393237342d313835373239333830312e706e67)

### this关键字

1.this关键字代表当前对象

　　this.属性 操作当前对象的属性

　　this.方法 调用当前对象的方法。

2.封装对象的属性的时候，经常会使用this关键字。

3.当getter和setter函数参数名和成员函数名重合的时候，可以使用this****区别。

## 多态

面向对象的多态性，即“一个接口，多个方法”。多态性体现在父类中定义的属性和方法被子类继承后，可以具有不同的属性或表现方式。

### JAVA中的多态

多态分为编译时多态和运行时多态：

- 编译时多态主要指方法的重载
- 运行时多态指程序中定义的对象引用所指向的具体类型在运行期间才确定

运行时多态有三个条件：

- 继承
- 覆盖（重写）
- 向上转型

#### 引用多态

父类的引用可以指向本类的对象；

父类的引用可以指向子类的对象；

#### 方法多态

根据上述创建的两个对象：本类对象和子类对象，同样都是父类的引用，当我们指向不同的对象时，它们调用的方法也是多态的。

　　创建本类对象时，调用的方法为本类方法；

　　创建子类对象时，调用的方法为子类重写的方法或者继承的方法；

#### 引用类型转换

1.向上类型转换(隐式/自动类型转换)，是小类型转换到大类型

2.向下类型转换(强制类型转换)，是大类型转换到小类型(有风险,可能出现数据溢出)。

3.instanceof运算符，来解决引用对象的类型，避免类型转换的安全性问题。

# 数据类型 #


## 基本类型 ##

八大基本类型

- byte/8
- char/16
- short/16
- int/32
- float/32
- long/64
- double/64
- boolean/~

## 包装类型 ##

基本类型都有对应的包装类型，基本类型与其对应的包装类型之间的赋值使用自动装箱与拆箱完成。

	Integer x = 2;     // 装箱 调用了 Integer.valueOf(2)
	int y = x;         // 拆箱 调用了 X.intValue()

自动装箱：从基本类型转换为包装类型，调用包装类型的静态方法valueOf();

自动拆箱：从包装类型转换为基本类型，调用基本类型的实例方法xxxValue(),比如int调用intValue()

## 缓存池 ##

new Integer(123) 与 Integer.valueOf(123) 的区别在于：

- new Integer(123) 每次都会新建一个对象；
- Integer.valueOf(123) 会使用缓存池中的对象，多次调用会取得同一个对象的引用。

		Integer x = new Integer(123);
		Integer y = new Integer(123);
		System.out.println(x == y);    // false
		Integer z = Integer.valueOf(123);
		Integer k = Integer.valueOf(123);
		System.out.println(z == k);   // true

valueOf() 方法的实现比较简单，就是先判断值是否在缓存池中，如果在的话就直接返回缓存池的内容。

在 Java 8 中，Integer 缓存池的大小默认为 -128~127。

编译器会在自动装箱过程调用 valueOf() 方法，因此多个值相同且值在缓存池范围内的 Integer 实例使用自动装箱来创建，那么就会引用相同的对象
	
	Integer m = 123;
	Integer n = 123;
	System.out.println(m == n); // true

基本类型对应的缓冲池如下：

- boolean values true and false
- all byte values
- short values between -128 and 127
- int values between -128 and 127
- char in the range \u0000 to \u007F

在使用这些基本类型对应的包装类型时，如果该数值范围在缓冲池范围内，就可以直接使用缓冲池中的对象。

在 jdk 1.8 所有的数值类缓冲池中，Integer 的缓冲池 IntegerCache 很特殊，这个缓冲池的下界是 - 128，上界默认是 127，但是这个上界是可调的，在启动 jvm 的时候，通过 -XX:AutoBoxCacheMax=<size> 来指定这个缓冲池的大小，该选项在 JVM 初始化的时候会设定一个名为 java.lang.IntegerCache.high 系统属性，然后 IntegerCache 初始化的时候就会读取该系统属性来决定上界。


# String #

## 概述 ##

String 被声明为 final，因此它不可被继承。(Integer 等包装类也不能被继承）

在 Java 8 中，String 内部使用 char 数组存储数据。

	public final class String
	    implements java.io.Serializable, Comparable<String>, CharSequence {
	    /** The value is used for character storage. */
	    private final char value[];
	}

## 不可变的好处 ##

### 1.可以缓存hash值

因为 String 的 hash 值经常被使用，例如 String 用做 HashMap 的 key。不可变的特性可以使得 hash 值也不可变，因此只需要进行一次计算。

### 2.String Pool 的需要

如果一个 String 对象已经被创建过了，那么就会从 String Pool 中取得引用。只有 String 是不可变的，才可能使用 String Pool。

### 3.安全性

String 经常作为参数，String 不可变性可以保证参数不可变。例如在作为网络连接参数的情况下如果 String 是可变的，那么在网络连接过程中，String 被改变，改变 String 的那一方以为现在连接的是其它主机，而实际情况却不一定是。

### 4.线程安全

String 不可变性天生具备线程安全，可以在多个线程中安全地使用。

## String, StringBuffer and StringBuilder

### 1.可变性


- String 不可变
- StringBuffer 和 StringBuilder 可变

### 2.线程安全 

- String 不可变，因此是线程安全的
- StringBuilder 不是线程安全的
- StringBuffer 是线程安全的，内部使用 synchronized 进行同步

### 3.使用场景

1、在字符串不经常发生变化的业务场景优先使用String(代码更清晰简洁)。如常量的声明，少量的字符串操作(拼接，删除等)。

2、在单线程情况下，如有大量的字符串操作情况，应该使用StringBuilder来操作字符串。不能使用String"+"来拼接而是使用，避免产生大量无用的中间对象，耗费空间且执行效率低下（新建对象、回收对象花费大量时间）。如JSON的封装等。

3、在多线程情况下，如有大量的字符串操作情况，应该使用StringBuffer。如HTTP参数解析和封装等。

## String Pool ##

字符串常量池（String Pool）保存着所有字符串字面量（literal strings），这些字面量在编译时期就确定。不仅如此，还可以使用 String 的 intern() 方法在运行过程将字符串添加到 String Pool 中。

当一个字符串调用 intern() 方法时，如果 String Pool 中已经存在一个字符串和该字符串值相等（使用 equals() 方法进行确定），那么就会返回 String Pool 中字符串的引用；否则，就会在 String Pool 中添加一个新的字符串，并返回这个新字符串的引用。

下面示例中，s1 和 s2 采用 new String() 的方式新建了两个不同字符串，而 s3 和 s4 是通过 s1.intern() 方法取得同一个字符串引用。intern() 首先把 s1 引用的字符串放到 String Pool 中，然后返回这个字符串引用。因此 s3 和 s4 引用的是同一个字符串。

	String s1 = new String("aaa");
	String s2 = new String("aaa");
	System.out.println(s1 == s2);           // false
	String s3 = s1.intern();
	String s4 = s1.intern();
	System.out.println(s3 == s4);           // true

如果是采用 "bbb" 这种字面量的形式创建字符串，会自动地将字符串放入 String Pool 中。
	
	String s5 = "bbb";
	String s6 = "bbb";
	System.out.println(s5 == s6);  // true

## new String("abc")

使用这种方式一共会创建两个字符串对象（前提是 String Pool 中还没有 "abc" 字符串对象）。

- "abc" 属于字符串字面量，因此编译时期会在 String Pool 中创建一个字符串对象，指向这个 "abc" 字符串字面量；
- 而使用 new 的方式会在堆中创建一个字符串对象。

## 方法

- charAt(int index): 获取指定位置的字符 
- toCharArray(): 获取对应的字符数组
- substring(int a, int b): 截取子字符串 
- split(" "): 根据分隔符进行分割
- trim(): 去掉首尾空格 
- toLowerCase(): 全部变成小写 
- toUpperCase(): 全部变成大写 
- indexOf(String s): 判断字符或者子字符串出现的位置
- contains(String s): 是否包含子字符串 
- replaceAll(String a, String b): 替换所有的 
- replaceFirst(String a, String b): 只替换第一个 

# 关键字 #

## final

1. final 修饰类，则该类不允许被继承。
2. final 修饰方法，则该方法不允许被重写。
3. final 修饰属性，则该类的该属性不会进行隐式的初始化，所以 该final 属性的初始化属性必须有值，或在**构造方法中赋值(但只能选其一，且必须选其一，因为没有默认值！)，**且初始化之后就不能改了，只能赋值一次。
4. final 修饰变量，则该变量的值只能赋一次值，在声明变量的时候才能赋值，即变为常量。

## static

### 1.静态变量

静态变量：又称为类变量，也就是说这个变量属于类的，类所有的实例都共享静态变量，可以直接通过类名来访问它。静态变量在内存中只存在一份。

实例变量：每创建一个实例就会产生一个实例变量，它与该实例同生共死

	private int x;         // 实例变量
    private static int y;  // 静态变量

### 2.静态方法

静态方法在类加载的时候就存在了，它不依赖于任何实例。所以静态方法必须有实现，也就是说它不能是抽象方法。

只能访问所属类的静态字段和静态方法，方法中不能有 this 和 super 关键字，因此这两个关键字与具体对象关联。

### 3.静态语句块

静态语句块在类初始化时运行一次。

	public class A {
	    static {
	        System.out.println("123");
	    }
	
	    public static void main(String[] args) {
	        A a1 = new A();
	        A a2 = new A();
	    }
	}

### 4.静态内部类

非静态内部类依赖于外部类的实例，也就是说需要先创建外部类实例，才能用这个实例去创建非静态内部类。而静态内部类不需要。

静态内部类不能访问外部类的非静态的变量和方法。

### 5.初始化顺序

静态变量和静态语句块优先于实例变量和普通语句块，静态变量和静态语句块的初始化顺序取决于它们在代码中的顺序。

存在继承的情况下，初始化顺序为：

- 父类（静态变量、静态语句块）
- 子类（静态变量、静态语句块）
- 父类（实例变量、普通语句块）
- 父类（构造函数）
- 子类（实例变量、普通语句块）
- 子类（构造函数）

# 内部类 #

## 什么是内部类？ ##

内部类是指在一个外部类的内部再定义一个类。内部类作为外部类的一个成员，并且依附于外部类而存在的。内部类可为静态，可用protected和private修饰（而外部类只能使用public和缺省的包访问权限）。内部类主要有以下几类：成员内部类、局部内部类、静态内部类、匿名内部类。

## 内部类的作用 ##

1. 内部类提供了更好的封装，可以把内部类隐藏在外部类之内，不允许同一个包中的其他类访问该类。

2. 内部类的方法可以直接访问外部类的所有数据，包括私有的数据。

3. 内部类所实现的功能使用外部类同样可以实现，只是有时使用内部类更方便。

4. 如果没有内部类提供的可以继承多个具体的或抽象的类的能力，一些设计与编程问题就很难解决。 从这个角度看，内部类使得多重继承的解决方案变得完整。接口解决了部分问题，而内部类有效地实现了"多重继承"。

## 内部类的好处 ##

### 静态内部类的作用： ###

1 只是为了降低包的深度，方便类的使用，静态内部类适用于包含类当中，但又不依赖与外在的类。

2 由于Java规定静态内部类不能用使用外在类的非静态属性和方法，所以只是为了方便管理类结构而定义。于是我们在创建静态内部类的时候，不需要外部类对象的引用。

### 非静态内部类的作用： ###

1 内部类继承自某个类或实现某个接口，内部类的代码操作创建其他外围类的对象。所以你可以认为内部类提供了某种进入其外围类的窗口。

2 使用内部类最吸引人的原因是:每个内部类都能独立地继承自一个(接口的)实现，所以无论外围类是否已经继承了某个(接口的)实现，对于内部类都没有影响

3 如果没有内部类提供的可以继承多个具体的或抽象的类的能力，一些设计与编程问题就很难解决。 从这个角度看，内部类使得多重继承的解决方案变得完整。接口解决了部分问题，而内部类有效地实现了"多重继承"。

## 内部类的共性 ##

1. 内部类仍然是一个独立的类，在编译之后内部类会被编译成独立的.class文件，但是前面冠以外部类的类名和$符号 。

1. 内部类不能用普通的方式访问。

1. 内部类声明成静态的，就不能随便的访问外部类的成员变量了，此时内部类只能访问外部类的静态成员变量 。

1. 外部类不能直接访问内部类的的成员，但可以通过内部类对象来访问

1. 内部类是外部类的一个成员，因此内部类可以自由地访问外部类的成员变量，无论是否是private的。因为当某个外围类的对象创建内部类的对象时，此内部类会捕获一个隐式引用，它引用了实例化该内部对象的外围类对象。通过这个指针，可以访问外围类对象的全部状态。

## 静态内部类与普通内部类有什么区别呢？ ##

1. 静态内部类不持有外部类的引用 在普通内部类中，我们可以直接访问外部类的属性、方法，即使是private类型也可以访问，这是因为内部类持有一个外部类的引用，可以自由访问。而静态内部类，则只可以访问外部类的静态方法和静态属性（如果是private权限也能访问，这是由其代码位置所决定的），其他则不能访问。

1. 静态内部类不依赖外部类 普通内部类与外部类之间是相互依赖的关系，内部类实例不能脱离外部类实例，也就是说它们会同生同死，一起声明，一起被垃圾回收器回收。而静态内部类是可以独立存在的，即使外部类消亡了，静态内部类还是可以存在的。

1. 普通内部类不能声明static的方法和变量 普通内部类不能声明static的方法和变量，注意这里说的是变量，常量（也就是final static修饰的属性）还是可以的，而静态内部类形似外部类，没有任何限制。

## 为什么普通内部类不能有静态变量呢？ ##

1 成员内部类 之所以叫做成员 就是说他是类实例的一部分 而不是类的一部分

2 结构上来说 他和你声明的成员变量是一样的地位 一个特殊的成员变量 而静态的变量是类的一部分和实例无关

3 你若声明一个成员内部类 让他成为主类的实例一部分 然后又想在内部类声明和实例无关的静态的东西 你让JVM情何以堪啊

4 若想在内部类内声明静态字段 就必须将其内部类本身声明为静态

## 例子 ##

下面的例子大概地介绍了

1 非静态内部类和静态内部类的区别。

2 不同访问权限的内部类的使用。

3 外部类和它的内部类之间的关系

	//本节讨论内部类以及不同访问权限的控制
	//内部类只有在使用时才会被加载。
	//外部类B
	public class B{
	    int i = 1;
	    int j = 1;
	    static int s = 1;
	    static int ss = 1;
	    A a;
	    AA aa;
	    AAA aaa;
	    //内部类A
	
	    public class A {
	//        static void go () {
	//
	//        }
	//        static {
	//
	//        }
	//      static int b = 1;//非静态内部类不能有静态成员变量和静态代码块和静态方法，
	        // 因为内部类在外部类加载时并不会被加载和初始化。
	        //所以不会进行静态代码的调用
	        int i = 2;//外部类无法读取内部类的成员，而内部类可以直接访问外部类成员
	
	        public void test() {
	            System.out.println(j);
	            j = 2;
	            System.out.println(j);
	            System.out.println(s);//可以访问类的静态成员变量
	        }
	        public void test2() {
	            AA aa = new AA();
	            AAA aaa = new AAA();
	        }
	
	    }
	    //静态内部类S，可以被外部访问
	    public static class S {
	        int i = 1;//访问不到非静态变量。
	        static int s = 0;//可以有静态变量
	
	        public static void main(String[] args) {
	            System.out.println(s);
	        }
	        @Test
	        public void test () {
	//            System.out.println(j);//报错，静态内部类不能读取外部类的非静态变量
	            System.out.println(s);
	            System.out.println(ss);
	            s = 2;
	            ss = 2;
	            System.out.println(s);
	            System.out.println(ss);
	        }
	    }
	
	    //内部类AA，其实这里加protected相当于default
	    //因为外部类要调用内部类只能通过B。并且无法直接继承AA，所以必须在同包
	    //的类中才能调用到(这里不考虑静态内部类)，那么就和default一样了。
	    protected class AA{
	        int i = 2;//内部类之间不共享变量
	        public void test (){
	            A a = new A();
	            AAA aaa = new AAA();
	            //内部类之间可以互相访问。
	        }
	    }
	    //包外部依然无法访问，因为包没有继承关系，所以找不到这个类
	    protected static class SS{
	        int i = 2;//内部类之间不共享变量
	        public void test (){
	
	            //内部类之间可以互相访问。
	        }
	    }
	    //私有内部类A，对外不可见，但对内部类和父类可见
	    private class AAA {
	        int i = 2;//内部类之间不共享变量
	
	        public void test() {
	            A a = new A();
	            AA aa = new AA();
	            //内部类之间可以互相访问。
	        }
	    }
	    @Test
	    public void test(){
	        A a = new A();
	        a.test();
	        //内部类可以修改外部类的成员变量
	        //打印出 1 2
	        B b = new B();
	
	    }
	}

	//另一个外部类 
	class C { 
		@Test public void test() { 
			//首先，其他类内部类只能通过外部类来获取其实例。 
			B.S s = new B.S(); //静态内部类可以直接通过B类直接获取，不需要B的实例，和静态成员变量类似。 
			//B.A a = new B.A(); //当A不是静态类时这行代码会报错。 
			//需要使用B的实例来获取A的实例 
			B b = new B(); 
			B.A a = b.new A(); 
			B.AA aa = b.new AA();
			//B和C同包，所以可以访问到AA 
			// B.AAA aaa = b.new AAA();AAA为私有内部类，外部类不可见 
			//当A使用private修饰时，使用B的实例也无法获取A的实例，这一点和私有变量是一样的。
			//所有普通的内部类与类中的一个变量是类似的。静态内部类则与静态成员类似。 
		} 
	}
​
## 局部内部类 ##

在方法中定义的内部类称为局部内部类。与局部变量类似，局部内部类不能有访问说明符，因为它不是外围类的一部分，但是它可以访问当前代码块内的常量，和此外围类所有的成员。

需要注意的是： 局部内部类只能在定义该内部类的方法内实例化，不可以在此方法外对其实例化。

## 匿名内部类 ##

定义时直接被实例化，不需要显式指定他的名字。

如果满足下面的一些条件，使用匿名内部类是比较合适的： 只用到类的一个实例。 类在定义后马上用到。 类非常小（SUN推荐是在4行代码以下） 给类命名并不会导致你的代码更容易被理解。

在使用匿名内部类时，要记住以下几个原则：

1 　匿名内部类不能有构造方法。

2 　匿名内部类不能定义任何静态成员、方法和类。

3 　匿名内部类不能是public,protected,private,static。

4 　只能创建匿名内部类的一个实例。

5 一个匿名内部类一定是在new的后面，用其隐含实现一个接口或实现一个类。

6 　因匿名内部类为局部内部类，所以局部内部类的所有限制都对其生效。

# Class类和Object类

## Class类

Java程序在运行时，Java运行时系统一直对所有的对象进行所谓的运行时类型标识，即所谓的RTTI。

这项信息纪录了每个对象所属的类。虚拟机通常使用运行时类型信息选准正确方法去执行，用来保存这些类型信息的类是Class类。Class类封装一个对象和接口运行时的状态，当装载类时，Class类型的对象自动创建。

Class类的对象内容是你创建的类的类型信息，比如你创建一个shapes类，那么，Java会生成一个内容是shapes的Class类的对象

Class类的对象不能像普通类一样，以 new shapes() 的方式创建，它的对象只能由JVM创建，因为这个类没有public构造函数

ClClass类的作用是运行时提供或获得某个对象的类型信息，和C++中的typeid()函数类似。这些信息也可用于反射。ass类的作用是运行时提供或获得某个对象的类型信息，和C++中的typeid()函数类似。这些信息也可用于反射。

### Class类原理

Class类中封装了类型的各种信息。在jvm中就是通过Class类的实例来获取每个Java类的所有信息的。

所有的java类都是继承了object这个类，在object这个类中有一个方法：getclass().这个方法是用来取得该类已经被实例化了的对象的该类的引用，这个引用指向的是Class类的对象。

所有的java类都是继承了object这个类，在object这个类中有一个方法：getclass().这个方法是用来取得该类已经被实例化了的对象的该类的引用，这个引用指向的是Class类的对象。

我们自己无法生成一个Class对象（构造函数为private)，而 这个Class类的对象是在当各类被调入时，由 Java 虚拟机自动创建 Class 对象，或通过类装载器中的 defineClass 方法生成。

### 如何获得一个Class类对象

请注意，以下这些方法都是值、指某个类对应的Class对象已经在堆中生成以后，我们通过不同方式获取对这个Class对象的引用。而上面说的DefineClass才是真正将字节码加载到虚拟机的方法，会在堆中生成新的一个Class对象。

1.Class类的forName函数

2.使用对象的getClass()函数

3.使用类字面常量 Class obj=String.class

### 使用Class类的对象来生成目标类的实例

获取一个Class类的对象后，可以用 newInstance() 函数来生成目标类的一个实例

## Object类

Object类是Java中其他所有类的祖先，没有Object类Java面向对象无从谈起。作为其他所有类的基类，Object具有哪些属性和行为，是Java语言设计背后的思维体现。

Object类位于java.lang包中，java.lang包包含着Java最基础和核心的类，在编译时会自动导入。Object类没有定义属性，一共有13个方法，13个方法之中并不是所有方法都是子类可访问的，一共有9个方法是所有子类都继承了的。

	1．clone方法
	保护方法，实现对象的浅复制，只有实现了Cloneable接口才可以调用该方法，否则抛出CloneNotSupportedException异常。
	2．getClass方法
	final方法，获得运行时类型。
	3．toString方法
	该方法用得比较多，一般子类都有覆盖。
	4．finalize方法
	该方法用于释放资源。因为无法确定该方法什么时候被调用，很少使用。
	5．equals方法
	该方法是非常重要的一个方法。一般equals和==是不一样的，但是在Object中两者是一样的。子类一般都要重写这个方法。
	6．hashCode方法
	该方法用于哈希查找，重写了equals方法一般都要重写hashCode方法。这个方法在一些具有哈希功能的Collection中用到。
	一般必须满足obj1.equals(obj2)==true。可以推出obj1.hash- Code()==obj2.hashCode()，但是hashCode相等不一定就满足equals。不过为了提高效率，应该尽量使上面两个条件接近等价。
	7．wait方法
	wait方法就是使当前线程等待该对象的锁，当前线程必须是该对象的拥有者，也就是具有该对象的锁。wait()方法一直等待，直到获得锁或者被中断。wait(long timeout)设定一个超时间隔，如果在规定时间内没有获得锁就返回。
	调用该方法后当前线程进入睡眠状态，直到以下事件发生。
	（1）其他线程调用了该对象的notify方法。
	（2）其他线程调用了该对象的notifyAll方法。
	（3）其他线程调用了interrupt中断该线程。
	（4）时间间隔到了。
	此时该线程就可以被调度了，如果是被中断的话就抛出一个InterruptedException异常。
	8．notify方法
	该方法唤醒在该对象上等待的某个线程。
	9．notifyAll方法
	该方法唤醒在该对象上等待的所有线程。

### clone()方法实现浅拷贝 ###

clone方法实现的是浅拷贝，只拷贝当前对象，并且在堆中分配新的空间，放这个复制的对象。但是对象如果里面有其他类的子对象，那么就不会拷贝到新的对象中。

深拷贝和浅拷贝的区别：

- 浅拷贝 浅拷贝是按位拷贝对象，它会创建一个新对象，这个对象有着原始对象属性值的一份精确拷贝。如果属性是基本类型，拷贝的就是基本类型的值；如果属性是内存地址（引用类型），拷贝的就是内存地址 ，因此如果其中一个对象改变了这个地址，就会影响到另一个对象。

- 深拷贝 深拷贝会拷贝所有的属性,并拷贝属性指向的动态分配的内存。当对象和它所引用的对象一起拷贝时即发生深拷贝。深拷贝相比于浅拷贝速度较慢并且花销较大。 现在为了要在clone对象时进行深拷贝， 那么就要Clonable接口，覆盖并实现clone方法，除了调用父类中的clone方法得到新的对象， 还要将该类中的引用变量也clone出来。如果只是用Object中默认的clone方法，是浅拷贝的。

一个对象在浅拷贝以后，只是把对象复制了一份放在堆空间的另一个地方，但是成员变量如果有引用指向其他对象，这个引用指向的对象和被拷贝的对象中引用指向的对象是一样的。当然，基本数据类型还是会重新拷贝一份的。

### equals()

- 对于基本类型，== 判断两个值是否相等，基本类型没有 equals() 方法。
- 对于引用类型，== 判断两个变量是否引用同一个对象，而 equals() 判断引用的对象是否等价。

		Integer x = new Integer(1);
		Integer y = new Integer(1);
		System.out.println(x.equals(y)); // true
		System.out.println(x == y);      // false

### hashCode() ###

hashCode() 返回哈希值，而 equals() 是用来判断两个对象是否等价。等价的两个对象散列值一定相同，但是散列值相同的两个对象不一定等价，这是因为计算哈希值具有随机性，两个值不同的对象可能计算出相同的哈希值。

在重写 equals() 方法时应当总是重写 hashCode() 方法，保证等价的两个对象哈希值也相等。

HashSet 和 HashMap 等集合类使用了 hashCode() 方法来计算对象应该存储的位置，因此要将对象添加到这些集合类中，需要让对应的类实现 hashCode() 方法。

理想的哈希函数应当具有均匀性，即不相等的对象应当均匀分布到所有可能的哈希值上。这就要求了哈希函数要把所有域的值都考虑进来。可以将每个域都当成 R 进制的某一位，然后组成一个 R 进制的整数。

R 一般取 31，因为它是一个奇素数，如果是偶数的话，当出现乘法溢出，信息就会丢失，因为与 2 相乘相当于向左移一位，最左边的位丢失。并且一个数与 31 相乘可以转换成移位和减法：31*x == (x<<5)-x，编译器会自动进行这个优化。

	@Override
	public int hashCode() {
	    int result = 17;
	    result = 31 * result + x;
	    result = 31 * result + y;
	    result = 31 * result + z;
	    return result;
	}





# 反射

## 什么是反射

反射是在运行状态中，对于任意一个类，都能够知道这个类的所有属性和方法；对于任意一个对象，都能够调用它的任意一个方法和属性；这种动态获取的信息以及动态调用对象的方法的功能称为 Java 语言的反射机制。

## 反射的作用

1.在运行时判断任意一个对象所属的类；

2.在运行时构造任意一个类的对象；

3.在运行时判断任意一个类所具有的成员变量和方法（通过反射甚至可以调用private方法）；

4.在运行时调用任意一个对象的方法

## 哪里用到反射机制

JDBC中，利用反射动态加载了数据库驱动程序。

Web服务器中利用反射调用了Sevlet的服务方法。

IDEA等开发工具利用反射动态刨析对象的类型与结构，动态提示对象的属性和方法。

很多框架都用到反射机制，注入属性，调用方法，如Spring。

## 反射机制的优缺点

优点：可以动态执行，在运行期间根据业务功能动态执行方法、访问属性，最大限度发挥了java的灵活性。

缺点：

- 对性能有影响，这类操作总是慢于直接执行java代码。
- 安全限制，使用反射技术要求程序必须在一个没有安全限制的环境中运行。
- 内部暴露

## 使用反射

Class 和 java.lang.reflect 一起对反射提供了支持，java.lang.reflect 类库主要包含了以下三个类：

- Field ：可以使用 get() 和 set() 方法读取和修改 Field 对象关联的字段；
- Method ：可以使用 invoke() 方法调用与 Method 对象关联的方法；
- Constructor ：可以用 Constructor 的 newInstance() 创建新的对象。

1.通过一个全限类名创建一个对象（Class类）

Class.forName(“全限类名”); 类名.class; 对象.getClass();

2.获取构造器对象，通过构造器new出一个对象（Constructor类）

Clazz.getConstructor([String.class]); Con.newInstance([参数]); 通过class对象创建一个实例对象（就相当与new类名（）无参构造器) Cls.newInstance();

3.通过class对象获得一个属性对象（Field类）

Field c=cls.getFields()：获得某个类的所有的公共（public）的字段，包括父类中的字段。 

Field c=cls.getDeclaredFields()：获得某个类的所有声明的字段，即包括public、private和proteced，但是不包括父类的声明字段

4.通过class对象获得一个方法对象（Method类）

# 回调 #

假设有两个类，分别是A和B，在A中有一个方法a()，B中有一个方法b()；在A里面调用B中的方法b()，而方法b()中调用了方法a()，这样子就同时实现了b()和a()两个方法的功能。

回调更像是一个约定，就是如果我调用了b()方法，那么就必须要回调，而不需要显示调用。

# 异常

Throwable 可以用来表示任何可以作为异常抛出的类，分为两种： Error 和 Exception。其中 Error 用来表示 JVM 无法处理的错误，Exception 分为两种：

- 受检异常 ：需要用 try...catch... 语句捕获并进行处理，并且可以从异常中恢复；
- 非受检异常 ：是程序运行时错误，例如除 0 会引发 Arithmetic Exception，此时程序崩溃并且无法恢复。

![](https://cs-notes-1256109796.cos.ap-guangzhou.myqcloud.com/PPjwP.png)

# 泛型

## 什么是泛型 ##

泛型，即“参数化类型”。就是将类型由原来的具体的类型参数化，类似于方法中的变量参数，此时类型也定义成参数形式（可以称之为类型形参），然后在使用/调用时传入具体的类型（类型实参）。

## 泛型的好处

它提供了编译期的类型安全，确保你只能把正确类型的对象放入 集合中，避免了在运行时出现ClassCastException。

## 特性 ##

Java中的泛型，只在编译阶段有效。在编译过程中，正确检验泛型结果后，会将泛型的相关信息擦出，并且在对象进入和离开方法的边界处添加类型检查和类型转换的方法。也就是说，泛型信息不会进入到运行时阶段。

泛型类型在逻辑上看以看成是多个不同的类型，实际上都是相同的基本类型。

## 泛型的使用方式

泛型有三种使用方式，分别为：泛型类、泛型接口、泛型方法

## 泛型上下边界 ##

在使用泛型的时候，我们还可以为传入的泛型类型实参进行上下边界的限制，如：类型实参只准传入某种类型的父类或某种类型的子类。

为泛型添加上边界，即传入的类型实参必须是指定类型的子类型

### 泛型通配符

类型通配符一般是使用？代替具体的类型实参，注意， 此处的？和Number、String、Integer一样都是一种实际的类型，可以把？看成所有类型的父类。是一种真实的类型。

可以解决当具体类型不确定的时候，这个通配符就是 ? ；当操作类型时，不需要使用类型的具体功能时，只使用Object类中的功能。那么可以用 ? 通配符来表未知类型


### 限定通配符

List<? extends T>可以接受任何继承自T的类型的List，而List<? super T>可以接受任何T的父类构成的List。

	public class 泛型通配符与边界 {
	    public void showKeyValue(Generic<Number> obj){
	        System.out.println("key value is " + obj.getKey());
	    }
	    @Test
	    public void main() {
	        Generic<Integer> gInteger = new Generic<Integer>(123);
	        Generic<Number> gNumber = new Generic<Number>(456);
	        showKeyValue(gNumber);
	        //泛型中的子类也无法作为父类引用传入
	//        showKeyValue(gInteger);
	    }
	    //直接使用？通配符可以接受任何类型作为泛型传入
	    public void showKeyValueYeah(Generic<?> obj) {
	        System.out.println(obj);
	    }
	    //只能传入number的子类或者number
	    public void showKeyValue1(Generic<? extends Number> obj){
	        System.out.println(obj);
	    }
	
	    //只能传入Integer的父类或者Integer
	    public void showKeyValue2(Generic<? super Integer> obj){
	        System.out.println(obj);
	    }
	
	    @Test
	    public void testup () {
	        //这一行代码编译器会提示错误，因为String类型并不是Number类型的子类
	        //showKeyValue1(generic1);
	        Generic<String> generic1 = new Generic<String>("11111");
	        Generic<Integer> generic2 = new Generic<Integer>(2222);
	        Generic<Float> generic3 = new Generic<Float>(2.4f);
	        Generic<Double> generic4 = new Generic<Double>(2.56);
	
	        showKeyValue1(generic2);
	        showKeyValue1(generic3);
	        showKeyValue1(generic4);
	    }
	
	    @Test
	    public void testdown () {
	
	        Generic<String> generic1 = new Generic<String>("11111");
	        Generic<Integer> generic2 = new Generic<Integer>(2222);
	        Generic<Number> generic3 = new Generic<Number>(2);
	//        showKeyValue2(generic1);本行报错，因为String并不是Integer的父类
	        showKeyValue2(generic2);
	        showKeyValue2(generic3);
	    }
	}

# 注解

## 什么是注解

Java 注解是附加在代码中的一些元信息，用于一些工具在编译、运行时进行解析和使用，起到说明、配置的功能。注解不会也不能影响代码的实际逻辑，仅仅起到辅助性的作用。


## 注解的用处

1、生成文档。这是最常见的，也是java 最早提供的注解。常用的有@param @return 等 

2、跟踪代码依赖性，实现替代配置文件功能。依赖注入，未来java开发，将大量注解配置，具有很大用处; 

3、在编译时进行格式检查。如@override 放在方法前，如果你这个方法并不是覆盖了超类方法，则编译时就能检查出。

## 注解的原理

注解本质是一个继承了Annotation的特殊接口，其具体实现类是Java运行时生成的动态代理类。而我们通过反射获取注解时，返回的是Java运行时生成的动态代理对象$Proxy1。通过代理对象调用自定义注解（接口）的方法，会最终调用AnnotationInvocationHandler的invoke方法。该方法会从memberValues这个Map中索引出对应的值。而memberValues的来源是Java常量池。

## 元注解

java.lang.annotation提供了四种元注解，专门注解其他的注解（在自定义注解的时候，需要使用到元注解）： 

@Documented –注解是否将包含在JavaDoc中 

@Retention –什么时候使用该注解 

@Target –注解用于什么地方 

@Inherited – 是否允许子类继承该注解

# 序列化和反序列化 #

序列化 (Serialization)是将对象的状态信息转换为可以存储或传输的形式的过程。一般将一个对象存储至一个储存媒介，例如档案或是记亿体缓冲等。在网络传输过程中，可以是字节或是XML等格式。而字节的或XML编码格式可以还原完全相等的对象。这个相反的过程又称为反序列化。

## Java对象的序列化与反序列化 ##

在Java中，我们可以通过多种方式来创建对象，并且只要对象没有被回收我们都可以复用该对象。但是，我们创建出来的这些Java对象都是存在于JVM的堆内存中的。

只有JVM处于运行状态的时候，这些对象才可能存在。一旦JVM停止运行，这些对象的状态也就随之而丢失了。

但是在真实的应用场景中，我们需要将这些对象持久化下来，并且能够在需要的时候把对象重新读取出来。Java的对象序列化可以帮助我们实现该功能。

> 对象序列化机制（object serialization）是Java语言内建的一种对象持久化方式，通过对象序列化，可以把对象的状态保存为字节数组，并且可以在有需要的时候将这个字节数组通过反序列化的方式再转换成对象。

对象序列化可以很容易的在JVM中的活动对象和字节数组（流）之间进行转换。

在Java中，对象的序列化与反序列化被广泛应用到RMI(远程方法调用)及网络传输中。

## 总结 ##

1、如果一个类想被序列化，需要实现Serializable接口。否则将抛出NotSerializableException异常，这是因为，在序列化操作过程中会对类型进行检查，要求被序列化的类必须属于Enum、Array和Serializable类型其中的任何一种。

2、通过ObjectOutputStream和ObjectInputStream对对象进行序列化及反序列化

3、虚拟机是否允许反序列化，不仅取决于类路径和功能代码是否一致，一个非常重要的一点是两个类的序列化 ID 是否一致（就是 private static final long serialVersionUID）

序列化 ID 在 Eclipse 下提供了两种生成策略，一个是固定的 1L，一个是随机生成一个不重复的 long 类型数据（实际上是使用 JDK 工具生成），在这里有一个建议，如果没有特殊需求，就是用默认的 1L 就可以，这样可以确保代码一致时反序列化成功。那么随机生成的序列化 ID 有什么作用呢，有些时候，通过改变序列化 ID 可以用来限制某些用户的使用。

4、序列化并不保存静态变量。

5、要想将父类对象也序列化，就需要让父类也实现Serializable 接口。

6、Transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

7、服务器端给客户端发送序列化对象数据，对象中有一些数据是敏感的，比如密码字符串等，希望对该密码字段在序列化时，进行加密，而客户端如果拥有解密的密钥，只有在客户端进行反序列化时，才可以对密码进行读取，这样可以一定程度保证序列化对象的数据安全。

8、在类中增加writeObject 和 readObject 方法可以实现自定义序列化策略，如ArrayList。

# JAVA8新特性

## Lambda表达式

Lambda允许把函数作为一个方法的参数（函数作为参数传递进方法中）

之前，Java程序员不得不使用毫无新意的匿名类来代替lambda。

在最简单的形式中，一个lambda可以由用逗号分隔的参数列表、–>符号与函数体三部分表示。例如：

	Arrays.asList( "a", "b", "d" ).forEach( e -> System.out.println( e ) );

Lambda和匿名内部类的区别：

匿名内部类仍然是一个类，只是不需要我们显式指定类名，编译器会自动为该类取名。比如有如下形式的代码：
	
	public class LambdaTest {
	    public static void main(String[] args) {
	        new Thread(new Runnable() {
	            @Override
	            public void run() {
	                System.out.println("Hello World");
	            }
	        }).start();
	    }
	}

## 接口的默认方法

JAVA8之前接口就是一个完全的抽象类。

Java 8 使我们能够使用default 关键字给接口增加非抽象的方法实现。这个特性也被叫做 扩展方法（Extension Methods）。

默认方法出现的原因是为了对原有接口的扩展，有了默认方法之后就不怕因改动原有的接口而对已经使用这些接口的程序造成的代码不兼容的影响。 

## 重复注解

Java8之前使用注解的一个限制是相同的注解在同一位置只能声明一次，不能声明多次。

Java 8打破了这条规则，引入了重复注解机制，这样相同的注解可以在同一地方声明多次。

## Optional

为了解决空指针异常

## Stream

Stream API极大简化了集合框架的处理

## Time API

新的java.time包涵盖了所有处理日期，时间，日期/时间，时区，时刻（instants），过程（during）与时钟（clock）的操作。

## 并行（parallel）数组

Java 8增加了大量的新方法来对数组进行并行处理。

可以说，最重要的是parallelSort()方法，因为它可以在多核机器上极大提高数组排序的速度。

# JRE or JDK #

- JRE：Java Runtime Environment，Java 运行环境的简称，为 Java 的运行提供了所需的环境。它是一个 JVM 程序，主要包括了 JVM 的标准实现和一些 Java 基本类库。
- JDK：Java Development Kit，Java 开发工具包，提供了 Java 的开发及运行环境。JDK 是 Java 开发的核心，集成了 JRE 以及一些其它的工具，比如编译 Java 源码的编译器 javac 等。