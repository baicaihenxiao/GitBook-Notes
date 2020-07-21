# Mybatis 框架下 SQL 注入攻击的 3 种方式，真是防不胜防！

{% embed url="https://mp.weixin.qq.com/s/rj4iXf2-Fy4ek8UPzHHvkg" %}





本文授权转载请注明来自FreeBuf.COM

来源 \| [https://www.freebuf.com/vuls/240578.html](https://www.freebuf.com/vuls/240578.html)

### **前言**

**SQL注入漏洞作为WEB安全的最常见的漏洞之一，在java中随着预编译与各种ORM框架的使用，注入问题也越来越少。**

**新手代码审计者往往对Java Web应用的多个框架组合而心生畏惧，不知如何下手，希望通过**[**Mybatis**](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490411&idx=2&sn=34db2fb1e7e3fad0bdccabb35c247ed7&chksm=eb539e5ddc24174b6b47e65dd26f914b931b1fd7fc27dd21bbfa5bec30fd3abd89cb78315755&scene=21#wechat_redirect)**框架使用不当导致的SQL注入问题为例，能够抛砖引玉给新手一些思路。**

### **一、Mybatis的SQL注入**

[Mybatis的SQL语句可以基于注解的方式写在类方法上面，更多的是以xml的方式写到xml文件。](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490411&idx=2&sn=34db2fb1e7e3fad0bdccabb35c247ed7&chksm=eb539e5ddc24174b6b47e65dd26f914b931b1fd7fc27dd21bbfa5bec30fd3abd89cb78315755&scene=21#wechat_redirect)

[Mybatis中SQL语句需要我们自己手动编写或者用generator自动生成。编写xml文件时，Mybatis支持两种参数符号，一种是\#，另一种是$。比如：](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490411&idx=2&sn=34db2fb1e7e3fad0bdccabb35c247ed7&chksm=eb539e5ddc24174b6b47e65dd26f914b931b1fd7fc27dd21bbfa5bec30fd3abd89cb78315755&scene=21#wechat_redirect)

```text
<select id="queryAll"  resultMap="resultMap">  SELECT * FROM NEWS WHERE ID = #{id}</select>
```

## **使用预编译，$使用拼接SQL。**

[Mybatis](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490411&idx=2&sn=34db2fb1e7e3fad0bdccabb35c247ed7&chksm=eb539e5ddc24174b6b47e65dd26f914b931b1fd7fc27dd21bbfa5bec30fd3abd89cb78315755&scene=21#wechat_redirect)框架下易产生SQL注入漏洞的情况主要分为以下三种：

#### 1、模糊查询

```text
Select * from news where title like ‘%#{title}%’
```

在这种情况下使用\#程序会报错，新手程序员就把\#号改成了$,这样如果java代码层面没有对用户输入的内容做处理势必会产生SQL注入漏洞。

正确写法：

```text
select * from news where tile like concat(‘%’,#{title}, ‘%’)
```

#### 2、in 之后的多个参数

in之后多个id查询时使用\# 同样会报错，

```text
Select * from news where id in (#{ids})
```

正确用法为使用foreach，而不是将\#替换为$

```text
id in<foreach collection="ids" item="item" open="("separatosr="," close=")">#{ids} </foreach>
```

#### 3、order by 之后

这种场景应当在Java层面做映射，设置一个字段/表名数组，仅允许用户传入索引值。这样保证传入的字段或者表名都在白名单里面。需要注意的是在mybatis-generator自动生成的SQL语句中，order by使用的也是$，而like和in没有问题。

### **二、实战思路**

[我们使用一个开源的cms来分析，java sql注入问题适合使用反推，先搜索xml查找可能存在注入的漏洞点→反推到DAO→再到实现类→再通过调用链找到前台URL，找到利用点，话不多说走起](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490210&idx=2&sn=22f4c6fec0c8a04dbf3c6dddb3434ea1&chksm=eb539f94dc241682fc294d49811b6cf4584fd7cab35466a9554bc48945c232d587f9ebb1833a&scene=21#wechat_redirect)

**1、idea导入项目**

Idea首页 点击Get from Version Control，输入[https://gitee.com/mingSoft/MCMS.git](https://gitee.com/mingSoft/MCMS.git)

下载完成，等待maven把项目下载完成

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084533755-084534.jpg)

**2、搜索$关键字**

Ctrl+shift+F 调出Find in Path，筛选后缀xml，搜索$关键字

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084533847-084534.jpg)

根据文件名带Dao的xml为我们需要的，以IContentDao.xml为例，双击打开，ctrl +F 搜索$,查找到16个前三个为数据库选择，跳过，

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084533899-084534.jpg)

继续往下看到疑似order by 暂时搁置

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084533953-084534.jpg)

继续往下看发现多个普通拼接，此点更容易利用，我们以此为例深入，只查找ids从前端哪里传入

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534005-084534.jpg)

**3、搜索映射对象**

[Mybatis](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490411&idx=2&sn=34db2fb1e7e3fad0bdccabb35c247ed7&chksm=eb539e5ddc24174b6b47e65dd26f914b931b1fd7fc27dd21bbfa5bec30fd3abd89cb78315755&scene=21#wechat_redirect) 的select id对应要映射的对象名，我们以getSearchCount为关键字搜索映射的对象

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534094-084534.jpg)

搜到了IContentDao.java,IContentDaoimpl.java和McmsAction.java，分别对应映射的对象，对象的实现类和前端controler，直接跳转到controler类

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534154-084534.jpg)

发现只有categoryIds与目标参数ids相似，需进一步确认，返回到IContentDao.java按照标准流继续反推

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534304-084534.jpg)

