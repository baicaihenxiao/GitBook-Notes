# Spring Boot 中 @EnableXXX 注解的驱动逻辑探讨

[https://mp.weixin.qq.com/s/oVVoghnf721OzUui6GXjQA](https://mp.weixin.qq.com/s/oVVoghnf721OzUui6GXjQA)

作者 \| 温安适

来源 \| [https://juejin.im/post/5efdd689e51d4534af686ca9](https://juejin.im/post/5efdd689e51d4534af686ca9)

工作中经常用到，如下注解：

* @EnableEurekaClient
* @EnableFeignClients
* @EnableCircuitBreaker
* @EnableHystrix

他们都是@Enable开头，各自实现不同的功能，解析这种@Enable的逻辑是什么呢？

## @Enable驱动逻辑

### 找入口

@Enable的模块驱动，依赖于@Import实现。

@Import作用是装载导入类，主要包括@Configuration class,ImportSelector实现类,ImportBeanDefinitionRegistrar实现类。

XML时代，经常是`@Import`,`<context:component-scan>`,`<context:annotation-config>`一起使用。

`<context:annotation-config>`（注解配置）中大概率有我们需要找的逻辑。

根据 Spring Framework 2.0引入的可扩展的XML编程机制，XML Schema命名空间需要与Handler建立映射关系。

该关系配置在相对于classpath下的`/META-INF/spring.handlers`中。

查看ContextNamespaceHandler 源码

```text
public class ContextNamespaceHandler extends NamespaceHandlerSupport {
   @Override
   public void init() {
       //省略其他代码
    registerBeanDefinitionParser("annotation-config", 
                    new AnnotationConfigBeanDefinitionParser());
  }
}复制代码
```

 **对应AnnotationConfigBeanDefinitionParser这个就是要找的入口**

### 找核心类

从AnnotationConfigBeanDefinitionParser的parse方法开始一路向下，找到

AnnotationConfigUtils.registerAnnotationConfigProcessors中注册了ConfigurationClassPostProcessor。

img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/30/640-20200730224017986-224018.gif)

ConfigurationClassPostProcessor类注释说明

> \1. 用于的引导处理@Configuration类
>
> \2. context:annotation-config/或 context:component-scan/时会注册
>
> 否则需要手工编程
>
> \3. ConfigurationClassPostProcessor第一优先级，保证
>
> @Configuration}类中声明@Bean，在其他 BeanFactoryPostProcessor执行之前被注册

扩展

> **AnnotationConfigApplicationContext中new AnnotationBeanDefinitionReader也调用了** AnnotationConfigUtils .
>
> registerAnnotationConfigProcessors

**从类注释中，可以看出ConfigurationClassPostProcessor就是要找的核心类**

**找核心方法**

查看 ConfigurationClassPostProcessor 的层级关系为

img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/30/640-20200730224018121-224018.jpg)

Aware系列注入相应资源，Ordered设置优先级，值得关注的就是

postProcessBeanDefinitionRegistry了。

**postProcessBeanDefinitionRegistry其内部有2个方法**

1. postProcessBeanDefinitionRegistry在BeanDefinition注册之后，BeanFactoryPostProcessor执行之前，修改或重写BeanDefinition
2. 继承自BeanFactoryPostProcessor的postProcessBeanFactory，BeanDefinition加载之后，Bean实例化之前，重写或添加BeanDefinition，修改BeanFactory

浏览2个方法，都有processConfigBeanDefinitions，从名称可以看出是处理配置类Bean定义

img

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/30/640-20200730224018699-224018.gif)

**ConfigurationClassPostProcessor\#processConfigBeanDefinitions就是要找的核心方法**

### 梳理流程

