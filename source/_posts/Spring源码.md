---
title: Spring源码
date: 2021-02-10 22:52:03
tags: Spring
category: Spring
---

![Photo by destex on wallhaven.cc](/Spring-source.png)


Spring 源码流程分析，主要包含 Spring 核心功能 IoC 和 AOP 部分代码，Spring 循环依赖等问题。
<!--more-->

## Bean生命周期
![生命周期](/bean.jpg)

## 🦸‍♀️IoC
控制反转：将创建对象的权利交给 Spring 容器来完成
### 两种容器
#### BeanFactory
「简单容器」 
#### ApplicationContext
「复杂容器」
### 流程图
下面是通过加载 XML 的形式加载 Bean 时 IoC 容器启动过程，包含初始化 Bean 和 循环依赖问题。
> ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");

![IoC](/IoC.png)

以注解扫描的方式启动和 XML 的类似。流程是一样的。
### 后置处理器🤖️
Spring 提供了很多空接口，供开发者自己实现，比如下面是 PostProcessor 「后置处理器」相关接口
#### BeanFactoryPostProcessor
beanFactory 的后置处理器：有个 postProcessBeanFactory(ConfigurableListableBeanFactory beanFactory) 方法，获取到的是 「Bean 工厂」，是在 invokeBeanFactoryPostProcessors() 方法中调用的，这个时候已经获取到了所有的 Bean 定义实例，但是还没有进行初始化，可以对所有 Bean 定义进行操作修改。
#### BeanPostProcessor
Bean 后置处理器：在 initializeBean() 方法中执行，会拦截所有的 Bean 进入当前处理器。此时 Bean 已经实例化完成。它提供了两个方法：
1. postProcessBeforeInitialization
invokeInitMethods() 方法之前的处理器
2. postProcessAfterInitialization
invokeInitMethods() 方法之后的处理器
#### InitializingBean
初始化 Bean 时在 Bean 的后置处理 before 方法之后执行（看上面的执行顺序），此时已经初始化完成。在调用 invokeInitMethods() 方法时执行，此时会有三种情况：
1. 只实现了 InitializingBean 接口，会执行 afterPropertiesSet() 方法。
2. 如果只设置了 init-method （@Bean(name = "user", initMethod = "init")），此时会执行init()方法
3. 如果以上都有，按照1，2的顺序执行
## 代码
### 启动类代码

```java
/**
 * @author ck
 * 基于 xml 获取 bean
 */
public class XmlBean {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
      
        Student student001 = (Student) context.getBean("student001");
        System.out.println(String.format("student001 ==>%s", student001.toString()));
        Student student002 = (Student) context.getBean("student002");
        System.out.println(String.format("student002 ==>%s", student002.toString()));
        School school = (School) context.getBean("school");
        System.out.println(String.format("school: ==>%s",school.toString()));
    }
}
```
#### 配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-3.2.xsd
                        http://www.springframework.org/schema/aop
                        https://www.springframework.org/schema/aop/spring-aop.xsd">
    
    <bean id="student001" class="com.github.oliverschen.springbean.Student">
        <property name="age" value="25" />
        <property name="name" value="ck" />
    </bean>
    
    <bean id="student002" class="com.github.oliverschen.springbean.Student">
        <property name="age" value="24" />
        <property name="name" value="fz" />
    </bean>
    <bean id="school" class="com.github.oliverschen.springbean.School">
        <property name="address" value="深圳"/>
        <property name="schoolName" value="深大"/>
        <property name="student" ref="student001"/>
    </bean>
