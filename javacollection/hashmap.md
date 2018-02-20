### HashMap -- 所属级别 ： 容器

###                  -- 主要作用  ： 装载映射关系的容器

_Note : _

* _父级关系 ： 继承自Map Interface ，与Collection  同属容器含义，但是却有着不同的装载方式_

1. [ ] #### 作为一个容器，最看重的基本上就是能装多少，以及是否可以扩容，还有在多线程下的安全性问题

#####      所以接下来介绍它的成员：

```
int capacity； 初始1<<4 = 16; 代表初始下hashmap的容量，为可装16个entry
float loadfactor； 负载因子； 代表达到这个初始容量最大可负载的值就准备要扩容了 eg：loadFactor = 0.75--> 则达到12就触发resize()
int modcount; fail-fast的检测值，代表只要在装的时候，这个值不对，就要抛出异常
Entry[] table;  重头戏，主要就是这个数组在装东西，只不过这个数组中装的是Entry对象，这个对象中有Entry next属性，代表可以成为链表连起来 
```



* [ ] #### 如何装？！key——value

####  



