# 你说，一个Java字符串到底有多少个字符?

[https://mp.weixin.qq.com/s/DnJO2AMYJmLHS25tX2-jeQ](https://mp.weixin.qq.com/s/DnJO2AMYJmLHS25tX2-jeQ)

作者 \| 鸟窝

来源 \| [https://urlify.cn/qYNR3q](https://urlify.cn/qYNR3q)

依照Java的文档， Java中的字符内部是以UTF-16编码方式表示的，最小值是 `\u0000` \(`0`\),最大值是`\uffff`\(`65535`\)， 也就是一个字符以2个字节来表示，难道Java最多只能表示 65535个字符？

> **char**: The char data type is a single 16-bit Unicode character. It has a minimum value of '\u0000' \(or 0\) and a maximum value of '\uffff' \(or 65,535 inclusive\).
>
> from The Java™ Tutorials

首先，让我们先看个例子：

```java
public class Main {
    public static void main(String[] args) {
        // 中文常见字
        String s = "你好";
        System.out.println("1. string length =" + s.length());
        System.out.println("1. string bytes length =" + s.getBytes().length);
        System.out.println("1. string char length =" + s.toCharArray().length);
        System.out.println();
        // emojis
        s = "👦👩";
        System.out.println("2. string length =" + s.length());
        System.out.println("2. string bytes length =" + s.getBytes().length);
        System.out.println("2. string char length =" + s.toCharArray().length);
        System.out.println();
        // 中文生僻字
        s = "𑃁妹";
        System.out.println("3. string length =" + s.length());
        System.out.println("3. string bytes length =" + s.getBytes().length);
        System.out.println("3. string char length =" + s.toCharArray().length);
        System.out.println();
    }
}
```

运行这个程序，你觉得输出结果是什么？

**输出结果:**

```text
1. string length =2
1. string bytes length =6
1. string char length =2
2. string length =4
2. string bytes length =8
2. string char length =4
3. string length =3
3. string bytes length =7
3. string char length =3
```

我们知道， `String.getBytes()`如果不指定编码格式，Java会使用操作系统的编码格式得到字节数组，在我的MacOS中，默认使用`UTF-8`作为字符编码\(`locale`命令可以查看操作系统的编码\)，所以在我的机器运行，`String.getBytes()`会返回`UTF-8`编码的字节数组。

`String.length`返回`Unicode code units`的长度。

`String.toCharArray`返回字符数组。

我们设置的字符串都是两个unicode字符，输出结果：

* 普通的中文字：字符串的长度是2，每个中文字按`UTF-8`编码是三个字节，字符数组的长度看起来也没问题
* emojis字符：我们设置了**两个**emojis字符，男女头像。结果字符串的长度是**4**, `UTF-8`编码8个字节，字符数组的长度是**4**
* 生僻的中文字：我们设置了两个中文字，其中一个是生僻的中文字。结果字符串的长度是**3**， `UTF-8`编码7个字节，字符数组的长度是**3**

看起来字符串的字符数和我们预期的有点不一样，我们的字符串只有两个unicode字符, 可是输出结果有时候是2，有时候是3， 有时候是4，为什么呢？这还得从Java的历史说起。

Java最初设计的`Charactor`用两个字节来表示unicode字符，这没有问题， 因为最初unicode中的字符还比较少， Java 1.1之前采用`Unicode version 1.1.5`, JDK 1.1中支持`Unicode 2.0`, JDK 1.1.7支持`Unicode 2.1`, Java SE 1.4 支持 `Unicode 3.0`, Java SE 5.0开始支持`Unicode 4.0`。

直到`Unicode 3.0`, Java用两个字节来表示unicode字符还没有问题，因为`Unicode 3.0`最多49,259个字符， 两个字节可以表示65,535个字符，还足够容的下所有的uicode3.0字符。

但是`Unicode 4.0`\(事实上自Unicode 3.1\), 字符集进行很大的扩充，已经达到了96,447个字符，`Unicode 11.0`已经包含137,374个字符。

在Unicode中，为每一个字符对应一个编码点\(一个整数\)，用 U+紧跟着十六进制数表示。所有字符按照使用上的频繁度划分为 17 个平面（编号为 0-16），即基本的多语言平面和增补平面。基本的多语言平面（英文为 Basic Multilingual Plane，简称 BMP）又称平面 0，收集了使用最广泛的字符。

这样一来，Java的Charactor的两个字节的设计，已经不足以容纳所有的Unicode 4的字符， 所以可能需要4个字节才能表示扩展字符，所以现在的Charactor代表的已经不再是**一个字符** \(代码点 code point\), 而是一个代码单元\(code unit\)。

* **Code Point**: 代码点，一个字符的数字表示。一个字符集一般可以用一张或多张由多个行和多个列所构成的二维表来表示。二维表中行与列交叉的点称之为代码点，每个码点分配一个唯一的编号数字，称之为码点值或码点编号，除开某些特殊区域\(比如代理区、专用区\)的非字符代码点和保留代码点，每个代码点唯一对应于一个字符。从`U+0000` 到 `U+10FFFF`。
* **Code Unit**：代码单元，是指一个已编码的文本中具有最短的比特组合的单元。对于 UTF-8 来说，代码单元是 8 比特长；对于 UTF-16 来说，代码单元是 16 比特长。换一种说法就是 UTF-8 的是以一个字节为最小单位的，UTF-16 是以两个字节为最小单位的。

Java的字符在内部以UTF-16编码方式来表示，`String.length`返回的是`Code Unit`的长度，而不再是Unicode中字符的长度。对于传统的`BMP`平面的代码点，`String.length`和我们传统理解的字符的数量是一致的，对于扩展的字符，`String.length`可能是我们理解的字符长度的两倍。

有可能你会问， 对于一个UTF-16编码的扩展字符，它以4个字节来表示，那么前两个字节会不会和`BMP`平面冲突，导致程序不知道它是扩展字符还是`BMP`平面的字符？

其实是不会的， 幸运的是， 在`BMP`平面中， `U+D800`到`U+DFFF`之间的码位是永久保留不映射到Unicode字符，UTF-16就利用保留下来的0xD800-0xDFFF区块的码位来对辅助平面的字符的码位进行编码。

UTF-16编码中，辅助平面中的码位从`U+10000`到`U+10FFFF`，共计FFFFF个,需要20位来表示。第一个整数（两个字节，称为前导代理）要容纳上述20位的前10位，第二个整数（称为后尾代理）容纳上述20位的后10位。前导代理的值的范围是`0xD800`到`0xDBFF`,后尾代理的`0xDC00`~`0xDFFF`。可以看到前导代理和后尾代理的范围都落在了`BMP`平面中不用来映射的码位，所以不会产生冲突，而且前导代理和后尾代理也没有重合。这样我们得到两个字节的，就可以直接判断它是否是`BMP`平面的字符，还是扩展字符中的前导代理还是后尾代码。

国外的有些用户用emojis字符做自己的昵称，导致有些系统不能正确的显示出来，这是因为这些系统粗暴的使用`Charactor`来表示，在显示的时候截断的时候有时候可能不是在正确的代码点上进行截断。

我们在进行字符串截取的时候,比如`String.substring`有可能会踩到一些坑，尤其经常使用的emojis字符。

自 Java 1.5 java.lang.String就提供了`Code Point`方法， 用来获取完整的Unicode字符和Unicode字符数量:

* public int codePointAt\(int index\)
* public int codePointBefore\(int index\)
* public int codePointCount\(int beginIndex, int endIndex\)

注意这些方法中的index使用的是`code unit`值。

