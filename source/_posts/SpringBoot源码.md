---
title: SpringBoot源码
date: 2021-02-14 15:04:06
tags: Spring
category: Spring
comments: true
---

![Photo by destex on JustJon](/SpringBoot.png)

Springboot 是 Spring 退出的一个快速集成开发框架，可以很容易的搭建一个 Web 项目，它采用 `starter` 的方式来引入需要的组件，自动装配需要的配置信息。很大程度的提高了开发效率，让开发者专注于具体业务的开发。这里是我在看 SpringBoot 源码时的一些分析和流程梳理。 
<!--more-->

## 注解

### @SpringBootApplication

这是 SpringBoot 启动动类注解，它是一个组合注解。

![SpringBootApplication 注解](/SpringBoot-anno.jpg)

#### @SpringBootConfiguration

它导入了 @Configuration 注解，拥有配置属性。所以 @SpringBootApplication 注解也拥有「配置类」属性。

#### @EnableAutoConfiguration

##### AutoConfigurationImportSelector

开启自动配置注解，这个注解 Import 了一个 AutoConfigurationImportSelector 自动配置导入选择器。

![实现类](/impl.png)

它实现了 Aware 接口，在 IoC 容器启动后，在初始化 Bean 时执行导入，Selector 类会去加载 `spring-boot-autoconfigure` 包下 `Resource/META-INFO/spring.factories` 文件中的  `Auto Configure` 自动配置类，这个文件已经默认将 Redis,RabbitMQ,Kafka,Mybatis,elasticsearch 等常见的第三方自动配置类配置了。只要引入依赖就可以直接使用了。

##### @AutoConfigurationPackage

自动配置基础扫描包，这个注解 Import 了 AutoConfigurationPackages.Registrar 类，负责将标注次注解的类所在的包名注册到 Bean 定义中

#### @ComponentScan

组建扫描注解，通过 ClassUtils.*getPackageName*(启动类) 扫描启动类包以及子包中的 @Component，@Service 等注解，将其加入到 IoC 容器中。

## 源码

下图是源码的大致调用和启动流程，蓝色为主干，其他均为各个分支。

![源码流程图](/flow.png)

### 重点

#### invokeBeanFactoryPostProcessors(beanFactory)

扫描 Bean 定义并且执行核心的自动配置都是在这里完成的，@SpringBootApplication 包含的所有注解涉及的步骤也是在这个方法中被加载使用。

##### AutoConfigurationImportSelector

![自动装配类](/auto-import.png)

它实现 Aware 和 ImportSelector 接口，在配置类记载是执行，并找到 `spring-boot-autoconfigure`

包中的 spring.factories 下所有的自动配置类，进行过滤并且自动装配。

##### 过滤规则

SpringBoot 自动装配时不会把所有的配置都加载进来 ，它会进行过滤，只添加项目需要的配置类，根据下面的注解：

1. @ConditionalOnProperty
2. @ConditionalOnClass
3. @ConditionalOnBean
4. @ConditionalOnWebApplication

通过自动配置类上面的注解进行过滤，筛选出项目依赖的配置类。

##### DispatcherServletAutoConfiguration

@ConditionalOnWebApplication(type = Type.SERVLET) 以 SERVLET 类型启动并且 @ConditionalOnClass(DispatcherServlet.class) 存在该类时会自动加载 DispatcherServlet 相关配置。

#### onRefresh()

如果依赖了 `spring-boot-starter-web` ，会在此时去创建 tomcat 相关的操作。

### refresh
refresh() 方法是容器启动的核心方法。
```java
@Override
public void refresh() throws BeansException, IllegalStateException {
    synchronized (this.startupShutdownMonitor) {
        // Prepare this context for refreshing.
        //1. 设置容器启动状态。创建早期事件监听，发布于多播器创建之前
        prepareRefresh();
        
        // Tell the subclass to refresh the internal bean factory.
        //2. 创建 beanFactory，加载 beanDefinitions(bean 定义)
        ConfigurableListableBeanFactory beanFactory = obtainFreshBeanFactory();
        
        // Prepare the bean factory for use in this context.
        //3. 准备工厂的类加载器等信息。设置 aware 类型的后置处理器，注册默认的环境 bean 到工厂中的一级缓存。
        prepareBeanFactory(beanFactory);
        try {
            // Allows post-processing of the bean factory in context subclasses.
            //4. 处理 BeanFactoryPostProcessor 后置处理器流程
            postProcessBeanFactory(beanFactory);
            
            // Invoke factory processors registered as beans in the context.
            //5. 调用执行后置处理器 BeanDefinitionRegistryPostProcessors, BeanFactoryPostProcessors 
            invokeBeanFactoryPostProcessors(beanFactory);
            
            // Register bean processors that intercept bean creation.
            //6. 注册 BeanPostProcessors 后置处理器
            registerBeanPostProcessors(beanFactory);
            
            //7. Initialize message source for this context.初始化国际化的一直资源
            initMessageSource();
            
            //8. Initialize event multicaster for this context.初始化事件多播器
            initApplicationEventMulticaster();
            
            //9. Initialize other special beans in specific context subclasses.留给子类的扩展点
            onRefresh();
            
            //10. Check for listener beans and register them.注册监听到多播器中
            registerListeners();
            
            // Instantiate all remaining (non-lazy-init) singletons.
            //11. 初始化所有的非懒加载单例 bean，重要
            finishBeanFactoryInitialization(beanFactory);
            
            //12. Last step: publish corresponding event.最后发布一些事件等
            finishRefresh();
        }
        catch (BeansException ex) {
            if (logger.isWarnEnabled()) {
                logger.warn("Exception encountered during context initialization - " + "cancelling refresh attempt: " + ex);
            }
            // Destroy already created singletons to avoid dangling resources.
            destroyBeans();
            // Reset 'active' flag.
            cancelRefresh(ex);
            // Propagate exception to caller.
            throw ex;
        }
        finally {
            // Reset common introspection caches in Spring's core, since we
            // might not ever need metadata for singleton beans anymore...
            resetCommonCaches();
        }
    }
}
```

---

<center>🌵</center>