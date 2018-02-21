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

在进行入座的时候，还要很多的事情是需要考虑的，首先要考虑的是这个大小是不是已经大于了threshold （= capability\*loadFactory） ，因为要考虑到扩容的问题

