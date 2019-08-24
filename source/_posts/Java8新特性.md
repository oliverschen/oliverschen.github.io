---
title: Java8新特性
date: 2019-07-27 00:03:44
tags: Java
category: Java
---

![Photo by Cristofer Jeschke](Java8新特性/Java8新特性.png)

Java8 相对之前版本来说是更新比较大的一个版本，不仅在 Java 语言本身，而且在编译器，类库，jvm 都引入了很多新的特性，工作中虽然偶尔会用到一些简单的特性，比如 forEach,stream,等相关功能，但是没有专门花时间去整理这些特性，最近新版本刚刚发布完成，抽空整理学习下这部分内容。用了 Java8 这么久，新特性面前不能瑟瑟发抖，哈哈哈....  <!--more-->


#### Lambda 表达式和函数式接口 FunctionInterface

Lambda 表达式是 Java8 引入的一个新特性，它允许将函数作为参数传递给调用方法，极大程度的简化了 Java 的代码量，这里举个例子：
``` java
/// 栗子
// Java8 之前遍历 List 实例
List<String> strings = Arrays.asList("a", "b", "c");
for (String s : strings) {
    System.out.println(s);
}
// Java8 
List<String> strings = Arrays.asList("a", "b", "c");
strings.forEach(s -> System.out.println(s));
```
上面这个例子比较简单，看起来也就省了一行代码，但是代码在整体的简洁程度有了很大的提高，Lambda 表达式主要符号是 `->`，在上面的例子中，参数 `s` 的类型是编译器自动去推断的，当然也可以自己声明参数的类型：
``` java
List<String> strings = Arrays.asList("a", "b", "c");
strings.forEach((String s )-> System.out.println(s));
```
Lambda 表达式引用成员变量或者局部变量时，会被隐式的转成 final 类型
```java
List<String> strings = Arrays.asList("a", "b", "c");
String ext = ",";
strings.forEach((String s )-> System.out.println(s + ext));
```
上面其实等价于
```java
List<String> strings = Arrays.asList("a", "b", "c");
final String ext = ",";
strings.forEach((String s )-> System.out.println(s + ext));
```

#### 方法引用

Java8 提供了方法引用的语法来直接访问类或者已经存在的方法或者构造方法，是语法更加简洁紧凑，一般主要有下面几种形式：
##### 构造方法引用
通过构造方法引用，可以实例化一个对象
```java
// Person 类【
public class Person {
    public static Person newPerson(final Supplier<Person> supplier) {
        return supplier.get();
    }
}
// test
@Test
public void testNew() {
    Person person = Person.newPerson(Person::new);
    System.out.println(person);
}

```
通过 Class::new 这种语法来创建对象，这里要提供一个无参构造器，否则会编译出错。
##### 类实例方法引用&静态方法引用
```java
public class Person {

    private Integer age;
    public Person() {
        age = new Random().nextInt(10);
    }
    public static Person newPerson(final Supplier<Person> supplier) {
        return supplier.get();
    }
    public Integer getAge() {
        System.out.println(age);
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }
}

/// test
@Test
public void testFor() {
    List<Person> list = new ArrayList<>();
    for (int i = 0; i < 4; i++) {
        list.add(Person.newPerson(Person::new));
    }
    // 类的实例方法引用
    list.forEach(Person::getAge);
}
```
从上面的例子可以看到，每次输出的都是不同的 age 属性，语法上更加简洁明了。**静态方法引用**和类的实例方法引用一样，也是 ClassName::methodName 的方式引用。一般用的最多的也就是这几种引用，后面有机会在补充上来。

#### Stream 流
Java8 提供了一种新的数据处理方式---流。流的处理的概念类似管道处理，结合 Lambda 表达式使得集合等数据处理变得简洁高效，还可以对数据进行筛选、排序、聚合等操作，简直不要太爽。

