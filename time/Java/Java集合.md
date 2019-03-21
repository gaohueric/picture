

- **是否保证线程安全**： Arraylist 和LinkedList 都不是同步的，也就是不保证线程安全。
- **底层数据结构**： Arraylist 底层使用的是Object数组；LinkedList 底层使用的是双向循环链表数据结构。
- **插入和删除是否受位置影响**： ① ArrayList 采用数组存储，所以插入和删除元素的时间复杂度受元素位置的影响，比如，执行add(E e) 方法的时间，ArrayList 会默认在将指定的元素追加到此列表的末尾，这种情况时间复杂度就是O(1). 但是如果要在指定位置i插入和删除元素的话(add(int index,E element))时间复杂度就是O(n-i),因为在进行上述操作的时间集合中第i和第i个元素之后的(n-i)个元素都要执行向后/向前移一位的操作。② LinkedList 采用链表存储，所以插入，删除元素时间复杂度不受元素位置的影响，都是近似O(1) 而 数组为近似O(n)
- 是否支持快速随机访问： LinkedList 不支持高效的随机元素访问，而ArrayList 支持，快速随机访问就是通过元素的序号快速获取元素对象(对应于get(int index))方法
- 内存空间占用： ArrayList 的空间浪费主要体现在list列表的结尾会预留一定的容量空间，而LinkedList 的空间花费则体现在它的每一个元素都需要消耗比ArrayList 更多的空间(因为要存放直接后继和直接前驱以及数据

##### 双向链表

双向链表也叫双链表，是链表的一种，它的每个数据结点中都有两个指针，分别指向直接后继和直接前驱。所以，从双向链表中的任意一个结点开始，都可以很方便地访问它的前驱结点和后继结点。一般我们都构造双向循环链表，如下图所示，同时下图也是LinkedList 底层使用的是双向循环链表数据结构。

[![img](https://camo.githubusercontent.com/42fe5da84eacd9b33e1143f3d6fe6d8153d46989/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d382d32312f38383736363732372e6a7067)](https://camo.githubusercontent.com/42fe5da84eacd9b33e1143f3d6fe6d8153d46989/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d382d32312f38383736363732372e6a7067)



##### ArrayList 和Vector的区别

Vector类的所有方法都是同步的，可以由两个线程安全的访问一个Vector对象，但是一个线程访问Vector的话代码要在同步操作上耗费大量的时间

Arraylist 不是同步的，所以在不需要线程安全时建议使用Arraylist .



##### HashMap的底层实现



##### jdk1.8之前

JDK1.8之前HashMap 底层是数组和链表结合在一起使用也就是链表散列。HashMap 通过key的hashCode 经过扰动函数处理过后得到hash值，然后通过(n-1) & hash 判断当前存放的位置(这里n指的是数组的长度）如果当前位置存在元素的话，就判断该元素与要存入的元素的hash值以及key是否相同，如果相同的话，直接覆盖，不相同就通过拉链法解决冲突。

所谓扰动函数指的就是HashMap 的hash方法，使用hash方法也就是扰动函数为了防止一些实现比较差的hashCode()方法，换句话说使用扰动函数之后可以减少碰撞。



JDK1.8 HashMap 的hash方法源码：

```
   static final int hash(Object key) {
      int h;
      // key.hashCode()：返回散列值也就是hashcode
      // ^ ：按位异或
      // >>>:无符号右移，忽略符号位，空位都以0补齐
      return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
  }
```

对比一下JDK1.7 的HashMap 的hash方法源码

```
static int hash(int h) {
    // This function ensures that hashCodes that differ only by
    // constant multiples at each bit position have a bounded
    // number of collisions (approximately 8 at default load factor).

    h ^= (h >>> 20) ^ (h >>> 12);
    return h ^ (h >>> 7) ^ (h >>> 4);
}
```

相比于 JDK1.8 的 hash 方法 ，JDK 1.7 的 hash 方法的性能会稍差一点点，因为毕竟扰动了 4 次。

所谓”拉链法"就是： 将链表和数组相结合，也就是说创建一个链表数组，数组中每一格就是一个链表，若 遇到哈希冲突，则将冲突的值加到链表中即可。

[![jdk1.8之前的内部结构](https://camo.githubusercontent.com/eec1c575aa5ff57906dd9c9130ec7a82e212c96a/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f332f32302f313632343064626363333033643837323f773d33343826683d34323726663d706e6726733d3130393931)](https://camo.githubusercontent.com/eec1c575aa5ff57906dd9c9130ec7a82e212c96a/68747470733a2f2f757365722d676f6c642d63646e2e786974752e696f2f323031382f332f32302f313632343064626363333033643837323f773d33343826683d34323726663d706e6726733d3130393931)



##### JDK1.8之后

相比于之前的版本，JDK1.8之后在解决哈希冲突时有了较大的变化，当链表长度大于阀值时，将链表转化为红黑树，以减少搜索时间。

[![JDK1.8之后的HashMap底层数据结构](https://camo.githubusercontent.com/20de7e465cac279842851258ec4d1ec1c4d3d7d1/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d382d32322f36373233333736342e6a7067)](https://camo.githubusercontent.com/20de7e465cac279842851258ec4d1ec1c4d3d7d1/687474703a2f2f6d792d626c6f672d746f2d7573652e6f73732d636e2d6265696a696e672e616c6979756e63732e636f6d2f31382d382d32322f36373233333736342e6a7067)

> TreeMap、TreeSet 以及JDK1.8之后的HashMap底层都用了红黑树，红黑树就是为了解决二叉查找数的缺陷，因为二叉查找数在某些情况下会退化成一个线性结构。



##### HashMap 与HashTable的区别 

- **线程是否安全**： HashMap是非线程安全的，HashTable是线程安全的；HashTable内部的方法基本经过synchronized 修饰(如果要保证线程安全建议使用ConcurrentHashMap).
- **效率**： 因为线程安全的问题，HashMap要比HashTable 效率高一点，另外，HashTable基本被淘汰，不要在代码中使用它。
- **对Null key 和Null value 的支持**，HashMap 中，null可以作为键，这样的键只有一个，可以由一个或者多个键所对应的值为null,但是在HashTable 中put进得键值只有一个null,直接抛出NullPointerException.
- **初始容量大小和每次扩容大小的不同**： ① 创建时如果不指定容量的初始值，HashTable 默认的初始大小为11，之后每次扩容，容量变为原来的2n+1,HashMap 默认的初始大小为16，之后每次扩容，容量变为原来的2倍。 ② 创建时间如果给定了容量初始值，那么HashTable 会直接使用你给定的大小，而HashMap 会将其扩充为2的幂次方大小(HashMap中的tableSizeFor()方法保证) 也就是说HashMap总是使用2的幂次方作为哈希表的大小，后面会介绍到什么为2的幂次方。
- **底层数据结构**： JDK1.8 以后的HashMap 在解决哈希冲突时有了较大的变化，当链表长度大于阀值(默认为8) 时，将链表转化为红黑树，以减少搜索时间，HashTable 没有这样的机制。



**HashMap中带有初始容量的构造函数**

```
    public HashMap(int initialCapacity, float loadFactor) {
        if (initialCapacity < 0)
            throw new IllegalArgumentException("Illegal initial capacity: " +
                                               initialCapacity);
        if (initialCapacity > MAXIMUM_CAPACITY)
            initialCapacity = MAXIMUM_CAPACITY;
        if (loadFactor <= 0 || Float.isNaN(loadFactor))
            throw new IllegalArgumentException("Illegal load factor: " +
                                               loadFactor);
        this.loadFactor = loadFactor;
        this.threshold = tableSizeFor(initialCapacity);
    }
     public HashMap(int initialCapacity) {
        this(initialCapacity, DEFAULT_LOAD_FACTOR);
    }
```

下面这个方法保证了HashMap 总是使用2的幂次方作为哈希表的大小

```
    /**
     * Returns a power of two size for the given target capacity.
     */
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
```

**HashMap 的长度为什么是2的幂次方**

为了让HashMap 能存取高效，尽量减少碰撞，也就是尽量把数据分配均匀，Hash值得范围值 2147483648到2147483648，前后加起来大概40亿的映射空间，只要哈希函数映射的比较均匀松散，一般应用是很难出现碰撞的，但问题是一个40亿长度的数组，内存是放不下的，所以这个狩猎值是不能直接拿来用的，用之前还要先做数组的长度取模运算，得到的余数才能用来要存放位置也就是对应的数组下标，这个数组下标的计算方法时 "(n-1) & hash" n 表示数组的长度，这个也就解释了HashMap 的长度为什么是2 的幂次方。

**这个算法设计方式**

我们首先可能会想到采用%取余的操作来实现，但是，重点来了，“取余(%)操作中如果除数是2 的幂次方则等价于其除数减一的与(&)操作，(也就是说hash%length== hash&(length-1))的前提是length是2的n次方；，并且采用二进制为操作&，相对于%能够提高运算效率，这就解释了HashMap 的长度为什么是2的幂次方。



##### HashMap多线程操作导致死循环问题

##### 