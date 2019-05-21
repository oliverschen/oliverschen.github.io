---
title: jvm-class文件结构
date: 2019-05-05 20:36:37
tags: openjdk
category: openjdk
---

![](jvm-class文件结构/class.png)

最近 manjaro 又更新挂了，算了，不折腾了，考虑真香 windows 了。（如果不忙的话还是很喜欢折腾的，但是最近手头也比较忙就先不换回去了）今年是准备偏向底层的学习，现在新技术层出不穷，让人眼花缭乱。有时候会很浮躁，担心学到的东西过不了多久就会被抛弃。也是瑟瑟发抖。不过看了很多大佬写的东西，都会有一个结论，就是偏底层技术永远是新技术的支撑，那些枯燥乏味的知识才是最生命力最强的技术。

<!-- more -->

我们都知道 Java 程序要运行的话，需要编译然后才能运行。Java 文件经过编译器编译之后会生产 class 文件，一般打开都是乱码的，建议使用 notepad++ 插件 HEX-editor 来进行查看字节码文件，打开文件就会看到一堆十六进制的代码，不过这样一堆代码也是有规可循的。

#### 文件基本组成

##### Java 魔数

这个相信学习 Java 的都会知道，JVM 如何知道这个文件是 class 文件呢，就是根据文件最开头的 4 个字节来判断的，这 4 个字节很有意思，分别是 `ca fe ba be` 咖啡贝比，都是十六进制数组成的，不知道当年 Java 之父想到这个时是多开心的...。当然，验证完是一个字节码文件之后还会验证很多东西，比如版本，全限定名，常量等等。否则会抛异常。

##### 主次版本号

跟在魔数后面的就是 Java 的 主次版本号，Java 语言都是向下兼容的，如果低版本跑在高版本的 JVM 就会抛错处出来。

##### 常量池计数器

在版本号后面会有 2 个字节的计数器，顾名思义，是用来记录当前 class 常量池大小计数器。

##### 常量池

常量池可以理解为整个文件的 `元数据`，就像是一个文件，文件名叫什么，什么时候创建的，文件多大，文件类型是什么。常量池包含了类的属性，方法，接口等一些类的描述信息。

***
以上就是对 class 文件基础信息的简单总结，这个会持续更新，我也在学习中，希望大家能在使用 Java 时，可以知其然，并知其所以然。（如果有问题，大佬们请多多指正）

***

#### 代码分析

##### .java 文件

我写了一个很简单的类，来大概分析下 class 文件的对应关系。

```java
public class ClassStructure extends Thread implements Serializable {
    private int i = 0;
    private static int a = 1;

    public void inc() {
        i++;
    }

    public int add() {
        return i + a;
    }

    public void test() {
        try {
            i = 1;
        } catch (Exception e) {
            a = 2;
        }
    }
}
```

首先使用 `javac ClassStructure.java` 编译文件，正常情况下会生成一个 ClassStructure.class 文件，然后使用  `javap -v ClassStructure.class` 命令查看文件信息，会生成下面的内容

```bash
G:\jvm\src\main\java\com\jihe\jvm\test>javap -v ClassStructure.class
Classfile /G:/jvm/src/main/java/com/jihe/jvm/test/ClassStructure.class
  Last modified 2019-5-21; size 617 bytes
  MD5 checksum e664b24bf12decb3e4ff2dbcf2857a55
  Compiled from "ClassStructure.java"
public class com.jihe.jvm.test.ClassStructure extends java.lang.Thread implements java.io.Serializable
  minor version: 0
  major version: 52
  flags: ACC_PUBLIC, ACC_SUPER
Constant pool:
   #1 = Methodref          #6.#24         // java/lang/Thread."<init>":()V
   #2 = Fieldref           #5.#25         // com/jihe/jvm/test/ClassStructure.i:I
   #3 = Fieldref           #5.#26         // com/jihe/jvm/test/ClassStructure.a:I
   #4 = Class              #27            // java/lang/Exception
   #5 = Class              #28            // com/jihe/jvm/test/ClassStructure
   #6 = Class              #29            // java/lang/Thread
   #7 = Class              #30            // java/io/Serializable
   #8 = Utf8               i
   #9 = Utf8               I
  #10 = Utf8               a
  #11 = Utf8               <init>
  #12 = Utf8               ()V
  #13 = Utf8               Code
  #14 = Utf8               LineNumberTable
  #15 = Utf8               inc
  #16 = Utf8               add
  #17 = Utf8               ()I
  #18 = Utf8               test
  #19 = Utf8               StackMapTable
  #20 = Class              #27            // java/lang/Exception
  #21 = Utf8               <clinit>
  #22 = Utf8               SourceFile
  #23 = Utf8               ClassStructure.java
  #24 = NameAndType        #11:#12        // "<init>":()V
  #25 = NameAndType        #8:#9          // i:I
  #26 = NameAndType        #10:#9         // a:I
  #27 = Utf8               java/lang/Exception
  #28 = Utf8               com/jihe/jvm/test/ClassStructure
  #29 = Utf8               java/lang/Thread
  #30 = Utf8               java/io/Serializable
{
  public com.jihe.jvm.test.ClassStructure();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: invokespecial #1                  // Method java/lang/Thread."<init>":()V
         4: aload_0
         5: iconst_0
         6: putfield      #2                  // Field i:I
         9: return
      LineNumberTable:
        line 10: 0
        line 12: 4

  public void inc();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=3, locals=1, args_size=1
         0: aload_0
         1: dup
         2: getfield      #2                  // Field i:I
         5: iconst_1
         6: iadd
         7: putfield      #2                  // Field i:I
        10: return
      LineNumberTable:
        line 16: 0
        line 17: 10

  public int add();
    descriptor: ()I
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=1, args_size=1
         0: aload_0
         1: getfield      #2                  // Field i:I
         4: getstatic     #3                  // Field a:I
         7: iadd
         8: ireturn
      LineNumberTable:
        line 20: 0

  public void test();
    descriptor: ()V
    flags: ACC_PUBLIC
    Code:
      stack=2, locals=2, args_size=1
         0: aload_0
         1: iconst_1
         2: putfield      #2                  // Field i:I
         5: goto          13
         8: astore_1
         9: iconst_2
        10: putstatic     #3                  // Field a:I
        13: return
      Exception table:
         from    to  target type
             0     5     8   Class java/lang/Exception
      LineNumberTable:
        line 25: 0
        line 28: 5
        line 26: 8
        line 27: 9
        line 29: 13
      StackMapTable: number_of_entries = 2
        frame_type = 72 /* same_locals_1_stack_item */
          stack = [ class java/lang/Exception ]
        frame_type = 4 /* same */

  static {};
    descriptor: ()V
    flags: ACC_STATIC
    Code:
      stack=1, locals=0, args_size=0
         0: iconst_1
         1: putstatic     #3                  // Field a:I
         4: return
      LineNumberTable:
        line 13: 0
}
SourceFile: "ClassStructure.java"

```

其实在这里看的话已经很明了了，在一些代码后面还有相应的注释，我们都知道一个类对象创建是在调用其构造方法的时候，那先从构造器开始看起