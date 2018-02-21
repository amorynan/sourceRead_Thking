### HashMap -- 所属级别 ： 容器

### -- 主要作用  ： 装载映射关系的容器

_Note : _

* _父级关系 ： 继承自Map Interface ，与Collection  同属容器含义，但是却有着不同的装载方式_

* [ ] #### 作为一个容器，最看重的基本上就是能装多少，以及是否可以扩容，还有在多线程下的安全性问题

##### 所以接下来介绍它的成员：

```
int capacity； 初始1<<4 = 16; 代表初始下hashmap的容量，为可装16个entry
float loadfactor； 负载因子； 代表达到这个初始容量最大可负载的值就准备要扩容了 eg：loadFactor = 0.75--> 则达到12就触发resize()
int modcount; fail-fast的检测值，代表只要在装的时候，这个值不对，就要抛出异常
Entry[] table;  重头戏，主要就是这个数组在装东西，只不过这个数组中装的是Entry对象，这个对象中有Entry next属性，代表可以成为链表连起来
```

* [ ] #### 如何装？！key——value

##### 既然是映射关系的容器，必然在存储上就不像Collection一样只需要把值存进去就行了，当然还有一个可以找到映射的方式，这个意思也就是说，可以有key去找到容器里面的value，同时反着也行，那要把这个抽象的“映射”装进去，怎么装？其实主要就是利用的数组的下标，现在就是面对万千的key值如何去计算对应的数组的下标？因为容器是以装东西为目的的，而key是否又能表示唯一对象呢？

在记录原理之前，需要对hashCode进行简单的回顾，在java Class 中，Object是万物之源，而他要去怎么定义自己的唯一性呢？就像人一样的基因，那就是对象的hashCode function去return 一个hash值，也就是对象的基因，这个值一般是由创建对象时的内部地址所计算出来的整型值，一般来说，new出来的不同对象是会有不同的hash值的，除了String有点特殊的类，也就是说对于key对象，我们在存储的时候需要它的hash值，同时做一些变形计划把它变成该hashMap的entry\[\] 数组的对应下标值，这个转换可是相当有来头，因为首先要满足尽可能多的让key放在不同的位置，其次还要能够在寻找的时候快

废话不多说，看源码

```
    public V put(K key, V value) {
        if (table == EMPTY_TABLE) {
            inflateTable(threshold);
        }
        if (key == null)
            return putForNullKey(value);
        int hash = hash(key);
        int i = indexFor(hash, table.length);
        for (Entry<K,V> e = table[i]; e != null; e = e.next) {
            Object k;
            if (e.hash == hash && ((k = e.key) == key || key.equals(k))) {
                V oldValue = e.value;
                e.value = value;
                e.recordAccess(this);
                return oldValue;
            }
        }

        modCount++;
        addEntry(hash, key, value, i);
        return null;
    }
```

Note ： 注意的看，当key == null 的时候，是没有抛异常的，而是，把这个value放到了一个专门的没有key的区域，其实也就是table\[0\]的位置，所以_**hashMap是允许存放的key为null的**_

```
final int hash(Object k) {
        int h = hashSeed;
        if (0 != h && k instanceof String) {
            return sun.misc.Hashing.stringHash32((String) k);
        }

        h ^= k.hashCode();

        // This function ensures that hashCodes that differ only by
        // constant multiples at each bit position have a bounded
        // number of collisions (approximately 8 at default load factor).
        h ^= (h >>> 20) ^ (h >>> 12);
        return h ^ (h >>> 7) ^ (h >>> 4);
    }
```

hashMap中求hash的方式，先对key对象日常求hashCode后，再进行了三次的不同层次的位运算，当然了，特别是后面两种是大神通过计算经验总结出来的，怎样可以让算出来的hash值尽可能的不同

得到了key的hash后，就相当于拿到了自己的独一的通行证，现在就是进入会场找座位了，也就是计算index In entry\[\] 中

```
    static int indexFor(int h, int length) {
        // assert Integer.bitCount(length) == 1 : "length must be a non-zero power of 2";
        return h & (length-1);
    }
```

