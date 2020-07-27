# 如果你每次面试前都要去背一篇Spring中Bean的生命周期，请看完这篇文章

[https://mp.weixin.qq.com/s?\_\_biz=MzI3NzE0NjcwMg==&mid=2650137013&idx=4&sn=548dc6916b6951edb358a300fb8dd46a&chksm=f36bfc94c41c7582b3935d43d6b26f133a5d435cf943c9bed229ea2ffd011f8aa57806b2aee1&mpshare=1&scene=1&srcid=0727b1iOlYJsMvVImcJlsgDW&sharer\_sharetime=1595820922969&sharer\_shareid=393f249533d421d13c2402bd43e74356\#rd](https://mp.weixin.qq.com/s?__biz=MzI3NzE0NjcwMg==&mid=2650137013&idx=4&sn=548dc6916b6951edb358a300fb8dd46a&chksm=f36bfc94c41c7582b3935d43d6b26f133a5d435cf943c9bed229ea2ffd011f8aa57806b2aee1&mpshare=1&scene=1&srcid=0727b1iOlYJsMvVImcJlsgDW&sharer_sharetime=1595820922969&sharer_shareid=393f249533d421d13c2402bd43e74356#rd)

**前言**

当你准备去复习Spring中Bean的生命周期的时候，这个时候你开始上网找资料，很大概率会看到下面这张图：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/27/640-20200727115901186-115901.jpg)

先不论这张图上是否全面，但是就说这张图吧，你是不是背了又忘，忘了又背？

究其原因在于，你没有理解为什么需要这些步骤，也不知道为什么要按这个顺序执行

笔者在阅读完整个`IOC`跟`AOP`的源码后，希望通过这篇文章讲一讲我的Spring中Bean生命周期的看法，帮助大家能理解性的记忆整个流程，而不是死记硬背！

## **基础知识补充**

所谓理解也是建立在有一定知识储备的基础上的，所以这里先补充一些基础概念

### **Bean创建的三个阶段**

Spring在创建一个Bean时是分为三个步骤的

* 实例化，可以理解为new一个对象
* 属性注入，可以理解为调用setter方法完成属性注入
* 初始化，你可以按照Spring的规则配置一些初始化的方法（例如，`@PostConstruct`注解）

### **生命周期的概念**

Bean的生命周期指的就是在上面三个步骤中后置处理器`BeanPostprocessor`穿插执行的过程

### **后置处理器的分析**

按照实现接口进行分类

1. 直接实现了`BeanPostProcessor`接口

最简单的后置处理器，也就是说直接实现了`BeanPostProcessor`接口，这种后置处理器只能在初始化前后执行

```text
public interface BeanPostProcessor {

 // 初始化前执行的方法
 @Nullable
 default Object postProcessBeforeInitialization(Object bean, String beanName) throws BeansException {
  return bean;
 }    

 // 初始化后执行的方法
 default Object postProcessAfterInitialization(Object bean, String beanName) throws BeansException {
  return bean;
 }

}
```

1. 直接实现了`InstantiationAwareBeanPostProcessor`接口

在第一种后置处理的基础上进行了一层扩展，可以在Bean的实例化阶段前后执行

```text
// 继承了BeanPostProcessor，额外提供了两个方法用于在实例化前后的阶段执行
// 因为实例化后紧接着就要进行属性注入，所以这个接口中还提供了一个属性注入的方法
public interface InstantiationAwareBeanPostProcessor extends BeanPostProcessor {

    // 实例化前执行
 default Object postProcessBeforeInstantiation(Class<?> beanClass, String beanName) throws BeansException {
  return null;
 }

    // 实例化后置
 default boolean postProcessAfterInstantiation(Object bean, String beanName) throws BeansException {
  return true;
 }

    // 属性注入
    default PropertyValues postProcessPropertyValues(
        PropertyValues pvs, PropertyDescriptor[] pds, Object bean, String beanName) throws BeansException {

        return pvs;
    }
}
```

1. Spring内部专用的后置处理器

可能有的小伙伴认为，第三种后置处理器肯定就是用来在属性注入前后执行了的吧。我只能说，大兄弟，太天真了，看看下面这张图

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/27/640-20200727115901258-115901.jpg)

这种情况下再为属性注入阶段专门提供两个方法是不是有点多余呢？实际上第三种后置处理器是Spring为了自己使用而专门设计的

