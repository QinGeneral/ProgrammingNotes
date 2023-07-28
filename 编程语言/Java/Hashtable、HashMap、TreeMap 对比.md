# Hashtable、HashMap、TreeMap 对比

对于哈希表原理，可以参考 [哈希表](../../算法与数据结构/5.%20哈希表.md)。

三种 Map 的对比如下：
- Hashtable：线程安全，效率低，不允许 null 键和 null 值；
- HashMap：线程不安全，效率高，put、get 操作时间复杂度为 O(1)，允许 null 键和 null 值；
- TreeMap：线程不安全，效率低，基于红黑树的一种提供顺序访问的 Map，put、get 操作时间复杂度为 O(logn)，不允许 null 键，允许 null 值。