```text
public void processConfigBeanDefinitions(BeanDefinitionRegistry registry) {
   List<BeanDefinitionHolder> configCandidates = new ArrayList<>();
   String[] candidateNames = registry.getBeanDefinitionNames();

   for (String beanName : candidateNames) {
      BeanDefinition beanDef = registry.getBeanDefinition(beanName);
      if (ConfigurationClassUtils.isFullConfigurationClass(beanDef) ||
            ConfigurationClassUtils.isLiteConfigurationClass(beanDef)) {
         if (logger.isDebugEnabled()) {
            logger.debug("Bean definition has already been processed as a configuration class: " + beanDef);
         }
      }
      else if (ConfigurationClassUtils.checkConfigurationClassCandidate(beanDef, this.metadataReaderFactory)) {
         configCandidates.add(new BeanDefinitionHolder(beanDef, beanName));
      }
   }

   // 没有找到 @Configuration classes 立即返回
   if (configCandidates.isEmpty()) {
      return;
   }
   //根据@Order 值进行排序
   configCandidates.sort((bd1, bd2) -> {
      int i1 = ConfigurationClassUtils.getOrder(bd1.getBeanDefinition());
      int i2 = ConfigurationClassUtils.getOrder(bd2.getBeanDefinition());
      return Integer.compare(i1, i2);
   });
   //通过封闭的应用程序上下文， 检测任何自定义bean名称生成策略
   supplied through the enclosing application context
   SingletonBeanRegistry sbr = null;
   if (registry instanceof SingletonBeanRegistry) {
      sbr = (SingletonBeanRegistry) registry;
      if (!this.localBeanNameGeneratorSet) {
         BeanNameGenerator generator = (BeanNameGenerator) sbr.getSingleton(CONFIGURATION_BEAN_NAME_GENERATOR);
         if (generator != null) {
            this.componentScanBeanNameGenerator = generator;
            this.importBeanNameGenerator = generator;
         }
      }
   }
   if (this.environment == null) {
      this.environment = new StandardEnvironment();
   }
   // 解析@Configuration class
   ConfigurationClassParser parser = new ConfigurationClassParser(
         this.metadataReaderFactory, this.problemReporter, this.environment,
         this.resourceLoader, this.componentScanBeanNameGenerator, registry);

   Set<BeanDefinitionHolder> candidates = new LinkedHashSet<>(configCandidates);
   Set<ConfigurationClass> alreadyParsed = new HashSet<>(configCandidates.size());
   do {
      parser.parse(candidates);
      parser.validate();

      Set<ConfigurationClass> configClasses = new LinkedHashSet<>(parser.getConfigurationClasses());
      configClasses.removeAll(alreadyParsed);
      // 读ConfigurationClass的信息，创建BeanDefinition
      if (this.reader == null) {
         this.reader = new ConfigurationClassBeanDefinitionReader(
               registry, this.sourceExtractor, this.resourceLoader, this.environment,
               this.importBeanNameGenerator, parser.getImportRegistry());
      }
      this.reader.loadBeanDefinitions(configClasses);
      alreadyParsed.addAll(configClasses);

      candidates.clear();
      if (registry.getBeanDefinitionCount() > candidateNames.length) {
         String[] newCandidateNames = registry.getBeanDefinitionNames();
         Set<String> oldCandidateNames = new HashSet<>(Arrays.asList(candidateNames));
         Set<String> alreadyParsedClasses = new HashSet<>();
         for (ConfigurationClass configurationClass : alreadyParsed) {
            alreadyParsedClasses.add(configurationClass.getMetadata().getClassName());
         }
         for (String candidateName : newCandidateNames) {
            if (!oldCandidateNames.contains(candidateName)) {
               BeanDefinition bd = registry.getBeanDefinition(candidateName);
               if (ConfigurationClassUtils.checkConfigurationClassCandidate(bd, this.metadataReaderFactory) &&
                     !alreadyParsedClasses.contains(bd.getBeanClassName())) {
                  candidates.add(new BeanDefinitionHolder(bd, candidateName));
               }
            }
         }
         candidateNames = newCandidateNames;
      }
   }
   while (!candidates.isEmpty());

   // 将ImportRegistry注册为bean以支持importware@Configuration类
   if (sbr != null && !sbr.containsSingleton(IMPORT_REGISTRY_BEAN_NAME)) {
      sbr.registerSingleton(IMPORT_REGISTRY_BEAN_NAME, parser.getImportRegistry());
   }
   if (this.metadataReaderFactory instanceof CachingMetadataReaderFactory) {
      // Clear cache in externally provided MetadataReaderFactory; this is a no-op
      // for a shared cache since it'll be cleared by the ApplicationContext.
      ((CachingMetadataReaderFactory) this.metadataReaderFactory).clearCache();
   }
}复制代码
```

