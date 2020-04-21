# JVM GC 垃圾回收测试

## Test JVM MinorGC 新生代清理测试

设置新生代大小30M，SurvivorRatio=1 使得 Eden、From Survivor和To Survivor大小都是10M

首先分配8M内存到Eden，然后让数组2变成可回收的状态，再新建一个4M的数组，

这时候触发MinorGC，Eden中一个数组被回收，一个被放到Surivor。完成一次垃圾回收

### 代码及JVM参数

```java
public class MinorGC {
    private static final int _1MB = 1024 * 1024;
/*

     -Xms60M -Xmx60M -Xmn30M -XX:SurvivorRatio=1 -XX:+UseSerialGC -XX:+PrintGCDetails
*/

    public static void MinorGC()
    {
        byte[] allocation1,allocation2,allocation3,allocation4;

        allocation1 = new byte[4 * _1MB];
        allocation2 = new byte[4 * _1MB];
    
        
        allocation2 = null;
        allocation3 = new byte[4 * _1MB];
    }

    public static void main(String[] args) {

        System.out.println("Test MinorGC");
        MinorGC();
    }

}

```

### 运行结果

Test MinorGC

[GC[DefNew: 8806K->4336K(20480K), 0.0035170 secs] 8806K->4336K(51200K), 0.0035430 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

Heap

 def new generation  total 20480K, used 8639K [0x00000007f7200000, 0x00000007f9000000, 0x00000007f9000000)

 eden space 10240K, 42% used [0x00000007f7200000, 0x00000007f7633d90, 0x00000007f7c00000)

 from space 10240K, 42% used [0x00000007f8600000, 0x00000007f8a3c0f0, 0x00000007f9000000)

 to  space 10240K,  0% used [0x00000007f7c00000, 0x00000007f7c00000, 0x00000007f8600000)

 tenured generation  total 30720K, used 0K [0x00000007f9000000, 0x00000007fae00000, 0x00000007fae00000)

  the space 30720K,  0% used [0x00000007f9000000, 0x00000007f9000000, 0x00000007f9000200, 0x00000007fae00000)

 compacting perm gen total 21248K, used 2879K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)

  the space 21248K, 13% used [0x00000007fae00000, 0x00000007fb0cfea0, 0x00000007fb0d0000, 0x00000007fc2c0000)

No shared spaces configured.



## 分配担保测试

设置新生代大小10M，SurvivorRatio=8 使得 Eden、From Survivor和To Survivor大小为8M，1M和1M

首先分配三个2M的数组到Eden，再分配第四个2M的数组的时候，

这时候触发MinorGC，Eden中没有需要回收的垃圾，在分配担保机制下，Eden空间中6M数组移动到了OLD

### 代码及JVM参数

```
public class HandlePromotion {
    private static final int _1MB = 1024 * 1024;
/*

     -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+UseSerialGC -XX:+PrintGCDetails
*/

    public static void HandlePromotion()
    {
        byte[] allocation1,allocation2,allocation3,allocation4;

        allocation1 = new byte[2 * _1MB];
        allocation2 = new byte[2 * _1MB]；
        allocation3 = new byte[2 * _1MB];
        allocation4 = new byte[2 * _1MB];
    }

    public static void main(String[] args) {

        System.out.println("Test MinorGC");
        HandlePromotion();
    }

}

```

### 运行结果

Test MinorGC

[GC[DefNew: 6826K->240K(9216K), 0.0048310 secs] 6826K->6384K(19456K), 0.0048590 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 

Heap

 def new generation  total 9216K, used 2454K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)

 eden space 8192K, 27% used [0x00000007f9a00000, 0x00000007f9c29810, 0x00000007fa200000)

 from space 1024K, 23% used [0x00000007fa300000, 0x00000007fa33c120, 0x00000007fa400000)

 to  space 1024K,  0% used [0x00000007fa200000, 0x00000007fa200000, 0x00000007fa300000)

 tenured generation  total 10240K, used 6144K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)

  the space 10240K, 60% used [0x00000007fa400000, 0x00000007faa00030, 0x00000007faa00200, 0x00000007fae00000)

 compacting perm gen total 21248K, used 2879K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)

  the space 21248K, 13% used [0x00000007fae00000, 0x00000007fb0cfea0, 0x00000007fb0d0000, 0x00000007fc2c0000)

No shared spaces configured.

# 大对象直接分配在老年代

