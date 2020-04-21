# JVM OOM 内存溢出测试

## Test JVM Heap OOM 堆内存溢出

设置堆大小为20M，并且不可扩展，新建三个10M的对象，发生堆溢出

### 代码及JVM参数

```java
public class HeapOOM {
private static final int _1MB = 1024 * 1024;
  /*
    -verbose:gc -Xms20M -Xmx20M
    
    */
public static void HeapOOM()
{
    byte[] allocation1,allocation2,allocation3,allocation4;

    allocation1 = new byte[10 * _1MB];
    allocation2 = new byte[10 * _1MB];
    allocation3 = new byte[10 * _1MB];
    // allocation2 = new byte[4 * _1MB];
    
    System.out.println("succseful");
}

public static void main(String[] args) {

    System.out.println("Test Start");
    HeapOOM();
    
}
}
```

### 运行结果

Test Start

852

Exception in thread "main" java.lang.StackOverflowError

​	at StackOOM.StackOOM(StackOOM.java:6)

​	at StackOOM.StackOOM(StackOOM.java:7)

​	at StackOOM.StackOOM(StackOOM.java:7)

​	at StackOOM.StackOOM(StackOOM.java:7)

​	at StackOOM.StackOOM(StackOOM.java:7)



## Test JVM Stack OOM 栈内存溢出1



栈中存放的是

* 本地变量表：基本数据类型和引用
* 操作数栈
* 动态连接和方法出口

当栈帧数量超过栈深度（栈内存大小）的时候就会StackOverflowError

允许动态扩展的情况下，用光了内存则会OOME

实验通过在一个方法中不断的调用自己，导致操作栈数量越来越大，并且方法没有出口直至栈溢出

### 代码及JVM参数

```java
public class StackOOM {
    private static int length = 0;
/*
    -Xss160K
    
    */
	public static void StackOOM()
	{
  		length++;
  		StackOOM();
	}

	public static void main(String[] args) {

  	System.out.println("Test Start");

  	try{
  		StackOOM();
  	}
  	catch (Throwable e)
  	{
  		System.out.println(length);
  	throw e;
  	}
	}
}
```

### 运行结果

Test Start

852

Exception in thread "main" java.lang.StackOverflowError

​	at StackOOM.StackOOM(StackOOM.java:6)

​	at StackOOM.StackOOM(StackOOM.java:7)

​	at StackOOM.StackOOM(StackOOM.java:7)

​	at StackOOM.StackOOM(StackOOM.java:7)

​	at StackOOM.StackOOM(StackOOM.java:7)

## Test JVM Stack OOM 栈内存溢出2

实验通过不断的新建本地变量（long类型），让本地变量表不断的增多，导致内存溢出

### 代码及JVM参数

```java
public class StackOOM2 {
    private static int length = 0;

	public static void StackOOM2()
	{
  		long l1,l2,l3,l4,l5,l6,l7,l8,l9,l10,l11,l12,l13,l14,l15,l16,l17l,l18,l19,l20,l21,l22,l23,l24,l25,l26,l27,l28,l29,l30;
  		length++;
  		StackOOM2();
	}

	public static void main(String[] args) {

  	System.out.println("Test Start");

  	try{
  		StackOOM2();
  	}
  	catch (Throwable e)
  	{
  		System.out.println(length);
  	throw e;
  	}
	}
}
```
### 运行结果

_每次新建30个long:_

Test Start

1701

Exception in thread "main" java.lang.StackOverflowError

​	at StackOOM2.StackOOM2(StackOOM2.java:11)

​	at StackOOM2.StackOOM2(StackOOM2.java:12)

​	at StackOOM2.StackOOM2(StackOOM2.java:12)

​	at StackOOM2.StackOOM2(StackOOM2.java:12)

_每次新建10个long:_

3970

Exception in thread "main" java.lang.StackOverflowError

​	at StackOOM2.StackOOM2(StackOOM2.java:7)

​	at StackOOM2.StackOOM2(StackOOM2.java:8)

​	at StackOOM2.StackOOM2(StackOOM2.java:8)

​	at StackOOM2.StackOOM2(StackOOM2.java:8)

​	at StackOOM2.StackOOM2(StackOOM2.java:8)

​	at StackOOM2.StackOOM2(StackOOM2.java:8)

## Test JVM Heap OOM 堆内存溢出3

上面说到的都是StackOverflowError，在栈内存没办法分配的情况下抛出的异常

还有一种是栈内存无法再扩展，即为无法申请到更多内存了的情况下抛出OOME

实验通过不断创建线程，导致OOME，必须在32位系统才可以

## Test JVM MethodArea OOM 方法区内存溢出

JDK7之前，字符串常量池在方法区，可以让字符串常量池不断增加导致方法区溢出，

JDK7之后，字符串常量池移到了java堆中，导致这种方法只能导致堆OOM

运行时常量池还是在方法区的！

所以想要实验就必须使用动态生成动态类的方法使方法区溢出。

JDK8以后，方法区或者说是永生代被元空间替代，元空间直接就是在内存里，所以只受硬件大小影响

但是为了防止内存溢出，JVM提供了设置最大值的参数：-XX：MaxMetaspaceSize

## Test JVM DirectMemory OOM 直接内存溢出

JVM native方法使用直接内存，可以使用-XX：MaxDirectMemorySize 参数指定大小

默认大小和堆最大值一样。

通过不断调用native方法，比如反射，可以导致直接内存溢出