**ConfigurationClassPostProcessor\#processConfigBeanDefinitions核心如下：**

1. 根据@Order 值进行排序
2. 解析@Configuration class 为ConfigurationClass对象
3. 读ConfigurationClass的信息，创建BeanDefinition
4. 将ImportRegistry注册为bean以支持importware@Configuration类

### 重点关注解析方法

ConfigurationClassParser\#parse方法负责解析@Configuration class 为ConfigurationClass对象

查阅其源码如下：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/gif/2020/07/30/640-20200730224019614-224019.gif)

ConfigurationClassParser\#doProcessConfigurationClass代码如下：

```text
protected final SourceClass doProcessConfigurationClass(ConfigurationClass configClass, SourceClass sourceClass)
      throws IOException {

   if (configClass.getMetadata().isAnnotated(Component.class.getName())) {
      // Recursively process any member (nested) classes first
      processMemberClasses(configClass, sourceClass);
   }

   // Process any @PropertySource annotations
   for (AnnotationAttributes propertySource : AnnotationConfigUtils.attributesForRepeatable(
         sourceClass.getMetadata(), PropertySources.class,
         org.springframework.context.annotation.PropertySource.class)) {
      if (this.environment instanceof ConfigurableEnvironment) {
         processPropertySource(propertySource);
      }
      else {
         logger.info("Ignoring @PropertySource annotation on [" + sourceClass.getMetadata().getClassName() +
               "]. Reason: Environment must implement ConfigurableEnvironment");
      }
   }

   // Process any @ComponentScan annotations
   Set<AnnotationAttributes> componentScans = AnnotationConfigUtils.attributesForRepeatable(
         sourceClass.getMetadata(), ComponentScans.class, ComponentScan.class);
   if (!componentScans.isEmpty() &&
         !this.conditionEvaluator.shouldSkip(sourceClass.getMetadata(), ConfigurationPhase.REGISTER_BEAN)) {
      for (AnnotationAttributes componentScan : componentScans) {
         // The config class is annotated with @ComponentScan -> perform the scan immediately
         Set<BeanDefinitionHolder> scannedBeanDefinitions =
               this.componentScanParser.parse(componentScan, sourceClass.getMetadata().getClassName());
         // Check the set of scanned definitions for any further config classes and parse recursively if needed
         for (BeanDefinitionHolder holder : scannedBeanDefinitions) {
            BeanDefinition bdCand = holder.getBeanDefinition().getOriginatingBeanDefinition();
            if (bdCand == null) {
               bdCand = holder.getBeanDefinition();
            }
            if (ConfigurationClassUtils.checkConfigurationClassCandidate(bdCand, this.metadataReaderFactory)) {
               parse(bdCand.getBeanClassName(), holder.getBeanName());
            }
         }
      }
   }

   // Process any @Import annotations
   processImports(configClass, sourceClass, getImports(sourceClass), true);

   // Process any @ImportResource annotations
   AnnotationAttributes importResource =
         AnnotationConfigUtils.attributesFor(sourceClass.getMetadata(), ImportResource.class);
   if (importResource != null) {
      String[] resources = importResource.getStringArray("locations");
      Class<? extends BeanDefinitionReader> readerClass = importResource.getClass("reader");
      for (String resource : resources) {
         String resolvedResource = this.environment.resolveRequiredPlaceholders(resource);
         configClass.addImportedResource(resolvedResource, readerClass);
      }
   }

   // Process individual @Bean methods
   Set<MethodMetadata> beanMethods = retrieveBeanMethodMetadata(sourceClass);
   for (MethodMetadata methodMetadata : beanMethods) {
      configClass.addBeanMethod(new BeanMethod(methodMetadata, configClass));
   }

   // Process default methods on interfaces
   processInterfaces(configClass, sourceClass);

   // Process superclass, if any
   if (sourceClass.getMetadata().hasSuperClass()) {
      String superclass = sourceClass.getMetadata().getSuperClassName();
      if (superclass != null && !superclass.startsWith("java") &&
            !this.knownSuperclasses.containsKey(superclass)) {
         this.knownSuperclasses.put(superclass, configClass);
         // Superclass found, return its annotation metadata and recurse
         return sourceClass.getSuperClass();
      }
   }

   // No superclass -> processing is complete
   return null;
}复制代码
```

