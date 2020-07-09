# 赶紧减少该死的 if else 嵌套？

{% embed url="https://mp.weixin.qq.com/s/ANoaM9XoYNdoAl8IUlEMoQ" %}





* 写在前面
* 正文
* 写在最后

## 写在前面

不知大家有没遇到过像“横放着的金字塔”一样的`if else`嵌套：

```text
if (true) {
    if (true) {
        if (true) {
            if (true) {
                if (true) {
                    if (true) {

                    }
                }
            }
        }
    }
}
```

我并没夸大其词，我是真的遇到过了！嵌套6、7层，一个函数几百行，简！直！看！死！人！

`if else`作为每种编程语言都不可或缺的条件语句，我们在编程时会大量的用到。但`if else`一般不建议嵌套超过三层，如果一段代码存在过多的`if else`嵌套，代码的可读性就会急速下降，后期维护难度也大大提高。所以，我们程序员都应该尽量避免过多的`if else`嵌套。下面将会谈谈我在工作中如何减少`if else`嵌套的。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/09/640-20200709110419186-110419.jpg)听说有图会更多人看

## 正文

在谈我的方法之前，不妨先用个例子来说明`if else`嵌套过多的弊端。

想象下一个简单分享的业务需求：支持分享链接、图片、文本和图文，分享结果回调给用户（为了不跑题，这里简略了业务，实际复杂得多）。当接手到这么一个业务时，是不是觉得很简单，稍动下脑就可以动手了：

先定义分享的类型、分享Bean和分享回调类：

```text
private static final int TYPE_LINK = 0;
private static final int TYPE_IMAGE = 1;
private static final int TYPE_TEXT = 2;
private static final int TYPE_IMAGE_TEXT = 3;

public class ShareItem {
    int type;
    String title;
    String content;
    String imagePath;
    String link;
}

public interface ShareListener {

    int STATE_SUCC = 0;
    int STATE_FAIL = 1;

    void onCallback(int state, String msg);
}
```

好了，然后在定义个分享接口，对每种类型分别进行分享就ok了：

```text
public void share (ShareItem item, ShareListener listener) {
    if (item != null) {
        if (item.type == TYPE_LINK) {
            // 分享链接
            if (!TextUtils.isEmpty(item.link) && !TextUtils.isEmpty(item.title)) {
                doShareLink(item.link, item.title, item.content, listener);
            } else {
                if (listener != null) {
                    listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
                }
            }
        } else if (item.type == TYPE_IMAGE) {
            // 分享图片
            if (!TextUtils.isEmpty(item.imagePath)) {
                doShareImage(item.imagePath, listener);
            } else {
                if (listener != null) {
                    listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
                }
            }
        } else if (item.type == TYPE_TEXT) {
            // 分享文本
            if (!TextUtils.isEmpty(item.content)) {
                doShareText(item.content, listener);
            } else {
                if (listener != null) {
                    listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
                }
            }
        } else if (item.type == TYPE_IMAGE_TEXT) {
            // 分享图文
            if (!TextUtils.isEmpty(item.imagePath) && !TextUtils.isEmpty(item.content)) {
                doShareImageAndText(item.imagePath, item.content, listener);
            } else {
                if (listener != null) {
                    listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
                }
            }
        } else {
            if (listener != null) {
                listener.onCallback(ShareListener.STATE_FAIL, "不支持的分享类型");
            }
        }
    } else {
        if (listener != null) {
            listener.onCallback(ShareListener.STATE_FAIL, "ShareItem 不能为 null");
        }
    }
}
```

到此，简单的分享模型就做出来了。有没问题？老实说，如果没什么追求的话，还真没什么问题，至少思路是清晰的。但一周后呢？一个月后呢？或者一年后呢？`share`方法的分支有15条，这意味着你每次回看代码得让自己的大脑变成微型的处理器，考虑15种情况。如果出现bug，你又得考虑15种情况，并15种情况都要测试下。再如果现在需要加多分享小视频功能，你又得添加多3个分支，还要改代码，一点都不“开放-闭合”。再再如果后面项目交接给他人跟进，他人又要把自己大脑变成处理器来想每个分支的作用，我敢肯定有百分之八十的人都会吐槽代码。

