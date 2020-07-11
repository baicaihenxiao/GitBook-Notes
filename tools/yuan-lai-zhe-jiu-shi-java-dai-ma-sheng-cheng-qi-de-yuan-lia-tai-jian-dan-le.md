# 原来这就是Java代码生成器的原理啊，太简单了

{% embed url="https://mp.weixin.qq.com/s/77qrA8azrnTDvBu7iUp1jg" %}



### 1. 前言

前几天写了篇关于代码生成器的文章（可查看历史文章），不少同学私下问我这个代码生成器是如何运作的，为什么要用到一些模板引擎，所以今天来说明下代码生成器的流程。

### 2. 代码生成器的使用场景

我们在编码中存在很多样板代码，格式较为固定，结构随着项目的迭代也比较稳定，而且数量巨大，这种代码写多了也没有什么技术含量，在这种情况下代码生成器可以有效提高我们的效率，其它情况并不适于使用代码生成器。

### 3. 代码生成器的制作流程

首先我们要制作模板，把样板代码的固定格式抽出来。然后把动态属性绑定到模板中，就像做填空题一样。所以在这个流程中模板引擎是最合适的。我们通过使用模板引擎的语法将数据动态地解析到静态模板中去，然后导出为编程中对应的文件就行了。

另外模板引擎有着丰富的绑定数据的指令集，可以让我们根据条件动态的绑定数据到模板中去。以**Freemarker**为例：

三元表达式：

```text
${true ? 'checked': ''}
```

还有我们等下要用的遍历列表：

```text
<#list  fields as field>
    private ${field.fieldType}  ${field.fieldName};
</#list>
```

在 Java 开发中我们常用的模板引擎有**Freemarker**、**Velocity**、**Thymeleaf** ，随着**Web**开发中前后端分离的流行模板引擎的使用场景正在被压缩，但是它依然是一门有用的技术。

### 4. 代码生成器演示

接下来，我们以**Freemarker**为例写一个简单的代码生成器，来生成**POJO**类。需要引入**Freemarker**的依赖。

```text
<dependency>
    <groupId>org.freemarker</groupId>
    <artifactId>freemarker</artifactId>
    <version>2.3.28</version>
</dependency>
```

#### 4.1 模板制作

**POJO**的结构可以分为以下几部分：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711181628368-181628.png)Java类的基本结构

> `java.lang` 包无需导入。

所以将这些规则封装到配置类中：

```text
public class JavaProperties {
    // 包名
    private final String pkg;
    // 类名
    private final String entityName;
    // 属性集合  需要改写 equals hash 保证名字可不重复 类型可重复
    private final Set<Field> fields = new LinkedHashSet<>();
    // 导入类的不重复集合
    private final Set<String> imports = new LinkedHashSet<>();


    public JavaProperties(String entityName, String pkg) {
        this.entityName = entityName;
        this.pkg = pkg;
    }

    public void addField(Class<?> type, String fieldName) {
        // 处理 java.lang
        final String pattern = "java.lang";
        String fieldType = type.getName();
        if (!fieldType.startsWith(pattern)) {
           // 处理导包
            imports.add(fieldType);
        }
        Field field = new Field();
        // 处理成员属性的格式
        int i = fieldType.lastIndexOf(".");
        field.setFieldType(fieldType.substring(i + 1));
        field.setFieldName(fieldName);
        fields.add(field);
    }

    public String getPkg() {
        return pkg;
    }


    public String getEntityName() {
        return entityName;
    }


    public Set<Field> getFields() {
        return fields;
    }

    public Set<String> getImports() {
        return imports;
    }


    /**
     * 成员属性封装对象.
     */
    public static class Field {
        // 成员属性类型
        private String fieldType;
        // 成员属性名称
        private String fieldName;

        public String getFieldType() {
            return fieldType;
        }

        public void setFieldType(String fieldType) {
            this.fieldType = fieldType;
        }

        public String getFieldName() {
            return fieldName;
        }

        public void setFieldName(String fieldName) {
            this.fieldName = fieldName;
        }

        /**
         * 一个类的成员属性 一个名称只能出现一次
         * 我们可以通过覆写equals hash 方法 然后放入Set
         *
         * @param o 另一个成员属性
         * @return 比较结果
         */
        @Override
        public boolean equals(Object o) {
            if (this == o) return true;
            if (o == null || getClass() != o.getClass()) return false;
            Field field = (Field) o;
            return Objects.equals(fieldName, field.fieldName);
        }

        @Override
        public int hashCode() {
            return Objects.hash(fieldType, fieldName);
        }
    }

}
```

