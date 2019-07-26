# HashMap
>作者：TinyDolphin
链接：https://www.jianshu.com/p/75adf47958a7
来源：简书
简书著作权归作者所有，任何形式的转载都请联系作者获得授权并注明出处。

### hashmap 的主要参数都有哪些
1. 桶容量
	即数组长度：默认值16 1<<4。即在不提供有参构造器的时候，声明的桶容量
	
2. 极限容量 
	hashmap 能承受的最大桶容量1<<30 超过后将发生碰撞
3. 负载因子
	默认0.75，负载因子有个奇特的效果，表示当当前容量大于（size/）时，将进行hashmap的扩容，扩容一般为扩容为原来的两倍。
4. 扩容阈值
	capacity*loadfactory
5. TREEIFY_THRESHOLD = 8
     树形阈值
6. UNTREEIFY_THRESHOLD = 6
    取消树形阈值
7. static final int MIN_TREEIFY_CAPACITY = 64;
    最小树形化的容量阈值
	
### HashMap 的数据结构？
哈希表结构（链表散列：数组+链表）实现，结合数组和链表的优点。当链表长度超过 8 时，链表转换为红黑树。

### HashMap 的工作原理？

>HashMap 底层是 hash 数组和单向链表实现，数组中的每个元素都是链表，由 Node 内部类（实现 Map.Entry<K,V>接口）实现，HashMap 通过 put & get 方法存储和获取。

>存储对象时，将 K/V 键值传给 put() 方法：
①、调用 hash(K) 方法计算 K 的 hash 值，然后结合数组长度，计算得数组下标；
②、调整数组大小（当容器中的元素个数大于 capacity * loadfactor 时，容器会进行扩容resize 为 2n）；
③、i.如果 K 的 hash 值在 HashMap 中不存在，则执行插入;
   ii.如果 K 的 hash 值在 HashMap 中存在，且它们两者 equals 返回 true，则更新键值对；
iii. 如果 K 的 hash 值在 HashMap 中存在，且它们两者 equals 返回 false，则插入链表的尾部（尾插法）或者红黑树中（树的添加方式）。
（JDK 1.7 之前使用头插法、JDK 1.8 使用尾插法）
（注意：当碰撞导致链表大于 TREEIFY_THRESHOLD =  8  MIN_TREEIFY_CAPACITY >64时，就把链表转换成红黑树）

>获取对象时，将 K 传给 get() 方法：
①、调用 hash(K) 方法（计算 K 的 hash 值）从而获取该键值所在链表的数组下标；
②、顺序遍历链表，equals()方法查找相同 Node 链表中 K 值对应的 V 值。
hashCode 是定位的，存储位置；equals是定性的，比较两者是否相等


### 当两个对象的 hashCode 相同会发生什么？
>因为 hashCode 相同，不一定就是相等的（equals方法比较），所以两个对象所在数组的下标相同，"碰撞"就此发生。又因为 HashMap 使用链表存储对象，这个 Node 会存储到链表中。

### 你知道 hash 的实现吗？为什么要这样实现？
>A：JDK 1.8 中，是通过 hashCode() 异或低 16 位实现的：(h = k.hashCode()) ^ (h >>> 16)，主要是从速度，功效和质量来考虑的，减少系统的开销，也不会造成因为高位没有参与下标的计算，从而引起的碰撞。
Q：为什么要用异或运算符？
A：保证了对象的 hashCode 的 32 位值只要有一位发生改变，整个 hash() 返回值就会改变。尽可能的减少碰撞。

### HashMap 的 table 的容量如何确定？loadFactor 是什么？ 该容量如何变化？这种变化会带来什么问题？
> ①、table 数组大小是由 capacity 这个参数确定的，默认是16，也可以构造时传入，最大限制是1<<30；
②、loadFactor 是装载因子，主要目的是用来确认table 数组是否需要动态扩展，默认值是0.75，比如table 数组大小为 16，装载因子为 0.75 时，threshold 就是12，当 table 的实际大小超过 12 时，table就需要动态扩容；
③、扩容时，调用 resize() 方法，将 table 长度变为原来的两倍（注意是 table 长度，而不是 threshold）
④、如果数据很大的情况下，扩展时将会带来性能的损失，在性能要求很高的地方，这种损失很可能很致命。