```text
public interface SmartInstantiationAwareBeanPostProcessor extends InstantiationAwareBeanPostProcessor {

    // 推测bean的类型，例如在属性注入阶段我们就需要知道符合依赖类型的Bean有哪些
    @Nullable
    default Class<?> predictBeanType(Class<?> beanClass, String beanName) throws BeansException {
        return null;
    }

    // 推断出所有符合要求的构造函数，在实例化对象的时候我们就需要明确到底使用哪个构造函数
    @Nullable
    default Constructor<?>[] determineCandidateConstructors(Class<?> beanClass, String beanName)
        throws BeansException {

        return null;
    }

    // 获取一个提前暴露的对象，用于解决循环依赖
    default Object getEarlyBeanReference(Object bean, String beanName) throws BeansException {
        return bean;
    }

}
```

一般我们在探究生命周期的时候都不会考虑这种后置处理器的执行

## 

## **生命周期详细介绍**

在了解了上面的概念后，我们再来看看这张图

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/27/640-20200727115901186-115901.jpg)

至少现在这张图上缺少了实例化前后后置处理器的执行流程，对吧？

再补充上这一点之后，我们再来看看，属性注入后紧接着已经是初始化的阶段，在初始化阶段开始前应该要调用`BeanPostProcessor`的预初始化方法（`postProcessBeforeInitialization`），然后调用自定义的初始化方法，最后调用`postProcessAfterInitialization`

这是没有问题，但是为什么要在初始前还要调用Aware接口的方法，如果你看了源码的话可能会说，源码就是这么写的，别人就是这么设计的，但是为什么要这么设计呢？**我们看源码到一定阶段后不应该仅仅停留在是什么的阶段，而应该多思考为什么，这样能帮助你更好的了解这个框架**

那么为什么Aware接口非要在初始化前执行呢？

这样做的目的是因为，初始化可能会依赖Aware接口提供的状态，例如下面这个例子

```text
@Component
public class A implements InitializingBean, ApplicationContextAware {

    ApplicationContext applicationContext;

    @Override
    public void afterPropertiesSet() throws Exception {
        // 初始化方法需要用到ApplicationContextAware提供的ApplicationContext
        System.out.println(applicationContext);
    }

    @Override
    public void setApplicationContext(ApplicationContext applicationContext) throws BeansException {
        this.applicationContext = applicationContext;
    }
}
```

这种情况下Aware接口当然要在初始化前执行啦！

另外，在讨论Bean的初始化的时候经常会碰到下面这个问题，`@PostConstruct`,`afterPropertiesSet`跟XML中配置的`init-method`方法的执行顺序。

请注意，`@PostConstruct`实际上是在`postProcessBeforeInitialization`方法中处理的，严格来说它不属于初始化阶段调用的方法，所以这个方法是最先调用的

其次我们思考下是调用`afterPropertiesSet`方法的开销大还是执行配置文件中指定名称的初始化方法开销大呢？我们不妨用伪代码演示下

```text
// afterPropertiesSet，强转后直接调用
((InitializingBean) bean).afterPropertiesSet()

// 反射调用init-method方法
// 第一步：找到这个方法
Method method = class.getMethod(methodName)
// 第二步：反射调用这个方法
method.invoke(bean,null)
```

相比而言肯定是第一种的效率高于第二种，一个只是做了一次方法调用，而另外一个要调用两次反射。

因此，`afterPropertiesSet`的优先级高于XML配置的方式

所以，这三个方法的执行顺序为：

1. `@PostConstruct`注解标注的方法
2. 实现了`InitializingBean`接口后复写的`afterPropertiesSet`方法
3. XML中自定义的初始化方法

在完成初始化，没什么好说的了，最后调用一下`postProcessAfterInitialization`，整个Bean的生命周期到此结束

## **总结**

本文的主要目的是想要帮助大家更好的理解整个Bean的生命周期，不过理解是建立在有一定知识存储的基础上的

你至少要对Bean的后置处理器跟Bean创建有一个大概的理解，那么通过本文你能理清一些细节方面的东西

例如，为什么Aware接口执行在初始化阶段之前？为什么初始化的三个方法会按

`@PostConstruct`，`afterPropertiesSet`，XML中定义的初始化方法这个顺序执行。

