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
