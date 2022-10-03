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
这个原则看似简单，其实也是经常违反的一个原则。你可能会想到一个类或者模块只负责完成一个职责，要设计粒度小，功能单一的类。毕竟要"高内聚、低耦合"嘛！在《敏捷软件开发：原则、实践与模式》中其定义是，“一个模块应该有且仅有一个变化的原因”；而到了《架构整洁之道》中，其定义就变成了“一个模块应该对一类且仅对一类行为者（actor）负责”。
单一职责原则和一个类只干一件事之间，最大的差别就是，考虑到了变化。
### 开放封闭原则
软件实体（模块、类、方法等）应该对扩展开放、对修改关闭，即应该在已有代码基础上拓展代码，而不是直接修改已有代码。当然这里并不不是完全不修改代码，这里要修改的是上层的业务代码，而非底层的实现代码，也就是说只修改业务变化的部分，从而以最小修改代码的代价来完成新功能开发。
### Liskov 替换原则
### 接口隔离原则
### 依赖倒置原则
