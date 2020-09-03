# 一文学会注解的正确使用姿势

[https://mp.weixin.qq.com/s/NpU0UdD32ak30wbAsXSnMA](https://mp.weixin.qq.com/s/NpU0UdD32ak30wbAsXSnMA)

### 前言

日志作为排查问题的重要手段，可以说是应用集成中必不可少的一环，但在日志中，又不宜暴露像电话，身份证，地址等个人敏感信息，去年 Q4 我司就开展了对 ELK 日志脱敏的全面要求。那么怎样快速又有效地实现日志脱敏呢。相信读者看完标题已经猜到了，没错，用注解！那么用注解该怎么实现日志脱敏呢，除了日志脱敏，注解还能用在哪些场景呢，注解的实现原理又是怎样的呢。本文将会为你详细介绍。

本文将会从以下几个方面来介绍注解。

* 日志脱敏场景简介
* 巧用注解解决这两类问题
* 1. 注解的定义与实现原理
     1. 使用注解解决日志脱敏
* 注解高级用法-解决银行中参数传递顺序要求

相信大家看了肯定有收获！

### 日志脱敏场景简介

在日志里我们的日志一般打印的是 model 的 Json string，比如有以下 model 类

```text
public class Request {
    /**
     *  用户姓名
     */
    private String name;
    /**
     *  身份证 
     */
    private String idcard;
    /**
     *  手机号
     */
    private String phone;

    /**
     *  图片的 base64
     */
    private String imgBase64;
}
```

有以下类实例

```text
Request request = new Request();
request.setName("爱新觉罗");
request.setIdcard("450111112222");
request.setPhone("18611111767");
request.setImgBase64("xxx");
```

我们一般使用 fastJson 来打印此 Request 的 json string：

```text
log.info(JSON.toJSONString(request));
```

这样就能把 Request 的所有属性值给打印出来，日志如下:

```text
{"idcard":"450111112222","imgBase64":"xxx","name":"张三","phone":"17120227942"}
```

这里的日志有两个问题

1. 安全性: name，phone, idcard 这些个人信息极其敏感，不应以明文的形式打印出来，我们希望这些敏感信息是以脱敏的形式输出的
2. 字段冗余：imgBase64 是图片的 base64，是一串非常长的字符串，在生产上，图片 base64 数据对排查问题帮助不大，反而会增大存储成本，而且这个字段是身份证正反面的 base64，也属于敏感信息，所以这个字段在日志中需要把它去掉。我们希望经过脱敏和瘦身（移除 imgBase64 字段）后的日志如下:

```text
{"idcard":"450******222","name":"爱**罗","phone":"186****1767","imgBase64":""}
```

可以看到各个字段最后都脱敏了，不过需要注意的这几个字段的脱敏规则是不一样的

* 身份证（idcard），保留前三位，后三位，其余打码
* 姓名（name）保留前后两位，其余打码
* 电话号码（phone）保持前三位，后四位，其余打码
* 图片的 base64（imgBase64）直接展示空字符串

该怎么实现呢，首先我们需要知道一个知识点，即 JSON.toJSONString 方法指定了一个参数 ValueFilter，可以定制要转化的属性。我们可以利用此 Filter 让最终的 JSON string 不展示或展示脱敏后的 value。大概逻辑如下

```text
public class Util {
    public static String toJSONString(Object object) {
        try {
            return JSON.toJSONString(object, getValueFilter());
        } catch (Exception e) {
            return ToStringBuilder.reflectionToString(object);
        }
    }

private static ValueFilter getValueFilter() {
        return (obj, key, value) -> {
            // obj-对象 key-字段名 value-字段值
            return  格式化后的value
        };
}
```

如上图示，我们只要在 **getValueFilter** 方法中对 value 作相关的脱敏操作，即可在最终的日志中展示脱敏后的日志。现在问题来了，该怎么处理字段的脱敏问题，我们知道有些字段需要脱敏，有些字段不需要脱敏，所以有人可能会根据 key 的名称来判断是否脱敏，代码如下:

```text
private static ValueFilter getValueFilter() {
        return (obj, key, value) -> {
            // obj-对象 key-字段名 value-字段值
            if (Objects.equal(key, "phone")) {
                return 脱敏后的phone
            }
            if (Objects.equal(key, "idcard")) {
                return 脱敏后的idcard
            }
            if (Objects.equal(key, "name")) {
                return 脱敏后的name
            }
            // 其余不需要脱敏的按原值返回
            return  value
        };
}
```

这样看起来确实实现了需求，但仅仅实现了需求就够了吗，这样的实现有个比较严重的问题：

**脱敏规则与具体的属性名紧藕合**，需要在 valueFilter 里写大量的 if else 判断逻辑，可扩展性不高，通用性不强，举个简单的例子，由于业务原因，在我们的工程中电话有些字段名叫 phone, 有些叫 tel，有些叫 telephone，它们的脱敏规则是一样的，但你不得不在上面的方法中写出如下丑陋的代码。

```text
private static ValueFilter getValueFilter() {
        return (obj, key, value) -> {
            // obj-对象 key-字段名 value-字段值
            if (Objects.equal(key, "phone") || Objects.equal(key, "tel") || Objects.equal(key, "telephone") || ) {
                return 脱敏后的phone
            }

            // 其余不需要脱敏的按原值返回
            return  value
        };
}
```

那么能否用一种通用的，可扩展性好的方法来解决呢，相信你看到文章的标题已经心中有数了，没错，就是用的注解，接下来我们来看看什么是注解以及如何自定义注解

### 注解的定义与实现原理

注解（Annotation）又称 Java 标注，是 JDK 5.0 引入的一种注释机制，如果说代码的注释是给程序员看的，那么注解就是给程序看的，程序看到注解后就可以在运行时拿到注解，根据注解来增强运行时的能力，常见的应用在代码中的注解有如下三个

* @Override 检查该方法是否重写了父类方法，如果发现父类或实现的接口中没有此方法，则报编译错误
* @Deprecated 标记过时的类，方法，属性等
* @SuppressWarnings - 指示编译器去忽略注解中声明的警告。

那这些注解是怎么实现的呢，我们打开 @Override 这个注解看看

```text
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
}
```

可以看到 Deprecated 注解上又有 @Documented, @Retention, @Target 这些注解，这些注解又叫元注解，即注解 Deprecated 或其他自定义注解的注解，其他注解的行为由这些注解来规范和定义，这些元注解的类型及作用如下

* @Documented 表明它会被 javadoc 之类的工具处理, 这样最终注解类型信息也会被包括在生成的文档中
* @Retention 注解的保存策略，主要有以下三种
* * RetentionPolicy.SOURCE 源代码级别的注解，表示指定的注解只在编译期可见，并不会写入字节码文件，Override, SuppressWarnings 就属于此类型，这类注解对于程序员来说主要起到在编译时提醒的作用，在运行保存意义并不大，所以最终并不会被编译入字节码文件中
    * RetentionPolicy.RUNTIME  表示注解会被编译入最终的字符码文件中，JVM 启动后也会读入注解，这样我们在运行时就可以通过反射来获取这些注解，根据这些注解来做相关的操作，**这是多数自定义注解使用的保存策略**，这里可能大家有个疑问，为啥 Deprecated 被标为 RUNTIME 呢，对于程序员来说，理论上来说只关心调用的类，方法等是否 Deprecated 就够了，运行时获取有啥意义呢，考虑这样一种场景，假设你想在生产上统计过时的方法被调用的频率以评估你工程的坏味道或作为重构参考，此时这个注解是不是派上用场了。
    * RetentionPolicy.CLASS 注解会被编译入最终的字符码文件，但并不会载入 JVM 中（在类加载的时候注解会被丢弃），这种保存策略不常用，主要用在字节码文件的处理中。
* @Target 表示该注解可以用在什么地方，默认情况下可以用在任何地方，该注解的作用域主要通过 value 来指定，这里列举几个比较常见的类型：
* * FIELD 作用于属性
    * METHOD 作用于方法
    * ElementType.TYPE: 作用于类、接口\(包括注解类型\) 或 enum 声明
* @Inherited - 标记这个注解是继承于哪个注解类\(默认 注解并没有继承于任何子类\)

再来看 @interface， 这个是干啥用的，其实如果你反编译之后就会发现在字节码中编译器将其编码成了如下内容。

```text
public interface Override extends Annotation {   
}
```

Annotation 是啥

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/09/03/640-20200903114043278-114043.png)