PretenureSizeThreshold=3000000参数设定，超过多大的对象直接放入老年代，只对Serial和PawNew有用

### 代码及JVM参数

```java

public class BigObject {
    private static final int _1MB = 1024 * 1024;


/*

    -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+UseSerialGC -XX:+PrintGCDetails -XX:PretenureSizeThreshold=3000000
    
    */

    public static void BigObject()
    {
        byte[] allocation1,allocation2,allocation3,allocation4;

        allocation1 = new byte[4 * _1MB];
        
    }

    public static void main(String[] args) {

        System.out.println("Test MinorGC");
        BigObject();
        
    }

}


```

### 运行结果

Test MinorGC

Heap

 def new generation  total 9216K, used 846K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)

 eden space 8192K, 10% used [0x00000007f9a00000, 0x00000007f9ad3950, 0x00000007fa200000)

 from space 1024K,  0% used [0x00000007fa200000, 0x00000007fa200000, 0x00000007fa300000)

 to  space 1024K,  0% used [0x00000007fa300000, 0x00000007fa300000, 0x00000007fa400000)

 tenured generation  total 10240K, used 4096K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)

  the space 10240K, 40% used [0x00000007fa400000, 0x00000007fa800010, 0x00000007fa800200, 0x00000007fae00000)

 compacting perm gen total 21248K, used 2879K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)

  the space 21248K, 13% used [0x00000007fae00000, 0x00000007fb0cfe90, 0x00000007fb0d0000, 0x00000007fc2c0000)



# 长期存活对象进入老年代

MaxTenuringThreshold设置多大年龄进入老年代

### 代码及JVM参数

```java
public class TestTenuringThreshold {
    private static final int _1MB = 1024 * 1024;
/*

     -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+UseSerialGC -XX:+PrintGCDetails -XX:MaxTenuringThreshold=1
*/

    public static void TestTenuringThreshold()
    {
        byte[] allocation1,allocation2,allocation3,allocation4;

        allocation1 = new byte[_1MB / 4];
        allocation2 = new byte[4 * _1MB];
        allocation3 = new byte[4 * _1MB];
        allocation3 = null;
        allocation3 = new byte[4 * _1MB];
    }

    public static void main(String[] args) {

        System.out.println("Test MinorGC");
        TestTenuringThreshold();
    }

}

```

### 运行结果

Test MinorGC

[GC[DefNew: 5034K->496K(9216K), 0.0038710 secs] 5034K->4592K(19456K), 0.0039000 secs] [Times: user=0.01 sys=0.00, real=0.00 secs] 

[GC[DefNew: 4677K->0K(9216K), 0.0011980 secs] 8773K->4591K(19456K), 0.0012150 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

Heap

 def new generation  total 9216K, used 4260K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)

 eden space 8192K, 52% used [0x00000007f9a00000, 0x00000007f9e28fd0, 0x00000007fa200000)

 from space 1024K,  0% used [0x00000007fa200000, 0x00000007fa200088, 0x00000007fa300000)

 to  space 1024K,  0% used [0x00000007fa300000, 0x00000007fa300000, 0x00000007fa400000)

 tenured generation  total 10240K, used 4590K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)

  the space 10240K, 44% used [0x00000007fa400000, 0x00000007fa87bbf0, 0x00000007fa87bc00, 0x00000007fae00000)

 compacting perm gen total 21248K, used 2879K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)

  the space 21248K, 13% used [0x00000007fae00000, 0x00000007fb0cfea8, 0x00000007fb0d0000, 0x00000007fc2c0000)

No shared spaces configured.



__还有两种对比情况__

__MaxTenuringThreshold=15__

Test MinorGC

[GC[DefNew: 5034K->496K(9216K), 0.0035950 secs] 5034K->4592K(19456K), 0.0036190 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 

[GC[DefNew: 4677K->495K(9216K), 0.0011320 secs] 8773K->4591K(19456K), 0.0011470 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

Heap

 def new generation  total 9216K, used 4811K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)

 eden space 8192K, 52% used [0x00000007f9a00000, 0x00000007f9e37070, 0x00000007fa200000)

 from space 1024K, 48% used [0x00000007fa200000, 0x00000007fa27bc68, 0x00000007fa300000)

 to  space 1024K,  0% used [0x00000007fa300000, 0x00000007fa300000, 0x00000007fa400000)

 tenured generation  total 10240K, used 4096K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)

  the space 10240K, 40% used [0x00000007fa400000, 0x00000007fa800010, 0x00000007fa800200, 0x00000007fae00000)

 compacting perm gen total 21248K, used 2879K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)

  the space 21248K, 13% used [0x00000007fae00000, 0x00000007fb0cfea8, 0x00000007fb0d0000, 0x00000007fc2c0000)

