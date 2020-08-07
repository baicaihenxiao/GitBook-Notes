# Spring中@Component，@Service等注解如何被解析？

[https://mp.weixin.qq.com/s/QaIftFx7B9qa48a0GVI5-g](https://mp.weixin.qq.com/s/QaIftFx7B9qa48a0GVI5-g)

作者 \| 温安适

来源 \| [http://8rr.co/EjqL](http://8rr.co/EjqL)



这个系列分为5篇，这是首篇

1.@Component，@Service等注解是如何被解析的

2.@Enableq驱动原理

3.@EnableAutoConfiguration处理逻辑

4.spring，springBoot事件（最晚20200719）

5.自定义springboot starter（最晚20200726）

## 前言

@Component和@Service都是工作中常用的注解，Spring如何解析？

## 1.@Component解析流程

### 找入口

Spring Framework2.0开始，引入可扩展的XML编程机制，该机制要求XML Schema命名空间需要与Handler建立映射关系。

该关系配置在相对于classpath下的/META-INF/spring.handlers中。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/07/640-20200807100101841-100102.jpg)

如上图所示 **ContextNamespaceHandler对应context:... 分析的入口。**

### 找核心方法

浏览ContextNamespaceHandler

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/07/640-20200807100101988-100102.jpg)

在parse中有一个很重要的注释

> // Actually scan for bean definitions and register them.
>
> ClassPathBeanDefinitionScanner scanner = configureScanner\(parserContext, element\);

大意是：**ClassPathBeanDefinitionScanner\#doScan是扫描BeanDefinition并注册的实现** **。**

ClassPathBeanDefinitionScanner 的源码如下：

```text
protected Set<BeanDefinitionHolder> doScan(String... basePackages) {
   Assert.notEmpty(basePackages, "At least one base package must be specified");
   Set<BeanDefinitionHolder> beanDefinitions = new LinkedHashSet<>();
   for (String basePackage : basePackages) {
      //findCandidateComponents 读资源装换为BeanDefinition
      Set<BeanDefinition> candidates = findCandidateComponents(basePackage);
      for (BeanDefinition candidate : candidates) {
         ScopeMetadata scopeMetadata = this.scopeMetadataResolver.resolveScopeMetadata(candidate);
         candidate.setScope(scopeMetadata.getScopeName());
         String beanName = this.beanNameGenerator.generateBeanName(candidate, this.registry);
         if (candidate instanceof AbstractBeanDefinition) {
            postProcessBeanDefinition((AbstractBeanDefinition) candidate, beanName);
         }
         if (candidate instanceof AnnotatedBeanDefinition) {
            AnnotationConfigUtils.processCommonDefinitionAnnotations((AnnotatedBeanDefinition) candidate);
         }
         if (checkCandidate(beanName, candidate)) {
            BeanDefinitionHolder definitionHolder = new BeanDefinitionHolder(candidate, beanName);
            definitionHolder =
                  AnnotationConfigUtils.applyScopedProxyMode(scopeMetadata, definitionHolder, this.registry);
            beanDefinitions.add(definitionHolder);
            registerBeanDefinition(definitionHolder, this.registry);
         }
      }
   }
   return beanDefinitions;
}
```

上边的代码，从方法名，猜测：

**findCandidateComponents：从classPath扫描组件，并转换为备选BeanDefinition，也就是要做的解析@Component的核心方法。**

### 概要分析

**findCandidateComponents在其父类ClassPathScanningCandidateComponentProvider 中。**

```text
public class ClassPathScanningCandidateComponentProvider implements EnvironmentCapable, ResourceLoaderAware {
//省略其他代码
public Set<BeanDefinition> findCandidateComponents(String basePackage) {
   if (this.componentsIndex != null && indexSupportsIncludeFilters()) {
      return addCandidateComponentsFromIndex(this.componentsIndex, basePackage);
   }
   else {
      return scanCandidateComponents(basePackage);
   }
}
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
   Set<BeanDefinition> candidates = new LinkedHashSet<>();
   try {
      String packageSearchPath = ResourcePatternResolver.CLASSPATH_ALL_URL_PREFIX +
            resolveBasePackage(basePackage) + '/' + this.resourcePattern;
      Resource[] resources = getResourcePatternResolver().getResources(packageSearchPath);
        //省略部分代码
      for (Resource resource : resources) {
        //省略部分代码
         if (resource.isReadable()) {
            try {
               MetadataReader metadataReader = getMetadataReaderFactory().getMetadataReader(resource);
               if (isCandidateComponent(metadataReader)) {
                  ScannedGenericBeanDefinition sbd = new ScannedGenericBeanDefinition(metadataReader);
                  sbd.setSource(resource);
                  if (isCandidateComponent(sbd)) {
                     candidates.add(sbd);
                //省略部分代码
      }
   }
   catch (IOException ex) {//省略部分代码 }
   return candidates;
}
}
```