我们程序员的脑力不应该花费在无止境的分支语句里的，应该专注于业务本身。所以我们很有必要避免写出多分支嵌套的语句。好的，我们来分析下上面的代码多分支的原因：

1. 空值判断
2. 业务判断
3. 状态判断

几乎所有的业务都离不开这几个判断，从而导致`if else`嵌套过多。那是不是没办法解决了？答案肯定不是的。

上面的代码我是用java写的，对于java程序员来说，空值判断简直使人很沮丧，让人身心疲惫。上面的代码每次回调都要判断一次`listener`是否为空，又要判断用户传入的`ShareItem`是否为空，还要判断`ShareItem`里面的字段是否为空......

对于这种情况，我采用的方法很简单：接口分层。

#### 减少 if else 方法一：接口分层

**所谓接口分层指的是：把接口分为外部和内部接口，所有空值判断放在外部接口完成，只处理一次；而内部接口传入的变量由外部接口保证不为空，从而减少空值判断。**

来，看代码更加直观：

```text
public void share(ShareItem item, ShareListener listener) {
    if (item == null) {
        if (listener != null) {
            listener.onCallback(ShareListener.STATE_FAIL, "ShareItem 不能为 null");
        }
        return;
    }

    if (listener == null) {
        listener = new ShareListener() {
            @Override
            public void onCallback(int state, String msg) {
                Log.i("DEBUG", "ShareListener is null");
            }
        };
    }

    shareImpl(item, listener);
}

private void shareImpl (ShareItem item, ShareListener listener) {
    if (item.type == TYPE_LINK) {
        // 分享链接
        if (!TextUtils.isEmpty(item.link) && !TextUtils.isEmpty(item.title)) {
            doShareLink(item.link, item.title, item.content, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else if (item.type == TYPE_IMAGE) {
        // 分享图片
        if (!TextUtils.isEmpty(item.imagePath)) {
            doShareImage(item.imagePath, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else if (item.type == TYPE_TEXT) {
        // 分享文本
        if (!TextUtils.isEmpty(item.content)) {
            doShareText(item.content, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else if (item.type == TYPE_IMAGE_TEXT) {
        // 分享图文
        if (!TextUtils.isEmpty(item.imagePath) && !TextUtils.isEmpty(item.content)) {
            doShareImageAndText(item.imagePath, item.content, listener);
        } else {
            listener.onCallback(ShareListener.STATE_FAIL, "分享信息不完整");
        }
    } else {
        listener.onCallback(ShareListener.STATE_FAIL, "不支持的分享类型");
    }
}
```

可以看到，上面的代码分为外部接口`share`和内部接口`shareImpl`，`ShareItem`和`ShareListener`的判断都放在`share`里完成，那么`shareImpl`就减少了`if else`的嵌套了，相当于把`if else`分摊了。这样一来，代码的可读性好很多，嵌套也不超过3层了。

但可以看到，`shareImpl`里还是包含分享类型的判断，也即业务判断，我们都清楚产品经理的脑洞有多大了，分享的类型随时会改变或添加。嗯说到这里相信大家都想到用多态了。多态不但能应付业务改变的情况，也可以用来减少`if else`的嵌套。

#### 减少 if else 方法二：多态

利用多态，每种业务单独处理，在接口不再做任何业务判断。把`ShareItem`抽象出来，作为基础类，然后针对每种业务各自实现其子类：

```text
public abstract class ShareItem {
    int type;

    public ShareItem(int type) {
        this.type = type;
    }

    public abstract void doShare(ShareListener listener);
}

public class Link extends ShareItem {
    String title;
    String content;
    String link;

    public Link(String link, String title, String content) {
        super(TYPE_LINK);
        this.link = !TextUtils.isEmpty(link) ? link : "default";
        this.title = !TextUtils.isEmpty(title) ? title : "default";
        this.content = !TextUtils.isEmpty(content) ? content : "default";
    }

    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}

public class Image extends ShareItem {
    String imagePath;

    public Image(String imagePath) {
        super(TYPE_IMAGE);
        this.imagePath = !TextUtils.isEmpty(imagePath) ? imagePath : "default";
    }

    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}

public class Text extends ShareItem {
    String content;

    public Text(String content) {
        super(TYPE_TEXT);
        this.content = !TextUtils.isEmpty(content) ? content : "default";
    }

    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}

public class ImageText extends ShareItem {
    String content;
    String imagePath;

    public ImageText(String imagePath, String content) {
        super(TYPE_IMAGE_TEXT);
        this.imagePath = !TextUtils.isEmpty(imagePath) ? imagePath : "default";
        this.content = !TextUtils.isEmpty(content) ? content : "default";
    }

    @Override
    public void doShare(ShareListener listener) {
        // do share
    }
}
```