我们可以看出**注解的本质其实是继承了 Annotation 这个接口的接口**，并且辅以 Retention，Target 这些规范注解运行时行为，作用域等的元注解。

Deprecated 注解中没有定义属性，其实如果需要注解是可以定义属性的，比如 Deprecated 注解可以定义一个 value 的属性，在声明注解的时候可以指定此注解的 value 值

```text
@Documented
@Retention(RetentionPolicy.RUNTIME)
@Target(value={CONSTRUCTOR, FIELD, LOCAL_VARIABLE, METHOD, PACKAGE, PARAMETER, TYPE})
public @interface Deprecated {
    String value() default "";
}
```

这样我将此注解应用于属性等地方时，可以指定此 value 值，如下所示

```text
public class Person {
    @Deprecated(value = "xxx")
    private String tail;
}
```

如果注解的保存策略为 RetentionPolicy.RUNTIME，我们就可以用如下方式在运行时获取注解，进而获取注解的属性值等

```text
field.getAnnotation(Deprecated.class);
```

### 巧用注解解决日志脱敏问题

上文简述了注解的原理与写法，接下来我们来看看如何用注解来实现我们的日志脱敏。

首先我们要定义一下脱敏的注解，由于此注解需要在运行时被取到，所以保存策略要为 **RetentionPolicy.RUNTIME**，另外此注解要应用于 phone，idcard 这些字段，所以@Target 的值为 ElementType.FIELD，另外我们注意到，像电话号码，身份证这些字段虽然都要脱敏，但是它们的脱敏策略不一样，所以我们需要为此注解定义一个属性，这样可以指定它的属性属于哪种脱敏类型，我们定义的脱敏注解如下：

