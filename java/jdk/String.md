了解jdk String的特性

https://www.jianshu.com/p/cf78e68e3a99

## String 的内存模型



## 声明字符串变量的方式

1、new String对象 String s1= new String("hello world");  

通过构造器方式创建的字符串对象存放在**堆内存、不具备不可变性**。除非需要拷贝此String 对象否则不推荐这种方式声明一个字符串。



2、字符串常量拼接 String s2 = "hello" + " world";  

JVM编译器在编译阶段，将String s2 = “hello” + " world"; 优化为 hello world



下面的测试代码和反编译后的jvm指令，可以看到第一行指令ldc向栈上变量赋值时已经被编译器优化为了hello world!，str3 str4都指向同一个字符串常量池中的常量，所以str3 == str4 为true

```java
// 测试代码 
public void test_1() {
        String str3 = "hello"+" word!";
        String str4 = "hello word!";
        System.out.println(str3 == str4);

    }
// 反编译后的jvm指令
public void test_1();
    Code:
       0: ldc           #17      // String hello word!
       2: astore_1
       3: ldc           #17      // String hello word!
       5: astore_2
       6: getstatic     #13      // Field java/lang/System.out:Ljava/io/PrintStream;
       9: aload_1
      10: aload_2
      11: if_acmpne     18
      14: iconst_1
      15: goto          19
      18: iconst_0
      19: invokevirtual #18      // Method java/io/PrintStream.println:(Z)V
      22: return

```



3、字符变量+ 常量拼接 String s3 = s2 + "";

字符变量+ 常量拼接 String 编译器编译器优化后通过StringBuilder apend()方法拼接，最终通过StringBuilder.toString()返回新的字符串对象

String str4 = str1 + " word!";jvm编译器在编译阶段将其优化成了通过StringBuilder.append()进行拼接，

最终通过StringBuilder.toString()返回新的字符串对象。所以str3 == str4返回false

```java
// 测试代码
public void test_2() {
        String str1 = "hello";
        String str2 = " word!";
        String str3 = "hello word!";
        String str4 = str1 + " word!";
        System.out.println(str3 == str4);
        //result: false
    }  

// 反编译后的jvm指令
public void test_2();
    Code:
       0: ldc           #19                 // String hello
       2: astore_1
       3: ldc           #20                 // String  word!
       5: astore_2
       6: ldc           #17                 // String hello word!
       8: astore_3
       9: new           #8                  // class java/lang/StringBuilder
      12: dup
      13: invokespecial #9                  // Method java/lang/StringBuilder."<init>":()V
      16: aload_1
      17: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      20: ldc           #20                 // String  word!
      22: invokevirtual #10                 // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      25: invokevirtual #12                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      28: astore        4
      30: getstatic     #13                 // Field java/lang/System.out:Ljava/io/PrintStream;
      33: aload_3
      34: aload         4
      36: if_acmpne     43
      39: iconst_1
      40: goto          44
      43: iconst_0
      44: invokevirtual #18                 // Method java/io/PrintStream.println:(Z)V
      47: return
}

// StringBuilder.toString();
	@Override     
    public String toString() {
        // Create a copy, don't share the array
        return new String(value, 0, count);
    }
```





## 不可变

### 什么是不可变

声明一个字符串对这个字符串进行任何操作都不改变这个字符串的value 和hashcode

```java
public void test_4() {
    String str = "hello";
    str.concat(" world!");
    System.out.println("str = " + str);
    // 打印 str = hello
}
```

**注意是value不变不是对象的引用不变**

```java
// 这里是str1指向了"world!"的内存地址 
public void test_3() {
    String str1 = "hello";
    str1 = " world!";
}
```



### 不可变如何体现

#### 1、成员变量不可修改

String 内部只有两个成员变量，分别是保存字符串的char数组、字符串hashcode的hash

```java
/** The value is used for character storage. */
private final char value[];

/** Cache the hash code for the string */
private int hash; // Default to 0
```

这两个成员变量是私有的且没有提供set和build方法用于修改这些值



#### 2、不可继承

String是final类，不可通过继承String从而重写关于value 和 hashCode的方法



#### 3、public 方法都是复制一份数据，原String 对象不变

String提供的公共修改方法，底层都是创建了一个新的字符串对象返回，将原String值value 拷贝到新的char[]数组进行操作后返回新的String对象。