（注意：上面每个子类的构造方法还对每个字段做了空值处理，为空的话，赋值`default`，这样如果用户传了空值，在调试就会发现问题。）

实现了多态后，分享接口的就简洁多了：

```text
public void share(ShareItem item, ShareListener listener) {
    if (item == null) {
        if (listener != null) {
            listener.onCallback(ShareListener.STATE_FAIL, "ShareItem 不能为 null");
        }
        return;
    }

    if (listener == null) {
        listener = new ShareListener() {
            @Override
            public void onCallback(int state, String msg) {
                Log.i("DEBUG", "ShareListener is null");
            }
        };
    }

    shareImpl(item, listener);
}

private void shareImpl (ShareItem item, ShareListener listener) {
    item.doShare(listener);
}
```

嘻嘻，怎样，内部接口一个`if else`都没了，是不是很酷~ 如果这个分享功能是自己App里面的功能，不是第三方SDK，到这里已经没问题了。但如果是第三方分享SDK的功能的话，这样暴露给用户的类增加了很多（各`ShareItem`的子类，相当于把`if else`抛给用户了），用户的接入成本提高，违背了“迪米特原则”了。

处理这种情况也很简单，再次封装一层即可。把`ShareItem`的子类的访问权限降低，在暴露给用户的主类里定义几个方法，在内部帮助用户创建具体的分享类型，这样用户就无需知道具体的类了：

```text
public ShareItem createLinkShareItem(String link, String title, String content) {
    return new Link(link, title, content);
}

public ShareItem createImageShareItem(String ImagePath) {
    return new Image(ImagePath);
}

public ShareItem createTextShareItem(String content) {
    return new Text(content);
}

public ShareItem createImageTextShareItem(String ImagePath, String content) {
    return new ImageText(ImagePath, content);
}
```

或者有人会说，这样用户也需额外了解多几个方法。我个人觉得让用户了解多几个方法好过了解多几个类，而已方法名一看就能知道意图，成本还是挺小，是可以接受的。

其实这种情况，更多人想到的是使用工厂模式。嗯，工厂模式能解决这个问题（其实也需要用户额外了解多几个`type`类型），但工厂模式难免又引入分支，我们可以用`Map`消除分支。

#### 减少 if else 方法三：使用Map替代分支语句

把所有分享类型预先缓存在`Map`里，那么就可以直接`get`获取具体类型，消除分支：

```text
private Map<Integer, Class<? extends ShareItem>> map = new HashMap<>();

private void init() {
    map.put(TYPE_LINK, Link.class);
    map.put(TYPE_IMAGE, Image.class);
    map.put(TYPE_TEXT, Text.class);
    map.put(TYPE_IMAGE_TEXT, ImageText.class);
}

public ShareItem createShareItem(int type) {
    try {
        Class<? extends ShareItem> shareItemClass = map.get(type);
        return shareItemClass.newInstance();
    } catch (Exception e) {
        return new DefaultShareItem(); // 返回默认实现，不要返回null
    }
}
```

这种方式跟上面分为几个方法的方式各有利弊，看大家取舍了~

## 写在最后

讲到这里大家有没收获呢？总结下减少`if else`的方法：

* 把接口分为外部和内部接口，所有空值判断放在外部接口完成；而内部接口传入的变量由外部接口保证不为空，从而减少空值判断。
* 利用多态，把业务判断消除，各子类分别关注自己的实现，并实现子类的创建方法，避免用户了解过多的类。
* 把分支状态信息预先缓存在`Map`里，直接`get`获取具体值，消除分支。

好了，到此就介绍完了，希望大家以后写代码能注意，有则避之，无则加勉。希望大家写的代码越来越简洁~

