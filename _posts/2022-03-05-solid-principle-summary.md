---
layout: post
title: "SOLID设计原则开篇"
date: 2022-03-05 18:09:12
categories: Design
tags: C# .NET SOLID
author: Michael
mathjax: true
---

* content
{:toc}

SOLID原则是一套比较成体系的设计原则。它不仅可以指导我们设计模块，有效帮助我们设计出易维护，好理解且灵活可扩展的软件，也可以减少软件的复杂性。当你在开发新功能时，不至于牵一发而动全身。在软件维护中，也不至于战战兢兢，因为你不会在担心一个改动会影响到其他组件。

它实际上是五个设计原则首字母的缩写，分别是：
- 单一职责原则（Single responsibility principle，SRP）
- 开放封闭原则（Open–closed principle，OCP）
- Liskov 替换原则（Liskov substitution principle，LSP）
- 接口隔离原则（Interface segregation principle，ISP）
- 依赖倒置原则（Dependency inversion principle，DIP）

<img style="width:150px;height:200px; margin:0px 50px" src="https://yuanzhitang.github.io/images/bob.jpeg"/>



这些设计原则是有由Robert C Martin(Bob大叔) 提出并逐步整理和完善的。他在 **《敏捷软件开发：原则、实践与模式》** 和 **《架构整洁之道》** 两本书中，对 SOLID 原则进行了两次比较完整的阐述。在这两本时隔近 20 年的书里，Robert C Martin 对 SOLID 原则的理解也是在一步步深化，这是两本非常不错的书，推荐阅读。

### 单一职责原则
这个原则看似简单，其实也是经常违反的一个原则。你可能会想到一个类或者模块只负责完成一个职责，要设计粒度小，功能单一的类。毕竟要"高内聚、低耦合"嘛！在《敏捷软件开发：原则、实践与模式》中其定义是
> “一个模块应该有且仅有一个变化的原因”

而到了《架构整洁之道》中，其定义就变成了
> “一个模块应该对一类且仅对一类行为者（actor）负责”

单一职责原则和一个类只干一件事之间，最大的差别就是，考虑到了变化。
### 开放封闭原则
> *软件实体（模块、类、方法等）应该对扩展开放、对修改关闭*

即应该在已有代码基础上拓展代码，而不是直接修改已有代码。当然这里并不是完全不修改代码，这里要修改的是上层的业务代码，而非底层的实现代码，也就是说只修改业务变化的部分，从而以最小修改代码的代价来完成新功能开发。我们需要在软件内部留好扩展点，而这正是需要我们去设计的地方。因为每一个扩展点都是一个需要设计的模型。
### Liskov 替换原则
1988 年，Barbara Liskov 在描述如何定义子类型时写下这样一段话：

> 这里需要如下替换性质：若每个类型 S 的对象 o1，都存在一个类型 T 的对象 o2，使得在所有针对 T 编程的程序 P 中，用 o1 替换 o2 后，程序 P 行为保持不变，则 S 是 T 的子类型。

用通俗的讲法来说，意思就是，*子类型（subtype）必须能够替换其父类型（base type）*。所以继承需要满足 IS-A 的关系，IS-A 的关键在于行为上的一致性。我们需要用父类的角度去思考，设计行为一致的子类。理解这一层，就比较容易理解长方形正方形问题，现实中正方形是长方形的一个特例，但是编码中正方形不应该是长方形的一个子类。这个原则
### 接口隔离原则
ISP是这样表述的：
> 不应强迫使用者依赖于它们不用的方法。
> No client should be forced to depend on methods it does not use.

很多接口设计得太“胖”了，里面包含了太多的内容，一个更好的设计是，关注不同的使用者，把大接口分解成一个一个的小接口，每个使用者面对不同的接口。
比如，一个接口有查询的函数又有添加或者修改的函数，如果一个类实现了这个接口，但只用到查询的函数，而其他函数都不用，其实就违反了接口隔离原则。
### 依赖倒置原则
依赖倒置原则（Dependency inversion principle，简称 DIP）是这样表述的：
> 高层模块不应依赖于低层模块，二者应依赖于抽象。
> High-level modules should not depend on low-level modules. Both should depend on abstractions.
> 
> 抽象不应依赖于细节，细节应依赖于抽象。
> Abstractions should not depend on details. Details (concrete implementations) should depend on abstractions.

这里的抽象主要是指接口和抽象类，高层模块直接引用抽象类或者接口，而具体类则实现接口或者抽象类。每次修改抽象接口的时候，一定也会去修改对应的具体实现。但是如果修改具体类时，却很少需要去修改相应的抽象接口，我们可以认为接口比实现更加稳定。想要设计上追求稳定，就必须多使用稳定的抽象接口，少依赖多变的具体实现。