```java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}

public String substring(int beginIndex) {
        if (beginIndex < 0) {
            throw new StringIndexOutOfBoundsException(beginIndex);
        }
        int subLen = value.length - beginIndex;
        if (subLen < 0) {
            throw new StringIndexOutOfBoundsException(subLen);
        }
        return (beginIndex == 0) ? this : new String(value, beginIndex, subLen);
    }
```



### 为什么设计成不可变的

#### 1、缓存需要

不可变意味着可以缓存，jvm将字符串缓存在常量池中，线程共享；	其中一个客户端修改了字符串常量会影响其它客户端的使用。因为字符串常量池对性能很重要，为了排除这种风险，String设计成不可变Immutable。



#### 2、线程安全

不可变意味着String 提供的操作方法不会出现线程并发问题。

如下方法：String 不可变 value不变，不会出现多线程可见性、原子性问题，因为自始至终都未对共享变量进行任何修改。

```java
public String concat(String str) {
    int otherLen = str.length();
    if (otherLen == 0) {
        return this;
    }
    int len = value.length;
    char buf[] = Arrays.copyOf(value, len + otherLen);
    str.getChars(buf, len);
    return new String(buf, true);
}
```



#### 3、缓存hashCode

与value一样hashcode是不可变的，可以缓存不用计算。

以HashMap<String, Object>为例，如果String可变那hashcode也会改变，就造成get()获取的值错误，hashcode变化定位到的数组位置就变了。



## 可变字符串 



### StringBuilder

非线程安全可变字符串



### StringBuffer

线程安全可变字符串



### 与String性能对比

字符串修改操作，变长字符串比String快很多



String进行修改操作，实际上是创建了新的String对象然后赋值给它；变长字符串【StringBuilder/StringBuffer】的修改操作通过数组拷贝，将修改后的值放到新的数组中来实现【System.arrayCopy()】，不需要创建新的字符串对象。



对字符串的修改操作，变长字符串性能远高于String，主要原因在于String是不变的对一个String对象进行修改操作会创建新的String对象并赋值。变长字符串【StringBuffer/StringBuilder】底层通过拷贝字符数组并重新赋值来完成【System.arraycopy】拷贝过程中创了一个新的数组用于存放修改后的字符串value，String每次修改操作都创建String对象的方式即慢又消耗内存资源。

```java
/**
 * 字符串拼接，stringBuilder 比 String的性能快1000倍，循环体越大性能差距越大
 */
public void test_stringBuilder() {
    StopWatch sw = new StopWatch();
    sw.start("sb");

    StringBuilder sb = new StringBuilder();
    for (int i = 0; i < 50000; i++) {
        sb.append(i);
    }
    sw.stop();

    sw.start("str");
    String str = "";
    for (int i = 0; i < 50000; i++) {
        str += i;
    }
    sw.stop();

    System.out.println("耗时 = " + sw.prettyPrint());
}

耗时 = StopWatch '': running time (millis) = 6584
-----------------------------------------
ms     %     Task name
-----------------------------------------
00005  000%  sb
06579  100%  str

```



for循环中使用变量对String进行赋值，编译器会优化成StringBuilder apend()进行拼接，最终通过StringBuilder.toString()返回一个新的String对象，所以还是在创建了一个新的Stirng对象返回。

```java
// 上述测试代码对应的JVM指令，JVM编译器将循环内部的str += i;优化为StringBuilder.append()；通过toString()返回新的字符串对象
public void test_7();
    Code:
       0: ldc           #31                 // String
       2: astore_1
       3: iconst_0
       4: istore_2
       5: iload_2
       6: ldc           #32                 // int 50000
       8: if_icmpge     36
      11: new           #7                  // class java/lang/StringBuilder
      14: dup
      15: invokespecial #8                  // Method java/lang/StringBuilder."<init>":()V
      18: aload_1
      19: invokevirtual #9                  // Method java/lang/StringBuilder.append:(Ljava/lang/String;)Ljava/lang/StringBuilder;
      22: iload_2
      23: invokevirtual #21                 // Method java/lang/StringBuilder.append:(I)Ljava/lang/StringBuilder;
      26: invokevirtual #11                 // Method java/lang/StringBuilder.toString:()Ljava/lang/String;
      29: astore_1
      30: iinc          2, 1
      33: goto          5
      36: return

```

