# Java反射是什么？看这篇绝对会了！

{% embed url="https://mp.weixin.qq.com/s/2oRXI6Pnj0ylP9nujVuAwA" %}



作者 \| 火星十一郎

来源 \| [https://www.cnblogs.com/hxsyl](https://www.cnblogs.com/hxsyl)

**一.概念**

反射就是把Java的各种成分映射成相应的Java类。

[Class](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247493426&idx=3&sn=22bab14abef8defb20abed4205cbf6a1&chksm=eb506204dc27eb1284be9fa9406a9afeb3dbbe7117c00db7f5a44500dddd3eb812bbdc61f3b1&scene=21#wechat_redirect)类的构造方法是private，由[JVM](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247489332&idx=2&sn=65de5886e13b98116c8432d7d10ae4bc&chksm=eb539202dc241b14010f70edf89dc37c7629b5e2b7add50fd3f58070ecf2c14260196bf147d8&scene=21#wechat_redirect)创建。

> 反射是java语言的一个特性，它允程序在运行时（注意不是编译的时候）来进行自我检查并且对内部的成员进行操作。例如它允许一个java的类获取他所有的成员变量和方法并且显示出来。Java 的这一能力在实际应用中也许用得不是很多，但是在其它的程序设计语言中根本就不存在这一特性。例如，Pascal、C 或者 C++ 中就没有办法在程序中获得函数定义相关的信息。（来自Sun）

JavaBean 是 reflection 的实际应用之一，它能让一些工具可视化的操作软件组件。这些工具通过 reflection 动态的载入并取得 Java 组件\(类\) 的属性。

反射是从1.2就有的，后面的三大框架都会用到反射机制,涉及到类"Class",无法直接new CLass\(\)，其对象是内存里的一份字节码.

Class 类的实例表示正在运行的 Java 应用程序中的类和接口。枚举是一种类，注释是一种接口。每个数组属于被映射为 Class 对象的一个类，所有具有相同元素类型和维数的数组都共享该 Class 对象。

基本的 Java类型（boolean、byte、char、short、int、long、float 和 double）和关键字 void 也表示为 Class 对象。Class 没有公共构造方法。

Class 对象是在加载类时由 Java 虚拟机以及通过调用类加载器中的 defineClass 方法自动构造的。

```text
Person p1 = new Person();  
//下面的这三种方式都可以得到字节码  
CLass c1 = Date.class();  
p1.getClass();   
//若存在则加载，否则新建,往往使用第三种,类的名字在写源程序时不需要知道，到运行时再传递过来  
Class.forName("java.lang.String");
```

[Class.forName\(\)](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247493426&idx=3&sn=22bab14abef8defb20abed4205cbf6a1&chksm=eb506204dc27eb1284be9fa9406a9afeb3dbbe7117c00db7f5a44500dddd3eb812bbdc61f3b1&scene=21#wechat_redirect)字节码已经加载到java虚拟机中，去得到字节码；java虚拟机中还没有生成字节码 用类加载器进行加载，加载的字节码缓冲到虚拟机中。

考虑下面这个简单的例子，让我们看看 reflection 是如何工作的。

```text
import java.lang.reflect.*;    

public class DumpMethods {    
   public static void main(String args[]) {    
      try {    
           Class c = Class.forName("java.util.Stack");    

           Method m[] = c.getDeclaredMethods();    

           for (int i = 0; i < m.length; i++)    
               System.out.println(m[i].toString());    
      }    
      catch (Throwable e){    
            System.err.println(e);    
      }    
   }    
}  

public synchronized java.lang.Object java.util.Stack.pop()   
public java.lang.Object java.util.Stack.push(java.lang.Object)   
public boolean java.util.Stack.empty()   
public synchronized java.lang.Object java.util.Stack.peek()   
public synchronized int java.util.Stack.search(java.lang.Object)
```

这样就列出了java.util.Stack 类的各方法名以及它们的限制符和返回类型。这个程序使用 [Class.forName](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247493426&idx=3&sn=22bab14abef8defb20abed4205cbf6a1&chksm=eb506204dc27eb1284be9fa9406a9afeb3dbbe7117c00db7f5a44500dddd3eb812bbdc61f3b1&scene=21#wechat_redirect) 载入指定的类，然后调用 getDeclaredMethods 来获取这个类中定义了的方法列表。java.lang.reflect.Methods 是用来描述某个类中单个方法的一个类。

以下示例使用 Class 对象来显示对象的类名：

```text
void printClassName(Object obj) {  
       System.out.println("The class of " + obj +  
                     " is " + obj.getClass().getName());  
}
```

还可以使用一个类字面值（JLS Section 15.8.2）来获取指定类型（或 void）的 Class 对象。例如：

```text
System.out.println("The name of class Foo is: "+Foo.class.getName());
```

在没有对象实例的时候，主要有两种办法。

```text
//获得类类型的两种方式          
Class cls1 = Role.class;          
Class cls2 = Class.forName("yui.Role");
```

注意第二种方式中，forName中的参数一定是完整的类名（包名+类名），并且这个方法需要捕获异常。现在得到cls1就可以创建一个Role类的实例了，利用Class的newInstance方法相当于调用类的默认的构造器。

```text
Object o = cls1.newInstance();   
//创建一个实例          
//Object o1 = new Role();   //与上面的方法等价
```

### 二.常用方法

**1.isPrimitive\(判断是否是基本类型的字节码\)**

```text
public class TestReflect {  
    public static void main(String[] args) {  
        // TODO Auto-generated method stub  
        String str = "abc";  
        Class cls1 = str.getClass();  
        Class cls2 = String.class;  
        Class cls3 = null;//必须要加上null  
        try {  
            cls3 = Class.forName("java.lang.String");  
        } catch (ClassNotFoundException e) {  
            // TODO Auto-generated catch block  
            e.printStackTrace();  
        }  
        System.out.println(cls1==cls2);  
        System.out.println(cls1==cls3);  

        System.out.println(cls1.isPrimitive());  
        System.out.println(int.class.isPrimitive());//判定指定的 Class 对象是否表示一个基本类型。  
        System.out.println(int.class == Integer.class);  
        System.out.println(int.class == Integer.TYPE);  
        System.out.println(int[].class.isPrimitive());  
        System.out.println(int[].class.isArray());  
    }  
}
```

结果：

```text
true  
true  
false  
true  
false  
true  
false  
true
```

**2.getConstructor和getConstructors\(\)**

java中构造方法没有先后顺序，通过类型和参数个数区分。

```text
public class TestReflect {  
    public static void main(String[] args) throws SecurityException, NoSuchMethodException {  
        // TODO Auto-generated method stub  
        String str = "abc";  

        System.out.println(String.class.getConstructor(StringBuffer.class));  
    }  
}
```

**3.Filed类代表某一类中的一个成员变量。**

```text
import java.lang.reflect.Field;  
public class TestReflect {  
    public static void main(String[] args) throws SecurityException, NoSuchMethodException, NoSuchFieldException, IllegalArgumentException, Exception {  
        ReflectPointer rp1 = new ReflectPointer(3,4);  
        Field fieldx = rp1.getClass().getField("x");//必须是x或者y  
        System.out.println(fieldx.get(rp1));  

        /*  
         * private的成员变量必须使用getDeclaredField，并setAccessible(true),否则看得到拿不到  
         */  
        Field fieldy = rp1.getClass().getDeclaredField("y");  
        fieldy.setAccessible(true);//暴力反射  
        System.out.println(fieldy.get(rp1));  

    }  
}  

class ReflectPointer {  

    public int x = 0;  
    private int y = 0;  

    public ReflectPointer(int x,int y) {//alt + shift +s相当于右键source  
        super();  
        // TODO Auto-generated constructor stub  
        this.x = x;  
        this.y = y;  
    }  
}
```

### 三.典型例题

**1.将所有String类型的成员变量里的b改成a。**

```text
import java.lang.reflect.Field;  
public class TestReflect {  
    public static void main(String[] args) throws SecurityException, NoSuchMethodException, NoSuchFieldException, IllegalArgumentException, Exception {  
        ReflectPointer rp1 = new ReflectPointer(3,4);  
        changeBtoA(rp1);  
        System.out.println(rp1);  

    }  

    private static void changeBtoA(Object obj) throws RuntimeException, Exception {  
        Field[] fields = obj.getClass().getFields();  

        for(Field field : fields) {  
            //if(field.getType().equals(String.class))  
            //由于字节码只有一份,用equals语义不准确  
            if(field.getType()==String.class) {  
                String oldValue = (String)field.get(obj);  
                String newValue = oldValue.replace('b', 'a');  
                field.set(obj,newValue);  
            }  
        }  
    }  
}  

class ReflectPointer {  

    private int x = 0;  
    public int y = 0;  
    public String str1 = "ball";  
    public String str2 = "basketball";  
    public String str3 = "itcat";  

    public ReflectPointer(int x,int y) {//alt + shift +s相当于右键source  
        super();  
        // TODO Auto-generated constructor stub  
        this.x = x;  
        this.y = y;  
    }  

    @Override  
    public String toString() {  
        return "ReflectPointer [str1=" + str1 + ", str2=" + str2 + ", str3="  
                + str3 + "]";  
    }  
}
```

**2.写一个程序根据用户提供的类名，调用该类的里的main方法。**

为什么要用反射的方式呢？

```text
import java.lang.reflect.Field;  
import java.lang.reflect.Method;  

public class TestReflect {  
    public static void main(String[] args) throws SecurityException, NoSuchMethodException, NoSuchFieldException, IllegalArgumentException, Exception {  
        String str = args[0];  
        /*  
         * 这样会数组角标越界，因为压根没有这个字符数组  
         * 需要右键在run as-configurations-arguments里输入b.Inter（完整类名）  
         *   
         */  
        Method m = Class.forName(str).getMethod("main",String[].class);  
        //下面这两种方式都可以,main方法需要一个参数  

        m.invoke(null, new Object[]{new String[]{"111","222","333"}});  
        m.invoke(null, (Object)new String[]{"111","222","333"});//这个可以说明，数组也是Object  
        /*  
         * m.invoke(null, new String[]{"111","222","333"})  
         * 上面的不可以,因为java会自动拆包  
         */  
    }  
}  

class Inter {  
    public static void main(String[] args) {  
        for(Object obj : args) {  
            System.out.println(obj);  
        }  
    }  
}
```

**3.模拟** [**instanceof**](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247483795&idx=1&sn=20613cdddf5446b0cdcc48ef440ad8c7&chksm=eb5384a5dc240db3a5070cedd0f764ecbe3056b609b6ff11854f2dbd91a017f71c52147c473a&scene=21#wechat_redirect) **操作符**

```text
class S {    
}     

public class IsInstance {    
   public static void main(String args[]) {    
      try {    
           Class cls = Class.forName("S");    
           boolean b1 = cls.isInstance(new Integer(37));    
           System.out.println(b1);    
           boolean b2 = cls.isInstance(new S());    
           System.out.println(b2);    
      }    
      catch (Throwable e) {    
           System.err.println(e);    
      }    
   }    
}
```

在这个例子中创建了一个S 类的 Class 对象，然后检查一些对象是否是S的实例。Integer\(37\) 不是，但 new S\(\)是。

推荐阅读：[instanceof、isInstance、isAssignableFrom](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247483795&idx=1&sn=20613cdddf5446b0cdcc48ef440ad8c7&chksm=eb5384a5dc240db3a5070cedd0f764ecbe3056b609b6ff11854f2dbd91a017f71c52147c473a&scene=21#wechat_redirect)

### 四.Method类

代表类（不是对象）中的某一方法。

```text
import java.lang.reflect.Field;  
import java.lang.reflect.Method;  
/*  
 * 人在黑板上画圆，涉及三个对象,画圆需要圆心和半径，但是是私有的，画圆的方法  
 * 分配给人不合适。  
 *   
 * 司机踩刹车，司机只是给列车发出指令，刹车的动作还需要列车去完成。  
 *   
 * 面试经常考面向对象的设计，比如人关门，人只是去推门。  
 *   
 * 这就是专家模式：谁拥有数据，谁就是专家,方法就分配给谁  
 */  
public class TestReflect {  
    public static void main(String[] args) throws SecurityException, NoSuchMethodException, NoSuchFieldException, IllegalArgumentException, Exception {  
        String str = "shfsfs";  
        //包开头是com表示是sun内部用的，java打头的才是用户的  
        Method mtCharAt = String.class.getMethod("charAt", int.class);  
        Object ch = mtCharAt.invoke(str,1);//若第一个参数是null，则肯定是静态方法  
        System.out.println(ch);  

        System.out.println(mtCharAt.invoke(str, new Object[]{2}));//1.4语法  

    }  

}
```

### 五.数组的反射

Array工具类用于完成数组的反射操作。

同类型同纬度有相同的字节码。

int.class和Integer.class不是同一份字节码，Integer.TYPE，TYPE代表包装类对应的基本类的字节码 int.class==Integer.TYPE。

```text
import java.util.Arrays;  

/*  
 * 从这个例子看出即便字节码相同但是对象也不一定相同,根本不是一回事  
 *   
 */  
public class TestReflect {  
    public static void main(String[] args) throws SecurityException, NoSuchMethodException, NoSuchFieldException, IllegalArgumentException, Exception {  
        int[] a = new int[3];  
        int[] b = new int[]{4,5,5};//直接赋值后不可以指定长度，否则ＣＥ  
        int[][] c = new int[3][2];  
        String[] d = new String[]{"jjj","kkkk"};  
        System.out.println(a==b);//false  
        System.out.println(a.getClass()==b.getClass());//true  
        //System.out.println(a.getClass()==d.getClass());    //比较字节码a和cd也没法比  
        System.out.println(a.getClass());//输出class [I  
        System.out.println(a.getClass().getName());//输出[I,中括号表示数组，I表示整数  

        System.out.println(a.getClass().getSuperclass());//输出class java.lang.Object  
        System.out.println(d.getClass().getSuperclass());//输出class java.lang.Object  

        //由于父类都是Object,下面都是可以的  
        Object obj1 = a;//不可是Object[]  
        Object obj2 = b;  
        Object[] obj3 = c;//基本类型的一位数组只可以当做Object，非得还可以当做Object[]  
        Object obj4 = d;  

        //注意asList处理int[]和String[]的区别  
        System.out.println(Arrays.asList(b));//1.4没有可变参数，使用的是数组,[[I@1bc4459]  
        System.out.println(Arrays.asList(d));//[jjj, kkkk]  

    }  
}
```

### 六.结束语

以上就是反射机制的简单的使用，显然学过spring的朋友一定明白了，为什么可以通过配置文件就可以让我们获得指定的方法和变量，在我们创建对象的时候都是通过传进string实现的，就好像你需要什么，我们去为你生产，还有我们一直在用Object,这就说明java语言的动态特性，依赖性大大的降低了。

