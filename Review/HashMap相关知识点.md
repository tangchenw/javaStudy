## 1、HashCode为什么使用31作为乘数？

固定乘积31的使用位置

```java
// 获取hashCode "abc".hashCode();
public int hashCode() {
    int h = hash;
    if (h == 0 && value.length > 0) {
        char val[] = value;
        for (int i = 0; i < value.length; i++) {
            h = 31 * h + val[i];
        }
        hash = h;
    }
    return h;
}
```

理由：

1. 31 是一个奇质数，如果选择偶数会导致乘积运算时数据溢出。
2. 另外在二进制中，2个5次方是32，那么也就是 `31 * i == (i << 5) - i`。这主要是说乘积运算可以使用位移提升性能，同时目前的JVM虚拟机也会自动支持此类的优化。

## 2、扰动函数的作用

`java 8`的散列值扰动函数，用于优化散列效果

```java
static final int hash(Object key) {
    int h;
    return (key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);
}
 
```

- 使用扰动函数就是为了增加随机性，让数据元素更加均衡的散列，减少碰撞。

## 3、初始化容量和负载因子

### 3.1 初始化容量

```java
public HashMap(int initialCapacity, float loadFactor) {
    ...
    this.loadFactor = loadFactor;
    this.threshold = tableSizeFor(initialCapacity);
}
```

- 阈值`threshold`，通过方法`tableSizeFor`进行计算，是根据初始化来计算的。
- 这个方法也就是要寻找比初始值大的，最小的那个2进制数值。比如传了17，我应该找到的是32（2的4次幂是16<17,所以找到2的5次幂32）。

```java
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

- 乍一看可能有点晕😵怎么都在向右移位1、2、4、8、16，这主要是为了把二进制的各个位置都填上1，当二进制的各个位置都是1以后，就是一个标准的2的幂次方减1了，最后把结果加1再返回即可。

### 3.2 负载因子

```java
static final float DEFAULT_LOAD_FACTOR = 0.75f;
```

所以，要选择一个合理的大小下进行扩容，默认值0.75就是说当阈值容量占了3/4时赶紧扩容，减少Hash碰撞。

同时0.75是一个默认构造值，在创建HashMap也可以调整，比如你希望用更多的空间换取时间，可以把负载因子调的更小一些，减少碰撞。

## 4、扩容元素拆分

为什么扩容，因为数组长度不足了。那扩容最直接的问题，就是需要把元素拆分到新的数组中。拆分元素的过程中，原jdk1.7中会需要重新计算哈希值，但是到jdk1.8中已经进行优化，不再需要重新计算，提升了拆分的性能，设计的还是非常巧妙的。

## 5、HashMap数据插入、查找、删除、遍历

### 5.1 插入

插入过程具体如下：

1. 首先进行哈希值的扰动，获取一个新的哈希值。`(key == null) ? 0 : (h = key.hashCode()) ^ (h >>> 16);`
2. 判断tab是否为空或者长度为0，如果是则进行扩容操作。

```java
if ((tab = table) == null || (n = tab.length) == 0)
    n = (tab = resize()).length;
```

3. 根据哈希值计算下标，如果对应下标正好没有存放数据，则直接插入即可否则需要覆盖。`tab[i = (n - 1) & hash])`
3. 判断tab[i]是否为树节点，否则向链表中插入数据，是则向树中插入节点。
3. 如果链表中插入节点的时候，链表长度大于等于8，则需要把链表转换为红黑树。`treeifyBin(tab, hash);`
3. 最后所有元素处理完成后，判断是否超过阈值；`threshold`，超过则扩容。
3. `treeifyBin`,是一个链表转树的方法，但不是所有的链表长度为8后都会转成树，还需要判断存放key值的数组桶长度是否小于64 `MIN_TREEIFY_CAPACITY`。如果小于则需要扩容，扩容后链表上的数据会被拆分散列的相应的桶节点上，也就把链表长度缩短了。

### 5.2 扩容机制

扩容的具体操作如下：

1. 扩容时计算出新的newCap、newThr，这是两个单词的缩写，一个是Capacity ，另一个是阀Threshold
2. newCap用于创新的数组桶 `new Node[newCap];`
3. 随着扩容后，原来那些因为哈希碰撞，存放成链表和红黑树的元素，都需要进行拆分存放到新的位置中。

### 5.3 链表树化

HashMap这种散列表的数据结构，最大的性能在于可以O(1)时间复杂度定位到元素，但因为哈希碰撞不得已在一个下标里存放多组数据，那么jdk1.8之前的设计只是采用链表的方式进行存放，如果需要从**链表**中定位到数据时间复杂度就是**O(n)**，链表越长性能越差。因为在jdk1.8中把过长的链表也就是8个，优化为**自平衡的红黑树**结构，以此让定位元素的时间复杂度优化**近似于O(logn)**，这样来提升元素查找的效率。但也不是完全抛弃链表，因为在元素相对不多的情况下，链表的插入速度更快，所以综合考虑下设定阈值为8才进行红黑树转换操作。



1. 链表树化的条件有两点；链表长度大于等于8、桶容量大于64，否则只是扩容，不会树化。
2. 链表树化的过程中是先由链表转换为树节点，此时的树可能不是一颗平衡树。同时在树转换过程中会记录链表的顺序，`tl.next = p`，这主要方便后续树转链表和拆分更方便。
3. 链表转换成树完成后，在进行红黑树的转换。先简单介绍下，红黑树的转换需要染色和旋转，以及比对大小。在比较元素的大小中，有一个比较有意思的方法，`tieBreakOrder`加时赛，这主要是因为HashMap没有像TreeMap那样本身就有Comparator的实现。

### 5.4 红黑树转链

红黑树转链表时候，直接把TreeNode转换为Node即可，源码如下；

```java
final Node<K,V> untreeify(HashMap<K,V> map) {
    Node<K,V> hd = null, tl = null;
    // 遍历TreeNode
    for (Node<K,V> q = this; q != null; q = q.next) {
    	// TreeNode替换Node
        Node<K,V> p = map.replacementNode(q, null);
        if (tl == null)
            hd = p;
        else
            tl.next = p;
        tl = p;
    }
    return hd;
}

