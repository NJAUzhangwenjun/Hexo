---
title: Spring源码-BeanDefinition
top: false
cover: true
author: 张文军
date: 2021-06-28 13:04:41
tags: 
 - Spring
 - Spring源码
category: 
 - Spring源码
 - Spring
summary: Spring源码-BeanDefinition
---
<center>更多内容请关注：</center>

![Java快速开发学习](https://zhangwenjun-1258908231.cos.ap-nanjing.myqcloud.com/njauit/1586869254.png)

<center><a href="https://wjhub.gitee.io">锁清秋</a></center>

----
## BeanDefinition 的定义

### 1. BeanDefinition 是什么？

1. BeanDefinition包含了我们对bean做的配置，比如XML<bean/>标签的形式进行的配置

2. 换而言之，Spring将我们对bean的定义信息进行了抽象，抽象后的实体就是BeanDefinition（将Bean的类信息封装成BenaDefinition对象）,**并且Spring会以此作为标准来对Bean进行创建**

### 2. BeanDefinition包含的元数据有哪些？

- BeanDefinition包含以下元数据：一个全限定类名，通常来说，就是对应的bean的全限定类名。
- bean的行为配置元素，这些元素展示了这个bean在容器中是如何工作的包括scope(作用域)，lifecycle callbacks(生命周期回调)等等
- 这个bean的依赖信息。
- 一些其他配置信息，比如我们配置了一个连接池对象，那么我们还会配置它的池子大小，最大连接数等等。

| Property                 | Explained in…                                                |
| :----------------------- | :----------------------------------------------------------- |
| Class                    | [Instantiating Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-class) |
| Name                     | [Naming Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-beanname) |
| Scope                    | [Bean Scopes](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-scopes) |
| Constructor arguments    | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Properties               | [Dependency Injection](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-collaborators) |
| Autowiring mode          | [Autowiring Collaborators](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-autowire) |
| Lazy initialization mode | [Lazy-initialized Beans](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lazy-init) |
| Initialization method    | [Initialization Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-initializingbean) |
| Destruction method       | [Destruction Callbacks](https://docs.spring.io/spring-framework/docs/current/reference/html/core.html#beans-factory-lifecycle-disposablebean) |

### 3. 创建Bean的方式比较

#### **正常的创建一个java bean:**

![image-20210627020045040](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702024825.png)



#### **Spring通过BeanDefinition来创建bean:**

![image-20210627020154006](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025351.png)

通过上面的比较，可以发现，相比于正常的对象的创建过程，Spring对其管理的bean没有直接采用new的方式，而是先通过解析配置数据以及根据对象本身的一些定义而获取其对应的beandefinition,并将这个beandefinition作为之后创建这个bean的依据。同时Spring在这个过程中提供了一些扩展点，例如我们在图中所提到了BeanfactoryProcessor（BeanFactory后置处理器）。



## BeanDefinition分析

### 1. BeanDefinition接口类图

![BeanDefinition](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025352.png)

![BeanDefinition类图结构](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025353.png)



### 2. BeanDefinition的方法分析

```java
    // 获取父BeanDefinition,主要用于合并，下节中会详细分析
    String getParentName();

    // 对于的bean的ClassName
    void setBeanClassName(@Nullable String beanClassName);

    // Bean的作用域，不考虑web容器，主要两种，单例/原型，见官网中1.5内容
    void setScope(@Nullable String scope);

    // 是否进行懒加载
    void setLazyInit(boolean lazyInit);

    // 是否需要等待指定的bean创建完之后再创建
    void setDependsOn(@Nullable String... dependsOn);

    // 是否作为自动注入的候选对象
    void setAutowireCandidate(boolean autowireCandidate);

    // 是否作为主选的bean
    void setPrimary(boolean primary);

    // 创建这个bean的类的名称
    void setFactoryBeanName(@Nullable String factoryBeanName);

    // 创建这个bean的方法的名称
    void setFactoryMethodName(@Nullable String factoryMethodName);

    // 构造函数的参数
    ConstructorArgumentValues getConstructorArgumentValues();

    // setter方法的参数
    MutablePropertyValues getPropertyValues();

    // 生命周期回调方法，在bean完成属性注入后调用
    void setInitMethodName(@Nullable String initMethodName);

    // 生命周期回调方法，在bean被销毁时调用
    void setDestroyMethodName(@Nullable String destroyMethodName);

    // Spring可以对bd设置不同的角色,了解即可，不重要
    // 用户定义 int ROLE_APPLICATION = 0;
    // 某些复杂的配置    int ROLE_SUPPORT = 1;
    // 完全内部使用   int ROLE_INFRASTRUCTURE = 2;
    void setRole(int role);

    // bean的描述，没有什么实际含义
    void setDescription(@Nullable String description);

    // 根据scope判断是否是单例
    boolean isSingleton();

    // 根据scope判断是否是原型
    boolean isPrototype();

    // 跟合并beanDefinition相关，如果是abstract，说明会被作为一个父beanDefinition，不用提供class属性
    boolean isAbstract();

    // bean的源描述，没有什么实际含义
    String getResourceDescription();

    // cglib代理前的BeanDefinition
    org.springframework.beans.factory.config.BeanDefinition getOriginatingBeanDefinition();

```

![image-20210627130538339](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025354.png)



### 3. BeanDefinition的继承关系:

![image-20210627132240116](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025355.png)

#### 1. BeanDefinition继承的接口：



- org.springframework.core.AttributeAccessor

  由接口上注释的Java doc可知

  ```java
  //Interface defining a generic contract for attaching and accessing metadata to/from arbitrary objects.
  ```

这个接口为从其它任意类中获取或设置元数据提供了一个通用的规范。

其实这就是访问者模式的一种体现，采用这方方法，可以将**数据接口**跟**操作方法**进行分离。

再来看这个接口中定义的方法：

```java
void setAttribute(String name, @Nullable Object value);

Object getAttribute(String name);

Object removeAttribute(String name);

boolean hasAttribute(String name);

String[] attributeNames();
```

就是提供了一个获取属性跟设置属性的方法

那么现在问题来了，在整个BeanDefiniton体系中，这个被操作的**数据结构**在哪呢？不要急，在后文中的AbstractBeanDefinition会介绍。



- org.springframework.beans.BeanMetadataElement

  由接口注释的Java doc可知

  ```java
  //Interface to be implemented by bean metadata elements that carry a configuration source object.
  ```

这个接口提供了一个方法去获取配置源对象，其实就是原文件。

这个接口只提供了一个方法：

```java
@Nullable
Object getSource();
```

可以理解为，当通过注解的方式定义了一个IndexService时，那么此时的IndexService对应的BeanDefinition通过getSource方法返回的就是IndexService.class这个文件对应的一个File对象。

如果我们通过@Bean方式定义了一个IndexService的话，那么此时的source是被@Bean注解所标注的一个Mehthod对象。

### 4. AbstractBeanDefinition

#### 1. **AbstractBeanDefinition的继承关系：**

![image-20210627140455666](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025356.png)



- org.springframework.core.AttributeAccessorSupport

  可以看到这个类实现了AttributeAccerror接口，我们在上文中已经提到过，AttributeAccerror采用了**访问者**的设计模式，将**数据结构**跟**操作方法**进行了分离，数据结构在哪呢？就在AttributeAccessorSupport这个类中，我们看下它的代码：

  ```java
  public abstract class AttributeAccessorSupport implements AttributeAccessor, Serializable {
  
  	/** Map with String keys and Object values. */
  	private final Map<String, Object> attributes = new LinkedHashMap<>();
  
  
  	@Override
  	public void setAttribute(String name, @Nullable Object value) {
  		Assert.notNull(name, "Name must not be null");
  		if (value != null) {
  			this.attributes.put(name, value);
  		}
  		else {
  			removeAttribute(name);
  		}
  	}
      //......省略下面的代
  ```

可以看到，在这个类中，维护了一个map，这就是BeanDefinition体系中，通过访问者模式所有操作的数据对象。



- org.springframework.beans.BeanMetadataAttributeAccessor

  这个类主要就是对上面的map中的数据操作做了更深一层的封装，下面就看其中的两个方法：

  ```java
  public void addMetadataAttribute(BeanMetadataAttribute attribute) {
      super.setAttribute(attribute.getName(), attribute);
  }
  public BeanMetadataAttribute getMetadataAttribute(String name) {
      return (BeanMetadataAttribute) super.getAttribute(name);
  }
  ```

  可以发现，它只是将属性统一封装成了一个BeanMetadataAttribute,然后就调用了父类的方法，将其放入到map中。

  我们的AbstractBeanDefinition通过继承了BeanMetadataAttributeAccessor这个类，可以对BeanDefinition中的属性进行操作。这里说的属性仅仅指的是BeanDefinition中的一个map，而不是它的其它字段。


  #### 2. 为什么需要AbstractBeanDefinition？

  对比BeanDefinition的源码我们可以发现，AbstractBeanDefinition对BeanDefinition的大部分方法做了实现（没有实现parentName相关方法）。同时定义了一系列的常量及默认字段。这是因为BeanDefinition接口过于顶层，如果我们依赖BeanDefinition这个接口直接去创建其实现类的话过于麻烦，所以通过AbstractBeanDefinition做了一个下沉，并给很多属性赋了默认值，例如：

```java
// 默认情况不是懒加载的
private boolean lazyInit = false;
// 默认情况不采用自动注入
private int autowireMode = AUTOWIRE_NO;
// 默认情况作为自动注入的候选bean
private boolean autowireCandidate = true;
// 默认情况不作为优先使用的bean
private boolean primary = false;
........
```

  这样可以方便我们创建其子类，如接下来的：ChildBeanDefinition,RootBeanDefinition等等



### 5. AbstractBeanDefinition的三个子类

![image-20210627144249811](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025357.png)



#### 1. GenericBeanDefinition:

- 替代了原来的ChildBeanDefinition，比起ChildBeanDefinition更为灵活，ChildBeanDefinition在实例化的时候必须要指定一个parentName,而GenericBeanDefinition不需要。我们通过注解配置的bean以及我们的配置类（除@Bena外）的BeanDefiniton类型都是GenericBeanDefinition。

#### 2. ChildBeanDefinition

- 现在已经被GenericBeanDefinition所替代了。在5.1.x版本没有找到使用这个类的代码。

#### 3. RootBeanDefinition

- Spring在启动时会实例化几个初始化的BeanDefinition,这几个BeanDefinition的类型都为RootBeanDefinition
- Spring在合并BeanDefinition返回的都是RootBeanDefinition
- 我们通过@Bean注解配置的bean，解析出来的BeanDefinition都是RootBeanDefinition（实际上是其子类ConfigurationClassBeanDefinition）

### 6. AnnotatedBeanDefinition

![image-20210627150544175](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025358.png)

AnnotatedBeanDefinition 接口扩展了BeanDefinition 添加新的功能:获取bean的注解元数据以及工厂方法的元数据。

```java
public interface AnnotatedBeanDefinition extends BeanDefinition {

    //获取bean的注解元数据
   AnnotationMetadata getMetadata();

    //工厂方法的元数据
   @Nullable
   MethodMetadata getFactoryMethodMetadata();

}
```

#### AnnotatedBeanDefinition的三个实现类



##### 1. AnnotatedGenericBeanDefinition:

- 通过形如下面的API注册的bean都是AnnotatedGenericBeanDefinition

```java
public static void main(String[] args) {
    AnnotationConfigApplicationContext ac = new AnnotationConfigApplicationContext();
    ac.register(Config.class);
}
```

这里的config对象，最后在Spring容器中就是一个AnnotatedGenericBeanDefinition。

- 通过@Import注解导入的类，最后都是解析为AnnotatedGenericBeanDefinition。

##### 2. ScannedGenericBeanDefinition:

- 都过注解扫描的类，如@Service,@Compent等方式配置的Bean都是ScannedGenericBeanDefinition

##### 3. ConfigurationClassBeanDefinition:

- 通过@Bean的方式配置的Bean为ConfigurationClassBeanDefinition



## BeanDefinition 小总结：



1. 什么是BeanDefinition，总结起来就是一句话，Spring创建bean时的建模对象。

2. BeanDefinition的具体使用的子类，以及Spring在哪些地方使用到了它们。画图总结如下：

   ![image-20210627155915148](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025359.png)

   

   ![image-20210627163124891](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025400.png)

   

## BeanDefinition的合并

   

在BeanDefinition众多属性中，有一下两个属性是和合并BeanDefinition

```java
//  是否抽象
boolean isAbstract();
// 获取父BeanDefinition的名称
String getParentName();
```



### 1. 什么是合并？

![image-20210627171709384](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025401.png)

一个BeanDefinition包含了很多的配置信息，包括构造参数，setter方法的参数还有容器特定的一些配置信息，比如初始化方法，静态工厂方法等等。一个子的BeanDefinition可以从它的父BeanDefinition继承配置信息，不仅如此，还可以覆盖其中的一些值或者添加一些自己需要的属性。使用BeanDefinition的父子定义可以减少很多的重复属性的设置，父BeanDefinition可以作为BeanDefinition定义的模板。

通过一个例子来观察下合并发生了什么，编写一个Demo如下：

```java
<?xml version="1.0" encoding="UTF-8"?>
<beans xmlns="http://www.springframework.org/schema/beans"
       xmlns:xsi="http://www.w3.org/2001/XMLSchema-instance"
       xsi:schemaLocation="http://www.springframework.org/schema/beans http://www.springframework.org/schema/beans/spring-beans.xsd">
    <bean id="parent" abstract="true"
          class="com.dmz.official.merge.TestBean">
        <property name="name" value="parent"/>
        <property name="age" value="1"/>
    </bean>
    <bean id="child"
          class="com.dmz.official.merge.DerivedTestBean"
          parent="parent" >
        <property name="name" value="override"/>
    </bean>
</beans>
public class DerivedTestBean {
    private String name;

    private int age;

    // 省略getter setter方法
}

public class TestBean {
    private String name;

    private String age;

     // 省略getter setter方法
}

public class Main {
    public static void main(String[] args) {
        ClassPathXmlApplicationContext cc = new ClassPathXmlApplicationContext("application.xml");
        DerivedTestBean derivedTestBean = (DerivedTestBean) cc.getBean("child");
        System.out.println("derivedTestBean的name = " + derivedTestBean.getName());
        System.out.println("derivedTestBean的age = " + derivedTestBean.getAge());
    }
}
```

运行：

------

derivedTestBean的name = override

derivedTestBean的age = 1

------

在上面的例子中，**我们将DerivedTestBean的parent属性设置为了parent,指向了我们的TestBean，同时将TestBean的age属性设置为1**，但是我们在配置文件中并没有直接设置DerivedTestBean的age属性。但是在最后运行结果，我们可以发现，DerivedTestBean中的age属性已经有了值，并且为1，就是我们在其parent Bean（也就是TestBean）中设置的值。也就是说，**子BeanDefinition会从父BeanDefinition中继承没有的属性**。另外，DerivedTestBean跟TestBean都指定了name属性，但是可以发现，这个值并没有被覆盖掉，也就是说，**子BeanDefinition中已经存在的属性不会被父BeanDefinition中所覆盖**。

**BeanDefinition合并：**

- **子BeanDefinition会从父BeanDefinition中继承没有的属性**
- **子BeanDefinition中已经存在的属性不会被父BeanDefinition中所覆盖**



### 2. 合并注意细节

- 子BeanDefinition中的class属性如果为null，同时父BeanDefinition又指定了class属性，那么子BeanDefinition也会继承这个class属性。
- 子BeanDefinition必须要兼容父BeanDefinition中的所有属性。这是什么意思呢？以我们上面的demo为例，我们在父BeanDefinition中指定了name跟age属性，但是如果子BeanDefinition中子提供了一个name的setter方法，这个时候Spring在启动的时候会报错。因为子BeanDefinition不能承接所有来自父BeanDefinition的属性
- 关于BeanDefinition中abstract属性的说明：并不是作为父BeanDefinition就一定要设置abstract属性为true，abstract只代表了这个BeanDefinition是否要被Spring进行实例化并被创建对应的Bean，如果为true，代表容器不需要去对其进行实例化。如果一个BeanDefinition被当作父BeanDefinition使用，并且没有指定其class属性。那么必须要设置其abstract为true ；abstract=true 一般会跟父BeanDefinition一起使用，因为当我们设置某个BeanDefinition的abstract=true时，一般都是要将其当作BeanDefinition的模板使用，否则这个BeanDefinition也没有意义，除非我们使用其它BeanDefinition来继承它的属性。

### 3. Spring在哪些阶段做了合并？

> **下文将所有BeanDefinition简称为bd**

#### 1、扫描并获取到bd：

这个阶段的操作主要发生在invokeBeanFactoryPostProcessors，对应方法的调用栈如下：

![image-20210627182558975](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025402.png)



```java
public static void invokeBeanFactoryPostProcessors(ConfigurableListableBeanFactory beanFactory,
                                                   List<BeanFactoryPostProcessor> beanFactoryPostProcessors) {
    // .....
    // 省略部分代码，省略的代码主要时用来执行程序员手动调用API注册的容器的后置处理器
    // .....

    // 发生一次bd的合并
    // 这里只会获取实现了BeanDefinitionRegistryPostProcessor接口的Bean的名字
    String[] postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
    for (String ppName : postProcessorNames) {
        // 筛选实现了PriorityOrdered接口的后置处理器
        if (beanFactory.isTypeMatch(ppName, PriorityOrdered.class)) {
            currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
            // 去重
            processedBeans.add(ppName);
        }
    }
    // .....
    // 只存在一个internalConfigurationAnnotationProcessor 处理器，用于扫描
    // 这里只会执行了实现了PriorityOrdered跟BeanDefinitionRegistryPostProcessor的后置处理器
    invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
    // .....
    // 这里又进行了一个bd的合并
    postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
    for (String ppName : postProcessorNames) {
        // 筛选实现了Ordered接口的后置处理器
        if (!processedBeans.contains(ppName) && beanFactory.isTypeMatch(ppName, Ordered.class)) {
            currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
            processedBeans.add(ppName);
        }
    }
    // .....
    // 执行的是实现了BeanDefinitionRegistryPostProcessor接口跟Ordered接口的后置处理器
    invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
    boolean reiterate = true;
    while (reiterate) {
        reiterate = false;
        // 这里再次进行了一次bd的合并
        postProcessorNames = beanFactory.getBeanNamesForType(BeanDefinitionRegistryPostProcessor.class, true, false);
        for (String ppName : postProcessorNames) {
            if (!processedBeans.contains(ppName)) {
                // 筛选只实现了BeanDefinitionRegistryPostProcessor的后置处理器
                currentRegistryProcessors.add(beanFactory.getBean(ppName, BeanDefinitionRegistryPostProcessor.class));
                processedBeans.add(ppName);
                reiterate = true;
            }
        }
        sortPostProcessors(currentRegistryProcessors, beanFactory);
        registryProcessors.addAll(currentRegistryProcessors);
        // 执行的是普通的后置处理器，即没有实现任何排序接口（PriorityOrdered或Ordered)
        invokeBeanDefinitionRegistryPostProcessors(currentRegistryProcessors, registry);
        currentRegistryProcessors.clear();
    }
    // .....
    // 省略部分代码，这部分代码跟BeanfactoryPostProcessor接口相关，这节bd的合并无关，下节容器的扩展点中我会介绍
    // .....

}
```

![image-20210627183341216](https://myblog-1258908231.cos.ap-shanghai.myqcloud.com/hexo/20210702025403.png)

大家可以结合我画的图跟上面的代码过一遍流程，只要弄清楚一点就行，即每次调用beanFactory.getBeanNamesForType都进行了一次bd的合并。getBeanNamesForType这个方法主要目的是为了获取指定类型的bd的名称，之后通过bd的名称去找到指定的bd，然后获取对应的Bean，比如上面方法三次获取的都是BeanDefinitionRegistryPostProcessor这个类型的bd。

#### 2、实例化

Spring在实例化一个对象也会进行bd的合并。

第一次：

org.springframework.beans.factory.support.DefaultListableBeanFactory#preInstantiateSingletons

```javascript
public void preInstantiateSingletons() throws BeansException {
    // .....
    // 省略跟合并无关的代码
    for (String beanName : beanNames) {
        RootBeanDefinition bd = getMergedLocalBeanDefinition(beanName);
        if (!bd.isAbstract() && bd.isSingleton() && !bd.isLazyInit()) {
            // .....
```

第二次：

org.springframework.beans.factory.support.AbstractBeanFactory#doGetBean

```javascript
protected <T> T doGetBean(final String name, @Nullable final Class<T> requiredType,
                          @Nullable final Object[] args, boolean typeCheckOnly) throws BeansException {
    // .....
    // 省略跟合并无关的代码
    final RootBeanDefinition mbd = getMergedLocalBeanDefinition(beanName);
    checkMergedBeanDefinition(mbd, beanName, args);

    // Guarantee initialization of beans that the current bean depends on.
    String[] dependsOn = mbd.getDependsOn();
    if (dependsOn != null) {
        // ....
    }
    if (mbd.isSingleton()) {
        // ....
    }
    // ....
```

我们可以发现这两次合并有一个共同的特点，就是在**合并之后立马利用了合并之后的bd我们简称为mbd做了一系列的判断**，比如上面的dependsOn != null和mbd.isSingleton()。基于上面几个例子我们来分析：为什么需要合并？

### 4. 为什么需要合并？

在扫描阶段，之所以发生了合并，是因为Spring需要拿到指定了实现了BeanDefinitionRegistryPostProcessor接口的bd的名称，也就是说，Spring需要用到bd的名称。所以进行了一次bd的合并。在实例化阶段，是因为Spring需要用到bd中的一系列属性做判断所以进行了一次合并。我们总结起来，其实就是一个原因：**Spring需要用到bd的属性，要保证获取到的bd的属性是正确的**。

那么问题来了，为什么获取到的bd中属性可能不正确呢？

主要两个原因：

1. 作为子bd,属性本身就有可能缺失，比如我们在开头介绍的例子，子bd中本身就没有age属性，age属性在父bd中
2. Spring提供了很多扩展点，在启动容器的时候，可能会修改bd中的属性。比如一个正常实现了BeanFactoryPostProcessor就能修改容器中的任意的bd的属性。在后面的容器的扩展点中我再介绍

#### 1. 合并的代码分析：

因为合并的代码其实很简单，所以一并在这里分析了，也可以加深对合并的理解：

```javascript
protected RootBeanDefinition getMergedLocalBeanDefinition(String beanName) throws BeansException {
    // Quick check on the concurrent map first, with minimal locking.
    // 从缓存中获取合并后的bd
    RootBeanDefinition mbd = this.mergedBeanDefinitions.get(beanName);
    if (mbd != null) {
        return mbd;
    }
    // 如何获取不到的话，开始真正的合并
    return getMergedBeanDefinition(beanName, getBeanDefinition(beanName));
}
    protected RootBeanDefinition getMergedBeanDefinition(
            String beanName, BeanDefinition bd, @Nullable BeanDefinition containingBd)
            throws BeanDefinitionStoreException {

        synchronized (this.mergedBeanDefinitions) {
            RootBeanDefinition mbd = null;

            // Check with full lock now in order to enforce the same merged instance.
            if (containingBd == null) {
                mbd = this.mergedBeanDefinitions.get(beanName);
            }

            if (mbd == null) {
                // 如果没有parentName的话直接使用自身合并
                // 就是new了RootBeanDefinition然后再进行属性的拷贝
                if (bd.getParentName() == null) {
                    if (bd instanceof RootBeanDefinition) {
                        mbd = ((RootBeanDefinition) bd).cloneBeanDefinition();
                    }
                    else {   
                        mbd = new RootBeanDefinition(bd);
                    }
                }
                else {
                    // 需要进行父子的合并
                    BeanDefinition pbd;
                    try {
                        String parentBeanName = transformedBeanName(bd.getParentName());
                        if (!beanName.equals(parentBeanName)) {
                            // 这里是递归，在将父子合并时，需要确保父bd已经合并过了
                            pbd = getMergedBeanDefinition(parentBeanName);
                        }
                        else {
                            // 一般不会进这个判断
                            // 到父容器中找对应的bean，然后进行合并，合并也发生在父容器中
                            BeanFactory parent = getParentBeanFactory();
                            if (parent instanceof ConfigurableBeanFactory) {
                                pbd = ((ConfigurableBeanFactory) parent).getMergedBeanDefinition(parentBeanName);
                            }
                            // 省略异常信息......
                        }
                    }
                    // 省略异常信息......
                    // 
                    mbd = new RootBeanDefinition(pbd);
                    //用子bd中的属性覆盖父bd中的属性
                    mbd.overrideFrom(bd);
                }

                // 默认设置为单例
                if (!StringUtils.hasLength(mbd.getScope())) {
                    mbd.setScope(RootBeanDefinition.SCOPE_SINGLETON);
                }
                // 当前bd如果内部嵌套了一个bd,并且嵌套的bd不是单例的，但是当前的bd又是单例的
                // 那么将当前的bd的scope设置为嵌套bd的类型
                if (containingBd != null && !containingBd.isSingleton() && mbd.isSingleton()) {
                    mbd.setScope(containingBd.getScope());
                }
                // 将合并后的bd放入到mergedBeanDefinitions这个map中
                // 之后还是可能被清空的，因为bd可能被修改
                if (containingBd == null && isCacheBeanMetadata()) {
                    this.mergedBeanDefinitions.put(beanName, mbd);
                }
            }

            return mbd;
        }
    }
```

上面这段代码整体不难理解，可能发生疑惑的主要是两个点：

1. pbd = getMergedBeanDefinition(parentBeanName);

这里进行的是父bd的合并，是方法的递归调用，这是因为在合并的时候父bd可能也还不是一个合并后的bd

1. containingBd != null && !containingBd.isSingleton() && mbd.isSingleton()

我查了很久的资料，经过验证后发现，如果进行了形如下面的嵌套配置，那么containingBd会不为null

```javascript
<bean id="luBanService" class="com.dmz.official.service.LuBanService" scope="prototype">
    <property name="lookUpService">
        <bean class="com.dmz.official.service.LookUpService" scope="singleton"></bean>
    </property>
</bean>
```

在这个例子中，containingBd为LuBanService，此时，LuBanService是一个原型的bd，但lookUpService是一个单例的bd，那么这个时候经过合并，LookUpService也会变成一个原型的bd。大家可以拿我这个例子测试一下。

### BeanDefinition合并小结：

这篇文章我觉得最重要的是，我们要明白Spring为什么要进行合并，之所以再每次需要用到BeanDefinition都进行一次合并，是为了每次都拿到最新的，最有效的BeanDefinition，因为利用容器提供了一些扩展点我们可以修改BeanDefinition中的属性。关于容器的扩展点，比如上文提到了BeanFactoryPostProcessor以及BeanDefinitionRegistryPostProcessor,我会在后面的几篇文章中一一介绍。

BeanDefinition的学习就到这里了，这个类很重要，是整个Spring的基石，希望大家可以多花时间多研究研究相关的知识。加油，共勉！	