接着就是静态模板`entity.ftl`

```text
package ${pkg};

<#list  imports as impt>
import ${impt};
</#list>

/**
 * the ${entityName} type
 * @author felord.cn
 */
public class ${entityName} {

<#list  fields as field>
    private ${field.fieldType}  ${field.fieldName};
</#list>

}
```

这里用到了**Freemarker**绑定数据的语法，比如`List`迭代渲染。

#### 4.2 生成器编写

**Freemarker**通过声明配置并获取模板对象`freemarker.template`，该对象的`process`方法可以将动态数据绑定到模板中并导出为文件，最终实现了代码生成器，核心代码如下：

```text
/**
 * 简单的代码生成器.
 *
 * @param rootPath       maven 的  java 目录
 * @param templatePath   模板存放的文件夹
 * @param templateName   模板的名称
 * @param javaProperties 需要渲染对象的封装
 * @throws IOException       the io exception
 * @throws TemplateException the template exception
 */
public static void autoCodingJavaEntity(String rootPath,
                                         String templatePath,
                                         String templateName,
                                         JavaProperties javaProperties) throws IOException, TemplateException {

    // freemarker 配置
    Configuration configuration = new Configuration(Configuration.DEFAULT_INCOMPATIBLE_IMPROVEMENTS);

    configuration.setDefaultEncoding("UTF-8");
    // 指定模板的路径
    configuration.setDirectoryForTemplateLoading(new File(templatePath));
    // 根据模板名称获取路径下的模板
    Template template = configuration.getTemplate(templateName);
    // 处理路径问题
    final String ext = ".java";
    String javaName = javaProperties.getEntityName().concat(ext);
    String packageName = javaProperties.getPkg();

    String out = rootPath.concat(Stream.of(packageName.split("\\."))
            .collect(Collectors.joining("/", "/", "/" + javaName)));

     // 定义一个输出流来导出代码文件
    OutputStreamWriter outputStreamWriter = new OutputStreamWriter(new FileOutputStream(out));
     // freemarker 引擎将动态数据绑定的模板并导出为文件
    template.process(javaProperties, outputStreamWriter);

}
```

通过执行以下代码即可生成一个`UserEntity`的**POJO**：

```text
// 路径根据自己项目的特点调整
String rootPath = "C:\\Users\\felord\\IdeaProjects\\codegenerator\\src\\main\\java";
String packageName = "cn.felord.code";
String templatePath = "C:\\Users\\felord\\IdeaProjects\\codegenerator\\src\\main\\resources\\templates";
String templateName = "entity.ftl";


JavaProperties userEntity = new JavaProperties("UserEntity", packageName);

userEntity.addField(String.class, "username");
userEntity.addField(LocalDate.class, "birthday");
userEntity.addField(LocalDateTime.class, "addTime");
userEntity.addField(Integer.class, "gender");
userEntity.addField(Integer.class, "age");


autoCodingJavaEntity(rootPath, templatePath, templateName, userEntity);
```

生成的效果是不是跟手写的差不多：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/11/640-20200711181628601-181628.png)生成的Java POJO

### 5. 总结

这就是大部分代码生成器的机制，希望可以解答一些网友的疑问。多多关注：**码农小胖哥** 获取更多干货，相关的**DEMO**可通过公众号回复**codegen**获取。如果你有疑问可以通过微信**MSW\_623**进行沟通。