// 替换方法
Node<K,V> replacementNode(Node<K,V> p, Node<K,V> next) {
    return new Node<>(p.hash, p.key, p.value, next);
}
 
```

### 5.5 查找

```java
public V get(Object key) {
    Node<K,V> e;
    // 同样需要经过扰动函数计算哈希值
    return (e = getNode(hash(key), key)) == null ? null : e.value;
}

final Node<K,V> getNode(int hash, Object key) {
    Node<K,V>[] tab; Node<K,V> first, e; int n; K k;
    // 判断桶数组的是否为空和长度值
    if ((tab = table) != null && (n = tab.length) > 0 &&
        // 计算下标，哈希值与数组长度-1
        (first = tab[(n - 1) & hash]) != null) {
        if (first.hash == hash && // always check first node
            ((k = first.key) == key || (key != null && key.equals(k))))
            return first;
        if ((e = first.next) != null) {
            // TreeNode 节点直接调用红黑树的查找方法，时间复杂度O(logn)
            if (first instanceof TreeNode)
                return ((TreeNode<K,V>)first).getTreeNode(hash, key);
            // 如果是链表就依次遍历查找
            do {
                if (e.hash == hash &&
                    ((k = e.key) == key || (key != null && key.equals(k))))
                    return e;
            } while ((e = e.next) != null);
        }
    }
    return null;
}
```

1. 扰动函数的使用，获取新的哈希值，这在上一章节已经讲过
2. 下标的计算，同样也介绍过 `tab[(n - 1) & hash])`
3. 确定了桶数组下标位置，接下来就是对红黑树和链表进行查找和遍历操作了

### 5.6 删除

```java
 public V remove(Object key) {
     Node<K,V> e;
     return (e = removeNode(hash(key), key, null, false, true)) == null ?
         null : e.value;
 }
 
final Node<K,V> removeNode(int hash, Object key, Object value,
                           boolean matchValue, boolean movable) {
    Node<K,V>[] tab; Node<K,V> p; int n, index;
    // 定位桶数组中的下标位置，index = (n - 1) & hash
    if ((tab = table) != null && (n = tab.length) > 0 &&
        (p = tab[index = (n - 1) & hash]) != null) {
        Node<K,V> node = null, e; K k; V v;
        // 如果键的值与链表第一个节点相等，则将 node 指向该节点
        if (p.hash == hash &&
            ((k = p.key) == key || (key != null && key.equals(k))))
            node = p;
        else if ((e = p.next) != null) {
            // 树节点，调用红黑树的查找方法，定位节点。
            if (p instanceof TreeNode)
                node = ((TreeNode<K,V>)p).getTreeNode(hash, key);
            else {
                // 遍历链表，找到待删除节点
                do {
                    if (e.hash == hash &&
                        ((k = e.key) == key ||
                         (key != null && key.equals(k)))) {
                        node = e;
                        break;
                    }
                    p = e;
                } while ((e = e.next) != null);
            }
        }
        
        // 删除节点，以及红黑树需要修复，因为删除后会破坏平衡性。链表的删除更加简单。
        if (node != null && (!matchValue || (v = node.value) == value ||
                             (value != null && value.equals(v)))) {
            if (node instanceof TreeNode)
                ((TreeNode<K,V>)node).removeTreeNode(this, tab, movable);
            else if (node == p)
                tab[index] = node.next;
            else
                p.next = node.next;
            ++modCount;
            --size;
            afterNodeRemoval(node);
            return node;
        }
    }
    return null;
} 
```

- 删除的操作也比较简单，这里面都没有太多的复杂的逻辑。
- 另外红黑树的操作因为被包装了，只看使用上也是很容易。

### 5.7 遍历

HashMap中的遍历也是非常常用的API方法，包括；

**KeySet**

```java
 for (String key : map.keySet()) {
     System.out.print(key + " ");
 }
```

**EntrySet**

```java
 for (HashMap.Entry entry : map.entrySet()) {
     System.out.print(entry + " ");
 }
 
```

从方法上以及日常使用都知道，KeySet是遍历是无序的，但每次使用不同方式遍历包括`keys.iterator()`，它们遍历的结果是固定的。

那么从实现的角度来看，这些种遍历都是从散列表中的链表和红黑树获取集合值，那么他们有一个什么固定的规律吗？

