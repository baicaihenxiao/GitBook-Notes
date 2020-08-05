# 一份工具清单就可以令 Python 变快

{% embed url="https://mp.weixin.qq.com/s/gd6NlEFwMfTpg-qBHFTYwA" %}

来源：linuxer小橡皮

https://zhuanlan.zhihu.com/p/31044229

这篇文章会提供一些**优化代码的工具**。会让代码变得更简洁，或者更迅速。

当然这些并不能代替算法设计，但是还是能让Python加速很多倍。

其实前面讲算法的文章，也有提到过。比如适用于双向队列的 deque，以及在合适的条件下运用 bisect 和 heapq 来提升算法的性能。

而且前面也提到过，Python提供了当今最高级也是最有效的**排序算法\(list.sort\)**。

另外还有一个功能多样又迅速的**散列表\(dict\)**。而且如果写迭代器封装、功能性代码或者是某种额外扩展的时候，或许 **CyToolz**可以用得到。当然在**itertools和 functools模块** 中，还有很多函数可以带来很高效的代码。

这篇文章主要讲**优化单处理器**的代码，下面会介绍一些一些高效的函数实现，也有已经封装好的拓展模块，还包括速度更快的Python解释器。

当然**多处理器版本**确实能大幅提高运行效率。如果想了解多核编程，可以从**multiprocessing模块**开始。而且也能找到非常多的关于分布式计算的第三方工具。这里可以看一下Python wiki上的关于Parallel Processing的内容。

接下来，会说一些关于**Python加速工具**的选单。

### **1.NumPy、SciPy、Sage和Pandas**

先说，NumPy。它的核心是一个多维数字数组的实现。除了这个数据结构之外，还实现了若干个函数和运算符，可以高效地进行数组运算。并且对于被调用的次数进行了精简。它可以被用来进行极其高效的数学运算。

SciPy和Sage都将NumPy内置为自身的一部分，同时内置了其他的不同的工具，从而可以用于特定科学、数学和高性能计算的模块。

Pandas是一个侧重于数据分析的工具。如果处理大量半结构化数据的时候，可能也会用到Pandas相关的工具，比如Blaze。

### **2.PyPy、Pyston、Parakeet、Psyco和Unladen Swallow**

让代码运行的更快，侵入性最小的就是使用实时编译器\(JIT编译\)。以前的话我们可以直接安装Psyco。安装之后导入psyco，然后调用psyco.full\(\)。代码运行速度就可以明显提升。运行Python代码的时候，它可以实时监控程序，会将一部分代码编译为了机器码。

现在好多Psyco等加速器的项目已经停止维护了，不过类似的功能在PyPy中得到了继承。

PyPy为了方便分析、优化和翻译，用Python语言将Python重新实现了一遍，这样就可以JIT编译。而且PyPy可以直接将代码翻译成像C那样的性能更高的语言。

Unladen Swallow是一个Python的JIT编译器。是Python解释器的一本版本，被称为底层虚拟机\(LLVM\)。不过这个开发已经停止了。

Pyston是一个与LLVM平台较为接近的Python的JIT编译器。很多时候已经优于Python的实现，但不过还有很多地方不完善。

### **3.GPULib、PyStream、PyCUDA和PyOpenCL**

这四个都是用在图像处理单元来实现代码的加速。前面讲的都是用代码优化来实现加速的。而这些都是从硬件层面上进行加速，如果有一个强大的GPU，我们可以用GPU来计算，从而减少CPU宝贵的资源。

PyStream古老一点。GPULib提供了基于GPU的各种形式的数据计算。

如果用GPU加速自己的代码，可以用PyCUDA和PyOpenCL。

### **4.Pyrex、Cython、Numba和Shedskin**

这四个项目都致力于将Python代码翻译为C、C++和LLVM的代码。Shedskin会将代码编译为C++语言。Pyrex、Cython编译的主要目标是C语言。Cython也是Pyrex的一个分支。

而且，Cython还有NumPy数组的额外支持。

如果面向数组和数学计算的时候，Numba是更好的选择导入时会自动生成相应的LLVM的代码。升级版本是NumbaPro，还提供了对GPU的支持。

### **5.SWIG、F2PY和Boost.Python**

这些工具可以将其他的语言封装为Python的模块。第一个可以封装C/C++语言。F2PY可以封装Fortran。Boost.Python可以封装C++语言。

SUIG只要启动一个命令行工具，往里面输入C或者C++的头文件，封装器代码就会自动生成。除了Python，而且可以成为其他语言的封装器，比如Java和PHP。

### **6.ctypes、llvm-py和CorePy2**

这些模块可以帮助我们实现Python底层对象的操作。ctypes模块可以用于在内存中构建编译C的对象。并且调用共享库中的C的函数。不过ctypes已经包含在Python的标准库里面了。

llvm-py主要提供LLVM的Python接口。以便于构建代码，然后编译他们。也可以在Python中构建它的编译器。当然搞出自己编程语言也是可以的。

CorePy2也可以进行加速，不过这个加速是运行在汇编层的。

### **7.Weave、Cinpy和PyInline**

这三个包，就可以让我们在Python代码中直接使用C语言或者其他的高级语言。混合代码，依然可以保持整洁。可以使用Python代码的字符串的多行特性，可以使其他的代码按照自身的风格来进行排版。

### **8.其他工具**

如果我们要节省内存，就不能使用JIT了。一般JIT都太耗费内存。有一句话说的很对，**时间和内存经常不能兼得，而我们在工程开发中，总是要寻找他们的平衡点。**

至于其他的一些东西，比如Micro Python项目，这个是用在嵌入式设备或者微控制器上面使用的。

如果只是想在Python环境中工作，然后想用别的语言，可以看看这个项目Julia。

