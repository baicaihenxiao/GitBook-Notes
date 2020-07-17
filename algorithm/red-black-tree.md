# 红黑树

> <https://blog.csdn.net/weixin_42786274/article/details/86557922>
>
> <https://mp.weixin.qq.com/s/hgVkeTnTubq-h0-7z3GLPQ>



# 前言

前段时间在研究 JDK1.8 的 hashmap 源码，看到 put 方法的插入环节，遇到了红黑树，不得不停止阅读源码的过程，因为还没掌握红黑树是无法完全读透 hashmap 源码的。红黑树作为一种数据结构，它被应用得非常多，可能很多人不认识它，但其实它已经在默默为我们的代码在发光发热。例如，你只要在 Java 中用到 map，基本上就是在用红黑树（当元素个数到达八个时链表转红黑树）。

> PS：在看这篇文章前，必须先了解普通的二叉查找树和平衡查找树（AVL）树、2-3-4树。不然看起来会非常吃力。

# 红黑树的性质

红黑树是一种自平衡树，它也是一颗二叉树。既然能保持平衡，说明它和 AVL 树类似，在插入或者删除时肯定有调整的过程，只不过这个调整过程并不像 AVL 树那样繁琐。为何红黑树使用得比 AVL 树更多，就是因为红黑树它的调整过程迅速且简介。红黑树有以下五个特性：

- 性质1：节点是红色或黑色
- 性质2：根是黑色
- 性质3：所有叶子都是黑色。叶子是 NIL 节点，也就是 Null 节点
- 性质4：如果一个节点是红的，则它的两个儿子都是黑的
- 性质5：从任一节点到其叶子的所有简单路径都包含相同数目的黑色节点。

下图展示了一棵红黑树。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-111006.jpg)

**分析：**

注意理解上，叶子并不等价于黑色节点，但是他们的颜色都是黑色。

**为何要给节点指定红或者黑的颜色？**

作者这种设计，只是为了从编程上达到一种便利的效果。另外可以让它们在插入时达到近似的平衡，并不像 AVL 树那样绝对平衡。实际上，红黑树是2-3树的一种变体，某种情况下，它又相当于2-3-4树。因为2-3树在编程上需要比较多的代码量，所以诞生了红黑树这种巧妙的设计。通过加了颜色来区分结点，这样编程上就可以当成二叉树来写程序，不用分别用三个指针表示左、中、右孩子了。

**看一下红黑树和2-3树的等价性联系**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111006762-111006.jpg)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111007058-111007.jpg)

从上面可以看到，把红色节点放到与父亲齐平，就是2-3树中的一个2-3节点。

# 红黑树的操作

数据结构的性质看完之后，就要掌握它到底会存在哪种操作？例如 hashmap，它最常见的操作就是 get 和 put、扩容。同理，红黑树也有它的基本操作。因为它本身上也是一棵二叉查找树，所以重点关注的操作无非就是查找、插入、删除。

## 1. 查找操作

红黑树的查找方式很简单，只要是树，查找的过程无非就是一个递归过程。如果查找的元素小于当前节点，那么查找其左子树；如果查找的元素大于当前元素，则查找其右子树。

## 2. 插入操作

插入操作首先需要通过查找操作找到合适的插入点，然后插入新节点。如果在插入节点后，发生了违背红黑树特性的情况时，需要对红黑树进行旋转染色等操作，使其重新满足特性。

### **2.1 插入新节点**

为了在插入新节点时尽可能少的违反红黑树特性且更容易调整红黑树，就先将新节点染成红色。这样就只可能会违反特性4。如果这里没有违反特性4，那么就不需要对红黑树进行调整，插入操作完成。

### **2.2 调整子树**

那么，在违反了特性4的时候，新节点的父节点为红色节点。根据特性2可知，父节点不是根节点，则新节点必有祖父节点。又根据特性3可推论出红色节点必有两个黑色子节点（空节点为黑色）。此时会出现两种情况：叔节点为红色、叔节点为黑色。

> 图例：C 表示当前节点，P 表示父节点，U 表示叔节点，G 表示祖父节点

**（1）父节点与叔节点都为红色的情况**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111007283-111007.jpg)

在这种情况下，需要将父节点和叔节点变为黑色，再将祖父节点变为红色。这样，图上所展示的子树就满足了红黑树的特性。如下图所示。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111007474-111007.jpg)

但是这里又可能会产生新的违反特性情况，因为祖父节点变成了红色，那么它可能会造成违反特性4的情况。所以，这里就将祖父节点作为当前节点，进行新一轮的调整操作。

**（2）父节点为红色，叔节点为黑色的情况**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111007678-111007.jpg)

在这种情况下，对其调整的核心就是保持父节点分支符合特性4，而叔节点分支保持符合特性5。

- 第一步，旋转。对祖父节点进行左旋或者右旋。如果父节点是祖父节点的右子节点，那么对祖父节点进行左旋；否则，对祖父节点进行右旋。
- 第二步，染色。将祖父节点染为红色，而父节点染为黑色。

进过这两步，上图的情况会转换为下图所示。

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111007907-111008.jpg)

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111008130-111008.jpg)

可以看出，父节点这一分支进过调整后，当前节点与父节点的颜色不再是连续红色，满足特性4。而叔节点这一分支的黑色节点数目没有发生变化，满足特性5。对原祖父节点的父节点来说，该子树没有发生违反特性的变化。该子树调整完成。

### **2.3 检查根节点**

当上述调整执行完后，还有最后一步，就是检查是否满足特性2。这一步只需要将根节点染成黑色就可以，无需再多加判断。

## 3. 删除操作

需要重点理解的就是删除操作，这个也是我觉得是红黑树中最难的部分。

删除操作要比插入操作略微复杂一些。因为删除的节点可能是出现在树的中间层的节点，此时删除该节点会遇到很复杂的情况。所以，在删除节点的时候，需要先对红黑树进行一些调整，使得删除节点对整个树的影响降到最低。

### **3.1 替换删除节点**

首先根据 BST 删除节点的规则，使用当前节点左子树的最大值节点或者右子树的最小值节点代替其删除。这两个节点是其子树中数值上最贴近当前节点数值的节点）。这一点，只要懂了二叉查找树的删除操作就明白了，在这里不多说了。如下图所示：

