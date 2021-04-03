---
title: 【java】泛型理解：协变、逆变、不变
date: 2021-04-03 20:55:23
tags: [java]
categories: java
---

泛型中的继承与扩展，`extends` 和 `super`

在开发中，关于泛型的操作离不开这两种实现的数据类型。使用泛型一方面是为了使程序能兼容多种数据类型，如使用继承型类 `ClassA<T extent B>` ，那么这个类可以接收 B 以及 B 的子类，而`CalssA` 内部能够直接对 B 进行操作，使之隔离参数B的影响；另一方面可以限定数据类型，如使用扩展型`List<T super C>` 限制输入的数据类型为C，或C的父类。

### 1.继承型（extends）

先说结论，继承类型数据结构的最终目的是 **取数据**。

举例

```java
abstract class Food{}

class Meat extends Food{}
class Beef extends Meat{}

class Fruit extends Food{}
class Apple extends Fruit{}
```

当一个泛型声明为继承型，如`ArrayList<T extent Food> list = new ArrayList<?>();` 那么后面声明的类型 **?** 必须是`Food`以及`Food`的子类，从一定程度来说，继承型也是限制了数据的可选范围，表明接收类型的上界。

进一步说明，通过`extends`方式申明限制的是这个大类型，即 `list`这个对象，这样我们能准确地知道，从这个`list`中取出的具体数据类型。如

````java
//正常来说，这个一般是一个赋值操作，而不是 new
//一般是这样
//class Adapter<T extends Food>{
//	private List<T> list;
//	public Adapter(List<T> list){
//    	list = this.list;    
//    }
//
//	public T get(int index){
//		return (T) list.get(index);
//	}
//
//	public Food get(int index){
//		return list.get(index);
//	}
//}

ArrayList<T extent Food> list = new ArrayList<Apple>();
Apple apple = list.get(0);
````

我们确切知道取出的数据类型就是 `Apple`。

我们并不在意数据源是如何构造，如何填充，我们只需关注的点在`Adapter` 中，只要你传入的数据是 `Food` 或者是`Food` 的子类，那么我们就能取出其中数据作为 `Food` 使用，也可以强转为其他确切子类进行操作。

### 2.扩展型（super）

先说结论，扩展型数据结构的目的是**写入数据**。

对于超类来说，使用的类至少是当前声明的类，也就是声明类及它的父类。所以，声明类是所定义泛型的最低标准，即下界，而上界是它的众多父类。如

```java
List<? super Fruit> list1 = new ArrayList<Fruit>();
List<? super Fruit> list2 = new ArrayList<Food>();

List<? super Food> list3 = new ArrayList<Food>();//只有 Food，没有其他父类
```

即 `<? super Fruit>` 它能接受 `Fruit` 本身，以及父类 `Food` 。

为什么说 `super` 的目的是 **写入数据？**

从上面的写法可以看出来，使用 `super` 定义的 `list` 我们可以往里面放 `Fruit` 以及它的子类，它放宽了数据的限制。如 `list3` 也能放入 `Meat`

```
list3.add(new Apple());
list3.add(new Fruit());
list3.add(new Meat());
```

但是它对数据的取出操作不友好。

`list1` 定义的数据类型是 `Fruit`,`list2` 定义的数据类型是 `Food`，但是他们声明却都是 `<? super Fruit>`，那仅仅通过定义的头声明，我们无法确切知道数组存储的数据类型是什么，它可以是 `Fruit`，也可以是 `Fruit` 的任意一个子类，所以我们取出的元素推断出来的最小类型是 `Fruit` ，当然这也是需要进一步强转才能知道的，所以在不经强转的前提下，我们最终所知最合适的数据类型只能定义为是 `Object`，这也进一步推论 `super` 的主要目的是**写入数据**，而不是**读取数据**。

虽然我们自己编程时候知道准确的数据类型，使用时强转即可，但是在一些扩展型代码中，往往使用一个标识符指代该类型，而该标识符在编译时被擦除（**编译泛型擦除**），所以无法追溯，这也是 `super` 多使用在写数据而不是取数据的原因。

### 3.确切类型

相对于泛型，还存在一种确切的数据类型，即不变。也就是声明具体的数据类型，写入读取都只能操作声明的类型。如

```java
//只能接收 String，取出的也是 String，当然也能当作 Object。万物基于 Object。
Array<String> data = new ArrayList<>();
```

### 4.协变、逆变、不变

这三种其实是上面的官方描述，表达`java`中的继承关系。

不变上面已经解释，不多说。

协变对应 `extends`, 理解为使用的数据类型可以为**定义的类型以及它的演化类，即子类，一个正向的推理，由父及子**

逆变对应 `super`，使用的数据类型可以为**定义的类型以及它的祖先类，即父类，逆向推理，由子及父的过程**

### 5.总结，记忆

在《Effective Java》中，有一个精炼的回答： **Producer-extends, Consumer-Super**, **PECS**。字面理解是，`extends` 生产者限制数据来源，`super` 消费者限制数据流出。

如何理解 `extends` 生产者限制数据来源？
数据来源已被限制，所以流出端数据是安全，你只管取出使用。

`super` 消费者限制数据流出？

数据流出被限制，那么数据流入端只要符合规则就可写入，也就是你只管写入。

怎么记忆 `PECS` 更好？

如果以生产者来记忆，容易产生 `extends` 等同于生产者的错觉，正确应该把它们理解为一个组合，即 `Producer（生产者）-extends（消费者）`，`extends` 的角色是消费者，它是来取数据，来消费数据的；同样类推，`Consumer（消费者）-Super（生产者）`，`super` 的角色是生产者，是生产数据，主要写入数据。

以后再编写泛型的代码时，只要先确定生产关系，那么就能准确选择出是 `extends` 还是 `super`。

