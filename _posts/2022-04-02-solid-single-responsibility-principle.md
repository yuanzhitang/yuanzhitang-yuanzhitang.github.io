---
layout: post
title: "SOLID之单一职责原则(SRP)"
date: 2022-04-02 21:54:14
categories: Design
tags: C# .NET SOLID SRP
author: Michael
mathjax: true
---

* content
{:toc}

单一职责原则（Single responsibility principle，SRP）是一个简单而直观的原则，但却是SOLID五大原则中最容易被误解的，也是经常违反的一个原则。可能是由于名字的原因，程序员们会想当然的认为这个原则就是指：每个模块都应该只负责一件事，也就是一个类或者模块只负责完成一个职责。我们要设计粒度小，功能单一的类。毕竟要"高内聚、低耦合"嘛！在重构中，我们也确实一致在强调每个函数值完成一个功能，但是这个方面并不是SRP的全部。


Robert C Martin(Bob大叔) 在他的著作 **《敏捷软件开发：原则、实践与模式》** 中初次将其定义为
> 一个模块应该有且仅有一个变化的原因
> A class should have only one reason to change.

而20你那，在他的新书 **《架构整洁之道》**中，其定义就变成了
> 一个模块应该对一类且仅对一类行为者（actor）负责