**findCandidateComponents大体思路如下：**

1. String packageSearchPath = ResourcePatternResolver.CLASSPATH\_ALL\_URL\_PREFIX resolveBasePackage\(basePackage\) + '/' + this.resourcePattern;                             将package转化为ClassLoader类资源搜索路径packageSearchPath，例如：com.wl.spring.boot转化为classpath_:com/wl/spring/boot/\*\*/_.class
2. Resource\[\] resources = getResourcePatternResolver\(\).getResources\(packageSearchPath\);  加载搜素路径下的资源。
3. isCandidateComponent 判断是否是备选组件
4. candidates.add\(sbd\); 添加到返回结果的list

ClassPathScanningCandidateComponentProvider\#isCandidateComponent其源码如下：

```text
protected boolean isCandidateComponent(MetadataReader metadataReader) throws IOException {
    //省略部分代码
   for (TypeFilter tf : this.includeFilters) {
      if (tf.match(metadataReader, getMetadataReaderFactory())) {
         return isConditionMatch(metadataReader);
      }
   }
   return false;
}
```

includeFilters由registerDefaultFilters\(\)设置初始值，有@Component，没有@Service啊？

```text
protected void registerDefaultFilters() {
   this.includeFilters.add(new AnnotationTypeFilter(Component.class));
   ClassLoader cl = ClassPathScanningCandidateComponentProvider.class.getClassLoader();
   try {
      this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils.forName("javax.annotation.ManagedBean", cl)), false));
      logger.trace("JSR-250 'javax.annotation.ManagedBean' found and supported for component scanning");
   }
   catch (ClassNotFoundException ex) {
      // JSR-250 1.1 API (as included in Java EE 6) not available - simply skip.
   }
   try {
      this.includeFilters.add(new AnnotationTypeFilter(
            ((Class<? extends Annotation>) ClassUtils.forName("javax.inject.Named", cl)), false));
      logger.trace("JSR-330 'javax.inject.Named' annotation found and supported for component scanning");
   }
   catch (ClassNotFoundException ex) {
      // JSR-330 API not available - simply skip.
   }
}
```

Spring如何处理@Service的注解的呢？？？？

## 2.查文档找思路

查阅官方文档，下面这话：

[https://docs.spring.io/spring/docs/5.0.17.RELEASE/spring-framework-reference/core.html\#beans-meta-annotations](https://docs.spring.io/spring/docs/5.0.17.RELEASE/spring-framework-reference/core.html#beans-meta-annotations)

> @Component is a generic stereotype for any Spring-managed component. @Repository, @Service, and @Controller are specializations of @Component

大意如下：

@Component是任何Spring管理的组件的通用原型。@Repository、@Service和@Controller是派生自@Component。

```text
@Target({ElementType.TYPE})
@Retention(RetentionPolicy.RUNTIME)
@Documented
// @Service 派生自@Component
@Component
public @interface Service {

   /**
    * The value may indicate a suggestion for a logical component name,
    * to be turned into a Spring bean in case of an autodetected component.
    * @return the suggested component name, if any (or empty String otherwise)
    */
   @AliasFor(annotation = Component.class)
   String value() default "";

}
```

@Component是@Service的元注解，Spring 大概率，在读取@Service，也读取了它的元注解，并将@Service作为@Component处理。

## 3. 探寻@Component派生性流程

回顾ClassPathScanningCandidateComponentProvider 中的关键的代码片段如下：

```text
private Set<BeanDefinition> scanCandidateComponents(String basePackage) {
 //省略其他代码
 MetadataReader metadataReader   
             =getMetadataReaderFactory().getMetadataReader(resource);  
   if(isCandidateComponent(metadataReader)){
       //....
   }         
}
public final MetadataReaderFactory getMetadataReaderFactory() {
   if (this.metadataReaderFactory == null) {
      this.metadataReaderFactory = new CachingMetadataReaderFactory();
   }
   return this.metadataReaderFactory;
}
```

### 1. 确定metadataReader

CachingMetadataReaderFactory继承自 SimpleMetadataReaderFactory，就是对SimpleMetadataReaderFactory加了一层缓存。

其内部的SimpleMetadataReaderFactory\#getMetadataReader 为：

```text
public class SimpleMetadataReaderFactory implements MetadataReaderFactory {
    @Override
public MetadataReader getMetadataReader(Resource resource) throws IOException {
   return new SimpleMetadataReader(resource, this.resourceLoader.getClassLoader());
}
    }
```

这里可以看出

**MetadataReader metadataReader =new SimpleMetadataReader\(...\);**

### 2.查看match方法找重点方法

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/08/07/640-20200807100102104-100102.jpg)