### HashMap 的遍历方式及其性能对比
主要四种方式：

##### NO.1：for-each map.keySet() -- 只需要K值的时候，推荐使用
```java
for (String key : map.keySet()) {
    map.get(key);
}
```

##### NO.2：for-each map.entrySet() -- 当需要V值的时候，推荐使用
```java
for (Map.Entry<String, String> entry : map.entrySet()) {
    entry.getKey();
    entry.getValue();
}
```

#### NO.3：for-each map.entrySet().iterator()
```java
Iterator<Map.Entry<String, String>> iterator = map.entrySet().iterator();
    while (iterator.hasNext()) {
        Map.Entry<String, String> entry = iterator.next();
        entry.getKey();
        entry.getValue();
}
```

### HashMap，LinkedHashMap，TreeMap 有什么区别？
> HashMap 参考其他问题；
LinkedHashMap 保存了记录的插入顺序，在用 Iterator 遍历时，先取到的记录肯定是先插入的；遍历比 HashMap 慢；LikedHashMap重写了newNode（）方法，在newNode方法中将每个新生成的节点与已经存在的节点用双向链表链接起来了，其实LinkedHashMap中put方法就是使用的父类HashMap的中的put方法，只是重新的newNode方法实现了节点的双向链表结构，因此可以理解为LinkedHashMap既采用了哈希表（拉链法和红黑树）存储，也采用了双向链表存储。此外LinkedHashMap的遍历方式与HashMap也是不一样的，LinkedHashMap的遍历方式是遍历双向链表。HashMap的遍历是根据节点数组（桶）来遍历拉链节点
TreeMap 实现 SortMap 接口，能够把它保存的记录根据键排序（默认按键值升序排序，也可以指定排序的比较器）

### HashMap & TreeMap & LinkedHashMap 使用场景？
>一般情况下，使用最多的是 HashMap。
HashMap：在 Map 中插入、删除和定位元素时；
TreeMap：在需要按自然顺序或自定义顺序遍历键的情况下；
LinkedHashMap：在需要输出的顺序和输入的顺序相同的情况下。

### HashMap 和 HashTable 有什么区别？
>①、HashMap 是线程不安全的，HashTable 是线程安全的；
②、由于线程安全，所以 HashTable 的效率比不上 HashMap；
③、HashMap最多只允许一条记录的键为null，允许多条记录的值为null，而 HashTable 不允许；
④、HashMap 默认初始化数组的大小为16，HashTable 为 11，前者扩容时，扩大两倍，后者扩大两倍+1；
⑤、HashMap 需要重新计算 hash 值，而 HashTable 直接使用对象的 hashCode

### Java 中的另一个线程安全的与 HashMap 极其类似的类是什么？同样是线程安全，它与 HashTable 在线程同步上有什么不同？

> ConcurrentHashMap 类（是 Java并发包 java.util.concurrent 中提供的一个线程安全且高效的 HashMap 实现）。
HashTable 是使用 synchronize 关键字加锁的原理（就是对对象加锁）；
而针对 ConcurrentHashMap，在 JDK 1.7 中采用 分段锁的方式；JDK 1.8 中直接采用了CAS（无锁算法）+ synchronized


###  HashMap & ConcurrentHashMap 的区别？

>除了加锁，原理上无太大区别。
另外，HashMap 的键值对允许有null，但是ConCurrentHashMap 都不允许。

