# static filed是否会被gc

## [Are static fields open for garbage collection?](https://stackoverflow.com/questions/453023/are-static-fields-open-for-garbage-collection)

即加载该类的类加载器被gc时，static field会被gc。

被bootstrap loader加载的类永远不会被回收。

Static variables cannot be elected for garbage collection while the class is loaded. They can be collected when the respective class loader \(that was responsible for loading this class\) is itself collected for garbage.

Check out the [JLS Section 12.7 Unloading of Classes and Interfaces](https://docs.oracle.com/javase/specs/jls/se8/html/jls-12.html#jls-12.7)

> A class or interface may be unloaded if and only if its defining class loader may be reclaimed by the garbage collector \[...\] Classes and interfaces loaded by the bootstrap loader may not be unloaded.