key对象先是根据自己的hashCode经过一系列的hash bit运算拿到唯一的hash通行证之后，再根据这个“会场”（容器）的大小去判断自己应该坐到哪个位置，也就是说，而这里用了非常好的位运算 -- 取模的与运算，而这个也能解释为什么要将length值置为2^n次方，因为这样使length-1后的int数值转换成二进制后，就能变成最多数量的1，可以加快装的速度，之后就是找到位置了后，对table\[i\]位置上的entry进行遍历，因为要讲究一个先来后到嘛，对于新加的entry的hash值如果和原位置上的hash一样，并且key一样，那么就让原来的让出位置给新来的，也就是说找到位置后先要去找到有没有跟我拿到一样id的，有的话就要替换，如果不是一样的，就进行addEntry操作

```
    void addEntry(int hash, K key, V value, int bucketIndex) {
        if ((size >= threshold) && (null != table[bucketIndex])) {
            resize(2 * table.length);
            hash = (null != key) ? hash(key) : 0;
            bucketIndex = indexFor(hash, table.length);
        }

        createEntry(hash, key, value, bucketIndex);
    }
```

在进行入座的时候，还要很多的事情是需要考虑的，首先要考虑的是这个大小是不是已经大于了threshold （= capability\*loadFactory） ，因为要考虑到扩容的问题，扩容的问题后面详见，现在看如何插入的

```
  void createEntry(int hash, K key, V value, int bucketIndex) {
        Entry<K,V> e = table[bucketIndex];
        table[bucketIndex] = new Entry<>(hash, key, value, e);
        size++;
    }
```

在这里是直接先将原位置上的entry记录下来，然后new Entry，注意在这里插入成为链表是用的头插入法

```
new Entry<>(hash, key, value, e);
```

```
    Entry(int h, K k, V v, Entry<K,V> n) {
            value = v;
            next = n;
            key = k;
            hash = h;
        }
```

记录下来的原位置的entry，作为next插入到新来的，也就是新来的占据了位置，原来的成为候补链表成员

好了，到这里基本上hashMap的插入就差不多了，但是光了解这些是不够的，我们还需要知道为什么hashMap是线程不安全的？！

* [ ] #### HashMap的线程不安全性

###### 对于hashMap的线程不安全性，主要有两个方面，一是对于插入entry来说，采用的头节点插入法，第二点是相对于resize的流程

首先对于头节点插入的问题，很有可能在线程1刚好进行到记录下原头节点的entry，准备插入的时候，线程2抢占到了资源，完成插入，这个时候，就会存在两个线程的数据不一致，线程1就会缺失线程2 的插入的数据，同时modcount也会不一样

其次对于resize来说

```
 void resize(int newCapacity) {
        Entry[] oldTable = table;
        int oldCapacity = oldTable.length;
        if (oldCapacity == MAXIMUM_CAPACITY) {
            threshold = Integer.MAX_VALUE;
            return;
        }

        Entry[] newTable = new Entry[newCapacity];
        transfer(newTable, initHashSeedAsNeeded(newCapacity));
        table = newTable;
        threshold = (int)Math.min(newCapacity * loadFactor, MAXIMUM_CAPACITY + 1);
    }
```

resize\(\)没有做到什么，最主要也就是扩了容，将原空间放大两倍，注意这里的扩容，还是进行了new Table的，所以扩容带来的副作用就是你得将原来的数据进行再一次的重新安排位置，也就是进入了transfer\(\)

```
    void transfer(Entry[] newTable, boolean rehash) {
        int newCapacity = newTable.length;
        for (Entry<K,V> e : table) {
            while(null != e) {
                Entry<K,V> next = e.next;
                if (rehash) {
                    e.hash = null == e.key ? 0 : hash(e.key);
                }
                int i = indexFor(e.hash, newCapacity);
                e.next = newTable[i];
                newTable[i] = e;
                e = next;
            }
        }
    }
```

在这里面主要就是这段代码会很有可能形成死循环

e.next = newTable\[i\];

newTable\[i\] = e;

e = next;

因为是复习笔记我就不多写，贴上我以前写过的blog有详解

[http://www.cnblogs.com/AmoryWang-JavaSunny/p/7487373.html](http://www.cnblogs.com/AmoryWang-JavaSunny/p/7487373.html)