> 集合，数组，I/O channel，generator 等。
> 集合中包含串行流和并行流，一般情况下并行流比串行流效率更高。
> 串行流：stream()。
> 并行流：parallelStream()。

下面是一个简单的列子：

```java
// javabean 添加了 name 属性
public class Person {

    private Integer age;
    private String name;
    public Person() {
        age = new Random().nextInt(10);
    }

    public Person(String name,Integer age) {
        this.age = age;
        this.name = name;
    }
    public static Person newPerson(final Supplier<Person> supplier) {
        return supplier.get();
    }
    public Integer getAge() {
        System.out.println(age);
        return age;
    }
    public void setAge(Integer age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }
}

// 测试
@Test
public void testStream() {
    List<Person> list = new ArrayList<>();
    list.add(new Person("小明", 20));
    list.add(new Person("李四", 25));
    list.add(new Person("二炮", 23));
    list.add(new Person("大同", 29));
    list.add(new Person("王五", 34));
    list.add(new Person("小二", 18));
    Predicate<Person> predicate = (person) -> person.getAge() > 25;
    list.forEach(person -> {
        if(predicate.test(person)){
            System.out.println(person.getName());
        }
    });
}
```
这里的串行流比较类似迭代器，使用 Predicate 创建了一个条件，在迭代时将满足条件的接口打印出来。

##### 聚合操作

###### forEach
> forEach() 在上面的例子中已经出现了很多次了，也是我经常用的方法之一，stream 流内部遍历简化了代码，在语法上面更加清晰。

###### map

> map 可以按照规则映射成另一个元素，简单来说就是可以给某个元素设置另外的值。
```java
@Test
public void testMap() {
    List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
    List<Integer> collect = integers.stream().map(i -> i+1).collect(Collectors.toList());
    System.out.println(collect);
}
```

###### filter

> filter 可以对 stream 中元素进行过滤

```java
@Test
public void filter() {
    List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5);
    long count = integers.stream().filter(integer -> integer > 3).count();
    System.out.println(count);
}
```
集合中数值大于 3 的数量统计。

###### limit 

> 此方法可以获取指定数量的流

```java
@Test
public void testLimit() {
    List<Integer> integers = Arrays.asList(1, 2, 3, 4, 5, 8, 9, 34);
    List<Integer> collect = integers.stream().limit(6).collect(Collectors.toList());
    System.out.println(collect);
}
```
打印前 6 条记录，这里 limit 指定想要的条数，从 0 开始取。

###### sorted

> 顾名思义，可以利用 sorted 方法对流数据进行排序

```java
@Test
public void testSorted() {
    List<Integer> integers = Arrays.asList(1, 99, 22, 4, 3, 11, 9, 34);
    List<Integer> collect = integers.stream().sorted().collect(Collectors.toList());
    System.out.println(collect);
}
```
这个使用还是很顺滑的，减少了很多代码就可以优雅进行排序了。

###### concat

> 对流进行合并操作

concat 方法是 Stream 接口的一个静态方法，如果合并时两个流中元素是经过排序的，则得到的元素也是排序的结果。

```java
@Test
public void testConcat() {
    List<Integer> integers = Arrays.asList(11, 9, 34);
    List<Integer> integerList = Arrays.asList(10,19, 2);
    List<Integer> collect = Stream.concat(integers.stream(), integerList.stream()).collect(Collectors.toList());
    System.out.println(collect);
}
```

###### distinct

> 对流中元素进行去重操作

```java
@Test
public void testDistinct() {
    List<Integer> integers = Arrays.asList(1, 99, 1, 4, 1, 11, 1, 34);
    List<Integer> collect = integers.stream().distinct().collect(Collectors.toList());
    System.out.println(collect);
}
```

这里有个坑，就是如果List 是一个对象集合，没有重写 equals() 方法的话，得到的答案是不正确的，因为它是基于 equals() 方法来实现的。

###### skip

> 跳过流中的某几个元素，经常和 limit 配置使用。