```text
// 敏感信息类型
public enum SensitiveType {
    ID_CARD, PHONE, NAME, IMG_BASE64
}

@Target({ ElementType.FIELD })
@Retention(RetentionPolicy.RUNTIME)
public @interface SensitiveInfo {
    SensitiveType type();
}
```

定义好了注解，现在就可以为我们的敏感字段指定注解及其敏感信息类型了，如下

```text
public class Request {
    @SensitiveInfo(type = SensitiveType.NAME)
    private String name;
    @SensitiveInfo(type = SensitiveType.ID_CARD)
    private String idcard;
    @SensitiveInfo(type = SensitiveType.PHONE)
    private String phone;
    @SensitiveInfo(type = SensitiveType.IMG_BASE64)
    private String imgBase64;
}
```

为属性指定好了注解，该怎么根据注解来实现相应敏感字段类型的脱敏呢，可以用反射，先用反射获取类的每一个 Field,再判定 Field 上是否有相应的注解，若有，再判断此注解是针对哪种敏感类型的注解，再针对相应字段做相应的脱敏操作，直接上代码，注释写得很清楚了，相信大家应该能看懂

```text
private static ValueFilter getValueFilter() {
        return (obj, key, value) -> {
            // obj-对象 key-字段名 value-字段值
            try {
                // 通过反射获取获取每个类的属性
                Field[] fields = obj.getClass().getDeclaredFields();
                for (Field field : fields) {
                    if (!field.getName().equals(key)) {
                        continue;
                    }
                    // 判定属性是否有相应的 SensitiveInfo 注解
                    SensitiveInfo annotation = field.getAnnotation(SensitiveInfo.class);
                    // 若有，则执行相应字段的脱敏方法
                    if (null != annotation) {
                        switch (annotation.type()) {
                            case PHONE:
                                return 电话脱敏;
                            case ID_CARD:
                                return 身份证脱敏;
                            case NAME:
                                return 姓名脱敏;
                            case IMG_BASE64:
                                return ""; // 图片的 base 64 不展示，直接返回空
                            default:
                                // 这里可以抛异常
                        }
                    }
                    }
                }
            } catch (Exception e) {
                log.error("To JSON String fail", e);
            }
            return value;
        };
    }
```