**ConfigurationClassParser\#doProcessConfigurationClass\(ConfigurationClass,AnnatationMetatdata\)将@PropertySource**， **@ComponentScan**， **@Import，@ImportResource，@Bean等一起处理了。**

看到这里基本逻辑已经理清 了，但是有一个疑问

@Configuration中的@Bean没有其他特殊处理吗？

### 浏览代码解决疑问

![img](data:image/svg+xml;utf8,)

从上边浏览的代码可以看到完全模式，会被AOP增强

那什么是完全模式呢？在ConfigurationClassUtils找到如下方法：

```text
public class ConfigurationClassUtils{
//省略其他方法
public static boolean isFullConfigurationCandidate(AnnotationMetadata metadata) {
   return metadata.isAnnotated(Configuration.class.getName());
}
}复制代码
```

即 **@Configuration class是完全模式，@Component，@Bean是轻量级模式**

那AOP增强了作用是什么呢？查看 ConfigurationClassEnhancer 的类注释如下：

> /\*\*
>
> \* Enhances {@link
>
> Configuration
>
> } classes by generating a CGLIB subclass which
>
> \* interacts with the Spring container to respect bean scoping semantics for
>
> \* {@code @Bean} methods. Each such {@code @Bean} method will be overridden in
>
> \* the generated subclass, only delegating to the actual {@code @Bean} method
>
> \* implementation if the container actually requests the construction of a new
>
> \* instance. Otherwise, a call to such an {@code @Bean} method serves as a
>
> \* reference back to the container, obtaining the corresponding bean by name.
>
> * \* @author Chris Beams
>
> \* @author Juergen Hoeller
>
> \* @since 3.0
>
> \* @see \#enhance
>
> \* @see
>
> ConfigurationClassPostProcessor
>
> \*/
>
> class ConfigurationClassEnhancer {

大概意思如下：

**通过CGLIB增强@Configuration class。**

**每个@Bean方法会生成子类。**

**首次被调用时，@Bean方法会被执行用于创建bean实例；**

**再次被调用时，不会再执行创建bean实例，而是根据bean名称返回首次该方法被执行时创建的bean实例。**

## 总结

1.ConfigurationClassPostProcessor负责筛选@Component Class、@Configuration Class以及@Bean定义的Bean，

2.ConfigurationClassParser从候选的Bean定义中解析出ConfigurationClass集合，随后被3.ConfigurationClassBeanDefinitionReader转换为BeanDefinition

4.ConfigurationClassParser的解析顺序，

@PropertySource-&gt;@ComponentScan-&gt;@Import-&gt;@ImportResource-&gt;@Bean-&gt;接口的默认方法-&gt;处理父类

5.@Configuration class是完全模式，@Component，@Bean是轻量级模式

6.CGLIB增强@Configuration class。每个@Bean方法会生成子类。

首次被调用时，@Bean方法会被执行用于创建bean实例；

再次被调用时，不会再执行创建bean实例，而是根据bean名称返回首次该方法被执行时创建的bean实例。

