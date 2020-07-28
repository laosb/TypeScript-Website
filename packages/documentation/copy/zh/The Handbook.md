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
  
  You should expect each chapter or page to provide you with a strong understanding of the given concepts. The TypeScript Handbook is not a complete language specification, but it is intended to be a comprehensive guide to all of the language's features and behaviors.

  A reader who completes the walkthrough should be able to:

  - Read and understand commonly-used TypeScript syntax and patterns
  - Explain the effects of important compiler options
  - Correctly predict type system behavior in most cases
  - Write a .d.ts declaration for a simple function, object, or class

  In the interests of clarity and brevity, the main content of the Handbook will not explore every edge case or minutiae of the features being covered. You can find more details on particular concepts in the reference articles.

- **参考手册**

  The handbook reference is built to provide a richer understanding of how a particular part of TypeScript works. You can read it top-to-bottom, but each section aims to provide a deeper explanation of a single concept - meaning there is no aim for continuity.

### 本书目标不是……

The Handbook is also intended to be a concise document that can be comfortably read in a few hours. Certain topics won't be covered in order to keep things short.

Specifically, the Handbook does not fully introduce core JavaScript basics like functions, classes, and closures. Where appropriate, we'll include links to background reading that you can use to read up on those concepts.

The Handbook also isn't intended to be a replacement for a language specification. In some cases, edge cases or formal descriptions of behavior will be skipped in favor of high-level, easier-to-understand explanations. Instead, there are separate reference pages that more precisely and formally describe many aspects of TypeScript's behavior. The reference pages are not intended for readers unfamiliar with TypeScript, so they may use advanced terminology or reference topics you haven't read about yet.

Finally, the Handbook won't cover how TypeScript interacts with other tools, except where necessary. Topics like how to configure TypeScript with webpack, rollup, parcel, react, babel, closure, lerna, rush, bazel, preact, vue, angular, svelte, jquery, yarn, or npm are out of scope - you can find these resources elsewhere on the web.

## 开始上手

Before getting started with [Basic Types](/docs/handbook/basic-types.html), we recommend reading one of the following introductory pages. These introductions are intended to highlight key similarities and differences between TypeScript and your favored programming language, and clear up common misconceptions specific to those languages.

- [TypeScript for New Programmers](/docs/handbook/typescript-from-scratch.html)
- [TypeScript for JavaScript Programmers](/docs/handbook/typescript-in-5-minutes.html)
- [TypeScript for OOP Programmers](/docs/handbook/typescript-in-5-minutes-oop.html)
- [TypeScript for Functional Programmers](/docs/handbook/typescript-in-5-minutes-func.html)
