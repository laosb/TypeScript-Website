---
title: TypeScript 手册
layout: docs
permalink: /docs/handbook/intro.html
oneline: 学习 TypeScript 的第一步
---

## 关于这本手册

在编程社群中创立至今20年，JavaScript 已是所有编程语言中最受广泛运用的跨平台语言之一。创立之初，JavaScript 只是一个用于在网页上添加简单的交互的脚本语言，而现在，JavaScript 已经成长为创建各种规模的前后端应用时的选择。不过尽管以 JavaScript 编写的程序之体积、领域和复杂度经历了指数级的增长，JavaScript 语言自身表达各代码单元之间关系的能力却没有得到相应的提升。加之 JavaScript 古怪的运行时行为，编程语言与应用程序之间复杂度的不对等已使得 JavaScript 难以胜任大规模应用的开发。

程序员们犯下的最常见的错误可以被归类为「类型错误」：某种类型的值被运用在需要另一类型的值的位置。造成这种情况的可能性有很多，比如纯粹的 typo，对依赖库 API 错误的理解，以及对运行时行为错误的假设等。TypeScript 的目标是成为 JavaScript 程序的静态类型检查器——换句话说，一个在代码运行前执行的（所谓「静态」）用于确保代码中的类型正确性（所谓「类型检查」）的工具。

如果你在来学习 TypeScript 之前没有 JavaScript 的背景，想将 TypeScript 作为你的编程母语，我们推荐您首先阅读 [Mozilla Web Docs 中的 JavaScript 相关章节](https://developer.mozilla.org/docs/Web/JavaScript/Guide)。
如果你有使用其它编程语言的经验，您应该能在阅读本书时很快领会 JavaScript 语法。

## 本书的结构

本书分为两个部分：

- **手册**

  TypeScript 手册意在作为综合性的文档向普通程序员阐释 TypeScript。你可以在左边的导航栏中自顶向下阅读本书。
  
  您可以期待每一章节或页面都能给你带来对给定概念较好的理解。本手册不是完整的语言规范，而是一本关于所有语言特性和行为的综合指南。
  
  读完本手册，读者将可以做到：
  
  - 阅读并理解常见的 TypeScript 语法和写法
  - 解释重要的编译器选项的效果
  - 在多数场景下能正确预测类型系统的行为
  - 为简单的函数、对象或类编写 .d.ts 类型声明

  为了保持简洁明了，这一部分内容不会讨论每一个边缘案例或是细枝末节的特性。你可以在「参考」部分中找到针对一些概念的详细讨论。

- **参考**

  编写参考部分是为了就某些 TypeScript 工作方式的具体方面提供更深刻的见解。您依然可以自上而下按顺序阅读，但每一章节旨在提供与某一个概念相关的更深刻的诠释——意味着我们不追求章节间的连惯性。
  
### 本书的目标不是……

本书也希望可以成为一个简要的、可以在几个小时内读完的文档。囿于篇幅，本书将不会涵盖一些特定的主题。

特别来说，TypeScript 手册不会完全阐释 JavaScript 核心基础概念，如函数、类和闭包。我们会在合适的位置包含一些有关这些概念的背景知识链接，供您参考。

本书并不意图成为编程语言规范的替代品。一些情况下，我们会以高层次、易于理解的解释来取代边缘案例的叙述和规范的行为描述。单独的参考文章中有对于 TypeScript 很多方面行为的更准确、更规范的描述。参考文章并不面向对 TypeScript 不熟悉的读者， 因此这些页面可能会运用一些高级的术语，或提及一些您未曾了解过的主题。

最后，本书不会涵盖 TypeScript 与其它工具互操作的相关内容，除非有必要这样做。有关怎样配置 TypeScript 使之和 webpack、rollup、parcel、react、babel、closure、lerna、rush、bazel、preact、vue、angular、svelte、jquery、yarn或 npm 一同工作的话题超出了本书讨论的范畴——您总是能在网络上别的地方找到这些内容。

## 开始入手

在开始阅读[基础类型](/docs/handbook/basic-types.html)之前，我们建议您先阅读下列介绍文章中的其中一篇。这些介绍旨在突出 TypeScript 和您所熟悉的编程语言之间的关键共通点和关键差异，并厘清来自那些语言的一些常见误解。

- [面向新程序员的 TypeScript 介绍](/docs/handbook/typescript-from-scratch.html)
- [面向 JavaScript 程序员的 TypeScript 介绍](/docs/handbook/typescript-in-5-minutes.html)
- [面向 OOP（面向对象语言）程序员的 TypeScript 介绍](/docs/handbook/typescript-in-5-minutes-oop.html)
- [面向函数式编程语言程序员的 TypeScript 介绍](/docs/handbook/typescript-in-5-minutes-func.html)