No shared spaces configured.

__MaxTenuringThreshold=1,但是只分配担保，垃圾清理了一次所以年龄不超过2__

__修改代码：__

allocation1 = new byte[_1MB / 4];
        allocation2 = new byte[4 * _1MB];
        allocation3 = new byte[4 * _1MB];
        allocation4 = new byte[2 * _1MB];

Test MinorGC

[GC[DefNew: 5034K->496K(9216K), 0.0036380 secs] 5034K->4592K(19456K), 0.0036640 secs] [Times: user=0.00 sys=0.00, real=0.01 secs] 

Heap

 def new generation  total 9216K, used 6896K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)

 eden space 8192K, 78% used [0x00000007f9a00000, 0x00000007fa040290, 0x00000007fa200000)

 from space 1024K, 48% used [0x00000007fa300000, 0x00000007fa37c168, 0x00000007fa400000)

 to  space 1024K,  0% used [0x00000007fa200000, 0x00000007fa200000, 0x00000007fa300000)

 tenured generation  total 10240K, used 4096K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)

  the space 10240K, 40% used [0x00000007fa400000, 0x00000007fa800010, 0x00000007fa800200, 0x00000007fae00000)

 compacting perm gen total 21248K, used 2879K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)

  the space 21248K, 13% used [0x00000007fae00000, 0x00000007fb0cfeb0, 0x00000007fb0d0000, 0x00000007fc2c0000)

No shared spaces configured.



# 动态判定对象年龄

当Survivor中相同年龄的对象超过Survivor空间一般的时候就会直接进入老年代

### 代码及JVM参数

```java
public class TestTenuringThreshold2 {
    private static final int _1MB = 1024 * 1024;
/*

     -Xms20M -Xmx20M -Xmn10M -XX:SurvivorRatio=8 -XX:+UseSerialGC -XX:+PrintGCDetails -XX:MaxTenuringThreshold=1
*/

    public static void TestTenuringThreshold2()
    {
        byte[] allocation1,allocation2,allocation3,allocation4;

        allocation1 = new byte[_1MB / 4];
        allocation2 = new byte[_1MB / 4];
        allocation3 = new byte[4 * _1MB];
        allocation4 = new byte[4 * _1MB];
        allocation3 = null;
        allocation3 = new byte[4 * _1MB];
    }

    public static void main(String[] args) {

        System.out.println("Test MinorGC");
        TestTenuringThreshold2();
    }

}

```

### 运行结果

Test MinorGC

[GC[DefNew: 5290K->752K(9216K), 0.0047480 secs] 5290K->4848K(19456K), 0.0047730 secs] [Times: user=0.01 sys=0.00, real=0.01 secs] 

[GC[DefNew: 4932K->0K(9216K), 0.0035020 secs] 9028K->8943K(19456K), 0.0035210 secs] [Times: user=0.00 sys=0.00, real=0.00 secs] 

Heap

 def new generation  total 9216K, used 4260K [0x00000007f9a00000, 0x00000007fa400000, 0x00000007fa400000)

 eden space 8192K, 52% used [0x00000007f9a00000, 0x00000007f9e28fd0, 0x00000007fa200000)

 from space 1024K,  0% used [0x00000007fa200000, 0x00000007fa200088, 0x00000007fa300000)

 to  space 1024K,  0% used [0x00000007fa300000, 0x00000007fa300000, 0x00000007fa400000)

 tenured generation  total 10240K, used 8943K [0x00000007fa400000, 0x00000007fae00000, 0x00000007fae00000)

  the space 10240K, 87% used [0x00000007fa400000, 0x00000007facbbc10, 0x00000007facbbe00, 0x00000007fae00000)

 compacting perm gen total 21248K, used 2879K [0x00000007fae00000, 0x00000007fc2c0000, 0x0000000800000000)

  the space 21248K, 13% used [0x00000007fae00000, 0x00000007fb0cfeb0, 0x00000007fb0d0000, 0x00000007fc2c0000)

No shared spaces configured.