> 图例：D 表示当前节点，P 表示父节点，B 表示兄弟节点，BR 表示兄弟节点的右子节点，BL 表示兄弟节点的左子节点

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111008357-111008.jpg)

既然待删除节点是要被移走的，那肯定有一个节点要替换到它的位置上去。如何找到这个替换节点，这个过程和二叉查找树一模一样，要么在它的左子树下一直往右找到最大节点，要么在右子树下找到最小节点。

**下面的描述过程采用的是右子树的最小值节点代替**

当找到替换节点之后，现在需要考虑的情况就减少了，只可能会出现以下几种情况（因为需要满足红黑树特性）：

1. 无子节点，节点为红色
2. 无子节点，节点为黑色
3. 只有右子节点，右子节点为红色，节点本身为黑色

> 上面这三种情况，说的是新待删除节点。新待删除节点，就是即将被替换到待删除位置的节点。

因为 D 节点就是即将要替换到待删节点位置的节点，它同时又是右子树的最小值，既然是最小值了，它就不再可能拥有左子树了，所以只有可能有右子节点。另外，假如它有右节点且右节点的颜色是黑色，它自身颜色是红色，根本不成立。因为假如它自身为红色且又有黑孩子，那它必须要有两个黑孩子才满足红黑树性质，所以不满足。那有没有可能，它自身是黑色且右孩子也为黑色呢？也不可能！因为它左孩子已经为空了，说明它从自身出发到左子树的叶子的距离就是1，假如它右孩子也为黑色，那它从自身出发到右子树叶子的距离肯定大于等于2了，明显不可能。

所以总的来说只可能有下面三种情况：

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111008555-111008.jpg)

- 情况1：只需要直接删除节点就可以。删了一个红色新待删节点，不会影响红黑树性质。
- 情况2：删除该 D 节点后，违反了红黑树特性5，需要调整（不考虑待删除节点为根节点的情况）
- 情况3：用右子节点 R 占据待删除节点 D，再将其染成黑色即可，不违反红黑树特性。因为左边本来就是空了，其实右子树下即使有多少个黑色节点，也不会影响整体特性。

在这三种情况中，情况1和情况3比较简单，不需要多余的调整。情况2则需要后续的调整步骤使其满足红黑树特性。

### **3.2 调整红黑树**

上述情况2的调整比较复杂。下面对各种情况进行讲解。

**根据红黑树的特性5，待删除节点必然有兄弟节点**。

为什么这么说呢？因为我们已经假设上面的 D 节点不为根了，那说明它肯定有父亲。首先它是没有孩子的，它下面直接就是叶子了，既然有父亲，不论它是父亲的左孩子或者右孩子，从父亲出发到它自身，黑色节点的个数为1。反证法：假如父亲只有它一个孩子，那说明父亲到另一边子树的叶子距离就为0，因为0个节点。这明显不符合，所以说明父亲肯定有两个孩子，那从而得知待删节点D必有兄弟。

下面根据其兄弟节点所在分支的不同，来分情况讨论。

> 以下是以关注待删节点为父节点的左子节点进行描述，如果遇到关注节点为父节点的右子节点的情况，则镜像处理。

**思路：**下面的任何调整只有一个目的，就是不断调整，直到调整到可以直接将 D 移除又不会影响红黑树特性的情况。但关键是调整过程中红黑树特性也不会发生改变。

> 图例：D 表示当前节点，P 表示父节点，B 表示兄弟节点，BR 表示兄弟节点的右子节点，BL 表示兄弟节点的左子节点

**（1）兄弟节点为红色**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111008754-111008.jpg)

将父节点染成红色，兄弟节点染成黑色，然后对父节点进行左旋操作。此时就转换为了下面的（4），之后按照（4）继续进行调整。

分析：这种情况，树的整体高度为2，变色左旋之后，整体高度还是保持在2。

**（2）兄弟节点为黑色，远侄节点为红色**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/17/640-111009.png)

这种情况下，不需要考虑父节点的颜色。

1. 将父节点 P 与兄弟节点 B 的颜色互换 ，这个过程父亲染黑
2. 将兄弟节点的右子节点 BR 染成黑色
3. 对父节点 P 进行左旋操作

可以看到，原本高度就是符合红黑树特性的，左右子树的高度都为1，因为黑色节点只有一个。经过这三步的调整后，直接删除节点 D 后仍然满足红黑树的特性，调整完成，跳出算法循环。

**（3）兄弟节点为黑色，远侄节点为黑色，近侄节点为红色**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/png/2020/07/17/640-20200717111009459-111009.png)

这种情况下，兄弟节点的左节点染成黑色。兄弟节点染红。然后对兄弟节点做右旋。此时的状况就和（2）一样了。之后就通过（2）的调整方式进行调整。

**（4）父节点为红色，兄弟节点为黑色，兄弟节点无子节点**

这种情况下，将父节点P染成黑色，再将兄弟节点染成红色。经过这样的操作后，除去节点D后，以P为根节点的子树的黑节点深度并没有发生变化。调整完成。

**怎么理解这个操作？**

可以看左边，没调整前，P 的左右子树的黑色结点的数目都是1，是相同的，符合红黑树的性质：从任一节点到其叶子的所有简单路径都包含相同数目的黑色节点。然后再看右边，调整后，删掉 D 之后，P 结点的左右子树的黑色结点都是0个，仍然满足性质，所以调整完成。

**（5）父节点为黑色，兄弟节点为黑色，兄弟节点无子节点**