找到ids为getSearchCount的最后一个参数，alt+f7查看调用链

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534354-084534.jpg)

调转到ContentBizImpl，确认前台参数为categoryIds

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534404-084534.jpg)

返回到McmsAction，参数由BasicUtil.getString接收，

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534490-084534.jpg)

跟进BasicUtil.getString

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534541-084534.jpg)

继续跳到SpringUtil.getRequest\(\)，前端未做处理，sql注入实锤

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534594-084534.jpg)

**4、漏洞确认**

项目运行起来，构造sql语句[http://localhost:8080/ms-mcms/mcms/search.do?categoryId=1%27\)  or+updatexml\(1,concat\(0x7e,\(SELECT+%40%40version\),0x7e\),1\)%23](http://localhost:8080/ms-mcms/mcms/search.do?categoryId=1%27%29%20%20or+updatexml%281,concat%280x7e,%28SELECT+%40%40version%29,0x7e%29,1%29%23) 得到mysql的版本5.7.27，验证注入存在。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/21/640-20200721084534800-084534.jpg)

### **三、总结**

以上就是[Mybatis](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490411&idx=2&sn=34db2fb1e7e3fad0bdccabb35c247ed7&chksm=eb539e5ddc24174b6b47e65dd26f914b931b1fd7fc27dd21bbfa5bec30fd3abd89cb78315755&scene=21#wechat_redirect)的sql注入审计的基本方法，我们没有分析的几个点也有问题，新手可以尝试分析一下不同的注入点来实操一遍，相信会有更多的收获。当我们再遇到类似问题时可以考虑：

> 1、[Mybatis](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490411&idx=2&sn=34db2fb1e7e3fad0bdccabb35c247ed7&chksm=eb539e5ddc24174b6b47e65dd26f914b931b1fd7fc27dd21bbfa5bec30fd3abd89cb78315755&scene=21#wechat_redirect)框架下审计SQL注入，重点关注在三个方面like，in和order by
>
> 2、xml方式编写sql时，可以先筛选xml文件搜索$,逐个分析，要特别注意mybatis-generator的order by注入
>
> 3、[Mybatis](http://mp.weixin.qq.com/s?__biz=MzI3ODcxMzQzMw==&mid=2247490411&idx=2&sn=34db2fb1e7e3fad0bdccabb35c247ed7&chksm=eb539e5ddc24174b6b47e65dd26f914b931b1fd7fc27dd21bbfa5bec30fd3abd89cb78315755&scene=21#wechat_redirect)注解编写sql时方法类似
>
> 4、java层面应该做好参数检查，假定用户输入均为恶意输入，防范潜在的攻击

