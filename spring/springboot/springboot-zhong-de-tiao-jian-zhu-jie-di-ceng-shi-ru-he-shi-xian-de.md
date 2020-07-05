# SpringBoot中的条件注解底层是如何实现的？

{% embed url="https://mp.weixin.qq.com/s/KG4iOr97j4Vql2v-gumUEQ" %}



## 

来源 \| [https://urlify.cn/bm2qqi](https://urlify.cn/bm2qqi)

SpringBoot内部提供了特有的注解：条件注解\(Conditional Annotation\)。比如：

* @ConditionalOnBean、
* @ConditionalOnClass、
* @ConditionalOnExpression、
* @ConditionalOnMissingBean等。

条件注解存在的意义在于动态识别\(也可以说是代码自动化执行\)。比如@ConditionalOnClass会检查类加载器中是否存在对应的类，如果有的话被注解修饰的类就有资格被Spring容器所注册，否则会被skip。

比如FreemarkerAutoConfiguration这个自动化配置类的定义如下：

```text
@Configuration
@ConditionalOnClass({ freemarker.template.Configuration.class,
    FreeMarkerConfigurationFactory.class })
@AutoConfigureAfter(WebMvcAutoConfiguration.class)
@EnableConfigurationProperties(FreeMarkerProperties.class)
public class FreeMarkerAutoConfiguration
```

这个自动化配置类被@ConditionalOnClass条件注解修饰，这个条件注解存在的意义在于判断类加载器中是否存在freemarker.template.Configuration和FreeMarkerConfigurationFactory这两个类，如果都存在的话会在Spring容器中加载这个FreeMarkerAutoConfiguration配置类；否则不会加载。

## 条件注解内部的一些基础

在分析条件注解的底层实现之前，我们先来看一下这些条件注解的定义。以@ConditionalOnClass注解为例，它的定义如下：

```text
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnClassCondition.class)
public @interface ConditionalOnClass {
  Class<?>[] value() default {}; // 需要匹配的类
  String[] name() default {}; // 需要匹配的类名
}
```

它有2个属性，分别是类数组和字符串数组\(作用一样，类型不一样\)，而且被@Conditional注解所修饰，这个@Conditional注解有个名为values的Class&lt;? extends Condition&gt;\[\]类型的属性。这个Condition是个接口，用于匹配组件是否有资格被容器注册，定义如下：

```text
public interface Condition {
  // ConditionContext内部会存储Spring容器、应用程序环境信息、资源加载器、类加载器
  boolean matches(ConditionContext context, AnnotatedTypeMetadata metadata);
}
```

也就是说@Conditional注解属性中可以持有多个Condition接口的实现类，所有的Condition接口需要全部匹配成功后这个@Conditional修饰的组件才有资格被注册。

Condition接口有个子接口ConfigurationCondition：

```text
public interface ConfigurationCondition extends Condition {
  ConfigurationPhase getConfigurationPhase();
  public static enum ConfigurationPhase {
    PARSE_CONFIGURATION,
    REGISTER_BEAN
  }
}
```

这个子接口是一种特殊的条件接口，多了一个getConfigurationPhase方法，也就是条件注解的生效阶段。只有在ConfigurationPhase中定义的两种阶段下才会生效。

Condition接口有个实现抽象类SpringBootCondition，SpringBoot中所有条件注解对应的条件类都继承这个抽象类。它实现了matches方法：

```text
@Override
public final boolean matches(ConditionContext context,
    AnnotatedTypeMetadata metadata) {
  String classOrMethodName = getClassOrMethodName(metadata); // 得到类名或者方法名(条件注解可以作用的类或者方法上)
  try {
    ConditionOutcome outcome = getMatchOutcome(context, metadata); // 抽象方法，具体子类实现。ConditionOutcome记录了匹配结果boolean和log信息
    logOutcome(classOrMethodName, outcome); // log记录一下匹配信息
    recordEvaluation(context, classOrMethodName, outcome); // 报告记录一下匹配信息
    return outcome.isMatch(); // 返回是否匹配
  }
  catch (NoClassDefFoundError ex) {
    throw new IllegalStateException(
        "Could not evaluate condition on " + classOrMethodName + " due to "
            + ex.getMessage() + " not "
            + "found. Make sure your own configuration does not rely on "
            + "that class. This can also happen if you are "
            + "@ComponentScanning a springframework package (e.g. if you "
            + "put a @ComponentScan in the default package by mistake)",
        ex);
  }
  catch (RuntimeException ex) {
    throw new IllegalStateException(
        "Error processing condition on " + getName(metadata), ex);
  }
}
```

## 基于Class的条件注解

SpringBoot提供了两个基于Class的条件注解：@ConditionalOnClass\(类加载器中存在指明的类\)或者@ConditionalOnMissingClass\(类加载器中不存在指明的类\)。

@ConditionalOnClass或者@ConditionalOnMissingClass注解对应的条件类是OnClassCondition，定义如下：

```text
@Order(Ordered.HIGHEST_PRECEDENCE) // 优先级、最高级别
class OnClassCondition extends SpringBootCondition {

  @Override
  public ConditionOutcome getMatchOutcome(ConditionContext context,
      AnnotatedTypeMetadata metadata) {

    StringBuffer matchMessage = new StringBuffer(); // 记录匹配信息

    MultiValueMap<String, Object> onClasses = getAttributes(metadata,
        ConditionalOnClass.class); // 得到@ConditionalOnClass注解的属性
    if (onClasses != null) { // 如果属性存在
      List<String> missing = getMatchingClasses(onClasses, MatchType.MISSING,
          context); // 得到在类加载器中不存在的类
      if (!missing.isEmpty()) { // 如果存在类加载器中不存在对应的类，返回一个匹配失败的ConditionalOutcome
        return ConditionOutcome
            .noMatch("required @ConditionalOnClass classes not found: "
                + StringUtils.collectionToCommaDelimitedString(missing));
      }
                // 如果类加载器中存在对应的类的话，匹配信息进行记录
      matchMessage.append("@ConditionalOnClass classes found: "
          + StringUtils.collectionToCommaDelimitedString(
              getMatchingClasses(onClasses, MatchType.PRESENT, context)));
    }
        // 对@ConditionalOnMissingClass注解做相同的逻辑处理(说明@ConditionalOnClass和@ConditionalOnMissingClass可以一起使用)
    MultiValueMap<String, Object> onMissingClasses = getAttributes(metadata,
        ConditionalOnMissingClass.class);
    if (onMissingClasses != null) {
      List<String> present = getMatchingClasses(onMissingClasses, MatchType.PRESENT,
          context);
      if (!present.isEmpty()) {
        return ConditionOutcome
            .noMatch("required @ConditionalOnMissing classes found: "
                + StringUtils.collectionToCommaDelimitedString(present));
      }
      matchMessage.append(matchMessage.length() == 0 ? "" : " ");
      matchMessage.append("@ConditionalOnMissing classes not found: "
          + StringUtils.collectionToCommaDelimitedString(getMatchingClasses(
              onMissingClasses, MatchType.MISSING, context)));
    }
        // 返回全部匹配成功的ConditionalOutcome
    return ConditionOutcome.match(matchMessage.toString());
  }

  private enum MatchType { // 枚举：匹配类型。用于查询类名在对应的类加载器中是否存在。

    PRESENT { // 匹配成功
      @Override
      public boolean matches(String className, ConditionContext context) {
        return ClassUtils.isPresent(className, context.getClassLoader());
      }
    },

    MISSING { // 匹配不成功
      @Override
      public boolean matches(String className, ConditionContext context) {
        return !ClassUtils.isPresent(className, context.getClassLoader());
      }
    };

    public abstract boolean matches(String className, ConditionContext context);

  }

}
```

比如FreemarkerAutoConfiguration中的@ConditionalOnClass注解中有value属性是freemarker.template.Configuration.class和FreeMarkerConfigurationFactory.class。在OnClassCondition执行过程中得到的最终ConditionalOutcome中的log message如下：

```text
1 @ConditionalOnClass classes found: freemarker.template.Configuration,org.springframework.ui.freemarker.FreeMarkerConfigurationFactory
```

## 基于Bean的条件注解

@ConditionalOnBean\(Spring容器中存在指明的bean\)、@ConditionalOnMissingBean\(Spring容器中不存在指明的bean\)以及ConditionalOnSingleCandidate\(Spring容器中存在且只存在一个指明的bean\)都是基于Bean的条件注解，它们对应的条件类是ConditionOnBean。

@ConditionOnBean注解定义如下：

```text
@Target({ ElementType.TYPE, ElementType.METHOD })
@Retention(RetentionPolicy.RUNTIME)
@Documented
@Conditional(OnBeanCondition.class)
public @interface ConditionalOnBean {
  Class<?>[] value() default {}; // 匹配的bean类型
  String[] type() default {}; // 匹配的bean类型的类名
  Class<? extends Annotation>[] annotation() default {}; // 匹配的bean注解
  String[] name() default {}; // 匹配的bean的名字
  SearchStrategy search() default SearchStrategy.ALL; // 搜索策略。提供CURRENT(只在当前容器中找)、PARENTS(只在所有的父容器中找；但是不包括当前容器)和ALL(CURRENT和PARENTS的组合)
}
```

OnBeanCondition条件类的匹配代码如下：

```text
@Override
public ConditionOutcome getMatchOutcome(ConditionContext context,
    AnnotatedTypeMetadata metadata) {
  StringBuffer matchMessage = new StringBuffer(); // 记录匹配信息
  if (metadata.isAnnotated(ConditionalOnBean.class.getName())) {
    BeanSearchSpec spec = new BeanSearchSpec(context, metadata,
        ConditionalOnBean.class); // 构造一个BeanSearchSpec，会从@ConditionalOnBean注解中获取属性，然后设置到BeanSearchSpec中
    List<String> matching = getMatchingBeans(context, spec); // 从BeanFactory中根据策略找出所有匹配的bean
    if (matching.isEmpty()) { // 如果没有匹配的bean，返回一个没有匹配成功的ConditionalOutcome
      return ConditionOutcome
          .noMatch("@ConditionalOnBean " + spec + " found no beans");
    }
    // 如果找到匹配的bean，匹配信息进行记录
    matchMessage.append(
        "@ConditionalOnBean " + spec + " found the following " + matching);
  }
  if (metadata.isAnnotated(ConditionalOnSingleCandidate.class.getName())) { // 相同的逻辑，针对@ConditionalOnSingleCandidate注解
    BeanSearchSpec spec = new SingleCandidateBeanSearchSpec(context, metadata,
        ConditionalOnSingleCandidate.class);
    List<String> matching = getMatchingBeans(context, spec);
    if (matching.isEmpty()) {
      return ConditionOutcome.noMatch(
          "@ConditionalOnSingleCandidate " + spec + " found no beans");
    }
    else if (!hasSingleAutowireCandidate(context.getBeanFactory(), matching)) { // 多了一层判断，判断是否只有一个bean
      return ConditionOutcome.noMatch("@ConditionalOnSingleCandidate " + spec
          + " found no primary candidate amongst the" + " following "
          + matching);
    }
    matchMessage.append("@ConditionalOnSingleCandidate " + spec + " found "
        + "a primary candidate amongst the following " + matching);
  }
  if (metadata.isAnnotated(ConditionalOnMissingBean.class.getName())) { // 相同的逻辑，针对@ConditionalOnMissingBean注解
    BeanSearchSpec spec = new BeanSearchSpec(context, metadata,
        ConditionalOnMissingBean.class);
    List<String> matching = getMatchingBeans(context, spec);
    if (!matching.isEmpty()) {
      return ConditionOutcome.noMatch("@ConditionalOnMissingBean " + spec
          + " found the following " + matching);
    }
    matchMessage.append(matchMessage.length() == 0 ? "" : " ");
    matchMessage.append("@ConditionalOnMissingBean " + spec + " found no beans");
  }
  return ConditionOutcome.match(matchMessage.toString()); //返回匹配成功的ConditonalOutcome
}
```

SpringBoot还提供了其他比如ConditionalOnJava、ConditionalOnNotWebApplication、ConditionalOnWebApplication、ConditionalOnResource、ConditionalOnProperty、ConditionalOnExpression等条件注解，有兴趣的读者可以自行查看它们的底层处理逻辑。

## 各种条件注解的总结

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/05/640-20200705094704671.jpg)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/05/640-20200705094704861.png)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/05/640-20200705094705179.png)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/05/640-20200705094705490.png)

