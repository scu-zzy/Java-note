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