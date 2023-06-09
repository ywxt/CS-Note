## 作用

### 防重排序

我们从一个最经典的例子来分析重排序问题。大家应该都很熟悉单例模式的实现，而在并发环境下的单例实现方式，我们通常可以采用双重检查加锁(DCL)的方式来实现。其源码如下：

```java
public class Singleton {
    public static volatile Singleton singleton;
    /**
     * 构造函数私有，禁止外部实例化
     */
    private Singleton() {};
    public static Singleton getInstance() {
        if (singleton == null) {
            synchronized (singleton.class) {
                if (singleton == null) {
                    singleton = new Singleton();
                }
            }
        }
        return singleton;
    }
}
```

现在我们分析一下为什么要在变量singleton之间加上volatile关键字。要理解这个问题，先要了解对象的构造过程，实例化一个对象其实可以分为三个步骤：

-   分配内存空间。
-   初始化对象。
-   将内存空间的地址赋值给对应的引用。

但是由于操作系统可以`对指令进行重排序`，所以上面的过程也可能会变成如下过程：

-   分配内存空间。
-   将内存空间的地址赋值给对应的引用。
-   初始化对象

如果是这个流程，多线程环境下就可能将一个未初始化的对象引用暴露出来，从而导致不可预料的结果。因此，为了防止这个过程的重排序，我们需要将变量设置为volatile类型的变量。

### 实现可见性

可见性问题主要指一个线程修改了共享变量值，而另一个线程却看不到。引起可见性问题的主要原因是每个线程拥有自己的一个高速缓存区——线程工作内存。

### 部分保證原子性

**只能保證單次讀寫的原子性**

*共享的 long 和 double 變量爲什麼要用 volatile?*

64 位分爲高 32 位和低 32 位，**可能**不是原子的，因此設爲 `volatile`。目前大部分虛擬機都把 64 位讀寫操作作爲原子操作，因此一般不需要 `volatile`。

## 原理

#### 可見性

由於 CPU Cache 對於每個線程的高速緩存是分開的，因此，在一個線程修改後其他線程並不能同步修改，用 `volatile` 修飾的變量在每次修改後都會把緩存內變量寫入內存，並使其他線程中的此變量無效。

在字節碼中，會在寫 `volatile` 變量之前加一個 `lock` 指令，其目的如上所述。

#### 禁止重排序





 