## SpringBoot条件注解的激活机制

分析完了条件注解的执行逻辑之后，接下来的问题就是SpringBoot是如何让这些条件注解生效的？

SpringBoot使用ConditionEvaluator这个内部类完成条件注解的解析和判断。

在Spring容器的refresh过程中，只有跟解析或者注册bean有关系的类都会使用ConditionEvaluator完成条件注解的判断，这个过程中一些类不满足条件的话就会被skip。这些类比如有AnnotatedBeanDefinitionReader、ConfigurationClassBeanDefinitionReader、ConfigurationClassParse、ClassPathScanningCandidateComponentProvider等。

比如ConfigurationClassParser的构造函数会初始化内部属性conditionEvaluator：

```text
public ConfigurationClassParser(MetadataReaderFactory metadataReaderFactory,
    ProblemReporter problemReporter, Environment environment, ResourceLoader resourceLoader,
    BeanNameGenerator componentScanBeanNameGenerator, BeanDefinitionRegistry registry) {

  this.metadataReaderFactory = metadataReaderFactory;
  this.problemReporter = problemReporter;
  this.environment = environment;
  this.resourceLoader = resourceLoader;
  this.registry = registry;
  this.componentScanParser = new ComponentScanAnnotationParser(
      resourceLoader, environment, componentScanBeanNameGenerator, registry);
  // 构造ConditionEvaluator用于处理条件注解
  this.conditionEvaluator = new ConditionEvaluator(registry, environment, resourceLoader);
}

ConfigurationClassParser对每个配置类进行解析的时候都会使用ConditionEvaluator：

if (this.conditionEvaluator.shouldSkip(configClass.getMetadata(), ConfigurationPhase.PARSE_CONFIGURATION)) {
  return;
}
```