使用 skip 配合 limit 可以更好的对流中元素进行分页操作。

```java
@Test
public void testSkip() {
    List<Integer> integers = Arrays.asList(1, 99, 12, 40, 8, 11);
    List<Integer> collect = integers.stream().skip(2).limit(3).collect(Collectors.toList());
    System.out.println(collect);
}
```

###### match

> 匹配指定的元素
> Stream 提供了 3 个 api 接口，分别是：allMatch()，anyMatch()，noneMatch() 方法的功能基本顾名思义的。

```java
 @Test
public void testMatch() {
    List<Integer> integers = Arrays.asList(1, 99, 12, 40, 8, 11);
    boolean b = integers.stream().allMatch(integer -> integer > 8);
    System.out.println(b);
    boolean anyMatch = integers.stream().anyMatch(integer -> integer > 8);
    System.out.println(anyMatch);
    boolean noneMatch = integers.stream().noneMatch(integer -> integer > 8);
    System.out.println(noneMatch);
}
```

#### Optional

代码中空指针异常是一个让人很头疼但是又不得不去检查的操作，在 Java8 之前都是用过 if 来判断的，Java8 则引入了 Optional 来解决，使代码不被空间查污染。
Optional 是一个容器，它可以保存任意类型的值，也可以保存 null，通过它提供的方法，就可以不用显式的进行空值判断。书面话的说明看着都是似懂非懂的感觉，还是看代码更有感觉一点。

##### Optional 创建

1. Optional.empty():创建一个空的 Optional 对象
2. Optional.of(T value):传入的 value 值为 null 的话，会抛空指针异常。
3. Optional.ofNullable(T value): 传入的 value 值允许为空。
```java
@Test
public void testOptional() {
    // empty()
    Optional<Person> empty = Optional.empty();
    // of(T value)
    Optional<Person> optional = Optional.of(new Person());
    // ofNullable(T value)
    Optional<Person> ofNullable = Optional.ofNullable(null);
}
```
###### orElse

> 有值则返回，没有则返回其他值

如果传入的值是 null 时，返回 orElse 方法中指定的结果，否则返回实例的值。

```java
@Test
public void testOrElse() {
    Person p = new Person();
    Optional<String> optional = Optional.ofNullable(p.getName());
    String name_is_null = optional.orElse("name is null");
    System.out.println(name_is_null);
    p.setName("jihe");
    Optional<String> optionalS = Optional.ofNullable(p.getName());
    String name_is_nullS = optionalS.orElse("name is null");
    System.out.println(name_is_nullS);
}
```

###### orElseGet

> 在功能上和 orElse() 方法类似，只是 orElseGet 指出传入一个 Lambda 表达式生产默认值

如果想要的实例为 null 时，就可以通过 orElseGet 获取一个非空实例

```java
@Test
public void testOrElseGet() {
    Person p = null;
    Optional<Person> optional = Optional.ofNullable(p);
    Person person = optional.orElseGet(() -> new Person("jihe", 23));
    System.out.println(person.getName());
}
```

以上是一些比较常用的 Java8 新特性，后面会一点点的完善上来。另外 Java8 对时间日期处理也引入了新的 API 来解决让人傻傻分不清的日期类。让日期和时间的处理更加人性化，易于理解。在 JVM 层面，PermGen 空间被移除了，取而代之的是 MetaSpace，与之对应的 -XX:PermSize 和 -XX:MaxPermSize 参数分别被 -XX:MetaSpaceSize 和 -XX:MaxMetaSpaceSize 所代替。[ PermSize 是用来存放 Class 类元数据的内存区域，如果在应用启动时加载的类信息比较的的话，可能会抛出 "java.lang.OutOfMemoryError: PermGen space" 异常，Java8 使用了本地内存来存储这部分信息，将这部分空间全部移除]



今晚去看了哪吒，超级好看。里面一句台词让人印象深刻。


<center>人的成见就像一座大山</center>