</beans>
```
## 🦸‍♂️AOP
AOP 是在原有代码的基础上，对代码进行「横向」增强。OOP 中继承都是「竖向」对代码进行增强，AOP 弥补了这个空白。

![aop-oop](/aop-oop.jpg)
### 实现
#### AspectJ
Eclipse 基金会提供的一个完整的解决方案。功能比 Spring AOP 要强大。它属于静态织入，有多种织入方式。
1. Compile-time weaving
编译期织入
2. Post-compile weaving
编译后织入
3. Load-time weaving
指的是在加载类的时候进行织入「自定义类加载器/启动是指定AspectJ 提供的 
agent：-javaagent:xxx/xxx/aspectjweaver.jar」
它在代码运行前已经将代码进行织入，所以执行起来就像是一个新增了方法的类一样，没有其他开销。
#### CGlib
Spring 中代理类没有接口，则用 CGlib 实现生成代理类。
CGlib是针对类实现代理，主要是对指定的类生成一个子类，覆盖其中的方法，并覆盖其中方法的增强，但是因为采用的是继承，所以该类或方法最好不要生成final，对于final类或方法，是无法继承的。采用 ASM 字节码技术。
在 JDK8 之前，大量调用时性能优于 JDK 动态代理。JDK8 之后，性能持平或稍逊于 JDK 动态代理。
JDK Proxy
JDK 内置的动态代理类。在 Spring 中如果目标类有接口就直接用 JDK 动态代理方式生成代理类。
### 流程
类关系
下图是自动代理主要的类 AnnotationAwareAspectJAutoProxyCreator 继承关系，它本质其实是一个拥有BeanPostProcessor 接口特性的类。
![](class-image.png)

调用流程
1. AOP 生成代理对象是依赖了 IoC 容器，在容器初始化完 Bean 之后，在 BeanPostProcessor 后置处理器的 after 方法中执行生成过程。
2. 调用时先拿到目标方法的拦截器链，然后执行对应的切面方法。
![AOP](/AOP.png)

#### 代码
##### 切面
```java
/**
 * @author ck
 */
@Aspect
@Component
public class AopLog{
    @Pointcut("execution(* com.github.oliverschen.springbean..*())")
    public void point() {}
    @Before("point()")
    public void before(JoinPoint joinPoint) {
        System.out.println("before 方法");
    }
    @After("point()")
    public void after() {
        System.out.println("after 方法");
    }
    @AfterReturning("point()")
    public void ret() {
        System.out.println("return 方法");
    }
    @AfterThrowing("point()")
    public void trw() {
        System.out.println("AfterThrowing 方法");
    }
    @Around("point()")
    public void around(ProceedingJoinPoint joinPoint) {
        System.out.println("around 方法进入");
        try {
            joinPoint.proceed();
            System.out.println("around 方法完成");
        } catch (Throwable throwable) {
            throwable.printStackTrace();
        }
    }
}
启动类
/**
 * @author ck
 * 基于 xml 获取 bean
 */
public class XmlBean {
    public static void main(String[] args) {
        ApplicationContext context = new ClassPathXmlApplicationContext("applicationContext.xml");
        OrderService orderService = (OrderService) context.getBean("orderService");
        orderService.order();
    }
}
```
##### 配置
```xml
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xmlns:context="http://www.springframework.org/schema/context"
       xmlns:aop="http://www.springframework.org/schema/aop"
       xsi:schemaLocation="http://www.springframework.org/schema/beans
                        http://www.springframework.org/schema/beans/spring-beans-3.2.xsd
                        http://www.springframework.org/schema/context
                        http://www.springframework.org/schema/context/spring-context-3.2.xsd
                        http://www.springframework.org/schema/aop
                        https://www.springframework.org/schema/aop/spring-aop.xsd">
    
    <bean id="student001" class="com.github.oliverschen.springbean.Student">
        <property name="age" value="25" />
        <property name="name" value="ck" />
    </bean>
    
    <bean id="student002" class="com.github.oliverschen.springbean.Student">
        <property name="age" value="24" />
        <property name="name" value="fz" />
    </bean>
    <bean id="school" class="com.github.oliverschen.springbean.School">
        <property name="address" value="深圳"/>
        <property name="schoolName" value="深大"/>
        <property name="student" ref="student001"/>
    </bean>
    <context:component-scan base-package="com.github.oliverschen.springbean" />
    <aop:aspectj-autoproxy/>
</beans>
```

<center>再见，2020，你好，2021</center>