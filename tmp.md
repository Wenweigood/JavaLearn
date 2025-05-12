## Java 集合补充

- fail-fast和fail-safe
  - fail-fast（快速失败）：一种**错误检测机制**，当多个线程并发修改集合（如一个线程遍历，另一个线程修改）时，会立即抛出 `ConcurrentModificationException`，防止数据不一致
    - 实现方式：`modCount`维护修改计数器，在迭代过程中判断`modCount == expectedModCount`是否被修改
  - fail-safe（安全失败）：是一种**线程安全的迭代机制**，允许在遍历时修改集合，不会抛出异常
    - 实现方式：
      - **拷贝原数据**如`CopyOnWriteArrayList`，遍历基于原集合的快照进行，修改操作在副本上进行
      - **使用并发数据结构**实现，如`ConcurrentHashMap`使用分段锁来支持并发操作

- Comparable 和 Comparator 的区别

  > `Comparable` 接口和 `Comparator` 接口都是 Java 中用于排序的接口，它们在实现类对象之间比较大小、排序等方面发挥了重要作用：

  - `Comparable` 接口实际上是出自`java.lang`包 它有一个 `compareTo(Object obj)`方法用来排序
  - `Comparator`接口实际上是出自 `java.util` 包它有一个`compare(Object obj1, Object obj2)`方法用来排序

- ArrayBlockingQueue 和 LinkedBlockingQueue 有什么区别？

  >  `ArrayBlockingQueue` 和 `LinkedBlockingQueue` 是 Java 并发包中常用的两种阻塞队列实现，它们都是线程安全的。不过，不过它们之间也存在下面这些区别：

  - 底层实现：`ArrayBlockingQueue` 基于数组实现，而 `LinkedBlockingQueue` 基于链表实现
  - 是否有界：`ArrayBlockingQueue` 是有界队列，必须在创建时指定容量大小。`LinkedBlockingQueue` 创建时可以不指定容量大小，默认是`Integer.MAX_VALUE`，也就是无界的。但也可以指定队列大小，从而成为有界的
  - 锁是否分离：
    -  `ArrayBlockingQueue`中的**锁是没有分离的**，**即生产和消费用的是同一个锁**，为了避免线程饥饿现象，设计为支持公平/非公平锁
    - `LinkedBlockingQueue`中的**锁是分离的**，即生产用的是`putLock`，消费是`takeLock`，这样可以防止生产者和消费者线程之间的锁竞争，由于锁竞争较少，因而设计成了非公平锁
  - 内存占用：`ArrayBlockingQueue` 需要提前分配数组内存，而 `LinkedBlockingQueue` 则是动态分配链表节点内存。这意味着，`ArrayBlockingQueue` 在创建时就会占用一定的内存空间，且往往申请的内存比实际所用的内存更大，而`LinkedBlockingQueue` 则是根据元素的增加而逐渐占用内存空间

-  `HashMap` 实现中，当用户通过构造函数指定初始容量时，会将该容量调整为**大于等于该值的最小的2的幂**

  - 为什么需要2的幂？

    - `HashMap` 使用 `(n - 1) & hash` 来计算元素在数组中的位置，其中 `n` 是数组长度。当 `n` 是2的幂时：
      - `hash & (n-1)` 相当于 `hash % n`，但位运算比模运算高效得多
      - 可以均匀利用hash值的所有低位，减少哈希冲突
      - 扩容时高效，新容量扩容翻倍仍然是2的幂，重新计算索引也很高效
      - 内存对齐

  - Java 8 实现算法解析

    ```java
    static final int tableSizeFor(int cap) {
        int n = cap - 1;
      	// 移位或的目的：将最高位1右边所有位都变成1
      	// 只需要五次移位（一次处理2，两次处理4，...，五次处理32）或即可将所有的32位处理完毕
        n |= n >>> 1;
        n |= n >>> 2;
        n |= n >>> 4;
        n |= n >>> 8;
        n |= n >>> 16;
        return (n < 0) ? 1 : (n >= MAXIMUM_CAPACITY) ? MAXIMUM_CAPACITY : n + 1;
    }
    ```

- HashMap与ConcurrentHashMap对null键值的不同处理机制

  - HashMap允许null的原因：
    - HashMap是Java早期引入的非线程安全集合类，为了提供更大的灵活性而允许null键值
    - 当key为null时，HashMap会特殊处理，将其哈希值固定为0，存储在数组的第一个位置
    - 单线程环境下，可以通过`containsKey()`明确区分key不存在，和值为null的情况
  - ConcurrentHashMap不允许null的原因：
    - 在多线程环境中，无法在`get()`时判断区分key不存在和值为null的情况，也就出现二义性
      - 即使使用`containsKey()`检查，在检查与获取`get()`之间可能有其他线程修改了Map状态
    - 不允许null可以降低复杂性，简化并发控制逻辑