ConditionEvaluator的skip方法：

```text
public boolean shouldSkip(AnnotatedTypeMetadata metadata, ConfigurationPhase phase) {
  // 如果这个类没有被@Conditional注解所修饰，不会skip
  if (metadata == null || !metadata.isAnnotated(Conditional.class.getName())) {
    return false;
  }
  // 如果参数中沒有设置条件注解的生效阶段
  if (phase == null) {
    // 是配置类的话直接使用PARSE_CONFIGURATION阶段
    if (metadata instanceof AnnotationMetadata &&
        ConfigurationClassUtils.isConfigurationCandidate((AnnotationMetadata) metadata)) {
      return shouldSkip(metadata, ConfigurationPhase.PARSE_CONFIGURATION);
    }
    // 否则使用REGISTER_BEAN阶段
    return shouldSkip(metadata, ConfigurationPhase.REGISTER_BEAN);
  }
  // 要解析的配置类的条件集合
  List<Condition> conditions = new ArrayList<Condition>();
  // 获取配置类的条件注解得到条件数据，并添加到集合中
  for (String[] conditionClasses : getConditionClasses(metadata)) {
    for (String conditionClass : conditionClasses) {
      Condition condition = getCondition(conditionClass, this.context.getClassLoader());
      conditions.add(condition);
    }
  }

  // 对条件集合做个排序
  AnnotationAwareOrderComparator.sort(conditions);
  // 遍历条件集合
  for (Condition condition : conditions) {
    ConfigurationPhase requiredPhase = null;
    if (condition instanceof ConfigurationCondition) {
      requiredPhase = ((ConfigurationCondition) condition).getConfigurationPhase();
    }
    // 没有这个解析类不需要阶段的判断或者解析类和参数中的阶段一致才会继续进行
    if (requiredPhase == null || requiredPhase == phase) {
      // 阶段一致切不满足条件的话，返回true并跳过这个bean的解析
      if (!condition.matches(this.context, metadata)) {
        return true;
      }
    }
  }

  return false;
}
```