有人可能会说了，使用注解的方式来实现脱敏代码量翻了一倍不止，看起来好像不是很值得，其实不然，之前的方式，脱敏规则与某个字段名强藕合，可维护性不好，而用注解的方式，就像工程中出现的 phone, tel，telephone 这些都属于电话脱敏类型的，只要统一标上 **@SensitiveInfo\(type = SensitiveType.PHONE\)**  这样的注解即可，而且后续如有新的脱敏类型，只要重新加一个 SensitiveType 的类型即可，**可维护性与扩展性大大增强**。所以在这类场景中，使用注解是强烈推荐的。

### 注解的高级应用-利用注解消除重复代码

在与银行对接的过程中，银行提供了一些 API 接口，对参数的序列化有点特殊，不使用 JSON，而是需要我们把参数依次拼在一起构成一个大字符串。

* 按照银行提供的 API 文档的顺序，把所有参数构成定长的数据，然后拼接在一起作为整个字符串。
* 因为每一种参数都有固定长度，未达到长度时需要做填充处理：
* * 字符串类型的参数不满长度部分需要以下划线右填充，也就是字符串内容靠左；
    * 数字类型的参数不满长度部分以 0 左填充，也就是实际数字靠右；
    * 货币类型的表示需要把金额向下舍入 2 位到分，以分为单位，作为数字类型同样进行左填充。
* 对所有参数做 MD5 操作作为签名（为了方便理解，Demo 中不涉及加盐处理）。简单看两个银行的接口定义

1、创建用户

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/09/03/640-20200903114043555-114043.png)在这里插入图片描述

2、支付接口![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/09/03/640-20200903114043773-114043.png)

常规的做法是为每个接口都根据之前的规则填充参数，拼接，验签，以以上两个接口为例，先看看常规做法

创建用户与支付的请求如下：

```text
// 创建用户 POJO
@Data
public class CreateUserRequest { 
    private String name; 
    private String identity; 
    private String mobile;
    private int age;
}

// 支付 POJO
@Data
public class PayRequest { 
    private long userId; 
    private BigDecimal amount;
}

public class BankService {

    //创建用户方法
    public static String createUser(CreateUserRequest request) throws IOException {
        StringBuilder stringBuilder = new StringBuilder();
        //字符串靠左，多余的地方填充_
        stringBuilder.append(String.format("%-10s", request.getName()).replace(' ', '_'));
        //字符串靠左，多余的地方填充_
        stringBuilder.append(String.format("%-18s", request.getIdentity()).replace(' ', '_'));
        //数字靠右，多余的地方用0填充
        stringBuilder.append(String.format("%05d", age));
        //字符串靠左，多余的地方用_填充
        stringBuilder.append(String.format("%-11s", mobile).replace(' ', '_'));
        //最后加上MD5作为签名
        stringBuilder.append(DigestUtils.md2Hex(stringBuilder.toString()));
        return Request.Post("http://baseurl/createUser")
                .bodyString(stringBuilder.toString(), ContentType.APPLICATION_JSON)
                .execute().returnContent().asString();
    }

    //支付方法
    public static String pay(PayRequest request) throws IOException {
        StringBuilder stringBuilder = new StringBuilder();
        //数字靠右，多余的地方用0填充
        stringBuilder.append(String.format("%020d", request.getUserId()));
        //金额向下舍入2位到分，以分为单位，作为数字靠右，多余的地方用0填充
 stringBuilder.append(String.format("%010d",request.getAmount().setScale(2,RoundingMode.DOWN).multiply(new                                                                                  BigDecimal("100")).longValue())); 
        //最后加上MD5作为签名
        stringBuilder.append(DigestUtils.md2Hex(stringBuilder.toString()));
        return Request.Post("http://baseurl//pay")
                .bodyString(stringBuilder.toString(), ContentType.APPLICATION_JSON)
                .execute().returnContent().asString();
    }
}
```