![img](https://gitee.com/baicaihenxiao/imageDB/raw/master/uPic/jpg/2020/07/17/640-20200717111007907-111008.jpg)

这种情况下，为了在删除节点 D 后使以 P 为根节点的子树能满足红黑树特性5，将兄弟节点 B 染成红色。但是这样操作后，以 P 为根节点的子树的黑色节点深度变小了。所以需要继续调整。

因为P节点子树的黑色深度发生了减少，可以把其当作待删除节点，那么此时就以 P 节点为关注节点进行进一步调整（继续向上调整）。这句话的意思我们再以 P 为起始点，继续根据情况进行平衡操作。就是把 P 当成 D，只是不要再删除 P 了。再看是这五种中的哪种情况，再进行对应的调整，这样一直向上，直到新的起始点为根节点或者关注节点不为黑色。

第五种情况，不会一直连续回溯的。假如能一直回溯，指针向上走之后，兄弟节点会一直都没有右孩子吗？不存在的。假如有这种情况，说明树的路径长度已经严重往左倾斜，肯定不可能。所以回溯这个情况只会回溯一次，不会连续回溯。第五个这种情况出现之后，下一次进入算法循环，肯定就是进入其他情况，直到遇到 break，跳出循环，终止整个算法过程。

### **3.3 检查根节点及删除节点**

经过上述的调整后，此时基本满足了红黑树的特性。但是存在根节点变成红色的情况。所以需要将根节点染成黑色的操作。最后，执行删除操作，将待删除节点删掉。

当然从编程的角度，你也可以调整指针先把待删除节点移掉，然后再开始平衡调整过程。注意这里说的平衡调整，并不是 AVL 树的绝对平衡调整，而是满足红黑树特性的平衡调整。红黑树的平衡和 AVL 的平衡是有区别的。

# Java实现

上面的操作篇幅比较长，假如没看明白，可以通过下面的代码继续看。

> 由于篇幅限制，源码请到原文链接查看 Java 实现
>
> <blog.csdn.net/weixin_42786274/article/details/86557922>



```java
package com.kun.kunspringbootweb.foo.tree;
 
import java.util.ArrayList;
import java.util.List;
import java.util.concurrent.ConcurrentHashMap;
 
 
/**
 * 红黑树-Java实现例子
 *
 *
 * @author xuyaokun
 * @date 2019/1/14 17:06
 */
public class MyRBTree<T extends Comparable<T>, D> {
 
    private RBNode<T, D> root;//根节点
    /**
     * 节点的颜色
     */
    private static final Boolean RED = false;
    private static final Boolean BLACK = true;
 
    public class RBNode<T extends Comparable<T>, D> {
 
        private Boolean color;//节点颜色
        private T key;//键值
        private D data;//具体的数据
        private RBNode<T, D> parent;
        private RBNode leftChild;
        private RBNode rightChild;
 
 
        public RBNode(Boolean col, T key, D data, RBNode paret, RBNode leftChild, RBNode rightChild) {
            this.color = col;
            this.key = key;
            this.data = data;
            this.parent = parent;
            this.leftChild = leftChild;
            this.rightChild = rightChild;
 
        }
 
    }
 
    /**
     * 获取父亲
     *
     * @param node
     * @return
     */
    public RBNode<T, D> parentOf(RBNode<T, D> node) {
        if (node != null) {
            return node.parent;
 
        }
 
        return null;
 
    }
 
    /**
     * 获取颜色
     *
     * @param node
     * @return
     */
    public Boolean colorOf(RBNode<T, D> node) {
        if (node != null) {
            return node.color;
 
        }
        return BLACK;
 
    }
 
    public void setParent(RBNode<T, D> node, RBNode<T, D> parent) {
        if (node != null) {
            node.parent = parent;
        }
 
    }
 
    public void setColor(RBNode<T, D> node, Boolean color) {
        if (node != null) {
            node.color = color;
 
        }
 
    }
 
    public Boolean isRed(RBNode<T, D> node) {
        return (node != null && node.color == RED) ? true : false;
 
    }
 
    public Boolean isBlack(RBNode<T, D> node) {
        return !isRed(node);
 
    }
 
    public void setRed(RBNode<T, D> node) {
        if (node != null) {
            node.color = RED;
        }
    }
 
    public void setBlack(RBNode<T, D> node) {
        if (node != null) {
            node.color = BLACK;
        }
 
    }
 
    /**
     * 根据key获取数据
     *
     * @param key
     * @return
     */
    public D get(T key){
        RBNode node = search(key, root);
        return node == null ? null : (D) node.data;
    }
 
    //寻找为key值的节点
    public RBNode<T, D> search(T key, RBNode<T, D> node) {
 
        if (node != null) {
            //查找的过程，就是一直递归比较到叶子为止
            int com = key.compareTo(node.key);
            if (com < 0) {
                return search(key, node.leftChild);
            } else if (com > 0) {
                return search(key, node.rightChild);
            } else {
                return node;
            }
 
        }
        return null;
 
 
    }
 
    //寻找后继节点，即大于该节点的最小节点
    public RBNode<T, D> min(RBNode<T, D> node) {
 
        //一直往左走，最左端的就是最小值，这是二叉树的性质
        if (node.leftChild == null) {
            return node;
        }
        while (node.leftChild != null) {
            node = node.leftChild;
        }
 
        return node;
    }
 
    /**
     * 寻找待删节点的后继节点
     * （因为这个节点即将要被删了，所以要选个后继节点补到这个位置来，
     *  选择节点的规则：
     *  这个规则和普通二叉树是一样的，要么就是找左子树的最大值，要么就是右子树的最小值）
     *
     * @param node
     * @return
     */
    public RBNode successor(RBNode<T, D> node) {
 
        if (node.rightChild != null) {
            return min(node.rightChild);
        }
        //下面这里是不会进入的，因为只有node的两个孩子都不为null时才会进入这个方法
        RBNode<T, D> y = node.parent;
        while ((y != null) && (y.rightChild == node)) {
            node = y;
            y = y.parent;
 
        }
        return y;
 
    }
 
    /**
     * 对某个节点进行左旋
     * （当前节点就是父亲节点，整体过程就是 父亲下沉，右孩子上升，然后右孩子的左节点变成了原父亲的右节点）
     *
     * @param x
     */
    public void leftRonate(RBNode<T, D> x) {
 
        //右孩子
        RBNode<T, D> y = x.rightChild;
 
        if (y.leftChild != null) {
            //当前节点 变成了 右孩子的左节点的父亲
            y.leftChild.parent = x;
        }
        x.rightChild = y.leftChild;
        y.leftChild = x;
        //当前的父亲变成了右孩子的父亲
        y.parent = x.parent;
 
        if (x.parent != null) {
            if (x.parent.leftChild == x) {
                x.parent.leftChild = y;
            } else {
                x.parent.rightChild = y;
            }
        } else {
            this.root = y;
        }
        x.parent = y;
 
    }
 
    //对某个节点进行右旋
    public void rightRonate(RBNode<T, D> x) {
        RBNode<T, D> y = x.leftChild;
 
        if (y.rightChild != null) {
            y.rightChild.parent = x;
 
        }
 
        y.parent = x.parent;
        x.leftChild = y.rightChild;
        y.rightChild = x;
 
        if (x.parent != null) {
            if (x.parent.leftChild == x) {
                x.parent.leftChild = y;
 
            } else {
                x.parent.rightChild = y;
 
            }
 
        } else {
            this.root = y;
 
        }
        x.parent = y;
 
    }
 
 
 
    /**
     *
     * 插入后的自平衡过程
     *
     * （为什么用这个方法名，因为HashMap源码也是用这个源码名）
     *
     * @param node 新插入的节点
     */
    public void balanceInsertion(RBNode<T, D> node) {
 
        RBNode<T, D> parent, gparent;//父亲与祖父
 
        //一开始插入的节点首先肯定是红色，假如父亲不为空且父亲也是红色，那就出现了“双红情况”
        //必须做调整，进入循环
        //假如父亲是黑色，没必要调整，为什么？
        // 因为加一个红节点，不会影响这条路径上的黑色节点个数发生变化，所以不会影响红黑树的性质
        while (((parent = parentOf(node)) != null) && isRed(parent)) {
 
            //拿到父亲的父亲，也就是祖父
            gparent = parentOf(parent);
 
            //根据祖父来判断，父亲是祖父的左子树还是右子树，确定了之后的目的是为了拿到叔父
            if (gparent.leftChild == parent) {
                //假如父亲是祖父的左孩子，那通过祖父的右孩子指针就能拿到叔父
                RBNode<T, D> uncle = gparent.rightChild;
                //进一步，再分两种情况，会发生以下两种情况的前提是父亲为红色 ：
                if (isRed(uncle)) {
                    //叔父是红色
                    //父亲变黑，叔父变黑，祖父变红 （为何要这三步，这就是规律，用笔和纸画画就知道）
                    // 实在看不懂参考我的笔记：http://note.youdao.com/noteshare?id=14243cb721319c19ef8f79cadd2a2c81&sub=0B35BC1D6FD747DAB0C373910D1DBAB5
                    setBlack(parent);
                    setBlack(uncle);
                    setRed(gparent);
                    //回溯，向上，这一步指针赋值，把祖父变成当前节点，继续向上判断，直到终止
                    node = gparent;
                    //插入操作，回溯过程进入这个代码if-else分支，最多一次，为何？
                    //因为叔父一开始是红色，那说明祖父必为黑色，那祖父的兄弟也肯定是黑色，因为黑色的兄弟节点，必然同色
                    //进入一次之后，父亲变成了当前节点，原来的祖父变成了父亲，那说明新父亲不会与新叔父同为红色。
                    continue;
 
                } else {
                    //叔父是黑色，父亲为红
                    if (parent.rightChild == node) {
                        //假如当前节点是父亲的右孩子
                        //父亲要左旋
                        leftRonate(parent);
                        RBNode<T, D> temp = node;
                        node = parent;
                        parent = temp;
 
                    }
                    //因为上面发生了指针调换，这里parent其实已经指当前节点的指针
                    // 所以是当前节点位置上升变黑，父亲下沉还是红色，祖父变红
                    setBlack(parent);
                    setRed(gparent);
                    //祖父右旋
                    //这里为什么祖父还要右旋一次呢？
                    /*
                             黑祖               黑祖
                            /    \             /   \
                         红父    黑叔  -->  红插   黑叔
                            \               /
                            红插          红父
                         可以看到上面，经过了一次旋转之后，平衡性仍然满足，但是颜色就不对了，仍然存在双红情况
                         其实理解了AVL树的平衡调整，这里其实也是一个“双旋转”的过程，只有被删节点与父亲不同侧时才需要双旋
                         继续接着上面的双红情况，因为颜色不对，这时需要变色
                                    黑祖          红祖
                                   /   \          /   \
                                红插   黑叔 --> 黑插  黑叔
                               /                /
                            红父              红父
                          可以看到，变色之后，右边的子树和左边子树的黑色节点个数保持一致，明显和原来不同，原来的图，看左上角，右子树的黑色节点要多一个
                          原来的树肯定是满足红黑树性质的，所以这里只需要做一次简单的右旋，就可以让右子树的黑色节点个数再次比左边多一个，满足了原来的情况
                     */
                    rightRonate(gparent);
 
                }
 
            } else {
                //同理，先找到叔父
                RBNode<T, D> uncle = gparent.leftChild;
                if (isRed(uncle)) {
                    //这个过程其实和上面的是 一致的，不需要旋转，只需要重新染色即可
                    setBlack(parent);
                    setBlack(uncle);
                    setRed(gparent);
                    node = gparent;
                    continue;
 
                } else {
                    if (parent.leftChild == node) {
                        //假如当前插入节点是父亲的左孩子，那就对父亲做右旋
                        //左孩子上升，父亲下沉
                        rightRonate(parent);
                        RBNode<T, D> temp = node;
                        node = parent;
                        parent = temp;
                    }
                    //同理，祖父左旋
                    setBlack(parent);
                    setRed(gparent);
                    leftRonate(gparent);
 
                }
 
            }
 
        }
 
        //调整后，假如都调整到根处了，必须要再次检验，让根为黑色，让它继续满足红黑树的性质
        if (root == node) {
            setBlack(node);
        }
 
    }
 
    /**
     * 红黑树删除后的平衡调整
     * （删除操作比较复杂，所以我加了很多过程状态追踪）
     *
     * @param node
     * @param parent
     *
     *               入参只可能是下面这两种情况：
     *               1. node=替换节点 parent=替换节点的父亲节点
     *               2. node=替换节点的孩子节点 parent=替换节点
     *               3. node=替换节点的孩子节点 parent=替换节点的父节点
     *
     *               无论是上面哪种情况，都满足：node和parent是儿子父亲的关系
     *
     *               首先要清楚，什么情况下会进入到这个方法？
     *               1.待删节点是黑色节点（且待删节点只有一个子树）
     *               或者
     *               2.替换节点是黑色节点(待删节点的左右子树都不为空)
     *
     *               下面仔细分析下上面两种情况，为何只有这两种情况下，才有必要进行调整。
     *               对于上面的情况1：
     *               待删节点只有一个子树，说明只有一条路径，待删节点移除掉之后，只会影响一条路径上的黑色节点个数。
     *               回忆下红黑树的性质，同一个节点出发直到叶子，每一个不同的路径下的黑色节点个数必须是相同的。
     *               那既然它只有一个子树，说明只有一条路径。假如待删节点是红色，它移除掉之后，不会影响这条路劲下的黑色节点个数，所以不需要调整。
     *               反之，假如它是黑色，它一旦被移除，这一条路径上的黑色个数就减少了，这样会导致这一条路径的黑色节点直接减一
     *               这样就会导致这一条路径和 其他由树根出发的路径相比，不再相等。
     *
     *               对于上面的情况2：
     *               因为待删节点下，有两个子树，那要让树继续保持二叉树的关系，那待删位置上必须放入一个符合规则的元素。
     *               符合什么规则呢？也就是必须比左子树大，比右子树小，所以我们编程上通常选右子树的最小值。
     *               待删元素被删，就要选右子树的最小值放到待删元素位置，那替换节点的指针就会丢失，因为做了移动。
     *               既然右子树中有个元素被移除了，假如这个被移除指针的替换节点，是红色，不会对整棵树的平衡性有任何影响。
     *               但假如这个替换节点是黑色，一旦被移除，会导致右子树的黑色个数减一，不再相等，所以需要调整。
     *
     *      对于一次调整过程，最多不会超过三次，这是为什么？
     *      上述的情况1比较简单，不需要做任何旋转，只需要染色即可。
     *      复杂的是情况2。。
     *          情况2又可以分为细分两种情况：
     *              1.替换节点在新父亲的左侧
     *              2.替换节点在新父亲的右侧
     *              (只要理解了其中一种情况，另一种其实也清楚了)
     *
     *
     *              下面只选其中一种情况，做总结
     *              对于替换节点在新父亲的左侧这种情况，又可以细分成几种情况（下面这几种情况，才是我觉得红黑树中最难理解的部分）：
     *                  1.兄弟节点是红色：兄弟变黑，父亲变红，父亲左旋
     *                  2.兄弟节点是黑色
     *                    2.1.兄弟节点的左右孩子都是黑色：兄弟染红，整体指针（包括替换节点和其parent）向上回溯一步
     *                    2.2.兄弟节点的左孩子是红色，右孩子是黑色：兄弟染红，左孩子染黑，兄弟右旋
     *                    2.3.兄弟节点的右孩子是红色：父亲的颜色赋值到兄弟,父亲染黑，兄弟的右孩子染黑，父亲左旋
     */
 
    public void balanceDeletion(RBNode<T, D> node, RBNode<T, D> parent) {
 
        //先看下调整之前的树结构
        System.out.println("先看下调整之前的树结构");
        this.printTreeLevel2();
 
        RBNode<T, D> other;
        while (isBlack(node) && node != this.root) {
 
            //假如替代节点的颜色也为黑，这里为什么要用“也”字，能进这个方法，肯定说明被删节点也是黑色
            //替代节点也是黑色，那肯定要做调整了，不然整条路径，就少了一个黑色节点，最终肯定是不符合红黑树条件的
 
            if (parent.leftChild == node) {
                //假如替代节点是其新父亲的左节点，那就通过右指针拿到其兄弟节点
                other = parent.rightChild;
 
                System.out.println("当前parent：" + parent.key + " other(兄弟节点):" + other.key);
 
                if (isRed(other)) {
                    //假如兄弟是红色，那么父亲肯定是黑色
 
                    System.out.println("兄弟当前是红色");
                    System.out.println("进入balanceDeletion的while（情况2-L-a:兄弟是红色）");
                    System.out.println("----父亲染红，other染黑，父亲左旋，然后continue");
 
 
                    //兄弟与父亲调换颜色，父亲做左旋
                    setRed(parent);
                    setBlack(other);
                    leftRonate(parent);
 
                    this.printTreeLevel2();
 
                    continue;
 
 
                } else {
                    if (isBlack(other.leftChild) && isBlack(other.rightChild)) {
                        //假如兄弟节点没有任何孩子节点，也会进入这个代码分支，因为叶子也相当于是黑色的
 
                        //other就是替换节点的兄弟节点
                        System.out.println("兄弟节点当前的左右孩子都是黑色");
                        System.out.println("进入balanceDeletion的while（情况2-L-b:兄弟的左右孩子都是黑色）");
                        System.out.println("----other染红，父亲指针向上回溯");
 
                        //other染红，父亲指针向上回溯
                        setRed(other);
                        node = parent;
                        parent = parentOf(node);
 
                        this.printTreeLevel2();
 
 
                    } else if (isRed(other.leftChild) && isBlack(other.rightChild)) {
 
                        System.out.println("other当前的左孩子是红色，右孩子是黑色");
                        System.out.println("进入balanceDeletion的while（情况2-L-c:兄弟的左孩子是红色，右孩子是黑色）");
                        System.out.println("----other染红，other的左节点染黑，other做右旋");
 
                        setRed(other);
                        setBlack(other.leftChild);
                        rightRonate(other);
 
                        this.printTreeLevel2();
 
 
                    } else if (isRed(other.rightChild)) {
 
                        System.out.println("other右孩子是红色");
                        System.out.println("进入balanceDeletion的while（情况2-L-d:兄弟的右孩子是红色）");
                        System.out.println("----父亲的颜色赋值到other,父亲染黑，other的右孩子染黑，父亲左旋，跳出while循环");
 
                        setColor(other, colorOf(parent));
                        setBlack(parent);
                        setBlack(other.rightChild);
                        leftRonate(parent);
 
                        this.printTreeLevel2();
 
                        break;
 
                    }
 
                }
 
            } else {
                other = parent.leftChild;
 
                System.out.println("当前parent：" + parent.key + " other:" + other.key);
 
                if (isRed(other)) {
 
                    System.out.println("other当前是红色");
                    System.out.println("进入balanceDeletion的while（情况2-R-a:兄弟是红色）----other染黑，parent变红,parent右旋");
 
                    setBlack(other);
                    setRed(parent);
                    rightRonate(parent);
 
                    this.printTreeLevel2();
 
                    continue;
 
                } else {
 
                    if (isBlack(other.leftChild) && isBlack(other.rightChild)) {
 
                        System.out.println("other当前的左孩子是黑色，other的右孩子是黑色");
                        System.out.println("进入balanceDeletion的while（情况2-R-b:兄弟的左右孩子都是黑色）----other变红，指针回溯");
 
                        setRed(other);
                        node = parent;
                        parent = parentOf(node);
 
                        this.printTreeLevel2();
 
 
                    } else if (isRed(other.rightChild) && isBlack(other.leftChild)) {
 
                        System.out.println("other当前的右孩子是红色，other的左孩子是黑色");
                        System.out.println("进入balanceDeletion的while（情况2-R-c:兄弟的右孩子是红色，左孩子是黑色）----parent变红，other的右孩子变黑，然后other做左旋");
 
                        setRed(parent);
                        setBlack(other.rightChild);
                        leftRonate(other);
 
                        this.printTreeLevel2();
 
 
                    } else if (isRed(other.leftChild)) {
 
                        System.out.println("other的左孩子是红色");
                        System.out.println("进入balanceDeletion的while（情况2-R-d:兄弟的左孩子是红色）----父亲的颜色赋值到other,父亲染黑，other的左孩子染黑，父亲右旋，跳出while循环");
 
                        setColor(other, colorOf(parent));
                        setBlack(parent);
                        setBlack(other.leftChild);
                        rightRonate(parent);
 
                        this.printTreeLevel2();
 
                        break;
 
                    }
 
                }
 
            }
 
        }
 
        //在这里，node其实是即将放入被删位置的替代节点
        //假如node是红色，那同时被删节点是黑色，
        // 那说明，直接把node的颜色由红变黑，就直接满足了，不需要做任何旋转
        if (node != null){
            System.out.println("节点：" + node.key + "染黑");
        }
        setBlack(node);
 
        this.printTreeLevel2();
 
        System.out.println("调整完成！！！！！！！");
 
    }
 
 
    //红黑树添加操作
    public void insertNode(T key, D data) {
 
        int com;
        RBNode<T, D> x = this.root;
        RBNode<T, D> y = null;
 
        //这个过程和二叉查找树的过程是一样的，从上循环到底，直到找到为止
        while (x != null) {
            y = x;
            com = key.compareTo(x.key);
 
            if(com == 0){
                //说明相等，找到了，直接替换新值，返回
                //TODO
 
                return ;
            }
 
            if (com < 0) {
                x = x.leftChild;
            } else {
                x = x.rightChild;
            }
        }
 
        //生成一个新的节点
        RBNode<T, D> node = new RBNode<T, D>(BLACK, key, data, null, null, null);
        //通过上面的比较，已经找到了父亲
        node.parent = y;
 
        if (y != null) {
            //再次做比较，决定要把新节点放在父亲的哪一边
            com = node.key.compareTo(y.key);
            if (com < 0) {
                y.leftChild = node;
            } else {
                y.rightChild = node;
            }
        } else {
            //假如找到的父亲为空，那说明肯定之前就没有根，是空树
            //把这个新节点作为根
            this.root = node;
 
        }
        //根据红黑树的性质，把默认节点设置为红色，向上回溯，更容易列举可能出现的情况，
        // 所以这里新节点都默认设置成红色
        setRed(node);
 
        //接下来这个就是最关键的方法 ，插入后的自平衡过程，调整让它保持红黑树的性质
        balanceInsertion(node);
 
    }
 
    public void insert(T key, D data) {
        insertNode(key, data);
    }
 
    public void add(T key, D data) {
        insertNode(key, data);
    }
 
 
    /**
     * 红黑树删除操作
     *
     * @param node  传入的是待删除节点
     */
    public void delete(RBNode<T, D> node) {
 
        RBNode<T, D> child, parent, replace;
        Boolean color = true;
 
        //删除从整体上也分两种情况，然后两种情况下再细分
 
        //假如待删除节点的双节点都不为空，这种情况较复杂
        if (node.leftChild != null && node.rightChild != null) {
 
            //找到了替换节点，就是要接替待删节点指针的新节点
            replace = successor(node);
            //找到替换节点的父亲节点
            parent = parentOf(replace);
            //因为替换节点已经是右子树中的最小值了，所以只有右孩子
            child = replace.rightChild;
 
            //在这里为什么要获取替换节点的颜色呢？
            //可以这样想，因为替换节点的指针最终肯定是会丢失的，因为替换节点即将接受待删节点的指针，所以替换节点的指针就不再保留了
            //既然不再保留，那说明原来替换节点这里肯定就少了一环，少了一个节点，所以这里就有判断它颜色的必要
            //假如少的恰恰是黑色，那说明它会影响整棵树的平衡性，不再满足红黑树性质
            color = colorOf(replace);
 
            if (node == parentOf(replace)) {
                //假如替换节点的父亲节点就是当前待删除节点
                //那就直接把待删除节点的指针赋值给parent
                parent = replace;
 
            } else {
                //假如替换节点的父亲不是待删除节点的父亲
                if (child != null) {
                    //因为替换节点待会是要被删掉的，因为它的值会被放置到待删除节点中，然后把替换节点删除就相当于完成整个删除操作
                    //所以要为替换节点的孩子节点找到新父亲
                    setParent(child, parentOf(replace));
                }
                //然后替换节点的右孩子设置成替换节点的父亲的左孩子
                replace.parent.leftChild = child;
                replace.rightChild = node.rightChild;
                setParent(node.rightChild, replace);
            }
 
            //把目标删除节点node的父亲设置成替换节点的父亲
            setParent(replace, parentOf(node));
            replace.leftChild = node.leftChild;
            setParent(node.leftChild, replace);
            //除了指针的调整，颜色也要覆盖，替换节点既然来到了待删节点的位置，那么颜色也要沿用之前的颜色，这样才能满足整棵树的性质
            setColor(replace, colorOf(node));
 
            if (parentOf(node) != null) {
                //待删节点的父亲节点假如不为空，那就要调整父亲节点的左右孩子指针
                if (node.parent.leftChild == node) {
                    node.parent.leftChild = replace;
                } else {
                    node.parent.rightChild = replace;
                }
 
            } else {
                this.root = replace;
            }
            //上面整个过程就是用replace的指针完全取代了node节点，到此为止，node节点就是一个孤立节点了，就算是删除了
 
            if (color == BLACK) {
                balanceDeletion(child, parent);
            }
 
        } else {
 
            //假如待删节点只有左子树或者右子树
            if (node.leftChild != null) {
                replace = node.leftChild;
            } else {
                replace = node.rightChild;
            }
            //找到待删节点的父亲节点
            parent = parentOf(node);
 
            if (parent != null) {
                //判断待删节点属于父亲的左还是右
                if (parent.leftChild == node) {
                    parent.leftChild = replace;
                } else {
                    parent.rightChild = replace;
                }
            } else {
                //假如待删节点的父亲为空，那说明它原来就是根
                //它被删了，所以孩子升为根
                this.root = replace;
            }
 
            //把parent设置成replace的父亲
            setParent(replace, parent);
 
            color = colorOf(node);
            child = replace;
            //假如待删节点是黑色节点，那说明本次删除肯定会影响红黑树的性质
            //删黑色节点，需要调整平衡，反之，删除红色节点，不需要调整
            if (color == BLACK) {
                balanceDeletion(child, parent);
            }
 
        }
 
 
    }
 
    public void delete(T key) {
        RBNode<T, D> node;
        if ((node = search(key, this.root)) != null) {
            delete(node);
        }
 
    }
 
    public void remove(T key) {
        RBNode<T, D> node;
        if ((node = search(key, this.root)) != null) {
            delete(node);
        }
 
    }
 
    //前序遍历
    public void preOrder(RBNode<T, D> node) {
        if (node != null) {
 
            System.out.print(node.key + " ");
            preOrder(node.leftChild);
            preOrder(node.rightChild);
 
        }
 
 
    }
 
    public void preOrder() {
        preOrder(this.root);
 
    }
 
    //中序遍历
    public void inOrder(RBNode<T, D> node) {
        if (node != null) {
            inOrder(node.leftChild);
            System.out.print(node.key + " ");
            inOrder(node.rightChild);
 
        }
 
    }
 
    public void inOrder() {
        inOrder(this.root);
 
    }
 
    //后序遍历
    public void postOrder(RBNode<T, D> node) {
        if (node != null) {
            postOrder(node.leftChild);
            postOrder(node.rightChild);
            System.out.print(node.key + " ");
 
        }
 
    }
 
    public void postOrder() {
        postOrder(this.root);
 
    }
 
    /**
     * 打印出整棵树的层级结构，为了方便跟踪旋转的过程
     *
     */
    public void printTreeLevel(){
 
        System.out.println("开始输出树的层级结构");
        ConcurrentHashMap<Integer, List<RBNode>> map = showTree();
        int size = map.size();
 
        for (int i = 0; i < map.size(); i++) {
            System.out.println();
            for (int j = 0; j < map.get(i).size(); j++) {
                System.out.print( makeSpace2(size, i) +
                        (map.get(i).get(j).key == null ? " " : (map.get(i).get(j).key) + (map.get(i).get(j).color? "(黑)":"(红)")) + makeSpace2(size, i));
 
            }
            System.out.println();
        }
        System.out.println("结束输出树的层级结构");
 
    }
 
    public void printTreeLevel2(){
 
        System.out.println("开始输出树的Graphviz结构");
        ConcurrentHashMap<Integer, List<RBNode>> map = showTree();
        int size = map.size();
        System.out.println("digraph kunghsu{");
        for (int i = 0; i < map.size(); i++) {
            for (int j = 0; j < map.get(i).size(); j++) {
 
                if(map.get(i).get(j).key != null){
                    System.out.println(map.get(i).get(j).key + " [color="  + (map.get(i).get(j).color == RED?"red":"black")  + " style=filled fontcolor=white] ");
                }
            }
        }
 
        for (int i = 0; i < map.size(); i++) {
            for (int j = 0; j < map.get(i).size(); j++) {
                String content = "";
 
                if(map.get(i).get(j).key != null){
                    if(map.get(i).get(j).leftChild != null){
                        System.out.println(map.get(i).get(j).key + "->" + map.get(i).get(j).leftChild.key + "[label=left]");
                    }
                    if(map.get(i).get(j).rightChild != null){
                        System.out.println(map.get(i).get(j).key + "->" + map.get(i).get(j).rightChild.key + "[label=right]");
                    }
                }
            }
        }
        System.out.println("}");
 
        System.out.println("结束输出树的Graphviz结构");
 
    }
 
    /**
     * 为了让输出更有结构感，在元素前拼接一些空格，对齐
     *
     * @param size
     * @param index
     * @return
     */
    public String makeSpace2(int size, int index){
        StringBuilder builder = new StringBuilder();
        for (int i = 0; i < 1 << (size - index); i++) {
            builder.append("  ");
        }
        return builder.toString();
    }
 
    public ConcurrentHashMap<Integer, List<RBNode>> showTree(){
 
        ConcurrentHashMap<Integer, List<RBNode>> map = new ConcurrentHashMap<>();
        showTree(root, 0, map);
        return  map;
    }
 
    public void showTree(RBNode root, int count, ConcurrentHashMap<Integer, List<RBNode>> map){
 
        if(map.get(count) == null){
            map.put(count, new ArrayList<>());
        }
        map.get(count).add(root);
 
        if(root.leftChild != null){
            showTree(root.leftChild, count+1 , map);
        }else{
            //假如为空，也添加到map中，因为我要做格式化控制，空的，我也要知道它这个位置是空的
            if(map.get(count+1) == null){
                map.put(count+1, new ArrayList<>());
            }
            map.get(count+1).add(new RBNode(false, null, null, null, null, null));
        }
        if(root.rightChild != null){
            showTree(root.rightChild, count+1 , map);
        }else{
            if(map.get(count+1) == null){
                map.put(count+1, new ArrayList<>());
            }
            map.get(count+1).add(new RBNode(false, null, null, null, null, null));
        }
    }
}
```



测试代码：

```java
package com.kun.kunspringbootweb.foo.tree;
 
 
 
import org.junit.jupiter.api.Test;
 
import java.util.ArrayList;
import java.util.List;
 
/**
 *
 * 测试代码
 *
 * @author xuyaokun
 * @date 2019/1/15 9:35
 */
public class TestRebBlackTree {
 
 
    public static void main(String[] args) {
 
        MyRBTree<Integer, String> tree = new MyRBTree<Integer, String>();
 
        //顺序插入
        for (int i = 0; i < 12; i++) {
 
            tree.add(i , "" + i);
            tree.printTreeLevel();
        }
 
        tree.remove(7);
        tree.printTreeLevel();
 
        tree.remove(4);
        tree.printTreeLevel();
 
//        System.out.println(tree.getKey(5));
 
        System.out.println("--------------随机插入----------------------");
        tree = new MyRBTree<>();
 
        List<Integer> list = new ArrayList<>();
 
        //随机插入
        for (int i = 0; i < 20; i++) {
 
            int ran = (int)(Math.random() * 100);
            System.out.println("新插入结点：" + ran);
            tree.add(ran, "" + i);
            tree.printTreeLevel();
 
            list.add(ran);
 
        }
 
        for (Integer integer : list){
            System.out.print(integer.intValue() + " ");
        }
    }
 
 
    @Test
    public void test1(){
 
        //55 94 0 8 75 5 82 78 43 47 75 33
        MyRBTree<Integer, String> tree = new MyRBTree<Integer, String>();
 
        tree.add(55, "#55");
        tree.add(94, "#94");
        tree.add(0, "#0");
        tree.add(8, "#8");
        tree.add(75, "#75");
        tree.add(5, "#5");
        tree.add(82, "#82");
        tree.add(78, "#78");
        tree.add(43, "#43");
        tree.add(47, "#47");
        tree.add(75, "#75");
        tree.add(33, "#33");
//        tree.add(95);
 
        tree.add(41, "#41");
        tree.add(23, "#23");
        tree.add(24, "#24");
        tree.add(32, "#32");
        tree.add(11, "#11");
 
 
        tree.printTreeLevel();
 
//        tree.remove(82);
        tree.remove(43);
 
        tree.printTreeLevel();
 
//        tree.remove(43);
//        tree.printTreeLevel();
 
        System.out.println(tree.get(55));
    }
 
 
    @Test
    public void test2(){
 
//       1 //                                        76
//       2 //                   36                                        116
//       4 //         16                   56                  96                    136
//       8 //    6         26         46        66        76        106        126         146
//       16 // 1   11    21  31     41 51     61  71    81  91   101  111   121  131    141  151
        MyRBTree<Integer, String> tree = new MyRBTree<Integer, String>();
 
        tree.add(76, "#55");
        tree.add(36, "#94");
        tree.add(116, "#0");
        tree.add(16, "#8");
        tree.add(56, "#75");
        tree.add(96, "#5");
        tree.add(1, "#82");
//
        tree.add(11, "#78");
        tree.add(21, "#43");
        tree.add(31, "#47");
        tree.add(41, "#75");
        tree.add(51, "#33");
//
        tree.add(61, "#41");
        tree.add(71, "#23");
        tree.add(81, "#24");
        tree.add(91, "#32");
        tree.add(101, "#11");
        tree.add(111, "#11");
        tree.add(121, "#11");
        tree.add(131, "#11");
        tree.add(141, "#11");
        tree.add(151, "#11");
 
 
        tree.printTreeLevel();
 
//        tree.remove(82);
//        tree.remove(43);
 
        tree.printTreeLevel();
 
//        tree.remove(43);
//        tree.printTreeLevel();
 
//        System.out.println(tree.get(55));
    }
 
 
    @Test
    public void test3(){
 
        MyRBTree<Integer, String> tree = new MyRBTree<Integer, String>();
 
        String str = "99 29 3 31 26 19 93 86 35 67 59 10 65 74 74 27 19 89 2 57 13 6 52 20 25 54 41 44 71 27 14 32 83 81 44 14 35 78 67 99 56 6 31 93 50 83 81 93 70 24";
        String[] strs = str.split(" ");
        for (String sss : strs){
 
            tree.add(Integer.valueOf(sss), sss);
        }
 
//        tree.printTreeLevel();
 
        tree.remove(26);
 
        tree.printTreeLevel2();
    }
 
    @Test
    public void test4(){
 
        MyRBTree tree = new MyRBTree<>();
 
        List<Integer> list = new ArrayList<>();
        //随机插入
        for (int i = 0; i < 50; i++) {
 
            int ran = (int)(Math.random() * 100);
            System.out.println("新插入结点：" + ran);
            tree.add(ran, "" + i);
            tree.printTreeLevel();
 
            list.add(ran);
 
        }
 
 
        for (Integer integer : list){
            System.out.print(integer.intValue() + " ");
        }
 
        System.out.println();
 
        tree.printTreeLevel2();
 
    }
}
```

上面总共两个类，一个红黑树实现类，一个测试类。都是测试通过的，可以直接拷贝下来运行，为了跟踪整个删除节点后的调整过程，我加了打印数层级结构的方法和输出Graphviz软件需要的文本的方法。（Graphviz是为了方便画二叉树的工具，在画图时还有可以为节点指定颜色等功能）。



# 总结

红黑树的删除操作是整个红黑树中最复杂的一部分，理解了这部分，红黑树就算基本拿下了。理解完一种数据结构，要能 get 到作者当初设计时的点，才算是一次积累。红黑树的删除操作，它非常地巧妙，整一个算法循环过程，它不会超过三次，调整过程基本都在子树内完成，指针不需要一直向上回溯，相比 AVL 树，AVL 树在删除节点时，指针有可能会一直回溯到根为止。

**- END -**