SpringBoot在条件注解的解析log记录在了ConditionEvaluationReport类中，可以通过BeanFactory获取\(BeanFactory是有父子关系的；每个BeanFactory都存有一份ConditionEvaluationReport，互不相干\)：

```text
ConditionEvaluationReport conditionEvaluationReport = beanFactory.getBean("autoConfigurationReport", ConditionEvaluationReport.class);
Map<String, ConditionEvaluationReport.ConditionAndOutcomes> result = conditionEvaluationReport.getConditionAndOutcomesBySource();
for(String key : result.keySet()) {
    ConditionEvaluationReport.ConditionAndOutcomes conditionAndOutcomes = result.get(key);
    Iterator<ConditionEvaluationReport.ConditionAndOutcome> iterator = conditionAndOutcomes.iterator();
    while(iterator.hasNext()) {
        ConditionEvaluationReport.ConditionAndOutcome conditionAndOutcome = iterator.next();
        System.out.println(key + " -- " + conditionAndOutcome.getCondition().getClass().getSimpleName() + " -- " + conditionAndOutcome.getOutcome());
    }
}
```

打印出条件注解下的类加载信息：

```text
.......
org.springframework.boot.autoconfigure.freemarker.FreeMarkerAutoConfiguration -- OnClassCondition -- required @ConditionalOnClass classes not found: freemarker.template.Configuration,org.springframework.ui.freemarker.FreeMarkerConfigurationFactory
org.springframework.boot.autoconfigure.groovy.template.GroovyTemplateAutoConfiguration -- OnClassCondition -- required @ConditionalOnClass classes not found: groovy.text.markup.MarkupTemplateEngine
org.springframework.boot.autoconfigure.gson.GsonAutoConfiguration -- OnClassCondition -- required @ConditionalOnClass classes not found: com.google.gson.Gson
org.springframework.boot.autoconfigure.h2.H2ConsoleAutoConfiguration -- OnClassCondition -- required @ConditionalOnClass classes not found: org.h2.server.web.WebServlet
org.springframework.boot.autoconfigure.hateoas.HypermediaAutoConfiguration -- OnClassCondition -- required @ConditionalOnClass classes not found: org.springframework.hateoas.Resource,org.springframework.plugin.core.Plugin
org.springframework.boot.autoconfigure.hazelcast.HazelcastAutoConfiguration -- OnClassCondition -- required @ConditionalOnClass classes not found: com.hazelcast.core.HazelcastInstance
.......
```