可以看到光写这两个请求，逻辑就有很多重复的地方：

1、 字符串，货币，数字三种类型的格式化逻辑大量重复，以处理字符串为例

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/09/03/640-20200903114044167-114044.png) 可以看到，格式化字符串的的处理只是每个字段的长度不同，其余格式化规则完全一样，但在上文中我们却为每一个字符串都整了一套相同的处理逻辑，这套拼接规则完全可以抽出来（因为只是长度不一样，拼接规则是一样的）

2、 处理流程中字符串拼接、加签和发请求的逻辑，在所有方法重复。

3、 由于每个字段参与拼接的顺序不一样，这些需要我们人肉硬编码保证这些字段的顺序，维护成本极大，而且很容易出错，想象一下如果参数达到几十上百个，这些参数都需要按一定顺序来拼接，如果要人肉来保证，很难保证正确性，而且重复工作太多，得不偿失

接下来我们来看看如何用注解来极大简化我们的代码。

1、 首先对于每一个调用接口来说，它们底层都是需要请求网络的，只是请求方法不一样，针对这一点 ，我们可以搞一个如下针对接口的注解

```text
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.TYPE)
@Documented
@Inherited
public @interface BankAPI { 
    String url() default "";
    String desc() default ""; 
}
```

这样在网络请求层即可统一通过注解获取相应接口的方法名

2、 针对每个请求接口的 POJO,我们注意到每个属性都有 **类型（字符串/数字/货币）**，**长度**，**顺序**这三个属性，所以可以定义一个注解，包含这三个属性，如下

```text
@Retention(RetentionPolicy.RUNTIME)
@Target(ElementType.FIELD)
@Documented
@Inherited
public @interface BankAPIField {
    int order() default -1;
    int length() default -1;
    String type() default ""; // M代表货币，S代表字符串，N代表数字
}
```

接下来我们将上文中定义的注解应用到上文中的请求 POJO 中

对于创建用户请求

```text
@BankAPI(url = "/createUser", desc = "创建用户接口")
@Data
public class CreateUserAPI extends AbstractAPI {
    @BankAPIField(order = 1, type = "S", length = 10)
    private String name;
    @BankAPIField(order = 2, type = "S", length = 18)
    private String identity;
    @BankAPIField(order = 4, type = "S", length = 11) //注意这里的order需要按照API表格中的顺序
    private String mobile;
    @BankAPIField(order = 3, type = "N", length = 5)
    private int age;
}
```

对于支付接口

```text
@BankAPI(url = "/bank/pay", desc = "支付接口")
@Data
public class PayAPI extends AbstractAPI {
    @BankAPIField(order = 1, type = "N", length = 20)
    private long userId;
    @BankAPIField(order = 2, type = "M", length = 10)
    private BigDecimal amount;
}
```

接下来利用注解来调用的流程如下

1. 根据反射获取类的 Field 数组，然后再根据 Field 的 BankAPIField 注解中的 order 值对 Field 进行排序
2. 对排序后的 Field 依次进行遍历，首先判断其类型，然后根据类型再对其值格式化，如判断为"S",则按接口要求字符串的格式对其值进行格式化，将这些格式化后的 Field 值依次拼接起来并进行签名
3. 拼接后就是发请求了，此时再拿到 POJO 类的注解，获取注解 BankAPI  的 url 值，将其与 baseUrl 组合起来即可构成一个完整的的 url，再加上第 2 步中拼接字符串即可构造一个完全的请求