AnnotationTypeFilter\#matchself方法如下：

```text
@Override
protected boolean matchSelf(MetadataReader metadataReader) {
   AnnotationMetadata metadata = metadataReader.getAnnotationMetadata();
   return metadata.hasAnnotation(this.annotationType.getName()) ||
         (this.considerMetaAnnotations && metadata.hasMetaAnnotation(this.annotationType.getName()));
}
```

**是metadata.hasMetaAnnotation法，从名称看是处理元注解，我们重点关注**

### 逐步分析

#### 找metadata.hasMetaAnnotation

metadata=metadataReader.getAnnotationMetadata\(\);

metadataReader =new SimpleMetadataReader\(...\)

metadata= new SimpleMetadataReader\#getAnnotationMetadata\(\)

```text
//SimpleMetadataReader 的构造方法
SimpleMetadataReader(Resource resource, @Nullable ClassLoader classLoader) throws IOException {
   InputStream is = new BufferedInputStream(resource.getInputStream());
   ClassReader classReader;
   try {
      classReader = new ClassReader(is);
   }
   catch (IllegalArgumentException ex) {
      throw new NestedIOException("ASM ClassReader failed to parse class file - " +
            "probably due to a new Java class file version that isn't supported yet: " + resource, ex);
   }
   finally {
      is.close();
   }

   AnnotationMetadataReadingVisitor visitor =
            new AnnotationMetadataReadingVisitor(classLoader);
   classReader.accept(visitor, ClassReader.SKIP_DEBUG);

   this.annotationMetadata = visitor;
   // (since AnnotationMetadataReadingVisitor extends ClassMetadataReadingVisitor)
   this.classMetadata = visitor;
   this.resource = resource;
}
```

metadata=new SimpleMetadataReader\(...\)**.**getAnnotationMetadata\(\)= new AnnotationMetadataReadingVisitor（。。）

也就是说

**metadata.hasMetaAnnotation=AnnotationMetadataReadingVisitor\#hasMetaAnnotation**

其方法如下：

```text
public class AnnotationMetadataReadingVisitor{
    // 省略部分代码
@Override
public boolean hasMetaAnnotation(String metaAnnotationType) {
   Collection<Set<String>> allMetaTypes = this.metaAnnotationMap.values();
   for (Set<String> metaTypes : allMetaTypes) {
      if (metaTypes.contains(metaAnnotationType)) {
         return true;
      }
   }
   return false;
}
}
```

逻辑很简单，就是判断该注解的元注解在，在不在metaAnnotationMap中，如果在就返回true。

这里面核心就是metaAnnotationMap，搜索AnnotationMetadataReadingVisitor类，没有发现赋值的地方？？！。

#### 查找metaAnnotationMap赋值

回到SimpleMetadataReader 的方法，

```text
//这个accept方法，很可疑，在赋值之前执行
SimpleMetadataReader(Resource resource, @Nullable ClassLoader classLoader) throws IOException {
//省略其他代码
AnnotationMetadataReadingVisitor visitor = new AnnotationMetadataReadingVisitor(classLoader);
classReader.accept(visitor, ClassReader.SKIP_DEBUG);
 this.annotationMetadata = visitor;
 }
```

发现一个可疑的语句：classReader.accept。

查看accept方法

```text
public class ClassReader {
        //省略其他代码
public void accept(..省略代码){
    //省略其他代码
    readElementValues(
    classVisitor.visitAnnotation(annotationDescriptor, /* visible = */ true),
    currentAnnotationOffset,
     true,
    charBuffer);
}
}
```

查看readElementValues方法