### 为什么 ConcurrentHashMap  比 HashTable 效率要高？
>HashTable 使用一把锁（锁住整个链表结构）处理并发问题，多个线程竞争一把锁，容易阻塞；
ConcurrentHashMap  
JDK 1.7 中使用分段锁（ReentrantLock + Segment + HashEntry），相当于把一个 HashMap 分成多个段，每段分配一把锁，这样支持多线程访问。锁粒度：基于 Segment，包含多个 HashEntry。
JDK 1.8 中使用 CAS + synchronized + Node + 红黑树。锁粒度：Node（首结点）（实现 Map.Entry<K,V>）。锁粒度降低了。

### 针对 ConcurrentHashMap 锁机制具体分析（JDK 1.7 VS JDK 1.8）？
> JDK 1.7 中，采用分段锁的机制，实现并发的更新操作，底层采用数组+链表的存储结构，包括两个核心静态内部类 Segment 和 HashEntry。
①、Segment 继承 ReentrantLock（重入锁） 用来充当锁的角色，每个 Segment 对象守护每个散列映射表的若干个桶；
②、HashEntry 用来封装映射表的键-值对；
③、每个桶是由若干个 HashEntry 对象链接起来的链表。
JDK 1.8 中，采用 Node + CAS + Synchronized 来保证并发安全。取消类 Segment，直接用 table 数组存储键值对；当 HashEntry 对象组成的链表长度超过 TREEIFY_THRESHOLD 时，链表转换为红黑树，提升性能。底层变更为数组 + 链表 + 红黑树。

### ConcurrentHashMap 在 JDK 1.8 中，为什么要使用内置锁 synchronized 来代替重入锁 ReentrantLock？
>①、粒度降低了；
②、JVM 开发团队没有放弃 synchronized，而且基于 JVM 的 synchronized 优化空间更大，更加自然。
③、在大量的数据操作下，对于 JVM 的内存压力，基于 API  的 ReentrantLock 会开销更多的内存。

### ConcurrentHashMap 简单介绍？
>
①、重要的常量：
private transient volatile int sizeCtl;
当为负数时，-1 表示正在初始化，-N 表示 N - 1 个线程正在进行扩容；
当为 0 时，表示 table 还没有初始化；
当为其他正数时，表示初始化或者下一次进行扩容的大小。
②、数据结构：
Node 是存储结构的基本单元，继承 HashMap 中的 Entry，用于存储数据；
TreeNode 继承 Node，但是数据结构换成了二叉树结构，是红黑树的存储结构，用于红黑树中存储数据；
TreeBin 是封装 TreeNode 的容器，提供转换红黑树的一些条件和锁的控制。
③、存储对象时（put() 方法）：
1.如果没有初始化，就调用 initTable() 方法来进行初始化；
2.如果没有 hash 冲突就直接 CAS 无锁插入；
3.如果需要扩容，就先进行扩容；
4.如果存在 hash 冲突，就加锁来保证线程安全，两种情况：一种是链表形式就直接遍历到尾端插入，一种是红黑树就按照红黑树结构插入；
5.如果该链表的数量大于阀值 8，就要先转换成红黑树的结构，break 再一次进入循环
6.如果添加成功就调用 addCount() 方法统计 size，并且检查是否需要扩容。
④、扩容方法 transfer()：默认容量为 16，扩容时，容量变为原来的两倍。
helpTransfer()：调用多个工作线程一起帮助进行扩容，这样的效率就会更高。
⑤、获取对象时（get()方法）：
1.计算 hash 值，定位到该 table 索引位置，如果是首结点符合就返回；
2.如果遇到扩容时，会调用标记正在扩容结点 ForwardingNode.find()方法，查找该结点，匹配就返回；
3.以上都不符合的话，就往下遍历结点，匹配就返回，否则最后就返回 null。



### ConcurrentHashMap 的并发度是什么？
>程序运行时能够同时更新 ConccurentHashMap 且不产生锁竞争的最大线程数。默认为 16，且可以在构造函数中设置。当用户设置并发度时，ConcurrentHashMap 会使用大于等于该值的最小2幂指数作为实际并发度（假如用户设置并发度为17，实际并发度则为32）