代码如下：

```text
private static String remoteCall(AbstractAPI api) throws IOException {
    //从BankAPI注解获取请求地址
    BankAPI bankAPI = api.getClass().getAnnotation(BankAPI.class);
    bankAPI.url();
    StringBuilder stringBuilder = new StringBuilder();
    Arrays.stream(api.getClass().getDeclaredFields()) //获得所有字段
            .filter(field -> field.isAnnotationPresent(BankAPIField.class)) //查找标记了注解的字段
            .sorted(Comparator.comparingInt(a -> a.getAnnotation(BankAPIField.class).order())) //根据注解中的order对字段排序
            .peek(field -> field.setAccessible(true)) //设置可以访问私有字段
            .forEach(field -> {
                //获得注解
                BankAPIField bankAPIField = field.getAnnotation(BankAPIField.class);
                Object value = "";
                try {
                    //反射获取字段值
                    value = field.get(api);
                } catch (IllegalAccessException e) {
                    e.printStackTrace();
                }
                //根据字段类型以正确的填充方式格式化字符串
                switch (bankAPIField.type()) {
                    case "S": {
                        stringBuilder.append(String.format("%-" + bankAPIField.length() + "s", value.toString()).replace(' ', '_'));
                        break;
                    }
                    case "N": {
                        stringBuilder.append(String.format("%" + bankAPIField.length() + "s", value.toString()).replace(' ', '0'));
                        break;
                    }
                    case "M": {
                        if (!(value instanceof BigDecimal))
                            throw new RuntimeException(String.format("{} 的 {} 必须是BigDecimal", api, field));
                        stringBuilder.append(String.format("%0" + bankAPIField.length() + "d", ((BigDecimal) value).setScale(2, RoundingMode.DOWN).multiply(new BigDecimal("100")).longValue()));
                        break;
                    }
                    default:
                        break;
                }
            });
    //签名逻辑
    stringBuilder.append(DigestUtils.md2Hex(stringBuilder.toString()));
    String param = stringBuilder.toString();
    long begin = System.currentTimeMillis();
    //发请求
    String result = Request.Post("http://localhost:45678/reflection" + bankAPI.url())
            .bodyString(param, ContentType.APPLICATION_JSON)
            .execute().returnContent().asString();
    log.info("调用银行API {} url:{} 参数:{} 耗时:{}ms", bankAPI.desc(), bankAPI.url(), param, System.currentTimeMillis() - begin);
    return result;
}
```

现在再来看一下创建用户和付款的逻辑

```text
//创建用户方法
public static String createUser(CreateUserAPI request) throws IOException {
    return remoteCall(request);
}
//支付方法
public static String pay(PayAPI request) throws IOException {
    return remoteCall(request);
}
```

可以看到所有的请求现在都只要统一调用 remoteCall 这个方法即可，remoteCall 这个方法统一了所有请求的逻辑，省略了巨量无关的代码，让代码的可维护性大大增强！**使用注解和反射让我们可以对这类结构性的问题进行通用化处理**，确实 Cool!

### 总结

如果说反射给了我们在不知晓类结构的情况下按照固定逻辑处理类成员的能力的话，注解则是扩展补充了这些成员的元数据的能力，使用得我们在利用反射实现通用逻辑的时候，可以从外部获取更多我们关心的数据，进而对这些数据进行通用的处理，巧用反射，确实能让我们达到事半功倍的效果，能极大的减少重复代码，有效解藕，使扩展性大大提升。

额外提一句，上文中注解的高级应用的例子来自文末的参考链接中的一个例子，如果大家没有订阅教程，可以加我好友（geekoftaste），邀请你试读此文

巨人的肩膀

[https://time.geekbang.org/column/article/228964](https://time.geekbang.org/column/article/228964)