```text
public class ClassReader{
    //省略其他代码
private int readElementValues(
    final AnnotationVisitor annotationVisitor,
    final int annotationOffset,
    final boolean named,
    final char[] charBuffer) {
  int currentOffset = annotationOffset;
  // Read the num_element_value_pairs field (or num_values field for an array_value).
  int numElementValuePairs = readUnsignedShort(currentOffset);
  currentOffset += 2;
  if (named) {
    // Parse the element_value_pairs array.
    while (numElementValuePairs-- > 0) {
      String elementName = readUTF8(currentOffset, charBuffer);
      currentOffset =
          readElementValue(annotationVisitor, currentOffset + 2, elementName, charBuffer);
    }
  } else {
    // Parse the array_value array.
    while (numElementValuePairs-- > 0) {
      currentOffset =
          readElementValue(annotationVisitor, currentOffset, /* named = */ null, charBuffer);
    }
  }
  if (annotationVisitor != null) {
    annotationVisitor.visitEnd();
  }
  return currentOffset;
}
}
```

这里面的核心就是 annotationVisitor.visitEnd\(\);

**确定annotationVisitor**

这里的annotationVisitor=AnnotationMetadataReadingVisitor\#visitAnnotation

源码如下，注意这里传递了metaAnnotationMap！！

```text
public class AnnotationMetadataReadingVisitor{
@Override
public AnnotationVisitor visitAnnotation(String desc, boolean visible) {
   String className = Type.getType(desc).getClassName();
   this.annotationSet.add(className);
   return new AnnotationAttributesReadingVisitor(
         className, this.attributesMap,
              this.metaAnnotationMap, this.classLoader);
}
}
```

**annotationVisitor=AnnotationAttributesReadingVisitor**

**查阅annotationVisitor.visitEnd\(\)**

**annotationVisitor=AnnotationAttributesReadingVisitor\#visitEnd\(\)**

```text
public class AnnotationAttributesReadingVisitor{
@Override
public void visitEnd() {
   super.visitEnd();

   Class<? extends Annotation> annotationClass = this.attributes.annotationType();
   if (annotationClass != null) {
      List<AnnotationAttributes> attributeList = this.attributesMap.get(this.annotationType);
      if (attributeList == null) {
         this.attributesMap.add(this.annotationType, this.attributes);
      }
      else {
         attributeList.add(0, this.attributes);
      }
      if (!AnnotationUtils.isInJavaLangAnnotationPackage(annotationClass.getName())) {
         try {
            Annotation[] metaAnnotations = annotationClass.getAnnotations();
            if (!ObjectUtils.isEmpty(metaAnnotations)) {
               Set<Annotation> visited = new LinkedHashSet<>();
               for (Annotation metaAnnotation : metaAnnotations) {
                  recursivelyCollectMetaAnnotations(visited, metaAnnotation);
               }
               if (!visited.isEmpty()) {
                  Set<String> metaAnnotationTypeNames = new LinkedHashSet<>(visited.size());
                  for (Annotation ann : visited) {
                     metaAnnotationTypeNames.add(ann.annotationType().getName());
                  }
                  this.metaAnnotationMap.put(annotationClass.getName(), metaAnnotationTypeNames);
               }
            }
         }
         catch (Throwable ex) {
            if (logger.isDebugEnabled()) {
               logger.debug("Failed to introspect meta-annotations on " + annotationClass + ": " + ex);
            }
         }
      }
   }
}
}
```

内部方法recursivelyCollectMetaAnnotations 递归的读取注解，与注解的元注解（读@Service，再读元注解@Component），并设置到metaAnnotationMap，也就是AnnotationMetadataReadingVisitor 中的metaAnnotationMap中。

## 总结

大致如下：

ClassPathScanningCandidateComponentProvider\#findCandidateComponents

1. 将package转化为ClassLoader类资源搜索路径packageSearchPath
2. 加载搜素路径下的资源。
3. isCandidateComponent 判断是否是备选组件。

   内部调用的TypeFilter的match方法：

   AnnotationTypeFilter\#matchself中metadata.hasMetaAnnotation处理元注解

   metadata.hasMetaAnnotation=AnnotationMetadataReadingVisitor\#hasMetaAnnotation

   就是判断当前注解的元注解在不在metaAnnotationMap中。

   AnnotationAttributesReadingVisitor\#visitEnd\(\)内部方法recursivelyCollectMetaAnnotations 递归的读取注解，与注解的元注解（读@Service，再读元注解@Component），并设置到metaAnnotationMap

4. 添加到返回结果的list

