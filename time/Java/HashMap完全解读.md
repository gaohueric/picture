##### 概念： 什么是HashMap

> 基于哈希表的Map接口实现。此实现提供所有可选的映射操作，并允许使用null值和null键。除了非同步和允许使用null外，HashMap 类与HashTable 大致相同，此类不保证映射的顺序，特别是它不保证该顺序亘久不变， 此实现假定哈希函数将元素适当地分布在各桶之间，可为基本操作（get 和 put）提供稳定的性能。迭代 collection 视图所需的时间与 HashMap 实例的“容量”（桶的数量）及其大小（键-值映射关系数）成比例。所以，如果迭代性能很重要，则不要将初始容量设置得太高（或将加载因子设置得太低）。



##### HashMap和HashTable的区别

> - **HashTable的方法是同步的**，在方法的前面都有synchronized来同步，**HashMap未经同步**，所以在多线程场合要手动同步
> - **HashTable不允许null值**(key和value都不可以) ,**HashMap允许null值**(key和value都可以)。
> - HashTable有一个contains(Object value)功能和containsValue(Object value)功能一样。
> - HashTable使用Enumeration进行遍历，HashMap使用Iterator进行遍历。
> - HashTable中hash数组默认大小是11，增加的方式是 old*2+1。HashMap中hash数组的默认大小是16，而且一定是2的指数。
> - 哈希值的使用不同，HashTable直接使用对象的hashCode，代码是这样的：

```
int hash = key.hashCode();
int index = (hash & 0x7FFFFFFF) % tab.length;
```

而 HashMap 重新计算hash值，而且用与代替求模

```
int hash = hash(k);
```

