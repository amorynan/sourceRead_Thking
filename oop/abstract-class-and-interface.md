### 面向对象程序设计之类的抉择 -- abstract class and interface

两个层面 -- 设计模式 / 语法

设计模式：

abstract class -- "is a"

interface -- “like a”

对与设计来说，abstract class 父类来说，这是一种属于的关系，也就说，能够继承此的类在抽象意义上的本质就应该是这个类，而对于什么时候设计interface，只是说在扩展功能的时候，需要对类的定义有添加

如果我们对于问题领域的理解是：AlarmDoor在概念本质上是Door，同时它有具有报警的功能。我们该如何来设计、实现来明确的反映出我们的意思呢？前面已经说过，abstract class在Java语言中表示一种继承关系，而继承关系在本质上是"is a"关系。所以对于Door这个概念，我们应该使用abstarct class方式来定义。另外，AlarmDoor又具有报警功能，说明它又能够完成报警概念中定义的行为，所以报警概念可以通过interface方式定义。如下所示：

```
abstract class Door{
    // basiclly function
    abstract void open(); // abstract means that as long as the class which extends it, must has these function
    abstract void close();
}

interface Alarm{
    // special and maybe the extend function later
    void alarm();
}

// so the alarm door is born
class AlarmDoor() extends Door, implements Alarm{
    void open(){
    }
    void close(){
    
    }
    void alarm(){
    }
}
```

so . the point is when u design a class , just think about this one is definlately a base class,  or just a function class to solve some   special requirement in client point



在语法上

abstract : the class which 

